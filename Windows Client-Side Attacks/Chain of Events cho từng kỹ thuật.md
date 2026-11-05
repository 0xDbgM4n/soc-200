### Phishing + Macro + Reverse Shell

```
1. User nhận email → Office Alerts 300 (Document opened)
2. User enable content → Sysmon 1: WINWORD.EXE (Process Create)
3. Macro chạy → Sysmon 1: powershell.exe (Parent: WINWORD.EXE)
   CommandLine: -nop -w hidden -e aQBmACgA...
4. PowerShell 4104: Script block với encoded payload
5. Sysmon 11: File created: __PSScriptPolicyTest_xxx.ps1
6. PowerShell download payload → Sysmon 3: NetworkConnect đến attacker:443
7. Sysmon 22: DNS query đến attacker domain
8. Reverse shell → Sysmon 1: cmd.exe (Parent: powershell.exe)
9. C2 connection → Sysmon 3: NetworkConnect
```

### AMSI Bypass

```
1. PowerShell 4104 (Warning): Script block với AmsiScanBuffer
2. Sysmon 11: File created: brfgb1cn.cmdline
3. Sysmon 11: File created: brfgb1cn.dll
4. Sysmon 1: csc.exe (Parent: powershell.exe)
5. Sysmon 1: cvtres.exe (Parent: csc.exe)
6. Sysmon 11: File created: brfgb1cn.dll (overwrite)
7. PowerShell 4104: Script block chạy payload (AMSI đã bị bypass)
```

### Obfuscation

```
1. PowerShell 4104: Script block obfuscated
2. PowerShell 4103: Get-CimInstance được gọi
3. PowerShell 4103: Select-Object được gọi
4. Sử dụng Revoke-Obfuscation để deobfuscate
```

