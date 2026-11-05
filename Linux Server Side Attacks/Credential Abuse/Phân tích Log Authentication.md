**Vị trí log:**

|OS|File Log|
|---|---|
|**Ubuntu/Debian**|`/var/log/auth.log`|
|**CentOS/RHEL**|`/var/log/secure`|

**Case 1: Successful Password Login**
```
Jul 15 04:31:05 linux01 sshd[2253]: Accepted password for offsec from 192.168.51.50 port 55398 ssh2
Jul 15 04:31:05 linux01 sshd[2253]: pam_unix(sshd:session): session opened for user offsec by (uid=0)
```

**Case 2: Failed Password Login**
```
Jul 15 04:45:37 linux01 sshd[2383]: pam_unix(sshd:auth): authentication failure; ... user=offsec
Jul 15 04:45:39 linux01 sshd[2383]: Failed password for offsec from 192.168.51.50 port 55400 ssh2
```

**Case 3: Successful Public-key Login**
```
Jul 15 05:48:53 linux01 sshd[1730]: Accepted publickey for offsec from 192.168.51.50 port 55420 ssh2: ED25519 SHA256:...
```

**Case 4: Failed Public-key Login** (cần tăng LogLevel lên DEBUG1)
```
Jul 15 07:24:05 linux01 sshd[1879]: Failed publickey for offsec from 192.168.51.50 port 55448 ssh2: ED25519 SHA256:...
```

**Lưu ý:** Khi `LogLevel INFO` (mặc định), failed pubkey chỉ log là:
```
Connection closed by authenticating user offsec ... [preauth]
```
→ Không nói rõ lý do. Cần nâng lên `LogLevel DEBUG1` trong `/etc/ssh/sshd_config` để thấy chi tiết.