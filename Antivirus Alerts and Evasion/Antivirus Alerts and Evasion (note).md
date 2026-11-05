**Tags:** `#SOC-200` `#Antivirus` `#WindowsDefender` `#AMSI` `#Evasion` `#Sysmon`

## 1. Kiến trúc và Cơ chế của Windows Defender Antivirus

Để đánh bại (hoặc phát hiện) kẻ thù, ta phải hiểu kẻ thù. Windows Defender (WD) hoạt động qua 3 lớp chính:

### 1.1. Signature-Based Detection (Phát hiện dựa trên chữ ký)

- **Cơ chế:** Quét file trên đĩa (On-demand) hoặc quét file ngay khi tải về/chạy (Real-time). Nó so sánh chuỗi byte của file với cơ sở dữ liệu chữ ký (Definitions) lưu tại `%PROGRAMDATA%\Microsoft\Windows Defender\Definition Updates`.
    
- **Cách Evasion (Né tránh):**
    
    - Obfuscation (Làm rối mã).
        
    - Thay đổi 1 byte trong file (Hash thay đổi, chữ ký gãy).
        
    - **Điểm yếu:** Kẻ tấn công có thể dùng lệnh `MpCmdRun.exe` để xóa các định nghĩa (definitions) đang nạp trong bộ nhớ, làm mù Defender.
        

### 1.2. Heuristic-Based Detection (Phát hiện dựa trên Heuristics)

- **Cơ chế:** Không cần chữ ký. Nó phân tích **mã lệnh** (instruction set) của file nhị phân, decompile mã nguồn để tìm các "mẫu hành vi" đáng ngờ (ví dụ: một đoạn code liên tục mở file, hoặc thao tác bộ nhớ lạ).
    

### 1.3. Behavior-Based Detection (Phát hiện dựa trên Hành vi)

- **Cơ chế:** Chạy file trong môi trường giả lập (Sandbox/Emulation) để xem nó _thực sự_ làm gì. Nếu thấy nó gọi API tạo tiến trình con, ghi vào Registry, hoặc kết nối mạng lạ -> Chặn.
    
- **Điểm mạnh:** Chặn được cả malware biến thể (chưa có signature).
    
- **Logging:** Tạo ra các event **1116** (Phát hiện) và **1117** (Hành động xử lý - Action taken) trong Event Log `Microsoft-Windows-Windows Defender/Operational`.
    

> **Góc nhìn SOC Analyst:**
> 
> - Event 1116: Phát hiện mối nguy (Severity: Severe/Critical).
>     
> - Event 1117: Đã xử lý (Remove, Quarantine).
>     
> - Sử dụng `Get-MpThreat` và `Get-MpThreatDetection` trong PowerShell để truy vấn nhanh.


## 2. Antimalware Scan Interface (AMSI)

Đây là "vũ khí tối thượng" của Microsoft để chống lại **Fileless Malware** (Malware không lưu file xuống đĩa, chạy trực tiếp trên RAM, thường qua PowerShell).

### 2.1. Cơ chế hoạt động của AMSI

- AMSI là một API (thư viện `amsi.dll`) được nạp vào các tiến trình có khả năng thực thi script (PowerShell, VBScript, JScript, Office Macros).
    
- Khi bạn chạy một lệnh PowerShell, AMSI sẽ chặn nội dung script đó lại, gửi qua RPC đến Windows Defender để phân tích.
    
- Nếu Defender báo "Malicious", AMSI sẽ chặn không cho PowerShell thực thi lệnh đó.
    
- **Hàm quan trọng:** `AmsiScanBuffer` và `AmsiOpenSession`. Nếu một trong hai hàm này trả về mã lỗi hoặc bị vô hiệu hóa, AMSI sẽ bị "mù".

![X](./Pasted%20image%2020261021235115.png)

### 2.2. Các kỹ thuật Bypass AMSI phổ biến (dành cho Pentester/Red Team)

