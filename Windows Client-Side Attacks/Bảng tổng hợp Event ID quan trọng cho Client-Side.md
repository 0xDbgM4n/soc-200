
|Event ID|Log Source|Ý nghĩa|Dùng để phát hiện|
|---|---|---|---|
|**300**|Office Alerts|Document opened|Phishing attachment|
|**301**|Office Alerts|Document closed|...|
|**4624**|Security|Login success|Lateral movement|
|**4625**|Security|Login failure|Brute force|
|**4688**|Security|Process creation|Macro execution|
|**4103**|PowerShell Operational|Module logging|Cmdlet được gọi|
|**4104**|PowerShell Operational|Script block logging|**Quan trọng nhất**|
|**400**|PowerShell Classic|Engine started|...|
|**800**|PowerShell Classic|Pipeline execution|...|
|**1116**|Defender|Malware detected|AMSI, macro|
|**1117**|Defender|Action taken|...|
|**1**|Sysmon|Process creation|Macro → PowerShell|
|**3**|Sysmon|Network connection|C2|
|**7**|Sysmon|Image loaded|DLL injection|
|**10**|Sysmon|Process access|Credential dumping|
|**11**|Sysmon|File creation|Malware drop|
|**12/13/14**|Sysmon|Registry events|Persistence|
|**22**|Sysmon|DNS query|C2 domain|