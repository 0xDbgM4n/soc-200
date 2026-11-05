- Các tệp log có thể được tìm thấy ở `C:\Windows\System32\winevt\Logs`. Chúng được lưu dưới dạng file `.evtx`
- Một số file có tên chung, 1 số khác có tên riêng của nhà cung cấp. Và tên nhà cung cấp rất quan trọng khi truy vấn `log` bằng `powershell`
- Các tệp log thì giới hạn quyền truy cập chỉ dành cho `(privileged users)` và data thì bị mã hoá dạng hex, vì vậy phải dùng công cụ `Windows Event Viewer`
- Có thể mở `Windows Event Viewer` bằng `CMD` thông qua lệnh `eventvwr.exe`

Các loại log:
- `Application Log`: nhật kí ứng dụng
- `Security Logs`: nhật kí an ninh
- `Setup Logs`: nhật kí cài đặt
- `System Logs`: nhật kí hệ thống

`Windows Event` có 1 level hoặc mức độ nghiêm trọng khác nhau, Các mức độ như sau: `Information`, `Warning`, `Error`, `Critical` có thể dùng thang level này để lọc ra các sự kiện mức high, critical

`Security Logs` gồm các sự kiện `Audit Success`, `Audit Failure`

---

![[Pasted image 20261013233939.png]]

Phân tích kĩ Event này (dạng XML)
- `TargetUserName`: cho biết ai đã đăng nhập vào thiết bị
- `LogonType`: là mã số (con số) được hệ điều hành ghi lại trong các sự kiện bảo mật (như Event ID 4624) nhằm xác định **hình thức hoặc phương thức** mà một người dùng hay tiến trình đã sử dụng để đăng nhập vào máy tính
- `IpAddress`: chứa IP của máy kết nối tới endpoint, nếu IP lạ ko có trong rule được phép kết nối thì xử lí ngay lập tức

