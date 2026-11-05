┌─────────────────────────────────────────────────────────────┐
│  SCRIPT BLOCK LOGGING — EVENT 4104                                             │
│  MITRE: Execution > Command and Scripting Interpreter                    │
└─────────────────────────────────────────────────────────────┘

📌 MÔ TẢ:
   - Ghi lại nội dung của mỗi script block
   - Bao gồm command đã deobfuscate (quan trọng!)
   - Giúp phát hiện macro độc hại ngay cả khi encode

📌 CẤU TRÚC EVENT 4104:
   - Message: "Creating Scriptblock text (1 of 1): <nội dung>"
   - ScriptBlock ID (GUID)
   - Path (nếu từ file)

📌 LEVELS:
   - **Verbose**: Script block bình thường
   - **Warning**: Script block đáng ngờ (thường là malicious)

📋 LOG ARTIFACT:

| Field            | Ý nghĩa         | Ví dụ              |
| ---------------- | --------------- | ------------------ |
| Message          | Nội dung script | function vjm {...} |
| ScriptBlock ID   | GUID            | e01d4655-...       |
| LevelDisplayName | Verbose/Warning | Warning            |

🔍 CÁCH PHÁT HIỆN:
   - Query: Event 4104 với Level = Warning
   - Tìm keyword: `Invoke-Expression`, `IEX`, `DownloadString`, `FromBase64String`
   - Tìm keyword: `AmsiUtils`, `AmsiScanBuffer`, `amsiutils` (AMSI bypass)
   - Tìm keyword: `VirtualAlloc`, `CreateThread`, `Marshal` (shellcode)

⚠️ FALSE POSITIVE:
   - Admin scripts phức tạp
   - Software installer

🛡️ MITIGATION:
   - Bật Script Block Logging
   - Forward log về SIEM
   - Alert trên Level=Warning