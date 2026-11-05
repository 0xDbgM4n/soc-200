**Nội dung:**

- **Khái niệm C2:** Server điều khiển, Agent trên máy nạn nhân.
    
- **Pyramid of Pain:** Giải thích các loại IOC (Hash, IP, Domain, Tools, TTPs) và mức độ khó khăn cho kẻ tấn công.
    
- **Domain Generation Algorithm (DGA):** Cách malware tạo domain ngẫu nhiên.
    
- **Phân tích ví dụ với Empire Framework:**
    
    - Snort phát hiện C2 của Empire thông qua:
        
        - URI cố định (`/news.php`).
            
        - User-Agent giả mạo (IE11 trên Windows 7).
            
    - Phân tích luật Snort mẫu: `content` để khớp URI và User-Agent.
        
- **Chiến lược phòng thủ:** Khi gặp C2 tùy chỉnh, cần phân tích endpoint (log Sysmon, PowerShell) để tìm ra agent, reverse engineer và tạo signature mới.

---
### 1.Khái niệm C2 (Command & Control)

- **Định nghĩa:** Kiến trúc tập trung mà kẻ tấn công dùng để điều khiển từ xa nhiều máy bị xâm nhập từ **một điểm truy cập duy nhất**.
    
- **Tên gọi khác:** C&C, CnC, C2.
    
- **MITRE ATT&CK Tactic:** `TA0011 - Command and Control`.
    
- **Thành phần kiến trúc:**
    
    1. **Stager (giai đoạn 1):** Payload nhẹ, chạy sau khi máy bị xâm nhập lần đầu. Nhiệm vụ: **beacon** (gọi về) C2 server để tải agent.
        
    2. **Agent (giai đoạn 2):** Payload chính, được nạp vào memory. Nhiệm vụ: nhận lệnh, thực thi, exfiltrate dữ liệu.
        
    3. **Operator:** Người điều khiển C2 master server.
        
    4. **C2 Master Server:** Máy chủ trung tâm quản lý toàn bộ agent.
        
- **Chuỗi lây nhiễm điển hình:**  
    `Phishing/Drive-by → Stager → Beacon C2 → Tải Agent → Nhận lệnh → Exfil/Lateral Movement`
    

> **Ghi nhớ:** C2 không chỉ là "kênh liên lạc" — nó là **toàn bộ hạ tầng** gồm server, domain/IP, giao thức, agent, và kênh mã hóa.

---
### 2.Pyramid of Pain (Kim tự tháp nỗi đau - David J. Bianco)

Đây là mô hình phân loại **IOC (Indicator of Compromise)** theo mức độ khó khăn mà kẻ tấn công phải đối mặt khi defender phát hiện & chặn chúng.

| Tầng | Loại IOC                                   | Mức độ khó (với Attacker) | Mô tả                                                                           |
| ---- | ------------------------------------------ | ------------------------- | ------------------------------------------------------------------------------- |
| Đỉnh | **TTPs** (Tactics, Techniques, Procedures) | **Tough** (Rất khó)       | Hành vi, chiến thuật. Phát hiện = buộc attacker phải thay đổi toàn bộ cách làm. |
| 4    | **Tools**                                  | **Challenging**           | Phần mềm, mã độc do attacker tạo. Phát hiện = attacker phải viết lại tool.      |
| 3    | **Network/Host Artifacts**                 | **Annoying**              | Byte đặc trưng, dấu vết trong traffic.                                          |
| 2    | **Domain Names**                           | **Simple**                | Domain/subdomain dùng để resolve C2.                                            |
| 1    | **IP Addresses**                           | **Easy**                  | IPv4/IPv6, CIDR của hạ tầng C2.                                                 |
| Đáy  | **Hash Values**                            | **Trivial**               | MD5, SHA256 của file. Chỉ cần đổi 1 byte là hash thay đổi.                      |

**Ý nghĩa phòng thủ:**

- Chặn ở tầng **Hash/IP/Domain** → attacker chỉ cần đổi infrastructure là xong (rẻ, nhanh).
    
- Phát hiện ở tầng **TTPs** → attacker phải **thay đổi cách vận hành** (đắt, chậm).
    
- **Mục tiêu của SOC analyst:** Cố gắng leo lên càng cao trên kim tự tháp càng tốt.
    
- **Với C2:** Domain/IP có thể bị block dễ dàng → attacker dùng **DGA** để né.

