# Báo cáo Bài 3: Cấu hình tường lửa UFW và chẩn đoán cổng mạng

- **Học viên:** [Điền họ và tên]
- **Mã học viên / MSSV:** [Điền MSSV]
- **Đường dẫn thư mục nộp bài:** `homework/session_06/ex3/`

---

## 1. Mục tiêu bài lab

- Cấu hình tường lửa **UFW (Uncomplicated Firewall)** trên máy chủ Cloud VPS nhằm thắt chặt bảo mật theo nguyên tắc quyền tối thiểu (*least privilege*).
- Thiết lập quy tắc tường lửa:
  - **Mặc định:** Chặn toàn bộ luồng kết nối đi vào (`deny incoming`), cho phép toàn bộ kết nối đi ra (`allow outgoing`).
  - **SSH (Port 22/tcp):** Mở để duy trì phiên quản trị từ xa, tránh bị khóa khỏi máy chủ (*lockout*).
  - **Web Application (Port 8080/tcp):** Mở để tiếp nhận lưu lượng truy cập dịch vụ web.
- Kích hoạt và xác thực cấu hình thông qua công cụ chẩn đoán mạng (`ufw status verbose`, `ss -tlnp`).

---

## 2. Các bước thực hiện

### Bước 1: Thiết lập chính sách mặc định
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
*Kết quả:* Hệ thống ghi nhận chính sách mặc định chặn toàn bộ kết nối vào và mở kết nối ra.

### Bước 2: Khai báo quy tắc mở cổng
```bash
# Mở cổng quản trị SSH
sudo ufw allow 22/tcp

# Mở cổng ứng dụng Web
sudo ufw allow 8080/tcp
```

### Bước 3: Kích hoạt tường lửa UFW
```bash
sudo ufw enable
```
*Hệ thống hiển thị cảnh báo ngắt kết nối SSH:*
```text
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

---

## 3. Bằng chứng kết quả (Screenshots & Logs)

### 3.1. Trạng thái chi tiết của tường lửa (`sudo ufw status verbose`)

#### Ảnh chụp màn hình:
![Ảnh chụp trạng thái UFW verbose](./ufw-status.png)

#### Log kết quả chi tiết:
```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                  
8080/tcp                   ALLOW IN    Anywhere                  
22/tcp (v6)                ALLOW IN    Anywhere (v6)             
8080/tcp (v6)              ALLOW IN    Anywhere (v6)             
```

---

### 3.2. Chẩn đoán cổng đang lắng nghe trên hệ thống (`ss -tlnp`)

#### Ảnh chụp màn hình:
![Ảnh chụp tiến trình và cổng lắng nghe](./ss-tlnp.png)

#### Log kết quả chi tiết:
```text
State      Recv-Q Send-Q Local Address:Port        Peer Address:Port Process                                            
LISTEN     0      128          0.0.0.0:22               0.0.0.0:*     users:(("sshd",pid=684,fd=3))                     
LISTEN     0      511          0.0.0.0:8080             0.0.0.0:*     users:(("node",pid=1420,fd=18))                   
LISTEN     0      128             [::]:22                  [::]:*     users:(("sshd",pid=684,fd=4))                     
LISTEN     0      511             [::]:8080                [::]:*     users:(("node",pid=1420,fd=19))                   
```

---

## 4. Đánh giá & Kết luận

1. **Trạng thái tường lửa:** UFW đã kích hoạt thành công (`Status: active`) và được cấu hình tự động bật mỗi khi khởi động lại máy chủ (*enabled on system startup*).
2. **Tính bảo mật:** Mọi cổng kết nối trái phép ngoài 22 và 8080 đều bị chặn hoàn toàn từ bên ngoài.
3. **Tính sẵn sàng:**
   - Kết nối SSH không bị gián đoạn.
   - Cổng `8080/tcp` đã sẵn sàng nhận kết nối từ Internet khi ứng dụng Web chạy thực tế.