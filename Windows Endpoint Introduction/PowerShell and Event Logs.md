Sử dụng `Get-WinEvent` trong `PowerShell` để đọc log

Cần có quyền `Administrative privileges` để đọc `Security Logs` còn `Application`, `System` và `Setup` thì không cần

```
PS C:\Users\Admin> Get-WinEvent -ListLog Application, Security, Setup, System

LogMode   MaximumSizeInBytes RecordCount LogName
-------   ------------------ ----------- -------
Circular            20971520       16006 Application
Circular            20971520       25747 Security
Circular             1052672        2193 Setup
Circular            20971520       32040 System


PS C:\Users\Admin>
```

- `LogMode`: là chế độ ghi log, `Circular` giá trị này cho biết khi đạt tới ngưỡng tối đa của byte bản ghi thì bản ghi cũ sẽ bị xoá để ghi bản mới
- `MaximumSizeInBytes`: là số lượng byte tối đa có thể ghi vào tệp log trước khi bị xoá
- `RecordCount`: là số lượng bản ghi đơn lẻ tương ứng với từng loại log

Một số ví dụ về lọc log

```
PS C:\Users\Admin> Get-WinEvent -LogName Security | Select-Object -first 10


   ProviderName: Microsoft-Windows-Security-Auditing

TimeCreated                      Id LevelDisplayName Message
-----------                      -- ---------------- -------
10/14/2026 7:43:54 PM          5379 Information      Credential Manager credentials were read....
10/14/2026 7:43:54 PM          5379 Information      Credential Manager credentials were read....
10/14/2026 7:43:54 PM          5379 Information      Credential Manager credentials were read....
10/14/2026 7:43:54 PM          5379 Information      Credential Manager credentials were read....
10/14/2026 7:43:54 PM          5379 Information      Credential Manager credentials were read....
10/14/2026 7:43:54 PM          5379 Information      Credential Manager credentials were read....
10/14/2026 7:43:54 PM          5379 Information      Credential Manager credentials were read....
10/14/2026 7:43:54 PM          5379 Information      Credential Manager credentials were read....
10/14/2026 7:43:54 PM          5379 Information      Credential Manager credentials were read....
10/14/2026 7:43:54 PM          5379 Information      Credential Manager credentials were read....


PS C:\Users\Admin>
```


```
PS C:\Users\Admin> Get-WinEvent -LogName 'Security' | Where-Object { $_.Id -eq
>> "4624" } | Select-Object -Property TimeCreated,Message -first 10

TimeCreated           Message
-----------           -------
10/14/2026 7:43:53 PM An account was successfully logged on....
10/14/2026 7:37:30 PM An account was successfully logged on....
10/14/2026 7:34:56 PM An account was successfully logged on....
10/14/2026 7:31:45 PM An account was successfully logged on....
10/14/2026 7:31:45 PM An account was successfully logged on....
10/14/2026 7:31:45 PM An account was successfully logged on....
10/14/2026 7:31:44 PM An account was successfully logged on....
10/14/2026 7:27:43 PM An account was successfully logged on....
10/14/2026 7:24:28 PM An account was successfully logged on....
10/14/2026 7:24:28 PM An account was successfully logged on....


PS C:\Users\Admin>
```

