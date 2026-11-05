### Rule 1: Office → PowerShell

```
event.code: 1 and process.parent.name: "WINWORD.EXE" and process.name: "powershell.exe"
```

**Mức độ**: Critical  
**Lý do**: Office không nên tạo process con PowerShell

### Rule 2: PowerShell Encoded Command

```
event.code: 1 and process.name: "powershell.exe" and process.command_line: *"-e "* or process.command_line: *"-EncodedCommand"*
```

**Mức độ**: High  
**Lý do**: Encoded command thường là malicious

### Rule 3: PowerShell Hidden Window

```
event.code: 1 and process.name: "powershell.exe" and process.command_line: *"-w hidden"*
```

**Mức độ**: High  
**Lý do**: User không cần hidden window

### Rule 4: AMSI Bypass

```
event.code: 4104 and powershell.file.script_block_text: *"AmsiScanBuffer"* or powershell.file.script_block_text: *"amsiutils"*
```

**Mức độ**: Critical  
**Lý do**: AMSI bypass là hành vi của attacker

### Rule 5: Script Block Warning

```
event.code: 4104 and log.level: "Warning"
```

**Mức độ**: Medium  
**Lý do**: PowerShell đánh dấu script block đáng ngờ

### Rule 6: csc.exe con của PowerShell

```
event.code: 1 and process.parent.name: "powershell.exe" and process.name: "csc.exe"
```

**Mức độ**: High  
**Lý do**: Compile C# code từ PowerShell thường là malicious

### Rule 7: DLL + .cmdline trong Temp

```
event.code: 11 and file.path: *"\\Temp\\*.dll"* or file.path: *"\\Temp\\*.cmdline"*
```

**Mức độ**: Medium  
**Lý do**: Artifact của C# compilation

### Rule 8: DNS query đến domain lạ

```
event.code: 22 and dns.question.name: *"kali"* or dns.question.name: *"attacker"*
```

**Mức độ**: High  
**Lý do**: Domain không phổ biến trong doanh nghiệp

