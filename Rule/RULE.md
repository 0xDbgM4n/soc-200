---

---
Trong wazuh thì có xây dựng sẵn kha khá rule mặc định nhưng nó cũng cho người dùng có thể cấu hình thêm các rule khác. Dưới đây là ví dụ về 1 rule được cấu hình  sẵn trong wazuh

![x](./Pasted%20image%2020261011155251.png)

Hãy phân tích rule này:
- `rule id`: Đây là mã định danh duy nhất cho quy tắc trong wazuh
- `level` : Mức độ phân loại tuy tắc nằm trong khoảng từ 0 đến 15, theo trang quy tắc https://documentation.wazuh.com/current/user-manual/ruleset/rules/rules-classification.html. mỗi con số hiển thị là 1 mức độ nguy hiểm khác nhau
- `if_sid`: Con số này là 1 ID của quy tắc khác (quy tắc cha), nếu "if" được thoả mãn thì sẽ bật quy tắc hiện tại
- `field name`: Tham số này chỉ định tên của trường được trích xuất từ ​​bộ giải mã. Giá trị được so khớp bằng biểu thức chính quy. Trong trường hợp này, chúng ta đang tìm kiếm tên trường `win.system.providerName` với giá trị là `PowerShell`
- `group`: Tệp này được sử dụng để sắp xếp các quy tắc Wazuh. Nó chứa danh sách các danh mục mà các quy tắc thuộc về. Chúng tôi đã sắp xếp quy tắc của mình trong nhóm `windows_powershell`.

Có rất nhiều kiểu quy tắc khác nhau, chúng ta có thể kiểm tra thêm tại https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xmlsyntax/rules.html)

