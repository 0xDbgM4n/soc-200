┌─────────────────────────────────────────────────────────────┐
│  OFFICE MACRO                                                                                     │
│  MITRE: Execution > User Execution > Malicious File                            │
└─────────────────────────────────────────────────────────────┘

📌 MÔ TẢ:
   - Macro = chuỗi lệnh VBA trong Office document
   - Attacker nhúng macro độc hại vào file
   - Khi user "Enable Content" → macro chạy
   - Macro thường gọi PowerShell để tải payload

📌 CÁC BƯỚC TẤN CÔNG:
   1. Attacker tạo file .docm với macro VBA
   2. Gửi qua email phishing
   3. User mở file → Protected View
   4. User click "Enable Editing"
   5. User click "Enable Content" → macro chạy
   6. Macro gọi PowerShell với payload encoded
   7. PowerShell tải reverse shell (Meterpreter)
   8. C2 connection được thiết lập

📌 DẤU HIỆU:
   - Word mở file từ Temp/INetCache
   - Word chạy PowerShell
   - PowerShell với `-nop -w hidden -e <base64>`
   - ParentProcess = WINWORD.EXE

📋 LOG ARTIFACT:

| Log Source | Event ID | Field             | Giá trị mẫu           |
| ---------- | -------- | ----------------- | --------------------- |
| Sysmon     | 1        | Image             | powershell.exe        |
| Sysmon     | 1        | ParentImage       | WINWORD.EXE           |
| Sysmon     | 1        | CommandLine       | -nop -w hidden -e ... |
| Sysmon     | 11       | TargetFilename    | *.ps1 trong Temp      |
| Sysmon     | 12       | QueryName         | kali (domain lạ)      |
| Sysmon     | 3        | DestinationPort   | 443, 80               |
| PowerShell | 4104     | ScriptBlockText   | Encoded payload       |
| PowerShell | 4103     | CommandInvocation | ...                   |

🔗 CHAIN OF EVENTS ĐIỂN HÌNH:

	15:10:38 → Sysmon 1: WINWORD.EXE (Process Create)  
	15:10:39 → Sysmon 1: powershell.exe (Parent: WINWORD.EXE)  
	CommandLine: -nop -w hidden -e aQBmACgAWwBJAG4A...  
	15:10:39 → Sysmon 11: File created: __PSScriptPolicyTest_xxx.ps1  
	15:10:40 → Sysmon 1: powershell.exe (second stage)  
	CommandLine: powershell -nop -w hidden -c &(scriptblock...)  
	15:10:40 → Sysmon 11: File created: __PSScriptPolicyTest_yyy.ps1  
	15:10:42 → Sysmon 22: DNS Query: kali (attacker domain)  
	15:10:43 → Sysmon 3: Network connection đến attacker:443


🔍 CÁCH PHÁT HIỆN:
- **Rule quan trọng nhất**: `ParentImage = WINWORD.EXE AND Image = powershell.exe`
- Rule: WINWORD.EXE tạo process con (bất kỳ)
- Rule: PowerShell với `-nop -w hidden -e` hoặc `-EncodedCommand`
- Rule: File .ps1 được tạo trong Temp
- Rule: DNS query đến domain không phổ biến

⚠️ FALSE POSITIVE:
- Macro hợp lệ (tài liệu nội bộ)
- Add-in Office hợp lệ
- Template có macro

🛡️ MITIGATION:
- **Disable macro by default** (Group Policy)
- **Mark of the Web** cho file từ internet
- **ASR rules**:
  • Block Office applications from creating child processes
  • Block Office applications from creating executable content
  • Block Win32 API calls from Office macros
- **Protected View** cho file từ internet
- User awareness training

https://support.microsoft.com/en-us/word/create-or-run-a-macro