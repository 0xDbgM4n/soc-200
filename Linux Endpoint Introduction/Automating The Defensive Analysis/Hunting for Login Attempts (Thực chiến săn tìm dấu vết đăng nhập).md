### Các loại log SSH cần thuộc lòng

| Trạng thái             | Log Message Pattern                           | Ý nghĩa                                  |
| ---------------------- | --------------------------------------------- | ---------------------------------------- |
| **Valid (Thành công)** | `Accepted password for [user] from [IP]`      | Đăng nhập bằng mật khẩu đúng.            |
| **Valid (Thành công)** | `Accepted publickey for [user] from [IP]`     | Đăng nhập bằng SSH Key.                  |
| **Invalid (Thất bại)** | `Failed password for [user] from [IP]`        | Nhập sai mật khẩu.                       |
| **Invalid (Thất bại)** | `Failed publickey for [user] from [IP]`       | Dùng sai SSH Key.                        |
| **Invalid (Thất bại)** | `Connection closed by authenticating user...` | Kết nối bị đóng trước khi xác thực xong. |
| **Cảnh báo**           | `Too many authentication failures`            | Vượt quá số lần thử (MaxAuthTries).      |

### Phát hiện Brute Force và Password Spraying

- **Brute Force**: Thử **1 user**, với **rất nhiều password** (dùng tool như Hydra, Medusa).
    
- **Password Spraying**: Thử **1 password** (hoặc vài password phổ biến), với **rất nhiều user** (tránh bị lockout).
    
- **Dấu hiệu trong log**:
    
    - Hàng loạt dòng `Failed password` từ cùng 1 IP.
        
    - Các dòng `Failed password` cách nhau vài giây.
        
    - Thông báo `maximum authentication attempts exceeded` hoặc `Too many authentication failures`.
        
    - Logout ngay sau khi login thành công (dấu hiệu tool tự động thử thành công rồi thoát).