---
### 3.Domain Generation Algorithm (DGA)

- **Vấn đề của C2 tĩnh:** IP/Domain cố định → defender block là chết C2.
    
- **Giải pháp của attacker:** Dùng DGA sinh domain **giả ngẫu nhiên tại runtime** → không có domain/IP hardcode.
    
- **MITRE ATT&CK:** `T1568.002 - Dynamic Resolution: Domain Generation Algorithms`.
    
- **Cách hoạt động:**
    
    - Input: seed (ngày, giờ, số nguyên tố, từ điển, URL khác...).
        
    - Output: chuỗi domain giả ngẫu nhiên (thường 16 ký tự + TLD).
        
    - Agent và C2 server dùng cùng seed → cùng sinh ra domain → agent biết gọi về đâu.

**Ví dụ DGA (dga.py trong lab):**
```
# Input: ngày dạng YYYY.MM.DD
# Output: domain 16 ký tự
# Ví dụ: python3 dga.py 11.02.2021 → bkjaewxytklxmnpb.com
```

- **Biến thể nguy hiểm:** DGA có thể dùng **social media platform** làm kênh C2 (VD: MiniDuke dùng Twitter).
    
- **Cách phát hiện DGA:**
    
    - Phân tích DNS query bất thường (nhiều domain ngẫu nhiên, không có ý nghĩa).
        
    - Domain có entropy cao (nhiều ký tự ngẫu nhiên).
        
    - NXDOMAIN rate cao (agent thử nhiều domain trước khi tìm được domain thật).

---
### 4.C2 qua Application Layer Protocol (MITRE T1071)

Attacker dùng nhiều giao thức để giấu C2:

|Giao thức|Đặc điểm|Ghi chú|
|---|---|---|
|**HTTP/HTTPS**|Phổ biến nhất, dễ blend với traffic thường|Empire dùng HTTP mặc định|
|**DNS**|Khó block (DNS hay bị bỏ qua)|DNS tunneling|
|**IRC**|Cũ, ít dùng|Dễ phát hiện|
|**WebSocket**|APT28 (Dvororub) dùng|Khó phát hiện vì ít ai giám sát|

> **Lưu ý với defender:** Admin thường bỏ qua **egress WebSocket** vì tập trung vào DNS/HTTP → đây là điểm mù cần chú ý.

---
### 5.Case Study: Phát hiện C2 của Empire Framework

#### 5.1. Bối cảnh lab

- **Attacker VM:** `attacker01` (Kali) — chạy `powershell-empire server/client`.
    
- **Nạn nhân:** `server02` (Windows) — chạy `launcher.bat` để gọi về C2.
    
- **IDS:** `snort01` — bắt gói tin trên port 8080, phát hiện C2.
    

#### 5.2. Empire hoạt động như thế nào?

- C2 server của Empire lắng nghe trên HTTP (port 8080).
    
- Agent gọi về bằng **GET request** tới URI `/news.php`.
    
- Server trả về **HTTP 200 OK** với payload đã mã hóa (gibberish).
    

#### 5.3. Dấu vết trong pcap (qua Wireshark/tshark)

|Trường|Giá trị bất thường|Tại sao đáng ngờ?|
|---|---|---|
|**URI**|`/news.php`|URI cố định — dấu hiệu beacon|
|**User-Agent**|`Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko`|Giả IE11 trên Windows 7 — lỗi thời, không khớp môi trường thực tế|
|**Server**|`Microsoft-IIS/7.5`|Empire giả mạo IIS 7.5 để blend|
|**Payload body**|Gibberish (đã obfuscate)|Không phải HTML/JSON bình thường|

 **Red flags:**

1. User-Agent lỗi thời (IE11/Windows 7) trong enterprise hiện đại.
    
2. URI cố định, không thay đổi.
    
3. Response body là dữ liệu mã hóa, không phải nội dung web bình thường.
    
4. Kết nối **lặp đi lặp lại** (beaconing) — dù lab chỉ chạy 1 lần.

####  5.4.Luật Snort phát hiện Empire C2

```
alert tcp $HOME_NET any -> $EXTERNAL_NET $HTTP_PORTS (
    msg:"MALWARE-CNC PowerShell Empire variant outbound connection";
    flow:to_server,established;
    content:"/news.php HTTP/1.1|0D 0A|",fast_pattern,nocase;
    content:"User-Agent: Mozilla/5.0 (Windows NT 6.1|3B| WOW64|3B| Trident/7.0|3B| rv:11.0) like Gecko";
    metadata:impact_flag red;
    service:http;
    reference:url,attack.mitre.org/techniques/T1086;
    reference:url,powershellempire.com;
    classtype:trojan-activity;
    sid:38259; rev:5;
)
```


