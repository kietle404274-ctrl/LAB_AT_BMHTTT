# LAB3 - Threats and Security Analysis

## 1. Thông tin sinh viên

- Họ và tên: Lê Vũ Kiệt
- MSSV: 11500080100
- Tên Lab: LAB3 - Threats, Attacks and Security Analysis

---

## 2. Thông tin môi trường thực hành

- Hệ điều hành máy ảo: Windows 11 Home 64-bit
- Nền tảng ảo hóa: VMware Workstation
- RAM cấp cho VM: 4 GB
- Python version: Python 3.14.7
- Microsoft Defender: Enabled
- Sysmon version: 15.22
- Wireshark version: 4.6.8

---

## 3. Cách dựng môi trường

### Chuẩn bị máy ảo
- Cài đặt Windows 11 trên VMware Workstation.
- Cấu hình mạng cho VM.
- Bật Microsoft Defender và cập nhật hệ thống.

### Cấu trúc thư mục Lab

```
C:\LAB3
├── Tools
├── Evidence
└── LAB3_Threats_Assets
    └── lab3_assets
```

### Công cụ sử dụng
- Wireshark 4.6.8
- Sysinternals Suite:
  - Sysmon
  - Autoruns
  - Process Explorer
- Python 3.14.7

---

# 4. Các tình huống đã thực hiện

## TH1 - Threat Classification
- Phân loại nguồn đe dọa:
  - Hành động vô ý
  - Hành động cố ý
  - Thảm họa tự nhiên
  - Lỗi kỹ thuật
  - Lỗi quản lý

Kết quả: PASS

---

## TH2 - Malware Detection và Defender
- Kiểm tra Microsoft Defender.
- Tạo file EICAR test.
- Kiểm tra log phát hiện.

Kết quả: PASS

---

## TH3 - Authentication Logging
- Tạo tài khoản lab3user.
- Kiểm tra Event ID:
  - 4624
  - 4625
  - 4648
- Thực hiện đổi mật khẩu và kiểm tra xác thực.

Kết quả: PASS

---

## TH4 - Persistence và Network Analysis
- Kiểm tra Registry Run Key.
- Tạo Scheduled Task.
- Chạy HTTP server Python trên 127.0.0.1:8080.
- Kiểm tra PID bằng PowerShell.
- Phân tích tiến trình bằng Process Explorer.

Kết quả: PASS

---

## TH5 - Network Traffic Analysis
- Capture HTTP và HTTPS bằng Wireshark.
- So sánh dữ liệu plaintext và dữ liệu được mã hóa TLS.

Kết quả: PASS

---

# 5. Các lỗi gặp phải và cách khắc phục

## Lỗi 1: Python không nhận lệnh python
**Hiện tượng:** Python was not found.

**Nguyên nhân:** Windows gọi Microsoft Store alias thay vì Python thật.

**Khắc phục:**
- Kiểm tra đường dẫn Python.
- Thêm Python vào PATH.
- Tắt App Execution Alias.

---

## Lỗi 2: Sysmon không đọc được file cấu hình
**Hiện tượng:** Failed to open xml configuration.

**Nguyên nhân:** Sai đường dẫn file sysmon-lab.xml.

**Khắc phục:**
- Kiểm tra lại vị trí file.
- Sử dụng đúng đường dẫn trong thư mục lab3_assets.

---

## Lỗi 3: Không tạo được tài khoản lab3user
**Hiện tượng:** Access Denied.

**Nguyên nhân:** PowerShell chưa chạy quyền Administrator.

**Khắc phục:**
- Mở PowerShell bằng Run as Administrator.

---

## Lỗi 4: Không tạo được file SHA256 Evidence
**Hiện tượng:** evidence_sha256.csv bị khóa.

**Nguyên nhân:** File CSV đang nằm trong thư mục được tính hash.

**Khắc phục:**
- Loại trừ file evidence_sha256.csv khi tạo hash.

---

# 6. Evidence đã thu thập

Các file bằng chứng gồm:
- baseline_os.txt
- baseline_defender.txt
- baseline_firewall.txt
- auth_events_lab3user.txt
- defender_eicar.txt
- autoruns_before.csv
- autoruns_after.csv
- sysmon_persistence.txt
- local_load_test.txt
- ddos_sources.txt
- mail_sender_counts.txt
- evidence_sha256.csv

---

# 7. Kết luận

Qua LAB3, các nội dung về nhận diện mối đe dọa, kiểm tra malware, phân tích xác thực, persistence, phân tích lưu lượng mạng và social engineering đã được thực hiện trong môi trường máy ảo.

Các bằng chứng được thu thập, lưu trữ và kiểm tra tính toàn vẹn bằng SHA-256.
