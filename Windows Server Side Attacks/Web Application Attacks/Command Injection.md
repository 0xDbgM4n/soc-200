
📌 MÔ TẢ:
   - Web app cho phép chèn command OS vào input
   - Ví dụ: Plainview Activity Monitor plugin
   - Payload: t.co|systeminfo

📌 DẤU HIỆU:
   - Pipe `|`, `;`, `&&`, `||` trong input
   - Command OS: whoami, systeminfo, ipconfig, net user
   - Download tool: certutil, powershell, wget
   - User-Agent bất thường

📋 LOG ARTIFACT:
   - IIS Log:
     • cs-uri-query chứa `|`, `;`, `%7C`, `%3B`
     • cs(User-Agent) = curl, python, sqlmap
     • time-taken cao bất thường (do command chạy lâu)
   - Sysmon Event 1 (ProcessCreate):
     • Image = cmd.exe, powershell.exe
     • ParentImage = php-cgi.exe, w3wp.exe (IIS)
     • CommandLine = payload
     • User = IUSR, IIS APPPOOL\...
   - Sysmon Event 11 (FileCreate):
     • File được tải về (nc.exe, mimikatz.exe)

🔍 CÁCH PHÁT HIỆN:
   - KQL: `url.original: *"|"* or url.original: *"systeminfo"*`
   - Sysmon: ParentImage = php-cgi.exe AND Image = cmd.exe
   - Rule: Process con của web server tạo cmd.exe/powershell.exe

⚠️ FALSE POSITIVE:
   - Web app hợp lệ gọi system command (ít gặp)

🛡️ MITIGATION:
   - Không dùng system() với input user
   - Escape/validate input
   - Chạy web server với quyền thấp