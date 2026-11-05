┌──────────────────────────────────────────────────────────────────────┐
│  AMSI BYPASS                                                                                                        │
│  MITRE: Defense Evasion > Impair Defenses > Disable or Modify Tools               │
└──────────────────────────────────────────────────────────────────────┘

📌 AMSI LÀ GÌ?
   - Antimalware Scan Interface
   - API cho phép AV scan script trước khi chạy
   - Được load vào mọi process PowerShell
   - Functions: AmsiInitialize, AmsiScanBuffer, AmsiScanString

📌 2 KỸ THUẬT BYPASS PHỔ BIẾN:
   1. **Patch AmsiScanBuffer** (Paul Laîné 2019)
      - Tìm function trong memory
      - Ghi đè bằng `xor rax, rax; ret` (return 0 = clean)
      - Kết quả: AMSI luôn trả về "clean"

   2. **Overwrite AMSI Context**
      - Tìm context structure của AmsiInitialize
      - Ghi đè bằng zero
      - Kết quả: AMSI không hoạt động

📌 DẤU HIỆU:
   - PowerShell script chứa `AmsiUtils`, `AmsiScanBuffer`, `amsiutils`
   - PowerShell script chứa `VirtualProtect`, `Marshal.Copy`
   - PowerShell script compile C# code (csc.exe)
   - File .dll và .cmdline trong Temp

📋 LOG ARTIFACT:

| Log Source | Event ID | Field          | Giá trị mẫu                   |
| ---------- | -------- | -------------- | ----------------------------- |
| PowerShell | 4104     | Message        | function vjm {...}            |
| PowerShell | 4104     | Level          | Warning                       |
| Sysmon     | 1        | Image          | csc.exe                       |
| Sysmon     | 1        | ParentImage    | powershell.exe                |
| Sysmon     | 11       | TargetFilename | *.dll trong Temp              |
| Defender   | 1116     | Name           | Trojan:Win32/AmsiTamper.A!ams |


🔗 CHAIN OF EVENTS:

```
12:35:07 → PowerShell 4104 (Warning): Script block với AMSI bypass  
12:35:07 → Sysmon 11: File created: brfgb1cn.cmdline  
12:35:07 → Sysmon 11: File created: brfgb1cn.dll  
12:35:08 → Sysmon 1: csc.exe (Parent: powershell.exe)  
12:35:08 → Sysmon 1: cvtres.exe (Parent: csc.exe)  
12:35:08 → Sysmon 11: File created: brfgb1cn.dll (overwrite)
```


🔍 CÁCH PHÁT HIỆN:
- **Rule quan trọng**: PowerShell 4104 với Level=Warning
- Rule: PowerShell 4104 chứa keyword AMSI
- Rule: csc.exe con của powershell.exe
- Rule: File .dll + .cmdline trong Temp cùng lúc
- Rule: Defender 1116 với source=AMSI

⚠️ FALSE POSITIVE:
- Developer compile C# code (hiếm)
- Software build tools

🛡️ MITIGATION:
- Bật Script Block Logging
- Bật AMSI
- Constrained Language Mode
- WDAC (Windows Defender Application Control)
- Alert trên Level=Warning