| Mục đích                       | Lệnh / Công cụ                                                                                 |
| ------------------------------ | ---------------------------------------------------------------------------------------------- |
| Xem log SSH (Ubuntu)           | `sudo tail -f /var/log/auth.log \| grep sshd`                                                  |
| Xem log SSH (CentOS)           | `sudo tail -f /var/log/secure \| grep sshd`                                                    |
| Tìm các IP bị fail nhiều nhất  | `grep "Failed password" /var/log/auth.log \| awk '{print $11}' \| sort \| uniq -c \| sort -rn` |
| Tìm các user bị brute force    | `grep "Failed password" /var/log/auth.log \| awk '{print $9}' \| sort \| uniq -c \| sort -rn`  |
| Xem log web Apache (lỗi 404)   | `cat /var/log/apache2/access.log \| grep " 404 "`                                              |
| Trích xuất URL path từ web log | `cat /var/log/apache2/access.log \| cut -d " " -f 7`                                           |
| Xem log bằng systemd           | `journalctl -u sshd --since "1 hour ago"`                                                      |
| Parse log tự động              | Python script kết hợp `re.finditer()`                                                          |
| Chạy script trên nhiều server  | Ansible Playbook (YAML)                                                                        |
