# Case-Study-SecOps-Engineer-2026

Repository này chứa các bài thực hành trong **SecOps Engineer Case Study 2026**, tập trung vào hai nội dung chính:

- **Task 1:** Kubernetes Security Hardening
- **Task 2:** ZTNA / Identity-aware Proxy

---

# Task 1: Kubernetes Security Hardening

## 📋 Yêu cầu hệ thống

### Operating System

- Windows OS

### Prerequisites

Cài đặt các công cụ sau trước khi chạy Lab:

1. **[Docker Desktop](https://www.docker.com/products/docker-desktop/)**
   - Docker Desktop phải đang chạy để Kind có thể tạo và quản lý các Kubernetes container.

2. **[Kind (Kubernetes IN Docker)](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)**
   - Dùng để tạo và chạy Kubernetes cluster local thông qua Docker.

3. **[kubectl](https://kubernetes.io/docs/tasks/tools/)**
   - Command-line tool dùng để quản lý và tương tác với Kubernetes cluster.

### Optional

- `trivy.exe`
  - Có thể đặt trực tiếp trong thư mục project nếu muốn script tự động thực hiện vulnerability/security scanning.

---

## 📁 Cấu trúc thư mục

Đảm bảo các file sau nằm trong cùng một thư mục:

```text
.
├── run.bat
├── lab.ps1
├── kind-config.yaml
├── deployment-vulnerable.yaml
├── deployment-secure.yaml
├── secret.yaml
├── app.py
└── trivy.exe                  # Optional
```

### Mô tả các file

| File | Mô tả |
|---|---|
| `run.bat` | Script khởi chạy Lab trên Windows. |
| `lab.ps1` | PowerShell script thực hiện các bước chính của Lab. |
| `kind-config.yaml` | Cấu hình Kubernetes cluster sử dụng Kind. |
| `deployment-vulnerable.yaml` | Deployment mô phỏng workload có các cấu hình bảo mật không an toàn. |
| `deployment-secure.yaml` | Deployment sau khi áp dụng các biện pháp hardening. |
| `secret.yaml` | Định nghĩa Kubernetes Secret được sử dụng bởi workload. |
| `app.py` | Mã nguồn ứng dụng Python giả lập được sử dụng trong Pod. |
| `trivy.exe` | Công cụ security/vulnerability scanning, tùy chọn. |

### Các vấn đề bảo mật được mô phỏng

`deployment-vulnerable.yaml` cố tình chứa một số cấu hình không an toàn, bao gồm:

- Chạy container với quyền `root`.
- Không giới hạn CPU và memory.
- Hardcode credential trong configuration.
- Mount host root filesystem.
- Các cấu hình khác được sử dụng để minh họa quá trình Kubernetes security hardening.

`deployment-secure.yaml` được sử dụng để triển khai phiên bản đã được harden.

---

## 🚀 Hướng dẫn chạy Lab

### Bước 1: Kiểm tra prerequisites

Đảm bảo Docker Desktop đang chạy và kiểm tra các công cụ:

```powershell
docker --version
kind --version
kubectl version --client
```

### Bước 2: Mở Terminal

Mở **Command Prompt** hoặc **PowerShell**.

Trong trường hợp cần thay đổi PowerShell Execution Policy, có thể mở terminal với quyền **Administrator**.

### Bước 3: Di chuyển đến thư mục project

```powershell
cd <path-to-project>
```

### Bước 4: Khởi chạy Lab

Chạy:

```text
run.bat
```

Script sẽ thực hiện các bước cần thiết để khởi tạo và triển khai Lab.

---

# Task 2: ZTNA / Identity-aware Proxy

## 🎯 Mục tiêu

Task này xây dựng một POC mô phỏng mô hình **Zero Trust Network Access (ZTNA)** sử dụng **Identity-aware Proxy** để kiểm soát quyền truy cập vào các tài nguyên nội bộ.

POC bảo vệ hai loại target:

1. **Internal Web App (HTTP)**
   - Ứng dụng web nội bộ sử dụng `httpbin`.

2. **Admin SSH Target (TCP)**
   - SSH server dùng cho mục đích quản trị.
   - SSH target không expose trực tiếp port SSH ra bên ngoài.
   - Việc truy cập được thực hiện thông qua Pomerium.

---

## 🏗️ Thành phần

Mô hình POC bao gồm:

```text
User
 │
 │ Authentication
 ▼
GitHub OAuth / IdP
 │
 ▼
Pomerium
 │
 ├──► Internal Web App
 │
 └──► Admin SSH Target
```

Pomerium đóng vai trò **Identity-aware Proxy**, thực hiện xác thực người dùng và áp dụng policy trước khi cho phép truy cập tới target.

---

## 🛠️ Yêu cầu hệ thống

Cần cài đặt:

- Docker
- Docker Compose
- `pomerium-cli`

`pomerium-cli` được sử dụng để kiểm thử luồng truy cập TCP/SSH.

---

## 🚀 Hướng dẫn chạy POC

### Bước 1: Cấu hình DNS local

Thêm các record sau vào file hosts.

**Windows:**

```text
C:\Windows\System32\drivers\etc\hosts
```

**Linux/macOS:**

```text
/etc/hosts
```

Thêm:

```text
127.0.0.1   auth.localhost.pomerium.io
127.0.0.1   webapp.localhost.pomerium.io
127.0.0.1   ssh.localhost.pomerium.io
```

---

### Bước 2: Thiết lập Identity Provider

POC sử dụng GitHub OAuth làm Identity Provider.

#### 2.1. Tạo OAuth App

Tạo một OAuth App trên GitHub.

Sử dụng callback URL:

```text
https://auth.localhost.pomerium.io/oauth2/callback
```

#### 2.2. Cấu hình credentials

Điền các thông tin OAuth vào `config.yaml`:

```text
Client ID
Client Secret
```

> Không commit `Client Secret` hoặc các credential thực tế vào Git repository.

#### 2.3. Cấu hình access policy

Trong phần `policy` của `config.yaml`, thay đổi các địa chỉ email được cấu hình sẵn thành email test tương ứng.

Policy sẽ được sử dụng để xác định user nào được phép truy cập các route được bảo vệ.

---

## Bước 3: Khởi động hệ thống

Từ thư mục chứa `docker-compose.yml`, chạy:

```bash
docker-compose up -d
```

Kiểm tra trạng thái container:

```bash
docker-compose ps
```

---

# 🔐 Bước 4: Kiểm thử Web App

Mở trình duyệt ở chế độ **Incognito/Private** và truy cập:

```text
https://webapp.localhost.pomerium.io
```

Luồng truy cập:

```text
Browser
   │
   ▼
Pomerium
   │
   ▼
GitHub Authentication
   │
   ▼
Access Policy
   │
   ├── Allowed ──► httpbin
   │
   └── Denied
```

Nếu tài khoản không đáp ứng policy được cấu hình trong `config.yaml`, request sẽ không được phép truy cập vào Web App.

---

# 🔐 Bước 5: Kiểm thử SSH Target

SSH target không expose trực tiếp SSH port ra bên ngoài.

Sử dụng `pomerium-cli` để tạo TCP tunnel:

```bash
pomerium-cli tcp ssh.localhost.pomerium.io:443
```

Sau khi authentication thành công, CLI sẽ bind SSH connection vào một local port.

Mở một terminal khác và kết nối tới local port đó:

```bash
ssh -p <port-được-bind> admin@127.0.0.1
```

Password của user test:

```text
secretpassword
```

> `<port-được-bind>` là port local được `pomerium-cli` hiển thị sau khi thiết lập tunnel.

Luồng truy cập:

```text
SSH Client
    │
    ▼
Local Port
    │
    ▼
pomerium-cli
    │
    ▼
Pomerium
    │
    ├── Authentication
    ├── Identity
    └── Access Policy
    │
    ▼
SSH Target
```

---

# 📌 Summary

| Task | Nội dung | Công nghệ chính |
|---|---|---|
| Task 1 | Kubernetes Security Hardening | Docker, Kind, Kubernetes, kubectl, Trivy |
| Task 2 | ZTNA / Identity-aware Proxy | Pomerium, GitHub OAuth, Docker Compose, SSH |

Hai task tập trung vào hai khía cạnh khác nhau của SecOps:

- **Task 1:** Hardening workload và giảm security risk trong Kubernetes.
- **Task 2:** Kiểm soát truy cập tài nguyên nội bộ dựa trên identity và access policy.
