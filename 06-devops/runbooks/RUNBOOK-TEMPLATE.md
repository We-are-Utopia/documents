# Runbook Template

| Field         | Value                                |
|---------------|--------------------------------------|
| **Version**   | 1.0.0                                |
| **Status**    | Draft                                |
| **Author**    | Vox                                  |
| **Reviewer**  | Vox                                  |
| **Created**   | 2026-03-27                           |
| **Updated**   | 2026-03-27                           |
| **Standard**  | DOCUMENTATION-STANDARD.md; SRE Best Practices |

---

## 1. Purpose

This document provides a standard template for operational runbooks in the Utopia project. All runbooks MUST follow this structure to ensure consistency during incident response.

## 2. When to Create a Runbook

A runbook SHOULD be created for:

- Any scenario that has caused a past incident
- Critical system recovery procedures
- Routine operational tasks that require specific steps
- Any procedure referenced by an alerting rule

## 3. Runbook Template

---

### `[RUNBOOK TITLE]`

| Field | Value |
|-------|-------|
| **Runbook ID** | RB-XXXX |
| **Version** | 1.0.0 |
| **Last Tested** | YYYY-MM-DD |
| **Severity** | SEV-1 / SEV-2 / SEV-3 / SEV-4 |
| **Related Alert** | `AlertName` (link to alert rule) |
| **Estimated Duration** | X minutes |

#### Symptoms

_Describe observable symptoms that indicate this runbook should be used._

- [ ] Symptom 1
- [ ] Symptom 2
- [ ] Symptom 3

#### Prerequisites

_List tools, access, and knowledge required before proceeding._

| Requirement | Details |
|-------------|---------|
| kubectl access | Configured for K3d cluster |
| Database access | PostgreSQL credentials from Vault |
| Permissions | Admin role |

#### Diagnosis

_Steps to confirm the issue and gather context._

```bash
# Step 1: Describe what this checks
<command>

# Step 2: Describe what this checks
<command>
```

#### Resolution Steps

> **IMPORTANT**: Follow steps in order. Do NOT skip steps.

**Step 1: [Action description]**

```bash
<command>
```

Expected output: _describe what success looks like_

**Step 2: [Action description]**

```bash
<command>
```

Expected output: _describe what success looks like_

**Step 3: [Action description]**

```bash
<command>
```

Expected output: _describe what success looks like_

#### Verification

_Steps to confirm the issue is resolved._

- [ ] Verification check 1
- [ ] Verification check 2
- [ ] Monitoring dashboards show normal values

#### Rollback

_If resolution makes things worse, describe how to undo._

```bash
<rollback command>
```

#### Escalation

| Condition | Action |
|-----------|--------|
| Steps fail after 2 attempts | Research external documentation |
| Data loss suspected | Stop and assess before continuing |
| Root cause unknown | Create post-incident review |

#### Post-Incident

- [ ] Update this runbook with any corrections
- [ ] File a post-incident review if SEV-1 or SEV-2
- [ ] Create follow-up tasks to prevent recurrence

---

## 4. Naming Convention

| Format | Example |
|--------|---------|
| `RUNBOOK-<SYSTEM>-<SCENARIO>.md` | `RUNBOOK-DATABASE-FAILOVER.md` |
| | `RUNBOOK-KUBERNETES-NODE-DRAIN.md` |
| | `RUNBOOK-RABBITMQ-QUEUE-STUCK.md` |
| | `RUNBOOK-KEYCLOAK-TOKEN-ISSUES.md` |

## 5. Storage Location

All runbooks are stored in `documents/06-devops/runbooks/`.

## 6. Testing Schedule

Runbooks SHOULD be tested periodically to ensure accuracy:

| Severity | Test Frequency |
|----------|---------------|
| SEV-1 | Every 3 months |
| SEV-2 | Every 6 months |
| SEV-3 / SEV-4 | Annually |

## 7. References

- [DOCUMENTATION-STANDARD.md](../../00-standards/DOCUMENTATION-STANDARD.md)
- [INCIDENT-RESPONSE-PLAN.md](../../04-security/INCIDENT-RESPONSE-PLAN.md)

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
