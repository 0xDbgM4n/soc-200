┌─────────────────────────────────────────────────────────────┐
│  BRUTE FORCE LOGINS                                                                         │
│  MITRE: Credential Access > Brute Force                                              │
└─────────────────────────────────────────────────────────────┘

📌 ĐẶC ĐIỂM:
   - Nhiều login failure liên tiếp
   - Cùng username, nhiều password
   - Hoặc nhiều username, cùng password (password spray)
   - Thời gian giữa các lần thử rất ngắn

🔧 CÔNG CỤ:
   - Hydra (RDP, SMB)
   - CrackMapExec
   - SprayPasswords.ps1

📋 LOG ARTIFACT:


| Event ID | Ý nghĩa                  | Field quan trọng                             |
| -------- | ------------------------ | -------------------------------------------- |
| 4625     | Login failure            | TargetUserName, IpAddress, Status, SubStatus |
| 4740     | Account lockout          | TargetUserName, CallerWorkstation            |
| 4771     | Kerberos pre-auth failed | ...                                          |
| 4776     | NTLM auth                | ...                                          |

   Status codes quan trọng:
   - 0xC000006D = Bad username or password
   - 0xC000006A = Bad password
   - 0xC0000064 = Username does not exist
   - 0xC000006F = Logon outside authorized hours
   - 0xC0000070 = Logon from unauthorized workstation
   - 0xC000015B = Logon type not granted

🔍 CÁCH PHÁT HIỆN:
   - Threshold rule: > 10 event 4625 trong 5 phút từ cùng IP
   - Group by source.ip
   - KQL: `event.code: 4625 | stats count by source.ip`

⚠️ FALSE POSITIVE:
   - User quên password, thử nhiều lần
   - Service account misconfigured
   - Ứng dụng retry

🛡️ MITIGATION:
   - Account lockout policy
   - Password phức tạp
   - MFA
   - Fail2ban / RDP Guard