**SSH Protocol:**

- Client-server protocol, mặc định port **22**
    
- 2 cơ chế auth chính:
    
    - **Password Authentication** (mặc định enabled)
        
    - **Public-key Authentication**
        
- PAM (Pluggable Authentication Modules) kiểm tra password với `/etc/shadow`

**Cấu hình kiểm tra:**
```
sudo cat /etc/ssh/sshd_config | grep -i "PasswordAuth\|PubkeyAuth"
```

**Public-key Auth flow:**

1. Client gửi username + public key association
    
2. Server gửi challenge message (random)
    
3. Client encrypt challenge bằng **private key** → gửi lại
    
4. Server decrypt bằng **public key** → xác thực thành công

**Tạo và deploy key:**
```
ssh-keygen -t Ed25519              # Tạo key pair
ssh-copy-id offsec@192.168.51.12   # Copy public key lên server
ssh offsec@192.168.51.12 -v        # Test với verbose
```

