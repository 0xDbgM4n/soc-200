SOC Analyst không thể đọc thủ công hàng ngàn dòng log. Phải dùng script.

- **Module cần nhớ**: `import re` (Regular Expression), `import os.path` (Kiểm tra file tồn tại).
    
- **Các Regex Operator cơ bản**:
    
    - `^` : Bắt đầu chuỗi
        
    - `$` : Kết thúc chuỗi
        
    - `.` : Bất kỳ ký tự nào
        
    - `*` : Lặp lại 0 hoặc nhiều lần
        
- **Regex mẫu bắt log SSH**:
    
    - `sshd\[.*\]` (Bắt chữ sshd theo sau là PID trong ngoặc vuông)
        
    - `(?i)` (Case-insensitive - không phân biệt hoa thường)
        
- **Workflow Script**:
    
    1. Xác định path file log (CentOS vs Ubuntu).
        
    2. Mở file (`with open(path, "r") as file:`).
        
    3. Duyệt từng dòng (`for line in file:`).
        
    4. Dùng `re.finditer()` hoặc `re.search()` để tìm pattern.
        
    5. In ra kết quả hoặc lưu vào file mới.

TEST:
file `ssh_log_parser.py`:

```
import re
import os

ubuntu_ssh_log_file_path = "/var/log/auth.log"
ssh_log_files = [ubuntu_ssh_log_file_path]

regex_pattern = r'sshd\[.*\]'

for log_file in ssh_log_files:
    if os.path.isfile(log_file):
        with open(log_file, "r") as file:
            for line in file:
                for match in re.finditer(regex_pattern, line, re.S):
                    print(line, end= '')
```

Output:

```
(venv) dbgman@machine:~$ sudo python3 ssh_log_parser.py 
Oct 18 22:05:37 machine sshd[12345]: Failed password for root from 1.1.1.1 port 56789 ssh2
Oct 18 22:05:37 machine sshd[12345]: Accepted publickey for ubuntu from 2.2.2.2 port 43210 ssh2
(venv) dbgman@machine:~$
```