**Phân tích từng phần:**

|Thành phần|Giá trị|Ý nghĩa|
|---|---|---|
|`alert tcp`|—|Cảnh báo traffic TCP|
|`$HOME_NET any -> $EXTERNAL_NET $HTTP_PORTS`|—|Từ mạng nội bộ ra ngoài qua HTTP|
|`flow:to_server,established`|—|Chỉ packet gửi đến server, kết nối đã thiết lập|
|`content:"/news.php HTTP/1.1\|0D 0A\|"`|`fast_pattern,nocase`|Khớp URI `/news.php` + CRLF (`\r\n`)|
|`content:"User-Agent: ..."`|—|Khớp toàn bộ User-Agent giả mạo (có `3B` = `;`)|
|`sid:38259`|—|Talos rule ID (có sẵn trong ruleset)|
|`classtype:trojan-activity`|—|Phân loại: hoạt động Trojan|

**Lưu ý lab:** Vì Snort trong lab **không bật HTTP preprocessor**, rule đã bị **bỏ keyword `http_header`** so với rule gốc của Talos. Trong production, nên bật preprocessor để dùng đầy đủ.

#### 5.5.Log Snort khi phát hiện
```
03/11-10:43:03.719729 [**] [1:38259:5] "MALWARE-CNC PowerShell Empire variant outbound connection"
[**] [Classification: A Network Trojan was detected] [Priority: 1] {TCP}
172.16.51.10:59935 -> 192.168.51.50:8080
```

- **Lưu ý NAT/PAT:** Do NAT, flow bị match **2 lần**:
    
    - Lần 1: từ IP gốc của client (`172.16.51.10`).
        
    - Lần 2: từ IP đã NAT (`192.168.51.40`).
        
    - Server (C2) chỉ thấy **1 flow** duy nhất (IP ngoài).

---
### 6. Hạn chế của signature-based detection với C2

- Rule trên **chỉ phát hiện Empire mặc định** (URI + User-Agent cố định).
    
- **Attacker nâng cao sẽ tùy biến:**
    
    - Đổi URI (`/news.php` → `/api/v1/status`).
        
    - Đổi User-Agent (giả Chrome, Firefox bản mới).
        
    - Dùng HTTPS (mã hóa).
        
    - Custom obfuscation algorithm.
        
    - Dùng DGA.
        
    - Dùng WebSocket/DNS tunneling.

**Chiến lược phòng thủ nhiều lớp (Defense-in-Depth) khi gặp C2 tùy chỉnh:**

1. **Network:** Snort với rule chung cho beaconing (khoảng thời gian đều đặn, kích thước packet cố định).
    
2. **Endpoint:** Bật **PowerShell Module/Script Block/Transcription logging** (Topic 6).
    
3. **Sysmon:** Giám sát `NetworkConnect` (Event ID 3) + `ProcessCreate` (Event ID 1).
    
4. **Reverse engineer:** Lấy agent từ endpoint, phân tích obfuscation, tạo signature mới.
    
5. **Threat Intel:** Chia sẻ IOC với cộng đồng (MISP, VirusTotal).


---
### Quy trình điều tra C2 (SOC Analyst Workflow)

```
Bước 1: Phát hiện bất thường
   ├── Snort alert (signature)
   ├── SIEM rule (beaconing pattern)
   └── Báo cáo từ user / EDR

Bước 2: Xác minh & Phân tích
   ├── Kiểm tra pcap (Wireshark/tshark) — User-Agent, URI, payload
   ├── Đối chiếu IOC với threat intel
   └── Xác định process gọi ra (qua Sysmon NetworkConnect)

Bước 3: Truy vết endpoint
   ├── PowerShell log (4103, 4104, transcription)
   ├── Sysmon (ProcessCreate, FileCreate, DNS)
   └── Tìm agent/stager trên đĩa hoặc trong memory

Bước 4: Đánh giá phạm vi (Scope)
   ├── Có bao nhiêu máy beacon về cùng C2?
   ├── C2 server ở đâu? (IP/Domain)
   └── Kênh mã hóa gì?

Bước 5: Ứng phó
   ├── Block IP/Domain tại firewall/proxy
   ├── Cách ly máy nhiễm
   ├── Reverse engineer agent (nếu cần)
   └── Tạo rule mới cho lần sau
```

