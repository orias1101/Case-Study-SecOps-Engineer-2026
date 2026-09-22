# ZTNA / Identity-Aware Proxy (IAP) Design & POC

## 🛠 Hướng dẫn chạy POC (Local)

POC mô phỏng việc bảo vệ 2 dịch vụ:
1. **Internal Web App (HTTP)**: Ứng dụng web nội bộ (`httpbin`).
2. **Admin SSH Target (TCP)**: Máy chủ quản trị SSH không expose trực tiếp port ra ngoài.

### Yêu cầu hệ thống
- Docker & Docker Compose.
- `pomerium-cli` (để test luồng TCP/SSH).

### Các bước chạy

**Bước 1: Cấu hình DNS local**
Thêm các record sau vào file `/etc/hosts` (hoặc `C:\Windows\System32\drivers\etc\hosts`):
\`\`\`text
127.0.0.1   auth.localhost.pomerium.io
127.0.0.1   webapp.localhost.pomerium.io
127.0.0.1   ssh.localhost.pomerium.io
\`\`\`

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