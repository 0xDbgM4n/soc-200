**Attack với Hydra – option `-u` (loop users trước passwords):**
```
hydra -L users.txt -P ./dict_bf_success.txt 192.168.51.12 -t 1 ssh -u
```

**Đặc điểm log:**

- Nhiều `Failed password` (15+ failures) cho NHIỀU users khác nhau
    
- Sau đó là nhiều `Accepted password` **liên tiếp cùng thời điểm**
    
- Source IP GIỐNG NHAU cho tất cả

**Ví dụ log điển hình:**
```
Jul 27 04:41:01 Failed password bob   192.168.51.50
Jul 27 04:41:04 Failed password alice 192.168.51.50
Jul 27 04:41:07 Failed password wendy 192.168.51.50
...
Jul 27 04:41:44 Accepted password bob   192.168.51.50
Jul 27 04:41:44 Accepted password alice 192.168.51.50
Jul 27 04:41:44 Accepted password wendy 192.168.51.50
```

**Phát hiện:** 15 failures rồi 3 successes cùng 1 phút, cùng IP = **Password Spraying**.