|Kỹ thuật|Cơ chế|Dấu vết để lại (Detection)|
|---|---|---|
|**1. Patching `AmsiScanBuffer`**|Ghi đè (Overwrite) byte đầu tiên của hàm `AmsiScanBuffer` trong bộ nhớ bằng lệnh `ret` (0xC3) hoặc `xor rax, rax` (0x31, 0xC0). Khiến hàm luôn trả về kết quả "sạch".|- Sysmon Event ID 1 (Process Create): Thấy `powershell.exe` gọi đến `csc.exe` (C# Compiler) và `cvtres.exe` (biên dịch mã C#) để tạo DLL động.  <br>- PowerShell Script Block Logging (4104): Ghi lại đoạn script chứa các byte patch (`0x31, 0xC0, 0xC3`).|
|**2. Overwriting AMSI Context**|Không sửa hàm, mà sửa vùng nhớ Context (ngữ cảnh) của AMSI bằng cách đổ toàn số 0 (`null`).|Tương tự như trên, nhưng script block sẽ chứa các lệnh `[Ref].Assembly.GetTypes()` và `[IntPtr]$ptr=$g` để can thiệp bộ nhớ.|
|**3. Obfuscation**|Dùng công cụ (như `Invoke-Obfuscation`) để làm rối mã (chèn ký tự, đảo ngược chuỗi).|PowerShell Script Block Logging (4104) sẽ ghi lại mã đã bị làm rối. Tuy nhiên, nếu bật **Script Block Logging**, PowerShell sẽ tự động "deobfuscate" (giải mã) và ghi lại nội dung gốc.|
|**4. Downgrade PowerShell**|Hạ cấp phiên bản PowerShell xuống v2 (không hỗ trợ AMSI).|Log sự kiện thay đổi cấu hình Registry (Sysmon Event 13) hoặc Event 400 (Engine state changed).|

> 🚩 **Red Flags (Dấu hiệu nhận biết):**
> 
> 1. Bất kỳ tiến trình nào gọi `powershell.exe` từ Word, Excel, hoặc một process lạ -> **Cực kỳ đáng ngờ**.
>     
> 2. PowerShell gọi `csc.exe` (C# compiler) -> Dấu hiệu của việc compile mã độc tại chỗ (In-memory compilation).
>     
> 3. Event 1116 từ Defender với `Detection Source: AMSI` và tên mã độc là `Trojan:Win32/AmsiTamper.A!ams`.
>     

---

## 3. Các Artifacts quan trọng trên Endpoint (Sysmon & Event Log)

Khi Malware/Attacker vượt qua được AV hoặc tấn công vào hệ thống, chúng sẽ để lại dấu vết. Là một SOC Analyst, bạn phải "bắt" được các dấu vết này.

### 3.1. Phân tích qua Sysmon (Event ID 1 - Process Create)

- **Dấu hiệu Malware chạy:**
    
    - `Image`: Đường dẫn của file thực thi (Ví dụ: `C:\Windows\Temp\nc.exe`, `C:\Users\Public\svchost.exe`).
        
    - `ParentImage`: Rất quan trọng. Ví dụ: `WINWORD.EXE` tạo ra `powershell.exe`, hoặc `cmd.exe` tạo ra `powershell.exe` với tham số `-enc` (encoded command).
        
    - `CommandLine`: Chứa Base64 strings (bắt đầu bằng `-e` hoặc `-enc`).
        
    - `Hashes`: MD5, SHA256, IMPHASH. Kiểm tra hash trên VirusTotal.
        

### 3.2. Phân tích qua PowerShell Logging (Event ID 4103 & 4104)

- **4103 (Module Logging):** Ghi lại việc gọi các lệnh (cmdlet) và tham số của chúng.
    
- **4104 (Script Block Logging):** Ghi lại **nội dung** của khối mã (script block). Đây là "bảo bối" để đọc các đoạn script đã bị obfuscate.
    
    - _Mẹo:_ Nếu thấy event 4104 chứa các lệnh như `IEX (New-Object Net.WebClient).DownloadString(...)` -> Đây là `Download cradle` kinh điển của fileless malware.
        

### 3.3. Phân tích qua Windows Defender Events

- **Event ID 1116:** Phát hiện malware.
    
    - `Detection Source`: Real-Time Protection / AMSI.
        
    - `Path`: Nơi file bị phát hiện.
        
- **Event ID 1117:** Đã thực hiện hành động (Action: Remove/Quarantine).
    
- **Event ID 5007:** Cấu hình Defender bị thay đổi (Kẻ tấn công dùng lệnh `Set-MpPreference` để tắt Real-time protection hoặc thêm loại trừ - Exclusion).
    

---

## 4. Quy trình Điều tra (SOC Workflow)

Khi nhận được cảnh báo về Antivirus/AMSI hoặc nghi ngờ có Evasion, hãy làm theo các bước sau:

1. **Xác minh (Verify):** Kiểm tra Event Log `Microsoft-Windows-Windows Defender/Operational`. Xem có Event 1116 (Detected) và 1117 (Action taken) không. Xem đường dẫn file và tiến trình tạo ra nó.
    
2. **Truy vết (Trace):**
    
    - Mở Sysmon Event 1 -> Tìm `ProcessId` của tiến trình độc hại -> Xem `ParentProcessId` để biết nó được sinh ra từ đâu.
        
    - Kiểm tra xem file đó có được tải từ đâu về không (Sysmon Event 3 - NetworkConnect, hoặc Event 11 - FileCreate).
        
3. **Phân tích Payload (Analyze):**
    
    - Nếu là PowerShell, kéo log 4104 ra xem script thực sự làm gì.
        
    - Sử dụng công cụ như `Revoke-Obfuscation` (trong lab) để giải mã script bị làm rối.
        
4. **Phản ứng (Respond):** Nếu phát hiện bị Bypass (ví dụ: PowerShell đã bị gỡ AMSI), tiến hành cô lập máy, thu thập bộ nhớ (Memory dump) để phân tích, và xóa các file độc hại.
    

---

### 📝 Tóm tắt (TL;DR)

> Antivirus/AMSI là lớp phòng thủ mạnh nhưng có thể bị vượt qua (Patching, Obfuscation). Để phát hiện, SOC Analyst phải dựa vào sự kết hợp của **Sysmon (Process Create)**, **PowerShell Script Block Logs (4104)**, và **Windows Defender Operational Logs (1116/1117)**. Kẻ tấn công càng cố gắng né tránh (Evasion) thì càng tạo ra nhiều Artifacts (như compiler C#, tiến trình lạ) để lại dấu vết.
