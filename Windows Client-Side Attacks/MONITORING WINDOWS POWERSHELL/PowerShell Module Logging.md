┌─────────────────────────────────────────────────────────────┐
│  MODULE LOGGING — EVENT 4103                                                     │
│  MITRE: Execution > Command and Scripting Interpreter                    │
└─────────────────────────────────────────────────────────────┘

📌 MÔ TẢ:
   - Ghi lại mỗi cmdlet được gọi trong pipeline
   - Bao gồm parameters và values
   - Giúp trace chính xác những gì script làm

📌 CẤU TRÚC EVENT 4103:
   - CommandInvocation (tên cmdlet)
   - ParameterBinding (name=..., value=...)
   - Context: 
     • Severity
     • Host Name (ConsoleHost, ServerRemoteHost)
     • Host Application
     • Engine Version
     • Runspace ID
     • Pipeline ID
     • Command Name
     • Command Type
     • Sequence Number
     • User

📋 LOG ARTIFACT:

| Field             | Ý nghĩa         | Ví dụ                               |
| ----------------- | --------------- | ----------------------------------- |
| CommandInvocation | Cmdlet được gọi | Get-WmiObject                       |
| ParameterBinding  | Tham số         | name="Class"; value="Win32_Process" |
| Pipeline ID       | ID của pipeline | 1                                   |
| Sequence Number   | Thứ tự thực thi | 16                                  |
| User              | User chạy       | CLIENT01\offsec                     |

🔍 CÁCH PHÁT HIỆN:
   - Query: Event 4103 với CommandInvocation đáng ngờ
   - Ví dụ: Invoke-Expression, IEX, DownloadString, Invoke-WebRequest
   - Group by Pipeline ID để xem toàn bộ pipeline

⚠️ FALSE POSITIVE:
   - Admin scripts hợp lệ
   - Software management tools

🛡️ MITIGATION:
   - Bật Module Logging cho tất cả modules
   - Forward log về SIEM
   - Monitor cmdlet đáng ngờ