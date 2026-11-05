### Đặc điểm khác so với `Server-Side`

|Tiêu chí|Server-Side|Client-Side|
|---|---|---|
|**Đối tượng**|Server (IIS, Apache)|Endpoint (Windows client)|
|**Vector**|Network service|Email, web, file|
|**Điều kiện**|Lỗ hổng service|User interaction (click, open)|
|**Log chính**|IIS Log, Sysmon|Windows Event Log, Sysmon, PowerShell Log|
|**Khó phát hiện**|Trung bình|**Khó hơn** (do user behavior)|

### Attack Chain điển hình

1. `Recon: Thu thập email, thông tin nhân viên`
2. `Delivery: Gửi email phishing có attachment`
3. `Exploitation: User mở attachment → macro chạy`
4. `Execution: Macro gọi PowerShell → tải payload`
5. `C2: Reverse shell kết nối về attacker`
6. `Actions: Duy trì, di chuyển ngang, đánh cắp dữ liệu`

### Log source cần nhớ

|Log Source|Vị trí|Event ID quan trọng|
|---|---|---|
|**Security Log**|Security.evtx|4624, 4625, 4688, 4648|
|**Sysmon**|Microsoft-Windows-Sysmon/Operational|1, 3, 7, 11, 22|
|**PowerShell Operational**|Microsoft-Windows-PowerShell/Operational|4103, 4104|
|**PowerShell Classic**|Windows PowerShell|400, 403, 600, 800|
|**Windows Defender**|Microsoft-Windows-Windows Defender/Operational|1116, 1117|
|**Office**|Microsoft Office Alerts|300, 301|
|**AMSI**|Microsoft-Windows-Windows Defender/Operational|1116 (source=AMSI)|
