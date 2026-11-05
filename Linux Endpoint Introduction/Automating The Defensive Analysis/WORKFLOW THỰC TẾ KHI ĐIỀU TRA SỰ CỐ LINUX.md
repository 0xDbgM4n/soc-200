Khi bạn nhận được cảnh báo "Có dấu hiệu tấn công SSH vào server Linux", bạn sẽ làm theo các bước sau:

1. **Bước 1: Xác định vị trí log** (Ubuntu hay CentOS?) -> Xem bảng file log ở trên.
    
2. **Bước 2: Tìm nguồn tấn công (IP, User)**
    
    - Dùng lệnh `grep "Failed password" /var/log/auth.log` để xem log.
        
    - Dùng `awk`, `sort`, `uniq` để đếm số lần thử.
        
3. **Bước 3: Kiểm tra xem có thành công không?**
    
    - Dùng `grep "Accepted" /var/log/auth.log` để xem có ai đăng nhập được từ IP đó không.
        
    - Nếu có, kiểm tra các lệnh đã chạy (qua `.bash_history` hoặc `journalctl`) để xem hacker đã làm gì.
        
4. **Bước 4: Phân tích lỗ hổng**
    
    - Tại sao hacker vào được? Do mật khẩu yếu? Do lộ SSH key? Do cấu hình sai?
        
5. **Bước 5: Viết Rule (Detection & Prevention)**
    
    - Viết script Python tự động bắt pattern `Failed password` và gửi cảnh báo.
        
    - Hoặc cấu hình `fail2ban` để tự động block IP sau N lần sai.