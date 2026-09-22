# Case-Study-SecOps-Engineer-2026

## Task 1: Kubernetes Security Hardening 

### 📋 Yêu cầu hệ thống (Prerequisites)
Môi trường: Windows OS
Cần cài đặt sẵn các công cụ sau:
1. **[Docker Desktop](https://www.docker.com/products/docker-desktop/)**: Phải đang chạy ngầm (running) để Kind có thể tạo container.
2. **[Kind (Kubernetes IN Docker)](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)**: Công cụ để chạy local Kubernetes cluster.
3. **[kubectl](https://kubernetes.io/docs/tasks/tools/)**: Command-line tool để tương tác với Kubernetes.

### 📁 Cấu trúc thư mục

Đảm bảo có các file sau trong cùng một thư mục:

* `run.bat`: Script khởi chạy cho Windows.
* `lab.ps1`: Script PowerShell thực hiện các bước chính của Lab.
* `kind-config.yaml`: Cấu hình cụm Kind (Cluster).
* `deployment-vulnerable.yaml`: File triển khai cố tình để lại các lỗ hổng bảo mật (chạy root, không giới hạn tài nguyên, hardcode mật khẩu, mount host root...).
* `deployment-secure.yaml`: File triển khai đã được áp dụng các tiêu chuẩn bảo mật.
* `secret.yaml`: Định nghĩa K8s Secret an toàn.
* `app.py`: Mã nguồn ứng dụng Python giả lập (đã được nhúng thẳng vào lệnh khởi chạy trong Pod).
* `trivy.exe` *(Tùy chọn)*: Đặt ở đây nếu muốn script tự động chạy rà quét bảo mật.

### 🚀 Hướng dẫn chạy Lab

1. Mở Terminal (Command Prompt hoặc PowerShell) với quyền Administrator (nếu cần thiết cho cấu hình execution policy).
2. Di chuyển đến thư mục chứa dự án.
3. Chạy file run.bat

## Task 2: ZTNA / Identity-aware Proxy

### 🛠 Hướng dẫn chạy POC (Local)

POC mô phỏng việc bảo vệ 2 dịch vụ:
1. **Internal Web App (HTTP)**: Ứng dụng web nội bộ (`httpbin`).
2. **Admin SSH Target (TCP)**: Máy chủ quản trị SSH không expose trực tiếp port ra ngoài.

### Yêu cầu hệ thống
- Docker & Docker Compose.
- `pomerium-cli` (để test luồng TCP/SSH).

### Các bước chạy

**Bước 1: Cấu hình DNS local**
Thêm các record sau vào file `/etc/hosts` (hoặc `C:\Windows\System32\drivers\etc\hosts`):
1. 127.0.0.1   auth.localhost.pomerium.io
2. 127.0.0.1   webapp.localhost.pomerium.io
3. 127.0.0.1   ssh.localhost.pomerium.io

**Bước 2: Thiết lập IdP (Identity Provider)**
1. Tạo một OAuth App trên GitHub.
2. Set Callback URL là: `https://auth.localhost.pomerium.io/oauth2/callback`
3. Điền `Client ID` và `Client Secret` vào file `config.yaml`.
4. Sửa các địa chỉ email trong phần `policy` của file `config.yaml` thành email test của bạn.

**Bước 3: Khởi động hệ thống**
\`\`\`bash
docker-compose up -d
\`\`\`

**Bước 4: Test các luồng truy cập**
- **Web App:** Truy cập trình duyệt ẩn danh tại `https://webapp.localhost.pomerium.io`. Hệ thống sẽ chuyển hướng xác thực qua GitHub.
- **SSH Target:** Mở terminal và chạy lệnh để mở một HTTPS tunnel an toàn:
  \`\`\`bash
  pomerium-cli tcp ssh.localhost.pomerium.io:443
  \`\`\`
  Sau khi trình duyệt xác thực thành công, CLI sẽ map SSH tới một port local ngẫu nhiên. Mở tab terminal khác và chạy:
  \`\`\`bash
  ssh -p <port-được-bind> admin@127.0.0.1
  # Mật khẩu: secretpassword
  \`\`\`
