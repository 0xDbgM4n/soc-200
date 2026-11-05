┌─────────────────────────────────────────────────────────────┐
│  POWERSHELL LOGGING — 3 LOẠI CHÍNH                                          │
└─────────────────────────────────────────────────────────────┘

📌 3 LOẠI LOGGING:
   1. **Module Logging** (Event 4103)
      - Ghi lại pipeline execution
      - Log tất cả cmdlet được gọi
      - Bật qua Group Policy: "Turn on Module Logging"
      - Log vào: Microsoft-Windows-PowerShell/Operational

   2. **Script Block Logging** (Event 4104)
      - Ghi lại nội dung script block
      - Bao gồm cả command đã deobfuscate
      - Bật qua Group Policy: "Turn on PowerShell Script Block Logging"
      - Log vào: Microsoft-Windows-PowerShell/Operational
      - **QUAN TRỌNG NHẤT** để phát hiện macro độc hại

   3. **Transcription** (file .txt)
      - Ghi lại toàn bộ input/output của session
      - Lưu vào: C:\Users\<user>\Documents\PowerShell_transcript...
      - Bật qua Group Policy: "Turn on PowerShell Transcription"
      - Có thể chọn "Include invocation headers" để có timestamp

📌 CÁCH BẬT (Group Policy):
   gpedit.msc → Computer Configuration → Administrative Templates
   → Windows Components → Windows PowerShell
   → Bật 3 settings trên

📌 VỊ TRÍ LOG:
   - Operational log: Event Viewer → Applications and Services Logs
     → Microsoft → Windows → PowerShell → Operational
   - Transcription: C:\Users\<user>\Documents\YYYYMMDD\