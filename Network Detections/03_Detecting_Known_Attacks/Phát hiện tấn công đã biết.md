**Nội dung:**

- Phân tích lỗ hổng Zerologon (CVE-2020-1472): Lỗi trong giao thức Netlogon.
    
- Phân tích luật Snort mẫu:
    
    - `dce_iface` và `dce_opnum` (lọc theo giao diện RPC và mã lệnh).
        
    - `content` để tìm chuỗi byte cụ thể (opnum 4).
        
    - `byte_test` để kiểm tra các byte lặp lại.
        
- **Thực hành:** Sử dụng `tshark` để bắt gói tin và phân tích offset byte, sau đó đối chiếu với luật Snort

---
# Rule Snort ZeroLogon (CVE-2020-1472)

```
alert tcp any any -> $HOME_NET any (
    msg:"OS-WINDOWS Microsoft Windows Netlogon crafted NetrServerReqChallenge elevation of privilege attempt";
    flow:to_server,established;
    dce_iface:12345678-1234-abcd-ef00-01234567cffb;
    dce_opnum:4;
    content:"|04 00|"; depth:2; offset:22; fast_pattern;
    content:"|00 00 00|"; distance:0;
    isdataat:7,relative;
    isdataat:!8,relative;
    byte_extract:1,0,first_cc_byte,relative;
    byte_test:1,=,first_cc_byte,0,relative;
    byte_test:1,=,first_cc_byte,1,relative;
    byte_test:1,=,first_cc_byte,2,relative;
    byte_test:1,=,first_cc_byte,3,relative;
    detection_filter:track by_src, count 10, seconds 10;
    metadata:policy balanced-ips drop, policy max-detect-ips drop, policy security-ips drop;
    service:dcerpc;
    reference:cve,2020-1472;
    reference:url,portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2020-1472;
    classtype:attempted-admin;
    sid:55703;
    rev:4;
)
```

### Header — `alert tcp any any -> $HOME_NET any`
| Thành phần      | Ý nghĩa                                                                           |
| --------------- | --------------------------------------------------------------------------------- |
| `alert`         | Sinh cảnh báo (không drop ở tầng rule; việc drop do policy quyết định)            |
| `tcp`           | Giao thức TCP — Netlogon/DCE-RPC chạy trên TCP (thường cổng 445 hoặc dynamic RPC) |
| `any any`       | Source bất kỳ (kẻ tấn công có thể từ ngoài hoặc trong)                            |
| `->`            | Hướng một chiều client → server                                                   |
| `$HOME_NET any` | Đích là mạng nội bộ, mọi cổng (RPC endpoint dynamic)                              |
**Vì sao không giới hạn cổng đích?** DCE/RPC endpoint mapper (135) cấp cổng động cho Netlogon; giới hạn cứng sẽ bỏ sót traffic thật.

### `flow:to_server,established`

- `to_server`: chỉ gói do client gửi tới server (request).
    
- `established`: phiên TCP đã bắt tay xong (loại SYN scan, handshake rác).
    

→ Đảm bảo rule chỉ xét **request NetrServerReqChallenge thực sự trong phiên hợp lệ**.

### `dce_iface` + `dce_opnum` — định danh RPC call

```
dce_iface:12345678-1234-abcd-ef00-01234567cffb;
dce_opnum:4;
```

| Trường      | Giá trị                                | Ý nghĩa                               |
| ----------- | -------------------------------------- | ------------------------------------- |
| `dce_iface` | `12345678-1234-abcd-ef00-01234567cffb` | UUID interface **Netlogon (MS-NRPC)** |
| `dce_opnum` | `4`                                    | Opnum 4 = **NetrServerReqChallenge**  |
**Tại sao đây là điểm mấu chốt?** ZeroLogon khai thác chính hàm `NetrServerReqChallenge` — kẻ tấn công gửi challenge toàn 0 thay vì giá trị ngẫu nhiên. Việc khớp đúng interface + opnum loại bỏ 99% traffic RPC không liên quan.

> Cơ chế: Snort dùng preprocessor `dcerpc` để parse header DCE/RPC (PDU type, call id, interface UUID, opnum) trước khi so khớp stub data. Đây là lý do các `content`/`offset` phía sau tính trên **stub data** chứ không phải toàn bộ packet.

