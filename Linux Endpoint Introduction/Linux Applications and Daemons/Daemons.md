- **Định nghĩa**: Là các chương trình chạy ngầm (background) không cần tương tác với người dùng. Tên gọi bắt nguồn từ "Maxwell's demon".
    
- **Đặc điểm**: Thường kết thúc bằng chữ `d` (ví dụ: `sshd`, `httpd`, `rsyslogd`, `systemd`).
    
- **Quản lý**: Sử dụng `systemctl` (trình quản lý dịch vụ systemd).
    
    - `systemctl status sshd` (Xem trạng thái)
        
    - `systemctl start sshd` (Khởi động)
        
    - `systemctl stop sshd` (Dừng)
        
    - `systemctl restart sshd` (Khởi động lại)