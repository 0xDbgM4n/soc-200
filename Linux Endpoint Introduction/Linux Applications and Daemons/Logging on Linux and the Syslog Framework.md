Đây là phần **quan trọng nhất** của topic này. Log không tự nhiên sinh ra, nó đi qua một luồng xử lý rất cụ thể.

#### a) Các thành phần của Syslog

1. **Syslog Daemon**: Nhận log từ ứng dụng (qua socket `/dev/log` hoặc `systemd-journald`), sau đó ghi ra file hoặc gửi qua mạng.
    
2. **Syslog Message Format**: Định dạng chuẩn để hiểu log (RFC3164 cũ, RFC5424 mới).
    
3. **Syslog Transport Protocol**: Giao thức truyền log qua mạng (UDP/TCP port 514, hoặc TCP/HTTPS 6514).
    

#### b) Luồng đi của Log (Log Flow) - Cực kỳ quan trọng

Các ứng dụng gửi log đến 2 nơi: `journald` (nhị phân) và `rsyslog` (text file).

1. **Sinh log**: Ứng dụng (vd: sshd) ghi log ra `/dev/log`.
    
2. **journald nhận**: `systemd-journald` thu thập log, lưu dưới dạng **binary** (không đọc bằng `cat` được, phải dùng `journalctl`).
    
3. **rsyslog nhận**: `rsyslogd` đọc log (thông qua module `imjournal` hoặc `imuxsock`), chuyển thành text và ghi vào các file tại `/var/log/`.
    

#### c) Định dạng chuẩn Syslog (RFC3164)

Một dòng log cơ bản có dạng:  
`<Priority>Timestamp Hostname App_Name[PID]: Message`  
_Ví dụ:_  
`<86>Jun 28 11:24:46 linux02 sshd[156159]: Failed password for offsec from 192.168.51.50 port 54209 ssh2`

- **Priority (PRI)**: Tính bằng công thức `PRI = (Facility * 8) + Severity`.
    
    - **Facility** (Nguồn sinh log): 0=kern, 1=user, 4=auth, 10=authpriv...
        
    - **Severity** (Mức độ): 0=Emergency, 1=Alert, 3=Error, 4=Warning, 6=Info, 7=Debug.
        
    - _Ví dụ trên: `<86>` = (10 * 8) + 6 = 86 (authpriv, Info)._
        
- **Timestamp**: `Jun 28 11:24:46`
    
- **Hostname**: `linux02`
    
- **App_Name[PID]**: `sshd[156159]`
    
- **Message**: `Failed password...`


**Facility**

| Facility code | Keyword         | Description                              |
| ------------- | --------------- | ---------------------------------------- |
| 0             | kern            | Kernel messages                          |
| 1             | user            | User-level messages                      |
| 2             | mail            | Mail system                              |
| 3             | daemon          | System daemons                           |
| 4             | auth            | Security/authentication messages         |
| 5             | syslog          | Messages generated internally by syslogd |
| 6             | lpr             | Line printer subsystem                   |
| 7             | news            | Network news subsystem                   |
| 8             | uucp            | UUCP subsystem                           |
| 9             | cron            | Cron subsystem                           |
| 10            | authpriv        | Security/authentication messages         |
| 11            | ftp             | FTP daemon                               |
| 12            | ntp             | NTP subsystem                            |
| 13            | security        | Log audit                                |
| 14            | console         | Log alert                                |
| 15            | solaris-cron    | Scheduling daemon                        |
| 16-23         | local0 – local7 | Locally used facilities                  |

![[Pasted image 20261018212227.png]]
#### d) Bảng vị trí File Log trên Linux (Phân biệt CentOS/RHEL vs Ubuntu/Debian)

Đây là bảng "bỏ túi" bắt buộc phải nhớ.

|Mục đích|Nguồn sinh|CentOS / RHEL|Ubuntu / Debian|
|---|---|---|---|
|**Authentication**|sshd, sudo, su|`/var/log/secure`|`/var/log/auth.log`|
|**Web Server (Apache)**|apache/httpd|`/var/log/httpd/access_log`|`/var/log/apache2/access.log`|
|**System Logs**|kernel, systemd|`/var/log/messages`|`/var/log/syslog`|
|**Package Mgmt**|yum/dpkg|`/var/log/yum.log`|`/var/log/dpkg.log`|
|**Audit Logs**|auditd|`/var/log/audit/audit.log`|`/var/log/audit/audit.log`|

#### e) Lệnh thao tác với Journal và Rsyslog

- **Xem log bằng journalctl**:
    
    - `journalctl -u sshd.service` (Xem log của riêng sshd)
        
    - `journalctl --since "1 hour ago"` (Xem log 1 giờ qua)
        
    - `journalctl -f` (Theo dõi log real-time - giống `tail -f`)
        
- **Đọc file log text**: `tail -f /var/log/secure` hoặc `grep "Failed password" /var/log/auth.log`.