---

### 8. Câu hỏi tự kiểm tra (Self-Check)

1. **Pyramid of Pain:** Tại sao phát hiện ở tầng TTP lại có giá trị hơn phát hiện ở tầng Hash?
    
2. **DGA:** Một domain có entropy cao và NXDOMAIN rate cao là dấu hiệu của gì? Làm sao phát hiện?
    
3. **Empire:** Kể tên 3 dấu hiệu trong pcap cho thấy traffic là C2 của Empire.
    
4. **Snort rule:** Trong rule Empire, tại sao lại có `fast_pattern`? `flow:to_server,established` có tác dụng gì?
    
5. **Giới hạn:** Nếu attacker đổi User-Agent và URI, rule `sid:38259` có còn phát hiện được không? Làm sao để phát hiện C2 tùy biến?

---
### 9. Tóm tắt 1 dòng (TL;DR)

> **C2 detection = hiểu kiến trúc C2 + nhận diện dấu vết (IOC) + leo thang lên Pyramid of Pain + không phụ thuộc vào signature tĩnh.**


---
```
import sys

def usage():
    print("Usage: " + sys.argv[0] + " [date]")
    print("Usage: " + sys.argv[0] + " 12.02.2021")
    sys.exit()

def generate_domain(year: int, month: int, day: int) -> str:
    domain = ""
    
    # Vòng lặp tạo ra 16 ký tự (0x10 = 16)
    for i in range(0x10):
        # Các phép toán Bitwise (XOR, Shift, AND) để tạo tính ngẫu nhiên
        year = ((year ^ 8 * year) >> 11) ^ ((year & 0xFFFFFFF0) << 17)
        month = ((month ^ 4 * month) >> 25) ^ 16 * (month & 0xFFFFFFF8)
        day = ((day ^ (day << 13)) >> 19) ^ ((day & 0xFFFFFFFE) << 12)
        
        # Tính toán ký tự từ kết quả bitwise và thêm vào domain
        domain += chr(((year ^ month ^ day) % 25) + 97)

    print(domain + ".com")


if __name__ == "__main__":
    if len(sys.argv) > 1:
        date = sys.argv[1]
        y, m, d = date.split('.')
        generate_domain(int(y), int(m), int(d))
    else:
        usage()
```
### Mục đích của Script

Đây là một script Python mô phỏng cách thức hoạt động của **DGA (Domain Generation Algorithm)**.

- **Input:** Một ngày tháng cụ thể (định dạng `YYYY.MM.DD`).
    
- **Output:** Một tên miền giả ngẫu nhiên dài 16 ký tự kết thúc bằng `.com`.
    
- **Ứng dụng trong Malware:** Kẻ tấn công nhúng thuật toán này vào C2 Agent. Mỗi ngày, Agent sẽ tính toán ra một domain mới để gọi về C2 Server, giúp né tránh các biện pháp chặn IP/Domain tĩnh của Defender.

**Phân tích kỹ thuật:**

1. **Vòng lặp `for i in range(0x10)`:** Lặp 16 lần để tạo ra 16 ký tự cho domain.
    
2. **Phép toán Bitwise (`^`, `>>`, `<<`, `&`):**
    
    - Đây là các phép toán thao tác trên bit: XOR (`^`), Dịch phải (`>>`), Dịch trái (`<<`), AND (`&`).
        
    - Các phép toán này liên tục biến đổi giá trị của `year`, `month`, `day` trong mỗi vòng lặp, tạo ra hiệu ứng "hỗn loạn" (pseudo-random).
        
    - **Điểm mấu chốt:** Kết quả của mỗi vòng lặp phụ thuộc vào kết quả của vòng lặp trước đó (tính chất _feedback_).
        
3. **Tạo ký tự:**
    
    - `((year ^ month ^ day) % 25)`: Lấy kết quả XOR của 3 biến, chia lấy dư cho 25 (kết quả từ `0` đến `24`).
        
    - `+ 97`: Trong bảng mã ASCII, `97` là ký tự `'a'`. Do đó, phép cộng này biến đổi số từ `0-24` thành các chữ cái thường từ `'a'` đến `'y'`.
        
    - `chr(...)`: Chuyển đổi số nguyên thành ký tự tương ứng.