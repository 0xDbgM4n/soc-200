
![x](./Pasted%20image%2020261020155156.png)

Các quy tắc của Snort bao gồm hai thành phần chính: `tiêu đề quy tắc` và `các tùy chọn quy tắc.`

Phân tích 1 chút:
- `Tiêu đề` trong rule này sẽ đưa ra cảnh báo đối với traffic ICMP từ src nội bộ đến any internet theo cơ chế hai chiều
- `Các tùy chọn quy tắc` đặt trong `ngoặc đơn` bao gồm `log message`, `snort ID`, `metadata tag`

![x](./Pasted%20image%2020261020155748.png)

Cần lưu ý rằng mỗi lệnh ping sẽ tạo ra hai mục nhật ký: 
- Mục thứ nhất tương ứng với yêu cầu ICMP Echo (ICMP Echo request) do máy Kali tạo ra, và 
- Mục thứ hai tương ứng với phản hồi ICMP Echo (ICMP Echo reply) do cổng kết nối Snort tạo ra. 
Điều này cho thấy một kịch bản trong đó không chỉ một cuộc tấn công được phát hiện, mà bản thân nạn nhân cũng tham gia vào quá trình trao đổi đó.

**Nội dung:**

- Cấu trúc: `action protocol src_ip src_port -> dst_ip dst_port (options)`
    
- **Rule Header:** Các action (`alert`, `log`, `pass`, `drop`).
    
- **Rule Options quan trọng:**
    
    - `msg`, `sid`, `rev`
        
    - `content`, `nocase`, `depth`, `offset`, `distance`, `within`
        
    - `flow`
        
    - `byte_test`, `byte_extract` (dùng để so sánh byte).
        
    - `detection_filter` (ngưỡng).
        
- **Ví dụ:** Luật ICMP cơ bản.

https://docs.snort.org/rules/
