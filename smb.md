Dưới đây là một dự án sử dụng Ansible để triển khai và quản lý dịch vụ SMB (Server Message Block) với Samba.

### Dự án: Triển khai và Quản lý Samba Server với Ansible

**Mục tiêu:**
Triển khai và cấu hình một máy chủ Samba để chia sẻ tập tin và thư mục trong mạng nội bộ bằng Ansible.

**Bước 1: Chuẩn bị môi trường Ansible**

Tạo một thư mục dự án và tạo các tệp sau:

1. `inventory.ini`: Chứa danh sách các máy chủ mục tiêu.
   ```
   [samba_servers]
   your_samba_server_ip ansible_ssh_user=your_ssh_user
   ```

2. `playbook.yml`: Playbook chính để triển khai Samba Server.
   ```yaml
   ---
   - name: Install and configure Samba Server
     hosts: samba_servers
     become: yes

     tasks:
       - name: Install Samba
         apt:
           name: samba
           state: present

       - name: Configure smb.conf
         template:
           src: "smb.conf.j2"
           dest: "/etc/samba/smb.conf"
         notify:
           - restart smbd

       - name: Create Samba user
         command: "smbpasswd -a {{ samba_user }}"
         vars:
           samba_user: "your_username"

   handlers:
     - name: restart smbd
       service:
         name: smbd
         state: restarted
   ```

3. `smb.conf.j2`: Mẫu cấu hình Samba.
   ```ini
   [global]
   workgroup = WORKGROUP
   security = user

   [share]
   comment = Shared Directory
   path = /srv/samba/share
   browseable = yes
   read only = no
   valid users = @{{ samba_user }}
   ```
   
**Bước 2: Thực hiện triển khai**

Chạy playbook Ansible để triển khai và cấu hình Samba Server:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Sau khi triển khai, bạn sẽ có một máy chủ Samba được cấu hình để chia sẻ thư mục `/srv/samba/share`. Sử dụng địa chỉ IP của máy chủ, bạn có thể truy cập thư mục chia sẻ từ các máy tính trong mạng nội bộ của bạn.

Dự án này có thể được mở rộng bằng cách thêm tính năng như quản lý quyền truy cập, chia sẻ thư mục từ xa, và tích hợp với các dịch vụ đăng nhập như LDAP. Ansible giúp quản lý cấu hình Samba một cách dễ dàng và linh hoạt.