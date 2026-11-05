Vị trí log: `C:\inetpub\logs\LogFiles\W3SVC1\u_exYYMMDD.log`

Các trường `W3C` và mô tả:
	• s-ip - Địa chỉ IP của máy chủ tiếp nhận hoạt động web
	• cs-method - Phương thức HTTP mà máy khách yêu cầu (ví dụ: GET)
	• cs-uri-stem - Tệp tin mục tiêu được yêu cầu từ máy chủ web
	• s-port - Cổng máy chủ của dịch vụ web (ví dụ: 80)
	• c-ip - Địa chỉ IP của máy khách yêu cầu trang web
	• cs(User-Agent) - Chuỗi User-Agent của trình duyệt web trên máy khách
	• cs(Referer) - HTTP Referer (trang trước đó đã dẫn người dùng đến tệp tin mục tiêu, nếu có)
	• sc-status - Mã trạng thái HTTP (HTTP Status Code) từ máy chủ (ví dụ: 200 nếu tìm thấy, 404 nếu không tìm thấy)

DEFAULT LOG LOCATIONS:
   - IIS: C:\inetpub\logs\LogFiles\W3SVC1\
   - Apache (CentOS): /var/log/httpd/access_log
   - Apache (Ubuntu): /var/log/apache2/access.log


