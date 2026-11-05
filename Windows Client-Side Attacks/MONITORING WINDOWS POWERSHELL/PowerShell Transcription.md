┌─────────────────────────────────────────────────────────────┐
│  POWERSHELL TRANSCRIPTION                                                            │
│  MITRE: Execution > Command and Scripting Interpreter                     │
└─────────────────────────────────────────────────────────────┘

📌 MÔ TẢ:
   - Ghi lại toàn bộ session PowerShell (input + output)
   - Lưu thành file .txt
   - Có timestamp (nếu bật invocation headers)
   - Hữu ích cho forensic

📌 VỊ TRÍ FILE:
   C:\Users\<user>\Documents\YYYYMMDD\PowerShell_transcript.<HOST>.<USER>.<TIMESTAMP>.txt

📌 CẤU TRÚC FILE:

	Windows PowerShell transcript start  
	Start time: 20210615154405  
	Username: CLIENT01\offsec  
	RunAs User: CLIENT01\offsec  
	Configuration Name:  
	Machine: CLIENT01 (Microsoft Windows NT 10.0.19041.0)  
	Host Application: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -nop -w hidden -e aQBmACgA...  
	Process ID: 6524  
	PSVersion: 5.1.19041.1151  
	PSEdition: Desktop  
	...
	********************** 
	Command start time: 20210615142013 
	********************** 
	PS C:\Users\offsec> CommandInvocation(Get-CimInstance): "Get-CimInstance" 
	>> ParameterBinding(Get-CimInstance): name="ClassName"; value="Win32_ComputerSystem" 
	>> CommandInvocation(Select-Object): "Select-Object" >> ParameterBinding(Select-Object): name="Property"; value="Name, PrimaryOwnerName, Domain, TotalPhysicalMemory, Model, Manufacturer" 
	>> >> ParameterBinding(Select-Object): name="InputObject"; value="Win32_ComputerSystem: CLIENT01 (Name = "CLIENT01")" Name : CLIENT01 PrimaryOwnerName : offsec Domain : WORKGROUP TotalPhysicalMemory : 2146418688 Model : VMware7,1 Manufacturer : VMware, Inc.


📋 LOG ARTIFACT:
- Header: Host Application chứa encoded command
- CommandInvocation: Cmdlet được gọi
- Output của command

🔍 CÁCH PHÁT HIỆN:
- Monitor file transcript mới trong Documents
- Tìm keyword trong transcript: `-nop -w hidden -e`
- Correlate với Process ID trong Sysmon

⚠️ FALSE POSITIVE:
- Admin session bình thường

🛡️ MITIGATION:
- Bật Transcription
- Forward file về SIEM (qua agent)
- Restrict quyền đọc file