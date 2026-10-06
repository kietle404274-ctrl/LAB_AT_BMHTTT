# LAB 4 – Khảo sát và đánh giá bề mặt mạng bằng Nmap

## Thông tin sinh viên

- **Họ và tên:** Lê Vũ Kiệt
- **Lớp:** 11ĐB_CNPM_2
- **MSSV:** 1150080100

---

## Giới thiệu

Đây là bài thực hành môn **An toàn mạng và Hệ thống thông tin**, tập trung vào việc sử dụng **Nmap** để khảo sát các host, cổng, dịch vụ và một số thông tin bảo mật cơ bản trong một môi trường lab cô lập.

> ⚠️ **Lưu ý:** Bài thực hành chỉ sử dụng trên các máy ảo do chính người học quản lý trong mạng **Host-Only**. Không quét các hệ thống, IP, Wi-Fi hoặc dịch vụ Internet bên ngoài khi chưa được cho phép.

## Mục tiêu

- Thiết lập môi trường lab bằng VMware.
- Cấu hình mạng Host-Only cho Kali Linux và Metasploitable 2.
- Xác định địa chỉ IP của các máy trong mạng lab.
- Phát hiện các host đang hoạt động.
- Khảo sát cổng TCP bằng nhiều kỹ thuật quét.
- So sánh TCP Connect Scan và SYN Scan.
- Thực hiện FIN, Xmas, NULL và ACK Scan.
- Quét UDP có kiểm soát.
- Nhận diện phiên bản dịch vụ và hệ điều hành.
- Sử dụng một số NSE script để thu thập thông tin SMB.
- Xuất kết quả quét phục vụ báo cáo.

---

## Mô hình lab

```text
Windows Host
     |
   VMnet1
 Host-Only
     |
--------------------------------
|                              |
Kali Linux                 Metasploitable 2
Máy quét                     Máy đích
192.168.184.x              192.168.184.101
```

Trong bài thực hành này:

```text
Metasploitable 2: 192.168.184.101
Subnet:           192.168.184.0/24
```

> IP của Kali có thể thay đổi tùy theo DHCP của VMware.

---

## 1. Thiết lập mạng Host-Only trên VMware

Trong VMware Workstation:

```text
Edit
→ Virtual Network Editor
→ VMnet1
→ Host-only
```

Có thể cấu hình:

```text
Subnet IP:   192.168.184.0
Subnet Mask: 255.255.255.0
```

Sau đó cấu hình cả Kali và Metasploitable 2:

```text
VM Settings
→ Network Adapter
→ Host-only
```

Không sử dụng **Bridged Adapter** cho Metasploitable 2.

---

## 2. Kiểm tra IP Kali

Trên Kali:

```bash
ip -br addr
```

Ví dụ:

```text
eth0    UP    192.168.184.128/24
```

---

## 3. Thiết lập IP Metasploitable 2

Kiểm tra interface:

```bash
ifconfig
```

Nếu interface là `eth0`, đặt IP:

```bash
sudo ifconfig eth0 192.168.184.101 netmask 255.255.255.0 up
```

Kiểm tra lại:

```bash
ifconfig
```

---

## 4. Kiểm tra kết nối

Từ Kali:

```bash
ping -c 4 192.168.184.101
```

Nếu có phản hồi thì môi trường Host-Only đã hoạt động.

---

# Nhiệm vụ 1 – Phát hiện các host đang hoạt động

## Host Discovery

Quét toàn bộ subnet:

```bash
sudo nmap -sn 192.168.184.0/24
```

Ý nghĩa:

```text
-sn = chỉ phát hiện host đang hoạt động, không quét port
```

Ví dụ kết quả:

```text
Nmap scan report for 192.168.184.1
Host is up

Nmap scan report for 192.168.184.101
Host is up

Nmap scan report for 192.168.184.128
Host is up
```

Có thể tổng hợp:

| IP | Vai trò |
|---|---|
| 192.168.184.1 | VMware Host-Only Adapter |
| 192.168.184.101 | Metasploitable 2 |
| 192.168.184.x | Kali Linux |

---

# Khảo sát cổng TCP

## 1. TCP Connect Scan

```bash
nmap -sT 192.168.184.101
```

`-sT` thực hiện kết nối TCP hoàn chỉnh.

Cơ chế:

```text
SYN →
← SYN/ACK
ACK →
```

Ví dụ các cổng thường xuất hiện trên Metasploitable 2:

```text
21/tcp    ftp
22/tcp    ssh
23/tcp    telnet
80/tcp    http
139/tcp   netbios-ssn
445/tcp   microsoft-ds
3306/tcp  mysql
```

---

## 2. SYN Scan

```bash
sudo nmap -sS 192.168.184.101
```

Cơ chế:

```text
SYN →
← SYN/ACK
RST →
```

Đây còn được gọi là half-open scan.

### So sánh `-sT` và `-sS`

| Tiêu chí | `-sT` | `-sS` |
|---|---|---|
| Cách hoạt động | TCP connect hoàn chỉnh | Không hoàn thành kết nối |
| Quyền | Thường không cần root | Thường cần sudo/root |
| Tốc độ | Có thể chậm hơn | Thường nhanh hơn |
| Cổng open | Phát hiện | Phát hiện |
| Cổng closed | Phát hiện | Phát hiện |
| Filtered | Có thể phát hiện | Có thể phát hiện |

