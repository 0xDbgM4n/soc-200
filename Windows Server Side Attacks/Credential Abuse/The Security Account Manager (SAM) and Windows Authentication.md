**[Trình quản lý tài khoản bảo mật (SAM) và xác thực Windows]**

Windows lưu password dưới dạng mã hoá thay vì plaintext bằng cách đưa qua hàm băm

Mật khẩu người dùng đã qua băm được lưu trữ trong cơ sở dữ liệu Security Account Manager (SAM)

SYSKEY: mã hoá 1 phần SAM nhưng đã lỗi thời Microsoft khiến nghị dùng BitLocker140

Windows dùng 2 loại hash:
- LM (LAN Manager) — yếu, DES, không salt
- NTLM — mạnh hơn, MD4


