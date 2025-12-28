Bước 1: Thiết lập Mạng Ảo (Virtual Network Editor)

•	VMnet8 (NAT): Dải IP 192.168.43.0/24. Đây là vùng mạng giả lập Internet/Mạng ngoài, nơi máy tấn công và máy nạn nhân giao tiếp.

•	VMnet2 (Host-only): Dải IP 192.168.148.0/24. Đây là vùng mạng nội bộ (Intranet) bảo mật, nơi đặt Domain Controller.
              ![alt](https://github.com/null1506/CDATHT-Setup-Env/blob/main/img/Picture1.png)

Bước 2: Cấu hình các máy

  1. Cấu hình Máy 1: Domain Controller (Mục tiêu)
Đây là máy chủ quan trọng nhất, chứa Active Directory và Certificate Services.
•	Hệ điều hành: Windows Server 2016
•	Cấu hình mạng (Network Adapter):
o	Chỉ gắn 1 card mạng vào VMnet2 (Host-only).
o	IP tĩnh: 192.168.148.131 
o	Subnet mask: 255.255.255.0
o	DNS: 127.0.0.1 


