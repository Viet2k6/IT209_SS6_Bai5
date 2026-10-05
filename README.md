# Bài 5: Thiết kế và Cấu hình Systemd Service cho ứng dụng Java

## 1. Mục tiêu

* Tạo system user `java-runner` không cho phép đăng nhập trực tiếp.
* Cấu hình systemd service cho ứng dụng Java.
* Chạy service dưới quyền user `java-runner`.
* Tự động restart khi service bị lỗi.

## 2. Tạo user java-runner

Đã tạo user hệ thống bằng lệnh:

```bash
sudo useradd -r -s /usr/sbin/nologin java-runner
```

Kiểm tra:

```text
java-runner:x:999:986::/home/java-runner:/usr/sbin/nologin
```

User sử dụng `/usr/sbin/nologin` nên không thể đăng nhập trực tiếp.

## 3. Cấu hình systemd service

File service:

```text
/etc/systemd/system/java-app.service
```

Nội dung:

```ini
[Unit]
Description=Java Application Service
After=network.target

[Service]
User=java-runner
ExecStart=/bin/sh -c 'echo "Java app started as $(id -un)"; exec /usr/bin/sleep 1000'
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Ứng dụng Java được mô phỏng bằng tiến trình `sleep 1000`.

## 4. Khởi động service

Nạp lại cấu hình systemd:

```bash
sudo systemctl daemon-reload
```

Khởi động service:

```bash
sudo systemctl start java-app.service
```

Kiểm tra:

```bash
sudo systemctl status java-app.service
```

Kết quả:

```text
Active: active (running)
Main PID: 945 (sleep)
```

Log cho thấy ứng dụng chạy dưới user `java-runner`:

```text
Java app started as java-runner
```

## 5. Kiểm tra journal

Lệnh:

```bash
journalctl -u java-app.service --no-pager
```

Kết quả:

```text
Started java-app.service - Java Application Service.
Java app started as java-runner
```

## 6. Kết quả

Service `java-app.service` đã được cấu hình và khởi động thành công. Service chạy dưới user giới hạn `java-runner` và được cấu hình tự động restart sau 10 giây khi xảy ra lỗi.
