┌─────────────────────────────────────────────────────────────┐
│  SOCIAL ENGINEERING & SPEARPHISHING                                          │
│  MITRE: Initial Access > Phishing > Spearphishing Attachment            │
└─────────────────────────────────────────────────────────────┘

📌 MÔ TẢ:
   - Tấn công vào yếu tố con người
   - Giả mạo email từ người/ tổ chức uy tín
   - Đính kèm file độc hại hoặc link
   - Kỹ thuật: Spearphishing Attachment, Spearphishing Link, Spearphishing via Service

📌 3 HÌNH THỨC CHÍNH:
   1. **User Solicitation**: Lừa user cung cấp thông tin qua reply email
   2. **Malicious Attachment**: Đính kèm file độc hại (Word, Excel, PDF)
   3. **Malicious Link**: Link đến trang web giả mạo hoặc tải malware

📌 DẤU HIỆU:
   - Email từ domain giả mạo (typosquatting)
   - Subject gây tò mò, khẩn cấp
   - Attachment: .doc, .docm, .xlsm, .pdf, .zip, .hta
   - Link rút gọn (bit.ly, t.co)
   - Yêu cầu enable macro, enable content

📋 LOG ARTIFACT:
   - Email gateway log (nếu có)
   - Office Event Log:
     • Event 300: Document opened
     • Event 301: Document closed
   - Sysmon Event 1: WINWORD.EXE khởi động
   - Sysmon Event 11: File attachment được ghi vào Temp

🔍 CÁCH PHÁT HIỆN:
   - Monitor file attachment trong Temp: 
     C:\Users\<user>\AppData\Local\Microsoft\Windows\INetCache\Content.Outlook\
   - Monitor WINWORD.EXE tạo process con bất thường
   - Monitor macro execution

⚠️ FALSE POSITIVE:
   - Email hợp lệ có attachment
   - User mở file Word bình thường

🛡️ MITIGATION:
   - Email filtering (ATP, Proofpoint)
   - User awareness training
   - Disable macro by default
   - Mark of the Web