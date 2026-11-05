- Định dạng: **Combined Log Format (CLF)**.
    
- **Cấu trúc**: `Source_IP - - [Timestamp] "Method Path HTTP_Version" Status_Code Size "Referer" "User_Agent"`
    
- _Ví dụ:_ `192.168.51.50 - - [12/Jul/2021:08:57:30 -0400] "GET /admin HTTP/1.1" 403 199691 "-" "Mozilla/5.0..."`
    
- **Kỹ thuật trích xuất (Parsing)**:
    
    - `cut -d " " -f 7` (Lấy cột thứ 7 - Path)
        
    - `grep " 404 "` (Lấy các request bị lỗi 404)