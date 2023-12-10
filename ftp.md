Dưới đây là một dự án sử dụng Ansible để triển khai và quản lý dịch vụ FTP (File Transfer Protocol) với vsftpd (Very Secure FTP Daemon).

### Dự án: Triển khai và Quản lý vsftpd FTP Server với Ansible

**Mục tiêu:**
Triển khai và cấu hình một máy chủ FTP sử dụng vsftpd để chia sẻ tập tin và thư mục trong mạng nội bộ bằng Ansible.

**Bước 1: Chuẩn bị môi trường Ansible**

Tạo một thư mục dự án và tạo các tệp sau:

1. `inventory.ini`: Chứa danh sách các máy chủ mục tiêu.
   ```
   [ftp_servers]
   your_ftp_server_ip ansible_ssh_user=your_ssh_user
   ```

2. `playbook.yml`: Playbook chính để triển khai vsftpd FTP Server.
   ```yaml
   ---
   - name: Install and configure vsftpd FTP Server
     hosts: ftp_servers
     become: yes

     tasks:
       - name: Install vsftpd
         apt:
           name: vsftpd
           state: present

       - name: Configure vsftpd.conf
         template:
           src: "vsftpd.conf.j2"
           dest: "/etc/vsftpd.conf"
         notify:
           - restart vsftpd

       - name: Create FTP user
         user:
           name: "{{ ftp_user }}"
           password: "{{ ftp_password }}"
         vars:
           ftp_user: "your_ftp_user"
           ftp_password: "your_ftp_password"

   handlers:
     - name: restart vsftpd
       service:
         name: vsftpd
         state: restarted
   ```

3. `vsftpd.conf.j2`: Mẫu cấu hình vsftpd.
   ```ini
   listen=NO
   listen_ipv6=YES
   anonymous_enable=NO
   local_enable=YES
   write_enable=YES
   local_umask=022
   dirmessage_enable=YES
   use_localtime=YES
   xferlog_enable=YES
   connect_from_port_20=YES
   chroot_local_user=YES
   secure_chroot_dir=/var/run/vsftpd/empty
   pam_service_name=vsftpd
   ```
   
**Bước 2: Thực hiện triển khai**

Chạy playbook Ansible để triển khai và cấu hình vsftpd FTP Server:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Sau khi triển khai, bạn sẽ có một máy chủ FTP sử dụng vsftpd, và bạn có thể truy cập nó từ các máy tính trong mạng nội bộ để tải lên và tải xuống tập tin.

Dự án này có thể được mở rộng bằng cách thêm tính năng như quản lý người dùng và quyền truy cập, tích hợp với TLS để bảo mật kết nối, và tối ưu hóa cấu hình để đáp ứng nhu cầu cụ thể của bạn. Ansible giúp quản lý cấu hình FTP Server một cách hiệu quả và linh hoạt.