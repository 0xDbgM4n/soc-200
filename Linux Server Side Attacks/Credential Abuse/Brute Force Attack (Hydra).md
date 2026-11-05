**Attack traditional brute force:**
```
hydra -l alice -P ./dict_bf.txt 192.168.51.12 -t 1 ssh
```

**Log footprint quan trọng:**
```
Received disconnect from 192.168.51.50 port ... [preauth]   # Hydra probing
pam_unix(sshd:auth): authentication failure; user=alice     # Generic failure
Failed password for alice from 192.168.51.50 port ... ssh2   # Specific failure
message repeated 5 times: [ Failed password ... ]           # Aggregation
error: maximum authentication attempts exceeded              # MaxAuthTries hit
Disconnecting authenticating user alice: Too many auth failures [preauth]
```

**Điểm quan trọng:** Default `MaxAuthTries = 6`. Nếu attacker chỉ thử **< 6 password** → không trigger "Too many authentication failures" → khó bị phát hiện.