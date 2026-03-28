# Runbook: Database Failover & Recovery

| Field | Value |
|-------|-------|
| **Runbook ID** | RB-0001 |
| **Version** | 1.0.0 |
| **Last Tested** | — |
| **Severity** | SEV-1 |
| **Related Alert** | `DatabaseDown`, `HighDiskUsage` |
| **Estimated Duration** | 15–30 minutes |
| **Author** | Vox |
| **Created** | 2026-03-27 |

---

## Symptoms

- [ ] Alert `DatabaseDown` firing (PostgreSQL unreachable)
- [ ] Application returns HTTP 500 with database connection errors
- [ ] Grafana PostgreSQL dashboard shows `pg_up == 0`
- [ ] Pod `postgresql-0` in `CrashLoopBackOff` or `Error` state
- [ ] Logs show `FATAL: could not open relation` or connection timeout

## Prerequisites

| Requirement | Details |
|-------------|---------|
| `kubectl` | Configured for K3d cluster |
| Database credentials | Available in Vault at `secret/utopia/dev/postgresql` |
| Backup location | PVC `postgresql-backup` or local backup at `/backups/postgresql/` |
| `psql` client | Installed locally or available via `kubectl exec` |

## Diagnosis

### Step 1: Check pod status

```bash
kubectl get pods -n utopia -l app=postgresql
kubectl describe pod postgresql-0 -n utopia
```

Look for: OOMKilled, CrashLoopBackOff, volume mount errors.

### Step 2: Check PostgreSQL logs

```bash
kubectl logs postgresql-0 -n utopia --tail=100
```

Look for: `FATAL`, `PANIC`, `out of disk space`, `corrupted`, `could not write`.

### Step 3: Check PVC status

```bash
kubectl get pvc -n utopia -l app=postgresql
kubectl exec postgresql-0 -n utopia -- df -h /var/lib/postgresql/data
```

Look for: PVC not bound, disk usage > 90%.

### Step 4: Check from application side

```bash
kubectl logs -n utopia -l app=utopia-api --tail=50 | grep -i "database\|connection\|postgres"
```

### Step 5: Verify connectivity

```bash
kubectl exec -n utopia deploy/utopia-api -- pg_isready -h postgresql -p 5432
```

## Resolution Scenarios

---

### Scenario A: Pod Restart (Most Common)

**When**: Pod crashed but data is intact.

**Step 1: Delete the pod** (StatefulSet will recreate it)

```bash
kubectl delete pod postgresql-0 -n utopia
```

**Step 2: Wait for pod to become ready**

```bash
kubectl wait --for=condition=Ready pod/postgresql-0 -n utopia --timeout=120s
```

**Step 3: Verify database is accepting connections**

```bash
kubectl exec postgresql-0 -n utopia -- pg_isready -h localhost -p 5432
```

Expected: `localhost:5432 - accepting connections`

---

### Scenario B: Data Corruption — Restore from Backup

**When**: PostgreSQL reports data corruption or WAL errors.

**Step 1: Scale down the application**

```bash
kubectl scale deployment utopia-api -n utopia --replicas=0
```

**Step 2: Stop the database pod**

```bash
kubectl delete pod postgresql-0 -n utopia
```

**Step 3: Identify most recent backup**

```bash
# List available backups (from CronJob backup)
kubectl exec -n utopia -it $(kubectl get pod -n utopia -l job-name -o jsonpath='{.items[0].metadata.name}' 2>/dev/null || echo "check-manually") -- ls -lt /backups/

# Or check backup PVC directly
kubectl run pg-debug --rm -it --image=alpine -n utopia --overrides='
{
  "spec": {
    "containers": [{
      "name": "pg-debug",
      "image": "alpine",
      "command": ["sh"],
      "volumeMounts": [{
        "name": "backup",
        "mountPath": "/backups"
      }]
    }],
    "volumes": [{
      "name": "backup",
      "persistentVolumeClaim": {
        "claimName": "postgresql-backup"
      }
    }]
  }
}' -- ls -lt /backups/
```

**Step 4: Clear corrupted data**

```bash
# CAUTION: This deletes existing data
kubectl exec postgresql-0 -n utopia -- rm -rf /var/lib/postgresql/data/*
```

**Step 5: Restore from backup**

```bash
# Start a restore job
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: pg-restore
  namespace: utopia
spec:
  template:
    spec:
      containers:
        - name: restore
          image: postgres:16-alpine
          command:
            - sh
            - -c
            - |
              LATEST_BACKUP=$(ls -t /backups/utopia_*.sql.gz | head -1)
              echo "Restoring from: $LATEST_BACKUP"
              gunzip -c "$LATEST_BACKUP" | psql -h postgresql -U utopia -d utopia_dev
              echo "Restore complete"
          env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgresql-credentials
                  key: postgres-password
          volumeMounts:
            - name: backup
              mountPath: /backups
      volumes:
        - name: backup
          persistentVolumeClaim:
            claimName: postgresql-backup
      restartPolicy: Never
  backoffLimit: 1
EOF

# Monitor restore progress
kubectl logs -n utopia job/pg-restore -f
```

