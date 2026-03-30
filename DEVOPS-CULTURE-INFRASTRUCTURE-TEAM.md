# QUY ĐỊNH VĂN HÓA DEVOPS & NGUYÊN TẮC LÀM VIỆC NỘI BỘ

## TEAM HẠ TẦNG – INFRASTRUCTURE TEAM

| Thông tin | Chi tiết |
|---|---|
| Phiên bản | 1.0 |
| Ngày ban hành | 28/03/2026 |
| Người soạn thảo | [Dương Nhật Khoa] |
| Phê duyệt | [Đoàn Hồng Đăng] |
| Phạm vi áp dụng | DevOps Engineer, SRE, Security Engineer, Infrastructure Engineer |
| Tần suất rà soát | Mỗi quý (Q1, Q2, Q3, Q4) |

---

## MỤC LỤC

1. [Mục đích và phạm vi](#1-mục-đích-và-phạm-vi)
2. [Thành phần team và phân chia trách nhiệm](#2-thành-phần-team-và-phân-chia-trách-nhiệm)
3. [Nguyên tắc văn hóa làm việc](#3-nguyên-tắc-văn-hóa-làm-việc)
4. [Nguyên tắc vận hành hệ thống](#4-nguyên-tắc-vận-hành-hệ-thống)
5. [Nguyên tắc triển khai và thay đổi](#5-nguyên-tắc-triển-khai-và-thay-đổi)
6. [Nguyên tắc giám sát và ứng cứu sự cố](#6-nguyên-tắc-giám-sát-và-ứng-cứu-sự-cố)
7. [Nguyên tắc bảo mật hạ tầng](#7-nguyên-tắc-bảo-mật-hạ-tầng)
8. [Nguyên tắc học tập và cải tiến liên tục](#8-nguyên-tắc-học-tập-và-cải-tiến-liên-tục)
9. [Quy trình phối hợp với các team khác](#9-quy-trình-phối-hợp-với-các-team-khác)
10. [Phụ lục](#10-phụ-lục)

---

## 1. MỤC ĐÍCH VÀ PHẠM VI

### 1.1 Mục đích

Tài liệu này quy định **văn hóa làm việc, nguyên tắc vận hành và quy trình nội bộ** của Team Hạ tầng, nhằm:

- **Thống nhất cách làm việc** giữa các vai trò trong team (DevOps, SRE, SE, Infrastructure).
- **Đảm bảo chất lượng và độ ổn định** của toàn bộ hệ thống hạ tầng.
- **Tạo cơ sở để các team khác** (Development, QA, Product…) tuân theo khi yêu cầu hỗ trợ hoặc phối hợp với Team Hạ tầng.
- **Thúc đẩy cải tiến liên tục** trong quy trình vận hành và triển khai.

### 1.2 Phạm vi áp dụng

- Áp dụng cho **toàn bộ thành viên** thuộc Team Hạ tầng.
- Áp dụng cho **các team khác** khi có yêu cầu liên quan đến hạ tầng, triển khai, bảo mật hạ tầng hoặc vận hành hệ thống.
- Bao gồm tất cả các môi trường: **Development, Staging, UAT, Production**.

### 1.3 Định nghĩa thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| DevOps | Văn hóa và tập hợp thực hành nhằm tăng cường hợp tác giữa phát triển và vận hành |
| CI/CD | Continuous Integration / Continuous Delivery – Tích hợp liên tục / Phân phối liên tục |
| IaC | Infrastructure as Code – Quản lý hạ tầng bằng code |
| SLA | Service Level Agreement – Cam kết mức dịch vụ |
| SLO | Service Level Objective – Mục tiêu mức dịch vụ |
| SLI | Service Level Indicator – Chỉ số đo lường mức dịch vụ |
| RCA | Root Cause Analysis – Phân tích nguyên nhân gốc rễ |
| MTTR | Mean Time To Recovery – Thời gian trung bình phục hồi sự cố |
| MTTD | Mean Time To Detect – Thời gian trung bình phát hiện sự cố |
| SOP | Standard Operating Procedure – Quy trình vận hành tiêu chuẩn |
| Runbook | Tài liệu hướng dẫn xử lý từng loại sự cố cụ thể |
| Postmortem | Báo cáo phân tích sau sự cố |

---

## 2. THÀNH PHẦN TEAM VÀ PHÂN CHIA TRÁCH NHIỆM

### 2.1 Cơ cấu vai trò

Team Hạ tầng bao gồm 4 vai trò chính:

| Vai trò | Viết tắt | Mô tả ngắn |
|---|---|---|
| DevOps Engineer | DevOps | Xây dựng và vận hành CI/CD pipeline, tự động hóa quy trình |
| Site Reliability Engineer | SRE | Đảm bảo độ tin cậy, ổn định và hiệu năng của hệ thống |
| Security Engineer | SE | Bảo mật hạ tầng, chính sách truy cập, kiểm tra lỗ hổng |
| Infrastructure Engineer | Infra | Thiết kế, xây dựng và quản lý hạ tầng vật lý/cloud |

### 2.2 Ma trận trách nhiệm chi tiết (RACI)

> **R** = Responsible (Thực hiện) | **A** = Accountable (Chịu trách nhiệm cuối cùng) | **C** = Consulted (Được tham vấn) | **I** = Informed (Được thông báo)

| Hạng mục công việc | DevOps | SRE | SE | Infra |
|---|---|---|---|---|
| **CI/CD Pipeline** | R, A | C | C | I |
| Thiết kế pipeline | R | C | C | I |
| Duy trì và tối ưu pipeline | R | C | I | I |
| Tích hợp security scan vào pipeline | R | I | A | I |
| **Hạ tầng** | C | C | C | R, A |
| Thiết kế kiến trúc hạ tầng | C | C | C | R, A |
| Provisioning server / cloud resource | I | I | C | R |
| Quản lý network, firewall, DNS | I | I | C | R |
| Quản lý IaC (Terraform, Ansible…) | R | I | C | A |
| **Độ ổn định hệ thống** | C | R, A | I | C |
| Định nghĩa SLO/SLI | I | R, A | I | C |
| Thiết kế và vận hành monitoring | C | R | I | C |
| On-call và ứng cứu sự cố | C | R | I | C |
| Capacity planning | I | R | I | C |
| **Bảo mật hạ tầng** | C | C | R, A | C |
| Chính sách truy cập (IAM, RBAC) | I | C | R | C |
| Quản lý secrets và certificates | C | I | R | C |
| Vulnerability scanning | I | I | R | C |
| Audit và compliance | I | C | R | C |
| **Triển khai ứng dụng** | R, A | C | C | I |
| Deploy lên các môi trường | R | C | I | I |
| Quản lý container/orchestration | R | C | I | C |
| Rollback khi có sự cố deploy | R | R | I | I |

### 2.3 Mô hình kỹ năng T-shaped

Mỗi thành viên trong team **PHẢI** có:

- **Chuyên sâu (Deep skill):** Thành thạo lĩnh vực chính theo vai trò được phân công.
- **Kiến thức tổng quát (Broad knowledge):** Hiểu biết cơ bản về các lĩnh vực của thành viên khác trong team để có thể hỗ trợ khi cần.

**Yêu cầu kiến thức tổng quát tối thiểu theo vai trò:**

| Vai trò | Chuyên sâu | Kiến thức tổng quát bắt buộc |
|---|---|---|
| DevOps | CI/CD, Container, Automation | Networking cơ bản, Security cơ bản, Monitoring cơ bản |
| SRE | Monitoring, Incident Response, Performance | CI/CD cơ bản, Networking cơ bản, Scripting |
| SE | Security Policy, Vulnerability, Compliance | CI/CD cơ bản, Infrastructure cơ bản, Monitoring cơ bản |
| Infra | Network, Server, Cloud, Storage | CI/CD cơ bản, Security cơ bản, Monitoring cơ bản |

---

## 3. NGUYÊN TẮC VĂN HÓA LÀM VIỆC

### 3.1 Nguyên tắc 1 – Văn hóa không đổ lỗi (Blameless Culture)

**Quy định:**

Khi xảy ra sự cố trong quá trình vận hành hoặc triển khai:

1. **KHÔNG** tìm cá nhân để quy trách nhiệm hoặc đổ lỗi.
2. **CẢ TEAM** cùng chịu trách nhiệm về sự cố.
3. **TẬP TRUNG** vào việc cải thiện quy trình, không phải trách con người.

**Khi sự cố xảy ra, team phải tự hỏi:**

| ❌ KHÔNG hỏi | ✅ PHẢI hỏi |
|---|---|
| "Ai gây ra lỗi này?" | "Vì sao lỗi này xảy ra?" |
| "Ai chịu trách nhiệm?" | "Quy trình nào cho phép lỗi này lọt qua?" |
| "Ai deploy sai?" | "Cần cải tiến gì để ngăn lỗi tương tự?" |

**Áp dụng cụ thể:**

- Mọi sự cố Production **PHẢI** có Postmortem Report (xem Mục 6.4) với tinh thần blameless.
- Postmortem **KHÔNG ĐƯỢC** nhắc tên cá nhân theo hướng quy lỗi. Chỉ mô tả hành động, timeline và quy trình.
- Thành viên báo cáo sự cố sớm **ĐƯỢC** ghi nhận tích cực, không bị phạt.

### 3.2 Nguyên tắc 2 – Tôn trọng lẫn nhau

**Quy định:**

- Tôn trọng chuyên môn và góc nhìn của mỗi vai trò trong team.
- Khi có ý kiến khác nhau về giải pháp kỹ thuật:
  1. Trình bày quan điểm kèm **dữ liệu hoặc bằng chứng** (benchmark, metrics, kinh nghiệm thực tế).
  2. Thảo luận dựa trên **kỹ thuật**, không dựa trên cấp bậc hoặc thâm niên.
  3. Nếu không thống nhất được, **người chịu trách nhiệm (Accountable)** theo ma trận RACI sẽ đưa ra quyết định cuối cùng.
  4. Sau khi quyết định được đưa ra, **toàn bộ team cam kết thực hiện** (Disagree and Commit).

### 3.3 Nguyên tắc 3 – Ra quyết định dựa trên dữ liệu (Data-Driven Decisions)

**Quy định:**

Mọi quyết định kỹ thuật quan trọng **PHẢI** được hỗ trợ bởi dữ liệu thực tế:

| Loại quyết định | Dữ liệu yêu cầu |
|---|---|
| Chọn công cụ/công nghệ mới | Benchmark, so sánh tính năng, chi phí, cộng đồng hỗ trợ |
| Thay đổi kiến trúc hạ tầng | Capacity metrics, growth forecast, cost analysis |
| Điều chỉnh alert/monitoring | Lịch sử alert, false positive rate, incident data |
| Mở rộng hạ tầng | Traffic trend, resource utilization (CPU, Memory, Disk, Network) |
| Rollback sau deploy | Error rate, latency metrics, user impact data |

**Cấm:**

- Ra quyết định dựa trên cảm tính hoặc "vì trước giờ làm vậy".
- Thay đổi hạ tầng Production mà không có dữ liệu chứng minh sự cần thiết.

### 3.4 Nguyên tắc 4 – Minh bạch và chia sẻ thông tin

**Quy định:**

- Mọi thay đổi hạ tầng, sự cố, quyết định kỹ thuật **PHẢI** được thông báo công khai trong kênh giao tiếp chung của team.
- **KHÔNG** giữ thông tin quan trọng cho riêng cá nhân.
- Tài liệu kỹ thuật **PHẢI** được cập nhật khi có thay đổi (xem Mục 8.2).

**Kênh giao tiếp:**

| Loại thông tin | Kênh | Thời gian thông báo |
|---|---|---|
| Sự cố Production | Channel #incident | Ngay lập tức |
| Thay đổi hạ tầng có kế hoạch | Channel #infra-changes | Trước ít nhất 24 giờ |
| Thay đổi khẩn cấp | Channel #incident + gọi điện on-call | Ngay lập tức |
| Cập nhật tài liệu | Channel #team-infra | Trong ngày |
| Bài học rút ra (Lessons Learned) | Channel #team-infra + Wiki | Trong 48 giờ sau sự cố |

---

## 4. NGUYÊN TẮC VẬN HÀNH HỆ THỐNG

### 4.1 Infrastructure as Code (IaC)

**Quy định bắt buộc:**

1. **100% hạ tầng** phải được quản lý bằng code (Terraform, Ansible, Helm, CloudFormation hoặc tương đương).
2. **NGHIÊM CẤM** thay đổi hạ tầng thủ công (manual change) trên môi trường Staging và Production, trừ trường hợp khẩn cấp (xem Mục 4.2).
3. Tất cả IaC code **PHẢI** được lưu trữ trong hệ thống version control (Git).
4. Mọi thay đổi IaC **PHẢI** qua quy trình review trước khi apply:

**Quy trình thay đổi IaC:**

```
Tạo branch → Viết/sửa IaC code → Tạo Pull Request → Review (tối thiểu 1 người) → Approve → Merge → Apply tự động hoặc có kiểm soát
```

**Cấu trúc thư mục IaC tiêu chuẩn (tham khảo):**

```
infrastructure/
├── terraform/
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   ├── modules/
│   │   ├── networking/
│   │   ├── compute/
│   │   ├── database/
│   │   └── security/
│   └── backend.tf
├── ansible/
│   ├── playbooks/
│   ├── roles/
│   └── inventories/
├── helm/
│   └── charts/
└── README.md
```

### 4.2 Quy định thay đổi thủ công khẩn cấp

Khi **bắt buộc** phải thay đổi thủ công (hotfix hạ tầng khẩn cấp):

1. **PHẢI** có ít nhất 2 người chứng kiến (pair operation hoặc screen share).
2. **PHẢI** ghi lại toàn bộ thao tác vào log (terminal recording hoặc ghi chú chi tiết).
3. **PHẢI** tạo ticket và cập nhật IaC code trong vòng **24 giờ** sau khi xử lý xong.
4. **PHẢI** báo cáo trong kênh #incident với nội dung:
   - Lý do thay đổi thủ công
   - Chi tiết thao tác đã thực hiện
   - Kế hoạch đưa vào IaC

### 4.3 Quản lý môi trường

**Các môi trường bắt buộc:**

| Môi trường | Mục đích | Ai có quyền truy cập | Ai có quyền deploy |
|---|---|---|---|
| **Development (Dev)** | Phát triển và test nội bộ | Tất cả team | DevOps, Developer |
| **Staging** | Test tích hợp, UAT | Team Hạ tầng, QA | DevOps (qua pipeline) |
| **Production** | Hệ thống thật, phục vụ người dùng | Team Hạ tầng (hạn chế) | DevOps (qua pipeline, có approval) |

**Nguyên tắc:**

- Staging **PHẢI** phản ánh gần nhất có thể cấu hình của Production (same architecture, scaled down resources).
- **NGHIÊM CẤM** sử dụng dữ liệu thật (PII) của Production trên các môi trường khác, trừ khi đã được anonymize và được SE phê duyệt.
- Mỗi môi trường **PHẢI** có cấu hình tách biệt (separate credentials, endpoints, resources).

### 4.4 Configuration as Code

**Quy định:**

- Tất cả cấu hình hệ thống (config files, environment variables, feature flags) **PHẢI** được quản lý bằng code và version control.
- **NGHIÊM CẤM** hardcode secrets, credentials, API keys trong source code hoặc IaC code.
- Secrets **PHẢI** được quản lý qua công cụ chuyên dụng (Vault, AWS Secrets Manager, Azure Key Vault, hoặc tương đương).

**Phân loại cấu hình:**

| Loại cấu hình | Cách quản lý | Ví dụ |
|---|---|---|
| Application config | ConfigMap / config files trong Git | Database connection string (không chứa password), feature flags |
| Infrastructure config | IaC code trong Git | Server spec, network config, firewall rules |
| Secrets | Secret management tool | Database password, API keys, certificates |
| Runtime config | Environment variables qua deployment tool | Log level, debug mode |

### 4.5 Tự động hóa (Automate Everything)

**Nguyên tắc:**

Mọi tác vụ **lặp lại hơn 2 lần** phải được xem xét để tự động hóa.

**Các hạng mục bắt buộc tự động hóa:**

| Hạng mục | Trạng thái yêu cầu | Công cụ tham khảo |
|---|---|---|
| Build và test | Bắt buộc tự động | Jenkins, GitHub Actions, GitLab CI |
| Deploy lên mọi môi trường | Bắt buộc tự động | ArgoCD, Jenkins, Spinnaker |
| Provisioning hạ tầng | Bắt buộc tự động | Terraform, Ansible, Pulumi |
| Security scanning | Bắt buộc tự động | Trivy, SonarQube, OWASP ZAP |
| Certificate renewal | Bắt buộc tự động | Cert-manager, Let's Encrypt |
| Backup | Bắt buộc tự động | Cron job, Velero, cloud-native backup |
| Scaling | Khuyến khích tự động | HPA, KEDA, Auto Scaling Group |
| Alerting | Bắt buộc tự động | Prometheus Alertmanager, PagerDuty |

---

## 5. NGUYÊN TẮC TRIỂN KHAI VÀ THAY ĐỔI

### 5.1 Quy trình CI/CD

**Pipeline tiêu chuẩn bắt buộc:**

```
Source Code Push
    │
    ▼
[1] Build ──────────── Compile, resolve dependencies
    │
    ▼
[2] Unit Test ──────── Chạy unit tests, coverage check
    │
    ▼
[3] Static Analysis ── Code quality + security scan (SonarQube, Trivy)
    │
    ▼
[4] Build Artifact ─── Docker image, package
    │
    ▼
[5] Deploy to Dev ──── Tự động
    │
    ▼
[6] Integration Test ─ Smoke test, API test trên Dev
    │
    ▼
[7] Deploy to Staging ─ Tự động hoặc manual trigger
    │
    ▼
[8] UAT / QA Testing ── Manual + automated testing
    │
    ▼
[9] Approval Gate ──── Yêu cầu approval trước Production
    │
    ▼
[10] Deploy to Prod ── Chiến lược deploy an toàn (xem 5.2)
    │
    ▼
[11] Post-deploy ───── Health check, smoke test, monitoring
```

**Quy định pipeline:**

- Pipeline **PHẢI** fail-fast: nếu bất kỳ step nào fail, pipeline dừng ngay và thông báo.
- Deploy Production **PHẢI** có ít nhất 1 approval từ DevOps hoặc SRE.
- Pipeline code **PHẢI** được version control và review như application code.

### 5.2 Chiến lược triển khai an toàn

**Các chiến lược được phép sử dụng trên Production:**

| Chiến lược | Mô tả | Khi nào dùng |
|---|---|---|
| **Blue-Green** | 2 môi trường song song, chuyển traffic khi sẵn sàng | Thay đổi lớn, cần rollback nhanh |
| **Canary** | Triển khai cho % nhỏ người dùng trước, mở rộng dần | Tính năng mới cần kiểm chứng |
| **Rolling Update** | Cập nhật từng nhóm instance dần dần | Thay đổi nhỏ, ít rủi ro |
| **Feature Flag** | Bật/tắt tính năng bằng config, không cần deploy lại | Tính năng cần kiểm soát linh hoạt |

**Quy trình Canary Deployment mẫu:**

```
Bước 1: Deploy phiên bản mới cho 5% traffic
Bước 2: Theo dõi metrics trong 15-30 phút
         ├── Error rate tăng > ngưỡng → Rollback ngay
         └── Metrics bình thường → Tiếp tục
Bước 3: Mở rộng lên 25% traffic
Bước 4: Theo dõi metrics trong 30-60 phút
Bước 5: Mở rộng lên 50% traffic
Bước 6: Theo dõi metrics trong 1-2 giờ
Bước 7: Mở rộng lên 100% traffic
```

**NGHIÊM CẤM:**

- Deploy trực tiếp 100% traffic trên Production mà không qua chiến lược an toàn (trừ hotfix khẩn cấp đã được approval).
- Deploy ngoài maintenance window mà không có approval từ SRE Lead hoặc Team Lead.

### 5.3 Quy trình Rollback

**Quy định:**

- Mọi deployment **PHẢI** có kế hoạch rollback trước khi thực hiện.
- Rollback **PHẢI** có thể thực hiện trong vòng **15 phút** sau khi phát hiện vấn đề.
- Sau rollback, **PHẢI** tạo incident report (xem Mục 6.4).

**Tiêu chí kích hoạt rollback tự động:**

| Metric | Ngưỡng rollback |
|---|---|
| Error rate (5xx) | Tăng > 5% so với baseline |
| Latency P99 | Tăng > 50% so với baseline |
| Health check | Fail liên tiếp 3 lần |

### 5.4 Change Management – Quản lý thay đổi

**Phân loại thay đổi:**

| Loại | Mô tả | Yêu cầu |
|---|---|---|
| **Standard** | Thay đổi đã có quy trình, rủi ro thấp | Không cần approval đặc biệt |
| **Normal** | Thay đổi mới, rủi ro trung bình | Cần PR review + approval |
| **Emergency** | Hotfix khẩn cấp cho Production | Approval nhanh từ Team Lead, tài liệu hóa sau |

**Mọi thay đổi Normal và Emergency PHẢI có:**

1. Mô tả thay đổi (What)
2. Lý do thay đổi (Why)
3. Phạm vi ảnh hưởng (Impact)
4. Kế hoạch rollback (Rollback plan)
5. Checklist kiểm tra sau deploy (Post-deploy checklist)

---

## 6. NGUYÊN TẮC GIÁM SÁT VÀ ỨNG CỨU SỰ CỐ

### 6.1 Giám sát toàn diện (Monitor Everything)

**Quy định:**

Mọi hệ thống trên Staging và Production **PHẢI** có monitoring đầy đủ.

**Các hạng mục giám sát bắt buộc:**

| Layer | Metrics bắt buộc | Công cụ tham khảo |
|---|---|---|
| **Infrastructure** | CPU, Memory, Disk I/O, Network I/O | Prometheus, Grafana, CloudWatch |
| **Application** | Request rate, Error rate, Latency (P50/P95/P99), Throughput | Prometheus, Datadog, New Relic |
| **Database** | Query latency, Connection pool, Slow queries, Replication lag | Database-specific exporters |
| **Message Queue** | Queue depth, Consumer lag, Processing rate | RabbitMQ/Kafka exporters |
| **Security** | Failed login attempts, Unauthorized access, Certificate expiry | SIEM, custom alerts |
| **Business** | Transaction success rate, User-facing errors | Application metrics |

### 6.2 Telemetry – Ba trụ cột

Team **PHẢI** triển khai đầy đủ 3 trụ cột quan sát (Observability):

| Trụ cột | Mục đích | Yêu cầu |
|---|---|---|
| **Logs** | Ghi nhận sự kiện chi tiết | Structured logging (JSON), centralized log (ELK/Loki), retention tối thiểu 30 ngày |
| **Metrics** | Đo lường hiệu năng theo thời gian | Time-series database (Prometheus), dashboard (Grafana), retention tối thiểu 90 ngày |
| **Traces** | Theo dõi luồng xử lý request | Distributed tracing (Jaeger/Zipkin/OpenTelemetry), sampling tối thiểu 10% |

### 6.3 Alerting – Phát hiện trước người dùng

**Nguyên tắc cốt lõi:**

> Hệ thống **PHẢI** phát hiện sự cố trước khi người dùng cuối nhận ra.

**Quy định alert:**

| Mức độ | Định nghĩa | Phản hồi yêu cầu | Kênh thông báo |
|---|---|---|---|
| **P1 – Critical** | Hệ thống Production down hoặc ảnh hưởng > 50% người dùng | Phản hồi trong 15 phút | Phone call + SMS + Chat |
| **P2 – High** | Suy giảm hiệu năng hoặc ảnh hưởng 10-50% người dùng | Phản hồi trong 30 phút | SMS + Chat |
| **P3 – Medium** | Vấn đề nhỏ, không ảnh hưởng trải nghiệm người dùng ngay | Phản hồi trong 4 giờ | Chat |
| **P4 – Low** | Informational, cần theo dõi | Xử lý trong giờ làm việc | Chat / Email |

**Quy định chống alert fatigue (mệt mỏi do alert):**

- Mỗi alert **PHẢI** gắn với một hành động cụ thể (actionable alert). Alert không có hành động → xóa hoặc giảm severity.
- Review alert rules **mỗi tháng**: loại bỏ alert không còn giá trị, điều chỉnh ngưỡng.
- False positive rate **PHẢI** dưới 20%. Nếu vượt → điều chỉnh ngay.

### 6.4 Quy trình ứng cứu sự cố (Incident Response)

**Quy trình:**

```
[1] Phát hiện ──────── Alert tự động hoặc báo cáo từ người dùng/team khác
    │
    ▼
[2] Phân loại ──────── Xác định severity (P1/P2/P3/P4)
    │
    ▼
[3] Thông báo ──────── Thông báo qua kênh phù hợp theo severity
    │
    ▼
[4] Ứng cứu ────────── On-call engineer bắt đầu xử lý
    │                   Escalate nếu cần
    │
    ▼
[5] Khắc phục ──────── Áp dụng fix/workaround
    │
    ▼
[6] Xác nhận ────────── Verify hệ thống hoạt động bình thường
    │
    ▼
[7] Thông báo kết thúc ─ Cập nhật kênh giao tiếp
    │
    ▼
[8] Postmortem ──────── Viết báo cáo phân tích (trong 48h với P1/P2)
```

### 6.5 Postmortem Report – Báo cáo phân tích sau sự cố

**Bắt buộc với sự cố P1 và P2. Khuyến khích với P3.**

**Template Postmortem:**

```
POSTMORTEM REPORT
=================

Tiêu đề: [Mô tả ngắn sự cố]
Ngày xảy ra: [DD/MM/YYYY]
Severity: [P1/P2/P3]
Thời gian phát hiện (MTTD): [X phút]
Thời gian phục hồi (MTTR): [X phút]

1. TÓM TẮT SỰ CỐ
   [Mô tả ngắn gọn sự cố và impact]

2. TIMELINE
   HH:MM - [Sự kiện]
   HH:MM - [Sự kiện]
   ...

3. NGUYÊN NHÂN GỐC RỄ (ROOT CAUSE)
   [Mô tả nguyên nhân thực sự, không phải biểu hiện]

4. IMPACT
   - Số người dùng bị ảnh hưởng: [X]
   - Thời gian downtime: [X phút]
   - Dữ liệu bị mất (nếu có): [mô tả]

5. HÀNH ĐỘNG KHẮC PHỤC ĐÃ THỰC HIỆN
   [Liệt kê các bước đã làm để khắc phục]

6. BÀI HỌC RÚT RA
   Điều gì hoạt động tốt:
   - [...]
   Điều gì cần cải thiện:
   - [...]

7. ACTION ITEMS
   | Hành động | Người phụ trách | Deadline | Trạng thái |
   |-----------|----------------|----------|------------|
   | [...]     | [...]          | [...]    | [...]      |
```

### 6.6 Root Cause Analysis (RCA)

**Quy định:**

- **KHÔNG** chỉ sửa biểu hiện lỗi. **PHẢI** tìm nguyên nhân gốc rễ.
- Sử dụng kỹ thuật **"5 Whys"** hoặc **Fishbone Diagram** để phân tích.

**Ví dụ phân tích 5 Whys:**

```
Vấn đề: API gateway trả về timeout cho 30% requests

Why 1: Vì sao API timeout?
→ Vì database query mất > 10 giây

Why 2: Vì sao database query chậm?
→ Vì table orders có 50 triệu records mà không có index phù hợp

Why 3: Vì sao không có index?
→ Vì không có quy trình review database schema khi data tăng

Why 4: Vì sao không có quy trình review?
→ Vì chưa thiết lập monitoring slow query và capacity review

Why 5: Vì sao chưa thiết lập?
→ Vì không có checklist bắt buộc khi đưa service lên Production

Nguyên nhân gốc rễ: Thiếu checklist Production Readiness
Giải pháp:
  - Thêm index cho table orders (ngắn hạn)
  - Thiết lập slow query alert (ngắn hạn)
  - Xây dựng Production Readiness Checklist (dài hạn)
  - Lên lịch capacity review hàng tháng (dài hạn)
```

---

## 7. NGUYÊN TẮC BẢO MẬT HẠ TẦNG

### 7.1 Nguyên tắc Least Privilege

- Mọi tài khoản (người dùng, service account) chỉ được cấp **quyền tối thiểu cần thiết** để thực hiện công việc.
- **NGHIÊM CẤM** sử dụng tài khoản root/admin cho công việc hàng ngày.
- Quyền truy cập **PHẢI** được review mỗi quý và thu hồi khi không còn cần thiết.

### 7.2 Kiểm soát truy cập môi trường Production

**Quy định:**

| Hạng mục | Quy định |
|---|---|
| Truy cập SSH/RDP | Chỉ qua bastion host, có log audit |
| Database access | Chỉ qua công cụ quản lý có audit trail, không direct connect |
| Deploy | Chỉ qua CI/CD pipeline (không deploy thủ công) |
| Đọc logs | Qua centralized logging system |
| Thay đổi config | Qua IaC và pull request, không sửa trực tiếp |

### 7.3 Quản lý Secrets

**Quy định bắt buộc:**

1. Secrets **NGHIÊM CẤM** lưu trong source code, config file, hoặc chat/email.
2. Secrets **PHẢI** lưu trong secret management tool (Vault, AWS Secrets Manager, Azure Key Vault).
3. Secrets **PHẢI** được rotate định kỳ theo bảng sau:

| Loại secret | Tần suất rotate |
|---|---|
| Database password | Mỗi 90 ngày |
| API keys | Mỗi 90 ngày |
| SSH keys | Mỗi 180 ngày |
| TLS certificates | Trước khi hết hạn (tự động qua cert-manager) |
| Service account tokens | Mỗi 90 ngày |

### 7.4 Security Scanning

**Quy định:**

| Loại scan | Tần suất | Ai chịu trách nhiệm |
|---|---|---|
| Container image scan | Mỗi lần build (trong CI/CD pipeline) | DevOps + SE |
| Infrastructure vulnerability scan | Hàng tuần | SE |
| Dependency scan | Mỗi lần build + hàng tuần | DevOps + SE |
| Penetration testing | Mỗi quý hoặc trước release lớn | SE (hoặc bên thứ ba) |
| Compliance audit | Mỗi 6 tháng | SE |

### 7.5 Audit và Logging bảo mật

- Mọi truy cập vào hệ thống Production **PHẢI** được ghi log audit.
- Audit log **PHẢI** được lưu trữ tối thiểu **1 năm** và **KHÔNG** được phép chỉnh sửa hoặc xóa.
- Review audit log **mỗi tháng** để phát hiện truy cập bất thường.

---

## 8. NGUYÊN TẮC HỌC TẬP VÀ CẢI TIẾN LIÊN TỤC

### 8.1 Learn Once and Share

**Quy định:**

- Khi phát hiện hoặc giải quyết một vấn đề kỹ thuật, thành viên **PHẢI** chia sẻ bài học cho toàn team.
- Hình thức chia sẻ:
  - Cập nhật **Runbook** nếu liên quan đến vận hành.
  - Viết **Knowledge Base article** trên Wiki nội bộ.
  - Chia sẻ trong buổi **Tech Sharing** hàng tuần/hai tuần.

**Mục tiêu: Không lặp lại cùng một sai lầm hai lần.**

### 8.2 Tài liệu hóa bắt buộc

**Các loại tài liệu team PHẢI duy trì:**

| Loại tài liệu | Mô tả | Tần suất cập nhật |
|---|---|---|
| **Runbook** | Hướng dẫn xử lý từng loại sự cố cụ thể | Cập nhật sau mỗi sự cố liên quan |
| **Architecture Diagram** | Sơ đồ kiến trúc hệ thống hiện tại | Cập nhật khi có thay đổi kiến trúc |
| **SOP** | Quy trình vận hành tiêu chuẩn | Review mỗi quý |
| **Onboarding Guide** | Hướng dẫn cho thành viên mới | Review mỗi 6 tháng |
| **Disaster Recovery Plan** | Kế hoạch phục hồi thảm họa | Review mỗi 6 tháng, test mỗi năm |
| **Postmortem Reports** | Báo cáo phân tích sau sự cố | Viết trong 48h sau sự cố P1/P2 |

**Quy định:**

- Tài liệu lỗi thời **nguy hiểm hơn** không có tài liệu. Team Lead chịu trách nhiệm đảm bảo tài liệu luôn cập nhật.
- Mỗi tài liệu **PHẢI** có metadata: ngày tạo, ngày cập nhật cuối, người chịu trách nhiệm.

### 8.3 Architecture Review

**Quy định:**

- Mọi thay đổi kiến trúc hạ tầng đáng kể **PHẢI** qua Architecture Review trước khi triển khai.
- Thành phần review: tối thiểu 1 SRE + 1 Infra + 1 SE.
- Kết quả review **PHẢI** được ghi lại dưới dạng **ADR (Architecture Decision Record)**.

### 8.4 Capacity Planning và Scale

**Quy định:**

- SRE và Infra **PHẢI** thực hiện capacity review **mỗi tháng**.
- Hệ thống **PHẢI** được thiết kế có khả năng mở rộng (scale) khi tải tăng.

**Các phương pháp scaling được áp dụng:**

| Phương pháp | Mô tả | Áp dụng khi |
|---|---|---|
| Horizontal Scaling | Thêm instance/node | Tăng lượng request |
| Vertical Scaling | Tăng resource (CPU/RAM) | Workload cần tính toán nặng |
| Auto Scaling | Tự động scale theo metrics | Traffic không đều, có peak hours |
| Load Balancing | Phân tải giữa các instance | Tất cả hệ thống Production |

### 8.5 Hoạt động cải tiến định kỳ

| Hoạt động | Tần suất | Mục đích |
|---|---|---|
| **Sprint Retrospective** | Mỗi 2 tuần | Rà soát quy trình làm việc, tìm điểm cải tiến |
| **Tech Sharing** | Mỗi 2 tuần | Chia sẻ kiến thức kỹ thuật trong team |
| **SOP Review** | Mỗi quý | Rà soát và cập nhật quy trình vận hành |
| **Disaster Recovery Drill** | Mỗi 6 tháng | Diễn tập phục hồi thảm họa |
| **Alert Review** | Mỗi tháng | Rà soát và tối ưu alerting rules |
| **Security Review** | Mỗi quý | Rà soát chính sách bảo mật và compliance |
| **Capacity Review** | Mỗi tháng | Đánh giá tài nguyên và dự báo tăng trưởng |

---

## 9. QUY TRÌNH PHỐI HỢP VỚI CÁC TEAM KHÁC

### 9.1 Nguyên tắc chung

Khi các team khác (Development, QA, Product…) có yêu cầu liên quan đến hạ tầng:

1. **PHẢI** gửi yêu cầu qua kênh chính thức (ticket system / form yêu cầu).
2. **KHÔNG** nhận yêu cầu qua tin nhắn cá nhân hoặc lời nói trực tiếp mà không có ticket.
3. Yêu cầu **PHẢI** có đầy đủ thông tin theo template (xem 9.2).

### 9.2 Template yêu cầu hạ tầng

**Các team khác PHẢI cung cấp đầy đủ thông tin sau khi gửi yêu cầu:**

```
YÊU CẦU HẠ TẦNG
================

1. Thông tin chung
   - Team yêu cầu: [Tên team]
   - Người yêu cầu: [Họ tên]
   - Ngày yêu cầu: [DD/MM/YYYY]
   - Mức độ ưu tiên: [Cao / Trung bình / Thấp]
   - Ngày cần hoàn thành: [DD/MM/YYYY]

2. Loại yêu cầu
   [ ] Cấp mới tài nguyên (server, database, storage…)
   [ ] Thay đổi cấu hình hạ tầng
   [ ] Cấp quyền truy cập
   [ ] Hỗ trợ CI/CD pipeline
   [ ] Hỗ trợ xử lý sự cố
   [ ] Khác: [mô tả]

3. Mô tả chi tiết
   [Mô tả yêu cầu cụ thể, bao gồm bối cảnh và mục đích]

4. Thông tin kỹ thuật
   - Môi trường: [Dev / Staging / Production]
   - Service/Application liên quan: [Tên service]
   - Resource cần thiết (nếu có): [CPU, RAM, Disk, Network…]
   - Kết nối cần thiết (nếu có): [Database, API, external service…]

5. Ảnh hưởng nếu không thực hiện
   [Mô tả impact nếu yêu cầu không được đáp ứng]
```

### 9.3 SLA xử lý yêu cầu

| Mức độ ưu tiên | Thời gian phản hồi | Thời gian xử lý |
|---|---|---|
| **Cao** (ảnh hưởng Production) | 1 giờ | 4 giờ |
| **Trung bình** (ảnh hưởng Staging/Dev) | 4 giờ | 2 ngày làm việc |
| **Thấp** (yêu cầu mới, không khẩn cấp) | 1 ngày làm việc | 5 ngày làm việc |

### 9.4 Quy định khi các team khác cần deploy

- Deploy **CHỈ** được thực hiện qua CI/CD pipeline do Team Hạ tầng quản lý.
- Các team khác **KHÔNG** được tự ý truy cập hạ tầng Production.
- Nếu cần deploy ngoài giờ hoặc ngoài lịch, **PHẢI** có approval từ Team Hạ tầng trước ít nhất 4 giờ.

---

## 10. PHỤ LỤC

### 10.1 Danh sách công cụ tiêu chuẩn

| Lĩnh vực | Công cụ | Mục đích |
|---|---|---|
| **IaC** | Terraform | Provisioning hạ tầng cloud |
| **Configuration** | Ansible | Configuration management |
| **Container** | Docker | Container runtime |
| **Orchestration** | Kubernetes (K3s/K8s) | Container orchestration |
| **CI/CD** | Jenkins / GitHub Actions | Build và deploy pipeline |
| **GitOps** | ArgoCD | Continuous deployment |
| **Monitoring** | Prometheus + Grafana | Metrics và dashboard |
| **Logging** | ELK Stack / Loki | Centralized logging |
| **Tracing** | Jaeger / OpenTelemetry | Distributed tracing |
| **Secret Management** | HashiCorp Vault | Quản lý secrets |
| **Security Scan** | Trivy, SonarQube | Vulnerability scanning |
| **Alerting** | Alertmanager / PagerDuty | Alert routing và escalation |

### 10.2 Checklist Production Readiness

Trước khi đưa bất kỳ service nào lên Production, **PHẢI** hoàn thành checklist sau:

| # | Hạng mục | Xác nhận |
|---|---|---|
| 1 | CI/CD pipeline hoạt động đầy đủ | ☐ |
| 2 | Monitoring và alerting đã thiết lập | ☐ |
| 3 | Logging đã tích hợp (structured + centralized) | ☐ |
| 4 | Health check endpoint hoạt động | ☐ |
| 5 | Rollback plan đã chuẩn bị | ☐ |
| 6 | Load testing đã thực hiện | ☐ |
| 7 | Security scan đã pass | ☐ |
| 8 | Secrets được quản lý qua secret manager | ☐ |
| 9 | Runbook đã viết cho các sự cố phổ biến | ☐ |
| 10 | Backup và restore đã test | ☐ |
| 11 | Resource limits (CPU/Memory) đã set | ☐ |
| 12 | Auto-scaling đã cấu hình (nếu cần) | ☐ |
| 13 | TLS/SSL đã cấu hình | ☐ |
| 14 | Access control đã thiết lập theo least privilege | ☐ |
| 15 | Architecture review đã hoàn thành | ☐ |

### 10.3 Lịch trình On-call

**Quy định:**

- On-call rotation áp dụng cho DevOps và SRE.
- Mỗi ca on-call kéo dài **1 tuần**, luân phiên giữa các thành viên.
- Người on-call **PHẢI** phản hồi alert P1/P2 trong thời gian quy định (xem Mục 6.3).
- Sau ca on-call, thành viên **ĐƯỢC** nghỉ bù nếu phải xử lý sự cố ngoài giờ làm việc.
- On-call handover **PHẢI** bao gồm: tóm tắt sự cố đã xử lý, các vấn đề đang theo dõi, lưu ý đặc biệt.

---

## LỊCH SỬ THAY ĐỔI

| Phiên bản | Ngày | Người thay đổi | Mô tả thay đổi |
|---|---|---|---|
| 1.0 | 28/03/2026 | [Dương Nhật Khoa] | Phiên bản đầu tiên |

---

> **Lưu ý:** Tài liệu này là tài liệu sống (living document), được rà soát và cập nhật mỗi quý. Mọi thành viên Team Hạ tầng có trách nhiệm đề xuất cải tiến khi phát hiện nội dung chưa phù hợp.
