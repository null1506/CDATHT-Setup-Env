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

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture27.png)

* **Join Domain:** Gia nhập miền `KMA.local` bằng user Administrator. Sau đó đăng nhập bằng `KMA\yen1`.

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture28.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture29.png)

* **Lưu ý:** Tắt Windows Defender và Firewall để thuận tiện cho việc thực tập ban đầu.

---

## 4. Cấu hình Máy 3: Attack Machine (Kẻ tấn công)

* **Hệ điều hành:** Kali Linux.
* **Mạng:** Gắn 1 card vào **VMnet8** (NAT). IP: `192.168.43.128`.

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture30.png)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture31.png)
### Cấu hình Proxychains
Mở file `/etc/proxychains4.conf` và thêm vào cuối file:
```text
socks4 127.0.0.1 9050
```

---

## 5. Tải công cụ trên máy tấn công (Kali Linux)

### **Kerbrute** 
Dùng để dò quét (brute-force) tên người dùng và mật khẩu qua giao thức Kerberos một cách nhanh chóng và ít bị phát hiện hơn.

Cách tải: Công cụ này được viết bằng Go, bạn cần tải file binary đã biên dịch sẵn (pre-compiled).
1.	Truy cập: https://github.com/ropnop/kerbrute/releases
2.	Tải file phù hợp (thường là kerbrute_linux_amd64).
`wget https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64`

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture32.png)

3.	Đổi tên và cấp quyền thực thi:
```cmd
mv kerbrute_linux_amd64 kerbrute
chmod +x kerbrute
```

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture33.png)

### **Metasploit Framework**
Dùng để tạo backdoor, điều khiển máy nạn nhân

Cách tải: Hai công cụ này đã được cài sẵn mặc định trên Kali Linux. Không cần tải thêm.

Kiểm tra Metasploit: Gõ `msfconsole`.

### **xFreeRDP**
Dùng để Remote Desktop vào máy nạn nhân khi đã có tài khoản (bước kiểm tra cuối cùng).

Cách tải: Gõ các lệnh sau
```
apt search freerdp
sudo apt install freerdp3-x11
```

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture34.png)

## 5. Tải công cụ trên máy tấn công (Kali Linux)

### **Python** 
Dùng để tải Certipy.

Cách tải: Truy cập vào đây để tải bản 3.13.11: https://www.python.org/downloads/windows/

### **Certify.exe (Optional)**
Dùng để quét lỗ hổng chứng chỉ từ bên trong máy nạn nhân

Cách tải: Đây là công cụ viết bằng C#. Tác giả (GhostPack) không cung cấp file .exe sẵn vì lý do an toàn.

1.	Truy cập GitHub chính thức của GhostPack: https://github.com/GhostPack/Certify
2.	Tải file zip về và giải nén
3.	Tải Visual Studio 2022 Community tại: https://www.windowsmode.com/download-visual-studio-2022-for-windows/
4.	Cấu hình biên dịch và biên dịch

4.1. File code sau khi giải nén, tìm file Certify.sln và mở bằng VS 2022 Community

4.2. Tìm ô đang để chữ Debug và đổi thành Release (Chế độ Release giúp file nhẹ hơn và bỏ các thông tin gỡ lỗi không cần thiết).

4.3. Chọn kiến trúc là Any CPU hoặc x64.

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture35.png)

4.4. Biên dịch ra file .exe

Trên thanh menu, chọn Build -> Build Solution (hoặc nhấn phím tắt Ctrl + Shift + B). (Nếu có warning thì tắt Visual đi bật lại)

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture36.png)

Nhìn xuống khung Output bên dưới, nếu thấy dòng Build succeeded là thành công.

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture37.png)

5. Lấy file sản phẩm

Vào thư mục dự án theo đường dẫn: Certify\bin\Release\. Ta sẽ thấy file Certify.exe. Đây chính là file mà chúng ta sẽ thực thi.

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture38.png)

## 6. Cấu hình máy Victim cho phép Remote##

Kiểm tra trạng thái của tính năng Remote Desktop (RDP) trên máy tính Windows thông qua Registry.

`yen1> reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections`

- `reg query`: Lệnh dùng để truy vấn/đọc thông tin từ Registry.

- `"HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server"`: Đường dẫn đến khóa Registry quản lý các thiết lập của Terminal Server (tên cũ của Remote Desktop Services).

- `/v fDenyTSConnections`: Yêu cầu hiển thị giá trị của biến cụ thể tên là fDenyTSConnections (viết tắt của Force Deny Terminal Server Connections).

Khi chạy lệnh này, bạn sẽ nhận được một giá trị số (thường là dạng Hexadecimal). Bạn cần chú ý vào số cuối cùng:

Giá trị là 0x1 (1): Remote Desktop đang bị TẮT (Từ chối kết nối).

Giá trị là 0x0 (0): Remote Desktop đang được BẬT (Cho phép kết nối).

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture39.png)

Ta thấy kết quả là 0x1, tức Remote Desktop đang bị tắt

Lệnh bật tính năng RDP trong Hệ điều hành
`C:\Windows\system32> reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f`

Mục đích: Ghi đè giá trị vào Registry để hệ thống "mở cửa" cho phép kết nối từ xa.

Phân tích:

- `fDenyTSConnections`: Như đã giải thích ở câu trước, giá trị 0 có nghĩa là "Không từ chối" (tức là Cho phép).

- `/t REG_DWORD`: Xác định kiểu dữ liệu là số nguyên.

- `/f`: (Force) Ép thực hiện thay đổi mà không cần hỏi xác nhận.

Lệnh mở cổng qua Firewall (Tường lửa)
`netsh advfirewall firewall set rule group="remote desktop" new enable=Yes`

Mục đích: Dù Windows đã cho phép RDP (ở bước 1), nhưng nếu Tường lửa vẫn chặn cổng 3389, bạn vẫn không thể kết nối được. Lệnh này sẽ mở tất cả các luật (rules) liên quan đến Remote Desktop trong Windows Firewall.

Lệnh kiểm tra thành viên nhóm Remote Desktop
`net localgroup "Remote Desktop Users"`

Mục đích: Liệt kê danh sách tất cả các tài khoản hiện đang có quyền đăng nhập vào máy này qua RDP.

Lệnh cấp quyền RDP cho một User cụ thể
`net localgroup "Remote Desktop Users" KMA\yen1 /add`

Mục đích: Thêm tài khoản miền KMA\yen1 vào nhóm cục bộ "Remote Desktop Users".

![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture40.png)
