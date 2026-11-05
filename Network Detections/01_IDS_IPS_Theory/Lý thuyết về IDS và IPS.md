
|             IDS             |                   IPS                   |
| :-------------------------: | :-------------------------------------: |
| Hệ thống phát hiện xâm nhập | Hệ thống phát hiện và **chặn** xâm nhập |
Cách triển khai IDS: gồm chủ động (inline) và thụ động (passive)
- `inline`:  
	+) vị trí: đặt nối tiếp trong ngăn xếp mạng
	+) có thể cung cấp dữ liệu cho system phía trước (upstream)

- `passive`:
	+) vị trí: đặt ở vị trí quan trọng mà ko làm gián đoạn lưu lượng mạng ( ko đặt inline )
	+) chi phí cao nên dành cho yêu cầu đặc thù

Khi có mối đe dọa `IDS` thông qua `next-generation firewall` để block `IP` tính năng chặn traffic này đã biến `IDS` thành `IPS` giúp chủ động ngăn chặn mối đe dọa

## Ưu/nhược điểm của Signature-based vs. Anomaly-based detection.

#### Cách thức hoạt động của IDS dựa trên chữ ký

![x](./Pasted%20image%2020261020145234.png)

- **1. Lưu trữ chữ ký tấn công**  
    Các chữ ký tấn công, là những mẫu duy nhất liên quan đến hoạt động độc hại, được lưu trữ trong cơ sở dữ liệu.
- **2. Giám sát lưu lượng mạng**  
    Hệ thống phát hiện xâm nhập (IDS) liên tục kiểm tra lưu lượng mạng đến.
- **3. Đối chiếu với các chữ ký đã biết**  
    IDS so sánh hoạt động mạng với cuộc tấn công được lưu trữ [chữ ký](https://fidelissecurity.com/glossary/signatures/).
- **4. Kích hoạt cảnh báo**  
    Hệ thống phát hiện dựa trên chữ ký sẽ kích hoạt cảnh báo khi tìm thấy sự trùng khớp giữa lưu lượng mạng và một chữ ký đã biết, thông báo cho quản trị viên về các mối đe dọa tiềm tàng.
- **5. Áp dụng các quy tắc đã được xác định trước**  
    Các công cụ bảo mật áp dụng các quy tắc cụ thể quy định cách phản hồi khi phát hiện chữ ký trùng khớp.
- **6. Cập nhật cơ sở dữ liệu chữ ký**  
    Việc cập nhật thường xuyên đảm bảo tính mới [các vectơ tấn công](https://fidelissecurity.com/cybersecurity-101/learn/what-is-an-attack-vector/) Hiệu quả của hệ thống được bổ sung.


| Ưu điểm                                                                                                                                                                                                                                                                                                                                                  | Nhược điểm                                                                                                                                                                                                                                                      |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Phát hiện dựa trên chữ ký rất hiệu quả trong việc nhanh chóng xác định và ngăn chặn các mối đe dọa đã biết thông qua cơ sở dữ liệu chữ ký được xác định trước. Phương pháp này thường cho thời gian phản hồi nhanh hơn đối với các mối đe dọa đã biết so với các phương pháp phát hiện dựa trên chữ ký, vốn có thể cần nhiều thời gian hơn để phân tích. | Một nhược điểm lớn của IDS dựa trên chữ ký là tính dễ bị tấn công bởi các kỹ thuật né tránh, có khả năng dẫn đến các cuộc xâm nhập không bị phát hiện. Họ cũng cần cập nhật thường xuyên cơ sở dữ liệu chữ ký để duy trì hiệu quả trước các mối đe dọa mới nổi. |

**Mặc dù phát hiện dựa trên chữ ký cung cấp khả năng bảo vệ gần như theo thời gian thực, nhưng nó có thể gặp khó khăn trong việc chống lại các mối đe dọa mới nổi thiếu chữ ký hiện có. Điều này nhấn mạnh sự cần thiết phải cập nhật liên tục và giám sát chặt chẽ để duy trì hiệu quả của hệ thống.**

#### Cơ chế hoạt động của IDS dựa trên bất thường

![x](./Pasted%20image%2020261020145829.png)

##### 1. Xác định các mốc cơ sở

- Thiết lập các điểm tham chiếu cho hoạt động bình thường của hệ thống bằng cách sử dụng dữ liệu lịch sử.
- Sử dụng các chỉ số thống kê như trung bình và độ lệch chuẩn để xác định tính chuẩn tắc.

##### 2. Học hỏi từ dữ liệu

- Các mô hình học máy phân tích dữ liệu lịch sử để thiết lập và tinh chỉnh các đường cơ sở.
- Các thuật toán phân cụm (ví dụ: K-means) nhóm các mẫu chuẩn và phát hiện các giá trị ngoại lệ.

##### 3. Phát hiện các bất thường

- Dữ liệu đầu vào được so sánh với dữ liệu cơ sở đã được thiết lập.
- Những sai lệch so với quy luật bình thường được đánh dấu là tiềm ẩn [dị thường](https://fidelissecurity.com/glossary/anomaly/).

##### 4. Phân loại mức độ nghiêm trọng

- Mức độ sai lệch quyết định mức độ nghiêm trọng của các cảnh báo bất thường.
- Độ lệch càng lớn thì mức độ rủi ro càng cao.

##### 5. Cập nhật các mốc cơ sở

- Thường xuyên cập nhật dữ liệu cơ sở với dữ liệu mới để thích ứng với các xu hướng đang thay đổi.
- Cập nhật liên tục đảm bảo [phát hiện bất thường](https://fidelissecurity.com/cybersecurity-101/learn/anomaly-detection/) vẫn hiệu quả.


| Ưu điểm                                                                                                                                                          | Nhược điểm                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Khả năng phát hiện các cuộc tấn công không xác định hoặc tấn công zero-day, Bảo mật chủ động và thích ứng hơn, Tăng cường khả năng nhận biết các mô hình hành vi | Việc phát hiện dựa trên bất thường tiêu tốn nhiều tài nguyên hơn do cần phải giám sát và phân tích liên tục các hành vi cơ bản. Một thách thức quan trọng với IDS dựa trên sự bất thường là số lượng kết quả dương tính giả cao hơn, điều này có thể tạo ra kết quả dương tính giả bằng cách có khả năng xác định sai hoạt động hợp pháp là đáng ngờ.<br><br>Các hệ thống dựa trên bất thường sẽ mất hiệu quả nếu các đường cơ sở hành vi bình thường không được thiết lập chính xác, làm suy yếu khả năng phát hiện bất thường thực sự. Việc học hỏi liên tục từ hoạt động mạng là rất quan trọng đối với việc phát hiện các bất thường nhằm giảm thiểu kết quả dương tính giả và cải thiện độ chính xác theo thời gian. |
https://fidelissecurity.com/cybersecurity-101/learn/signature-based-vs-anomaly-based-ids/
