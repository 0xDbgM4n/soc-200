- Sử dụng `Get-WinEvent` để kiểm tra tất cả Sysmon event
- Chỉ định tham số `-LogName` với tên nhà cung cấp (provider) "`Microsoft-WindowsSysmon/Operational`"

![x](./Pasted%20image%2020261014215819.png)

Vì chúng ta sử dụng câu lệnh `Get-WinEvent` khá nhiều lên hãy đơn giản hoá nó bằng cách bọc nó trong một `PowerShell Module` hãy mở text editor nên và lưu lại nội dung như bên dưới vào file `C:\Sysmon\Get-Sysmon.psm1`

```
function Get-SysmonEvent { Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" }
```

Việc lưu ở dạng `psm1` thay vì `ps1` có hai lợi ích:
- Nhận diện đây là một module cần `import` chứ không phải file độc lập
- Có thể thêm vào useful module trong powershell profile để có sẵn khi mở dấu nhắc lệnh lên

Sau khi lưu xong sử dụng `Import-Module` để thêm vào phiên làm việc hiện tại, để xác nhận dùng `Get-Module`

![x](./Pasted%20image%2020261014220605.png)

![x](./Pasted%20image%2020261014220716.png)

Nâng cấp file `psm1`

![x](./Pasted%20image%2020261014221524.png)

Sử dụng `$null` để giữ chỗ cho `ID` vì hiện tại chưa cần `ID`
![x](./Pasted%20image%2020261014221610.png)

