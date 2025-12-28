# Hướng dẫn Thiết lập Môi trường Lab Active Directory Security (AD CS Focus)

Tài liệu này hướng dẫn chi tiết các bước thiết lập mạng ảo, cấu hình Domain Controller, máy nạn nhân và máy tấn công để thực hiện bài lab khai thác Active Directory Certificate Services.

---

## 1. Thiết lập Mạng Ảo (Virtual Network Editor)

Cấu hình trong VMware Workstation / VirtualBox:

| Tên Mạng | Chế độ (Type) | Dải IP (Subnet) | Mục đích |
| :--- | :--- | :--- | :--- |
| **VMnet8** | NAT | `192.168.43.0/24` | Mạng ngoài (Internet giả lập). Nơi Attacker và Victim (Card 1) giao tiếp. |
| **VMnet2** | Host-only | `192.168.148.0/24` | Mạng nội bộ (Intranet). Nơi đặt Domain Controller, cực kỳ bảo mật. |

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture1.png)
---

## 2. Cấu hình Máy 1: Domain Controller (Mục tiêu)

* **Hệ điều hành:** Windows Server 2016.
* **Mạng:** Gắn 1 card vào **VMnet2** (Host-only).
* **Thông số IP:**
    * IP tĩnh: `192.168.148.131`
    * Subnet mask: `255.255.255.0`
    * DNS: `127.0.0.1`

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture2.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture3.png)
### Cài đặt Roles & Policy
1.  **AD DS:** Promote lên Domain Controller với tên miền `KMA.local`. Mật khẩu DSRM: `Admin@123`.

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture4.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture5.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture6.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture7.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture8.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture9.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture10.png)

2.  **AD CS:**

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture11.png)
    
   * Chọn role **Certification Authority**.

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture12.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture13.png)
    
   * Cấu hình: **Enterprise CA** -> **Root CA**.

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture14.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture15.png)
   
   * Tên CA: `YENKMA-KMASERVER-CA`.

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture16.png)

3.  **Tạo User:** Tạo người dùng `yen1` (Pass: `Admin@123`) trong *AD Users and Computers*.

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture17.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture18.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture19.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture20.png)

4.  **Kiểm tra MachineAccountQuota:**
    Đảm bảo giá trị bằng `10` để phục vụ tấn công tạo máy ảo giả mạo. Kiểm tra bằng PowerShell:
    ```powershell
    ([adsisearcher]"(objectClass=domainDNS)").FindOne().Properties["ms-ds-machineaccountquota"]
    ```
    ![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture21.png)

---

## 3. Cấu hình Máy 2: Victim Machine (Pivot)

Máy này đóng vai trò máy trạm của người dùng và là bàn đạp (pivot) để tấn công vào mạng nội bộ.

* **Hệ điều hành:** Windows 10 Pro / Enterprise.
* **Cấu hình mạng (Dual-Homed):**
    * **Card 1 (VMnet8 - NAT):** IP `192.168.43.130` (Dùng để nhận payload từ Attacker).
    * **Card 2 (VMnet2 - Host-only):** IP `192.168.148.133`, DNS `192.168.148.131`.

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture22.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture23.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture24.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture25.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture26.png)

* **Join Domain:** Gia nhập miền `KMA.local` bằng user Administrator. Sau đó đăng nhập bằng `KMA\yen1`.
* **Lưu ý:** Tắt Windows Defender và Firewall để thuận tiện cho việc thực tập ban đầu.

---

## 4. Cấu hình Máy 3: Attack Machine (Kẻ tấn công)

* **Hệ điều hành:** Kali Linux.
* **Mạng:** Gắn 1 card vào **VMnet8** (NAT). IP: `192.168.43.128`.

### Cấu hình Proxychains
Mở file `/etc/proxychains4.conf` và thêm vào cuối file:
```text
socks4 127.0.0.1 9050