### Phát hiện mẫu byte đặc trưng

```
content:"|04 00|"; depth:2; offset:22; fast_pattern;
content:"|00 00 00|"; distance:0;
```

**Phân tích offset 22:**

Stub của `NetrServerReqChallenge` có cấu trúc xấp xỉ:

```
[ PrimaryName (UNICODE_STRING) ]  ~ 12-16 byte
[ ComputerName (UNICODE_STRING) ] ~ biến đổi
[ ClientChallenge (8 byte) ]       ← tại đây
```

- `offset:22; depth:2` → tìm `04 00` trong 2 byte tại vị trí 22 của stub.
    
- `04 00` là **giá trị length/flag** đặc trưng trong cấu trúc challenge bị craft (độ dài chuỗi Unicode = 4).
    
- `content:"|00 00 00|"; distance:0` → ngay sau `04 00` phải là 3 byte 0 liên tiếp.
    

→ Kết hợp lại thành mẫu 5 byte `04 00 00 00 00` — **signature của challenge bị zero hoá**.

`fast_pattern`: đánh dấu content này làm mẫu ưu tiên cho bộ máy Aho-Corasick, tăng tốc so khớp.

### Kiểm soát độ dài stub còn lại

```
isdataat:7,relative;
isdataat:!8,relative;
```

|Điều kiện|Nghĩa|
|---|---|
|`isdataat:7,relative`|Còn **ít nhất 7 byte** tính từ vị trí hiện tại|
|`isdataat:!8,relative`|Còn **ít hơn 8 byte**|
→ Kết hợp: **chính xác 7 byte** còn lại sau mẫu `00 00 00`.

**Vì sao 7 byte?** Credential ZeroLogon gồm 8 byte (challenge) — sau khi trừ 1 byte đã bị "ăn" bởi bước so khớp trước, còn 7 byte cuối. Con số này khớp với cấu trúc exploit đã biết, giúp phân biệt với challenge hợp lệ (thường dài hơn hoặc khác mẫu).

### Kiểm tra byte lặp — "zeroed credential"

```
byte_extract:1,0,first_cc_byte,relative;
byte_test:1,=,first_cc_byte,0,relative;
byte_test:1,=,first_cc_byte,1,relative;
byte_test:1,=,first_cc_byte,2,relative;
byte_test:1,=,first_cc_byte,3,relative;
```

**Luồng thực thi:**

1. `byte_extract:1,0,first_cc_byte,relative`
    
    - Lấy **1 byte** tại vị trí 0 (tương đối so với con trỏ hiện tại).
        
    - Lưu vào biến `first_cc_byte`.
        
2. Bốn lệnh `byte_test:1,=,first_cc_byte,N,relative` (N = 0,1,2,3)
    
    - Kiểm tra byte tại vị trí N **bằng** giá trị `first_cc_byte`.
        

→ Nếu **cả 4 byte đầu tiên giống hệt nhau**, rule coi đây là credential bị zero hoá (toàn `0x00`) — dấu hiệu kinh điển của ZeroLogon.

**Tại sao không dùng `content:"|00 00 00 00|"` trực tiếp?** Vì kẻ tấn công có thể dùng giá trị lặp khác 0 (ví dụ `0xAA`), và cách kiểm tra "4 byte giống nhau" tổng quát hơn — bắt được biến thể exploit tinh chỉnh.

### `detection_filter` — chống false positive

```
detection_filter:track by_src, count 10, seconds 10;
```

- Theo dõi **theo source IP** (`by_src`).
    
- Chỉ alert khi cùng IP đạt **≥ 10 lần khớp** trong **10 giây**.
    

**Vì sao cần?** Netlogon hợp lệ có thể gửi `NetrServerReqChallenge` với challenge ngẫu nhiên — đôi khi tình cờ trùng mẫu byte. Việc yêu cầu lặp nhanh 10 lần/10s lọc bỏ nhiễu ngẫu nhiên, vì exploit ZeroLogon thường brute-force lặp lại nhiều lần.

### Metadata & policy

```
metadata:policy balanced-ips drop, policy max-detect-ips drop, policy security-ips drop;
```

- Rule được **drop** (không chỉ alert) trong 3 policy: `balanced-ips`, `max-detect-ips`, `security-ips`.
    
