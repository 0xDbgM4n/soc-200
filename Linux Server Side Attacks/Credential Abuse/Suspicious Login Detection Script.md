**Script `ssh_suspicious_logons.py`** – Nhận 3 tham số:

1. `[AUTHENTICATION METHOD]`: `pubkey` | `password` | `all`
    
2. `[SCOPE]`: `valid` | `failed` | `all`
    
3. `[DEBUG]`: `verbose` | `off`

**4 Regex patterns cốt lõi:**
```
password_valid_login  = 'sshd\[.*\]*Accepted password'
password_failed_login = 'sshd\[.*\]*Failed password'
pubkey_valid_login    = 'sshd\[.*\]*Accepted publickey'
pubkey_failed_login   = 'sshd\[.*\]*Failed publickey'
```

**Cách sử dụng:**
```
python3 ssh_suspicious_logons.py pubkey failed off
# Output: Jul 20 04:41:0 Failed publickey offsec 192.168.51.50
```