**Step 6: Verify restored data**

```bash
kubectl exec postgresql-0 -n utopia -- psql -U utopia -d utopia_dev -c "\dt identity.*"
kubectl exec postgresql-0 -n utopia -- psql -U utopia -d utopia_dev -c "\dt catalog.*"
kubectl exec postgresql-0 -n utopia -- psql -U utopia -d utopia_dev -c "SELECT COUNT(*) FROM identity.users;"
```

**Step 7: Scale application back up**

```bash
kubectl scale deployment utopia-api -n utopia --replicas=1
```

**Step 8: Clean up restore job**

```bash
kubectl delete job pg-restore -n utopia
```

---

### Scenario C: Disk Full

**When**: PVC is out of space.

**Step 1: Check current usage**

```bash
kubectl exec postgresql-0 -n utopia -- df -h /var/lib/postgresql/data
```

**Step 2: Clean WAL files and temporary data**

```bash
kubectl exec postgresql-0 -n utopia -- psql -U utopia -c "SELECT pg_size_pretty(pg_database_size('utopia_dev'));"
kubectl exec postgresql-0 -n utopia -- psql -U utopia -c "VACUUM FULL;"
```

**Step 3: If still insufficient, expand PVC**

```bash
kubectl patch pvc postgresql-data -n utopia -p '{"spec":{"resources":{"requests":{"storage":"15Gi"}}}}'
```

> Note: PVC expansion requires the StorageClass to support `allowVolumeExpansion: true`.

**Step 4: Restart pod to pick up new size**

```bash
kubectl delete pod postgresql-0 -n utopia
kubectl wait --for=condition=Ready pod/postgresql-0 -n utopia --timeout=120s
```

---

### Scenario D: Connection Pool Exhaustion

**When**: Application cannot get connections but PostgreSQL is running.

**Step 1: Check active connections**

```bash
kubectl exec postgresql-0 -n utopia -- psql -U utopia -c "
  SELECT count(*), state 
  FROM pg_stat_activity 
  WHERE datname = 'utopia_dev' 
  GROUP BY state;"
```

**Step 2: Kill idle connections**

```bash
kubectl exec postgresql-0 -n utopia -- psql -U utopia -c "
  SELECT pg_terminate_backend(pid) 
  FROM pg_stat_activity 
  WHERE datname = 'utopia_dev' 
  AND state = 'idle' 
  AND pid <> pg_backend_pid();"
```

**Step 3: Restart application to reset connection pool**

```bash
kubectl rollout restart deployment utopia-api -n utopia
```

---

## Verification

After any resolution scenario:

- [ ] `pg_isready` returns "accepting connections"
- [ ] Application pods are Running and Ready
- [ ] Application logs show successful database queries
- [ ] Grafana PostgreSQL dashboard shows `pg_up == 1`
- [ ] Alert `DatabaseDown` has resolved
- [ ] Run a quick API health check: `curl -k https://api.utopia.local/health/ready`

## Rollback

If a restore makes things worse:

1. Stop the application: `kubectl scale deployment utopia-api -n utopia --replicas=0`
2. Try an older backup (second-to-latest in `/backups/`)
3. If no backup works, re-initialize the database and run EF Core migrations to create a clean schema

## Escalation

| Condition | Action |
|-----------|--------|
| All backups are corrupted | Investigate root cause, check backup CronJob logs |
| PVC cannot be expanded | Create a new PVC and migrate data |
| Issue recurs within 24 hours | Investigate root cause, review resource limits |

## Post-Incident

- [ ] Update this runbook with any corrections
- [ ] File a post-incident review (see [INCIDENT-RESPONSE-PLAN.md](../../04-security/INCIDENT-RESPONSE-PLAN.md))
- [ ] Verify backup CronJob is running correctly
- [ ] Review PostgreSQL resource allocation
- [ ] Add/update monitoring alerts if gaps found

## References

- [STORAGE-BACKUP.md](../../05-infrastructure/STORAGE-BACKUP.md)
- [DATA-ARCHITECTURE.md](../../02-architecture/DATA-ARCHITECTURE.md)
- [RUNBOOK-TEMPLATE.md](./RUNBOOK-TEMPLATE.md)
- [INCIDENT-RESPONSE-PLAN.md](../../04-security/INCIDENT-RESPONSE-PLAN.md)

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