Đo thời gian:

```bash
time nmap -sT 192.168.184.101
```

```bash
time sudo nmap -sS 192.168.184.101
```

---

# FIN / Xmas / NULL Scan

## FIN Scan

```bash
sudo nmap -sF 192.168.184.101
```

## Xmas Scan

```bash
sudo nmap -sX 192.168.184.101
```

## NULL Scan

```bash
sudo nmap -sN 192.168.184.101
```

Một số kết quả có thể xuất hiện:

```text
open|filtered
```

Điều này **không có nghĩa chắc chắn là open**.

Nó có nghĩa Nmap không phân biệt được giữa:

- cổng đang mở
- hoặc firewall/filter đang làm packet bị im lặng

---

# ACK Scan

```bash
sudo nmap -sA 192.168.184.101
```

ACK Scan chủ yếu dùng để quan sát chính sách lọc của firewall.

Các trạng thái thường gặp:

```text
filtered
unfiltered
```

`-sA` không dùng để kết luận một port là `open`.

---

# UDP Scan

Quét 20 cổng UDP phổ biến:

```bash
sudo nmap -sU --top-ports 20 192.168.184.101
```

UDP thường chậm hơn TCP vì không có cơ chế handshake.

---

# Nhận diện dịch vụ

## Version Detection

```bash
sudo nmap -sV 192.168.184.101
```

`-sV` giúp xác định:

- tên dịch vụ
- phần mềm đang chạy
- phiên bản dịch vụ

---

# Nhận diện hệ điều hành

```bash
sudo nmap -O 192.168.184.101
```

Kết quả OS detection chỉ mang tính fingerprinting và không nên coi là tuyệt đối chính xác.

---

# Aggressive Scan

```bash
sudo nmap -A 192.168.184.101
```

`-A` có thể bao gồm:

- OS detection
- version detection
- default NSE scripts
- traceroute

Scan này tạo nhiều lưu lượng hơn nên dễ bị phát hiện hơn.

---

# SMB Information Gathering

## Kiểm tra port 445

```bash
sudo nmap -p 445 192.168.184.101
```

## SMB OS Discovery

```bash
sudo nmap -p 445 --script smb-os-discovery 192.168.184.101
```

Script có thể trả về:

- hệ điều hành
- hostname
- domain/workgroup

---

# Kiểm tra MS17-010

```bash
sudo nmap -p 445 --script smb-vuln-ms17-010 192.168.184.101
```

Nếu script báo:

```text
VULNERABLE
```

thì chỉ kết luận rằng mục tiêu **có dấu hiệu dễ bị ảnh hưởng theo kết quả NSE**.

Nếu script timeout hoặc không kết nối được thì không được tự kết luận rằng mục tiêu đã được vá.

---

# Xuất kết quả quét

## Normal Text

```bash
sudo nmap -sV 192.168.184.101 -oN scan.txt
```

## XML

```bash
sudo nmap -sV 192.168.184.101 -oX scan.xml
```

## Grepable

```bash
sudo nmap -p 445 192.168.184.101 -oG smb.txt
```

Lọc kết quả:

```bash
grep "445/open" smb.txt
```

## Xuất cả 3 định dạng

```bash
sudo nmap -sV 192.168.184.101 -oA lab4
```

Các file tạo ra:

```text
lab4.nmap
lab4.xml
lab4.gnmap
```

---

# Quét toàn bộ 65535 cổng

Quét mặc định:

```bash
sudo nmap -sS 192.168.184.101
```

Quét toàn bộ:

```bash
sudo nmap -sS -p- 192.168.184.101
```

`-p-` tương ứng với tất cả port từ `1` đến `65535`.

---

# Ảnh minh chứng cần chuẩn bị

- `ip -br addr` trên Kali.
- `ifconfig` trên Metasploitable 2.
- Kết quả `nmap -sn`.
- Kết quả `-sT` hoặc `-sS`.
- Kết quả `-sV`.
- Kết quả `-O` hoặc `-A`.
- Kết quả một NSE script.
- File kết quả `.txt`, `.xml`, `.nmap`, `.gnmap` hoặc HTML.

---

# Một số trạng thái Nmap quan trọng

| State | Ý nghĩa |
|---|---|
| `open` | Có dịch vụ đang lắng nghe |
| `closed` | Host phản hồi nhưng không có dịch vụ tại port |
| `filtered` | Firewall/filter khiến Nmap không xác định được |
| `unfiltered` | Packet đi qua filter nhưng chưa xác định port open |
| `open|filtered` | Không phân biệt được port mở hay bị filter làm im lặng |

---

# Ghi chú an toàn

Các câu lệnh và kỹ thuật trong repository này chỉ phục vụ mục đích:

- học tập
- thực hành an toàn mạng
- nghiên cứu trong môi trường lab cá nhân

Không sử dụng để quét hoặc kiểm tra hệ thống không thuộc quyền quản lý khi chưa có sự cho phép.

---

## Công cụ sử dụng

- VMware Workstation
- Kali Linux
- Metasploitable 2
- Nmap

---

## Nguồn bài thực hành

README này được biên soạn dựa trên nội dung **LAB 4 – Khảo sát và đánh giá bề mặt mạng bằng Nmap** của môn An toàn Hệ thống Thông tin, đồng thời điều chỉnh phần mạng Host-Only từ VirtualBox sang VMware để phù hợp với môi trường thực hành.
