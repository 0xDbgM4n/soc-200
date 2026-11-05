
📌 ĐẶC ĐIỂM:
   - Login ngoài giờ làm việc
   - Login từ IP lạ
   - Login Type bất thường (10 = RDP)
   - Nhiều login thất bại liên tiếp

📋 LOG ARTIFACT:

| Event ID | Ý nghĩa               | Field quan trọng                     |
| -------- | --------------------- | ------------------------------------ |
| 4624     | Login success         | LogonType, IpAddress, TargetUserName |
| 4625     | Login failure         | Status, SubStatus, LogonType         |
| 4634     | Logoff                | LogonId                              |
| 4647     | User initiated logoff | LogonId                              |

   LogonType quan trọng:
   - 2 = Interactive (local)
   - 3 = Network (share, SMB)
   - 10 = RemoteInteractive (RDP)

🔍 CÁCH PHÁT HIỆN:
   - Query: Event 4624 với LogonType=10 ngoài giờ
   - Query: Event 4625 với nhiều failure từ cùng IP

⚠️ FALSE POSITIVE:
   - Admin login ngoài giờ (bảo trì)
   - Service account login tự động

🛡️ MITIGATION:
   - MFA
   - Giới hạn giờ login
   - GeoIP blocking