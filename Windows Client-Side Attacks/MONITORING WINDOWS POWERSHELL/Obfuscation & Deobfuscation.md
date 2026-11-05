┌─────────────────────────────────────────────────────────────┐
│  POWERSHELL OBFUSCATION                                                               │
│  MITRE: Defense Evasion > Obfuscated Files or Information                │
└─────────────────────────────────────────────────────────────┘

📌 MÔ TẢ:
   - Attacker làm rối code PowerShell để tránh detection
   - Khác với encoding (encoding là để chuyển đổi, obfuscation là để che giấu)
   - Tools: Invoke-Obfuscation (Daniel Bohannon)

📌 CÁC KỸ THUẬT OBFUSCATION:
   1. **Token Obfuscation**:
      - COMMAND: Ticks (`Ne\w- O\Bject`)
      - ARGUMENT: Random case, Reorder, Concatenate
      - STRING: Chia nhỏ string
      - MEMBER: Ticks
      - VARIABLE: Đổi tên
      - TYPE: Ticks
      - WHITESPACE: Chèn space ngẫu nhiên

   2. **AST Obfuscation** (PS 3.0+)
   3. **Encoding**: Base64
   4. **Compression**: Gzip + Base64
   5. **Launcher**: Các kỹ thuật launch khác nhau

📌 VÍ DỤ OBFUSCATED:

```
gE'T- ci'mIns'TANce ("5)[6](https://0.0.0.4/)[0](https://0.0.0.2/)[1](3]" - f'Co', 'puterSy', 'm', 'stem', '32', 'Wi', 'n') | s'ele'ct- OBJe'CT - Property ("1)[0]" - f'e', 'Nam'), ...
```


📌 DEOBFUSCATION TOOLS:
- **Revoke-Obfuscation** (Daniel Bohannon + Lee Holmes)
- `Get-RvoScriptBlock` để tái tạo script block từ event log
- Import: `Import-Module Revoke-Obfuscation.psm1`
- Export log: `wevtutil export-log Microsoft-Windows-PowerShell/Operational pwsh_export.evtx`
- Chạy: `Get-RvoScriptBlock -Path pwsh_export.evtx -Verbose`

📋 LOG ARTIFACT:

| Log Source | Event ID | Field | Giá trị mẫu |
|------------|----------|-------|-------------|
| PowerShell | 4104     | Message | Obfuscated script |
| PowerShell | 4103     | CommandInvocation | Get-CimInstance |

🔍 CÁCH PHÁT HIỆN:
- Rule: Script block có nhiều `'`, `"`, `+`, `-f` bất thường
- Rule: Random case trong cmdlet name
- Rule: Nhiều ký tự đặc biệt trong command
- Dùng **Revoke-Obfuscation** để score script block

⚠️ FALSE POSITIVE:
- Script phức tạp từ admin
- Generated code

🛡️ MITIGATION:
- Bật Script Block Logging (quan trọng nhất)
- Bật Module Logging
- Sử dụng Revoke-Obfuscation
- Alert trên script block có obfuscation score cao