- Đây là mức xử lý nghiêm ngặt — phù hợp lỗ hổng CVSS 10.0.

### `service:dcerpc` + references + classtype

|Trường|Ý nghĩa|
|---|---|
|`service:dcerpc`|Gắn rule với service DCE/RPC, giúp engine tối ưu & phân loại|
|`reference:cve,2020-1472`|Liên kết CVE ZeroLogon|
|`reference:url,...`|Link advisory MSRC|
|`classtype:attempted-admin`|Phân loại: nỗ lực chiếm quyền admin|
|`sid:55703; rev:4`|Định danh rule & phiên bản (rev 4 = đã qua 4 lần chỉnh sửa)|

## Sơ đồ logic phát hiện

```
[Packet TCP to_server, established]
            │
            ▼
[Parse DCE/RPC] ──► interface = Netlogon? ──► Không ──► Bỏ qua
            │              │ Có
            │              ▼
            │         opnum = 4? ──► Không ──► Bỏ qua
            │              │ Có
            ▼              ▼
[Stub offset 22 có "04 00"?] ──► Không ──► Bỏ qua
            │ Có
            ▼
[Sau đó có "00 00 00"?] ──► Không ──► Bỏ qua
            │ Có
            ▼
[Còn đúng 7 byte?] ──► Không ──► Bỏ qua
            │ Có
            ▼
[4 byte đầu giống nhau?] ──► Không ──► Bỏ qua
            │ Có
            ▼
[≥10 lần/10s từ cùng src?] ──► Không ──► Không alert (chỉ đếm)
            │ Có
            ▼
        🚨 ALERT (và DROP theo policy)
```

## Đánh giá chất lượng rule

**Điểm mạnh:**

- Kết hợp **3 lớp kiểm tra độc lập**: định danh RPC (interface+opnum) → mẫu byte cấu trúc → kiểm tra ngữ nghĩa (byte lặp).
    
- `isdataat` khống chế độ dài chính xác → giảm match nhầm.
    
- `detection_filter` chống brute-force nhiễu.
    
- `fast_pattern` tối ưu hiệu năng.
    
- Áp dụng `drop` ở policy nghiêm ngặt.
    

**Hạn chế cần lưu ý:**

- Chỉ phát hiện **attempt**, không phân biệt thành công/thất bại.
    
- Phụ thuộc preprocessor `dcerpc` — nếu traffic mã hoá (RPC over TLS) sẽ không parse được.
    
- `offset:22` cứng — nếu Microsoft thay đổi cấu trúc stub trong bản vá mới, cần cập nhật.
    
- Cần tune `detection_filter` nếu môi trường có nhiều Netlogon hợp lệ tần suất cao.
    

**Khuyến nghị vận hành:**

1. Vá KB4571756 (Server) / KB4577015 — bản vá ZeroLogon.
    
2. Bật **Netlogon enforcement mode** sau khi rà soát.
    
3. Giám sát Event ID **5827/5828/5829** trên DC.
    
4. Kết hợp rule này với rule phát hiện **NetrServerAuthenticate3** (opnum 26) để bắt trọn chuỗi exploit.

## Bản rút gọn (nếu cần tối ưu hiệu năng)

Nếu môi trường ít nhiễu và ưu tiên tốc độ, có thể lược bớt phần kiểm tra byte lặp (giữ nguyên logic cốt lõi):

```
alert tcp any any -> $HOME_NET any (
    msg:"OS-WINDOWS Netlogon ZeroLogon NetrServerReqChallenge EoP attempt";
    flow:to_server,established;
    dce_iface:12345678-1234-abcd-ef00-01234567cffb;
    dce_opnum:4;
    content:"|04 00 00 00 00|"; offset:22; depth:5; fast_pattern;
    isdataat:7,relative;
    isdataat:!8,relative;
    detection_filter:track by_src, count 10, seconds 10;
    metadata:policy security-ips drop;
    service:dcerpc;
    reference:cve,2020-1472;
    classtype:attempted-admin;
    sid:5570301;
    rev:1;
)
```

> Bản này gộp 2 content thành 1 mẫu 5 byte, bỏ 4 lệnh `byte_test` — nhanh hơn nhưng **kém tổng quát** hơn (không bắt được biến thể credential lặp giá trị khác 0). Khuyến nghị dùng bản đầy đủ ở mục 1 cho production.

