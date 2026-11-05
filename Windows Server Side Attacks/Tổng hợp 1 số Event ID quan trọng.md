
|Event ID|Log|Ý nghĩa|Dùng để phát hiện|
|---|---|---|---|
|**4624**|Security|Login success|Suspicious login, PTH|
|**4625**|Security|Login failure|Brute force|
|**4634**|Security|Logoff|Correlate với login|
|**4647**|Security|User initiated logoff|...|
|**4648**|Security|Logon with explicit credentials|PTH, lateral movement|
|**4672**|Security|Special privileges assigned|Admin login|
|**4688**|Security|Process creation|Command injection|
|**4697**|Security|Service installed|Persistence|
|**4720**|Security|User account created|Persistence|
|**4728**|Security|Member added to global group|Privilege escalation|
|**4732**|Security|Member added to local group|Privilege escalation|
|**4740**|Security|Account lockout|Brute force|
|**4768**|Security|Kerberos TGT requested|Pass the Ticket|
|**4769**|Security|Kerberos TGS requested|Kerberoasting|
|**4776**|Security|NTLM auth|PTH|
|**5140**|Security|Network share accessed|Lateral movement|
|**5145**|Security|Network share object checked|Lateral movement|
|**7045**|System|Service installed|PsExec, persistence|
|**1116**|Defender|Malware detected|Malware|
|**1117**|Defender|Action taken|Malware|
|**1**|Sysmon|Process creation|Hầu hết attack|
|**3**|Sysmon|Network connection|C2, reverse shell|
|**7**|Sysmon|Image loaded|DLL injection|
|**10**|Sysmon|Process access|Credential dumping|
|**11**|Sysmon|File creation|File upload, malware|
|**12/13/14**|Sysmon|Registry events|Persistence|
|**22**|Sysmon|DNS query|C2|
