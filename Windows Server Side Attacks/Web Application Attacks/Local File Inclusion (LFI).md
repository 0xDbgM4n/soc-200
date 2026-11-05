

📌 MÔ TẢ:
   - Lỗi web app cho phép đọc file tùy ý trên server
   - Ví dụ: WordPress Site Import plugin
   - URL: /wp-content/plugins/siteimport/admin/page.php?url=../../etc/passwd

📌 DẤU HIỆU:
   - Path traversal: ../, ..\, %2e%2e%2f
   - Đọc file nhạy cảm: wp-config.php, /etc/passwd, web.config
   - User-Agent bất thường: Python, curl, sqlmap

📋 LOG ARTIFACT:
   - IIS Log:
     • cs-uri-stem chứa ../ hoặc ..
     • cs(User-Agent) = Python-urllib, curl, sqlmap
     • sc-status = 200 (nếu thành công)
   - Sysmon Event 11: FileCreate nếu attacker ghi file

🔍 CÁCH PHÁT HIỆN:
   - KQL: `url.path: *"../"* or url.path: *"..\\"*`
   - Regex: `\.\./|\.\.\\|%2e%2e`
   - User-Agent đáng ngờ: python, curl, wget, sqlmap

⚠️ FALSE POSITIVE:
   - Web app hợp lệ dùng ../ (ít gặp)
   - Scanner bảo mật nội bộ

🛡️ MITIGATION:
   - Validate input
   - Whitelist file path
   - Cập nhật plugin