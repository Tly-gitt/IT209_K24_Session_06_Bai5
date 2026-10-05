# Báo Cáo:

## 1. Phân tích các khối cấu hình trong `java-app.service`

- **Khối `[Unit]`**:
  - `Description`: Định danh mô tả mục đích của dịch vụ.
  - `After=network.target`: Đảm bảo toàn bộ hạ tầng mạng của hệ điều hành đã sẵn sàng trước khi ứng dụng khởi chạy.
- **Khối `[Service]`**:
  - `User=java-runner` & `Group=java-runner`: Phân quyền thực thi cho tài khoản dịch vụ non-root, hạn chế rủi ro bảo mật leo thang đặc quyền khi ứng dụng bị khai thác mã độc.
  - `ExecStart=/opt/java-app/run.sh`: Đường dẫn tuyệt đối đến kịch bản/nhị thức thực thi ứng dụng.
  - `Restart=on-failure`: Cơ chế tự phục hồi khi ứng dụng sập bất thường (exit code khác 0 hoặc bị kill bởi tín hiệu bất ngờ).
  - `RestartSec=10s`: Đặt thời gian chờ 10 giây trước khi Systemd thử khởi tạo lại tiến trình.
- **Khối `[Install]`**:
  - `WantedBy=multi-user.target`: Định nghĩa target chạy mức hệ thống hoàn chỉnh (runlevel tương đương chế độ đa người dùng có mạng).

---

## 2. Quy trình thực hiện

1. **Tạo tài khoản dịch vụ:**
   ```bash
   sudo useradd -r -s /usr/sbin/nologin java-runner
   ```
2. **Triển khai ứng dụng và service file:**
   - Cấu hình file tại `/etc/systemd/system/java-app.service`.
3. **Nạp và khởi động dịch vụ:**
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable --now java-app.service
   ```

---

## 3. Minh chứng kết quả thực tế

### Trạng thái dịch vụ (`systemctl status java-app.service`):
```text
● java-app.service - Java Spring Boot Application Service
     Loaded: loaded (/etc/systemd/system/java-app.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2026-10-06 01:25:00 +07; 12s ago
   Main PID: 24680 (run.sh)
      Tasks: 2 (limit: 4614)
     Memory: 1.2M
        CPU: 4ms
     CGroup: /system.slice/java-app.service
             ├─24680 /bin/bash /opt/java-app/run.sh
             └─24682 sleep 3600
```

### Nhật ký log ghi nhận chạy dưới user `java-runner`:
```text
-- Boot 1 --
systemd[1]: Started Java Spring Boot Application Service.
run.sh[24680]: Java Spring Boot application starting...
run.sh[24680]: Running as user: java-runner
```