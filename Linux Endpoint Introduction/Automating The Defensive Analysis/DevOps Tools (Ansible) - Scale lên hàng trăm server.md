- **Vấn đề**: Chạy script Python trên 1 máy thì dễ, chạy trên 100 máy thì rất mệt.
    
- **Giải pháp**: Dùng Ansible Playbook (viết bằng YAML).
    
- **Cấu trúc cơ bản của Playbook**:

``` yaml
- name: logparser
  hosts: soc200
  become: yes
  become_user: root
  tasks:
    - name: list files in folder
      script: /path/to/ssh_log_parser.py
      args:
        executable: python3
      register: output
    - debug:
        var=output.stdout_lines
```

- **Lệnh chạy**: `ansible-playbook ./log_parser.yml -u offsec --key-file='/path/key' -K`

---
## Tự động hóa thu thập Log tập trung (Ansible)

Bao gồm: 
- 1 máy kali (đóng vai trò manager)
- 1 máy ubuntu (đóng vai trò server, nới có log)

Bước 1: tạo key: `ssh-keygen -t rsa -f ~/.ssh/ansible_rsa -N ""`

Bước 2: copy Public Key sang máy Ubuntu`ssh-copy-id -i ~/.ssh/ansible_rsa.pub dbgman@192.168.174.128`

Bước 3: kết nối thử để hosts key: `ssh -i ~/.ssh/ansible_rsa dbgman@192.168.174.128 "exit"`

Bước 4: `sudo nano /etc/ansible/hosts` 

```
[victim]
192.168.174.128 ansible_user=<user_name_ubuntu_machine>
```

Bước 5: `ansible-playbook ./log_parser.yml -u <user_name_ubuntu_machine> --key-file='~/.ssh/ansible_rsa' -K`

Nếu cấu hình file `hosts` rồi thì ko cần tham số `-u <user_name_ubuntu_machine>` nữa và ngược lại.

---
####  Reset cấp độ Ansible (Xóa Key & Inventory)

Nếu bạn chỉ muốn làm lại phần Ansible mà không cài lại máy ảo:

```
# Xóa SSH Key cũ
rm -f ~/.ssh/ansible_rsa ~/.ssh/ansible_rsa.pub

# Xóa file inventory cũ (hoặc sửa lại)
sudo rm -f /etc/ansible/hosts

# Xóa file ansible.cfg (nếu có)
rm -f ~/ansible.cfg

# Xóa known_hosts (để xóa dấu vân tay máy Ubuntu cũ)
ssh-keygen -R 192.168.174.128
```


#### Reset cấp độ máy Ubuntu (Xóa Key đã copy)

Nếu bạn muốn xóa Public Key đã copy sang Ubuntu để test lại từ đầu:

Cách 1: **Dùng lệnh trên Kali (nhanh)**
```
ssh dbgman@192.168.174.128 "rm -f ~/.ssh/authorized_keys"
```

Cách 2: **Vào trực tiếp máy Ubuntu**
```
# Đăng nhập vào Ubuntu, sau đó chạy:
rm -f ~/.ssh/authorized_keys
# Hoặc xóa từng dòng key của Kali bằng nano:
nano ~/.ssh/authorized_keys
```

