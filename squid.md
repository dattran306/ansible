Dưới đây là một dự án sử dụng Ansible để triển khai và quản lý dịch vụ Squid Proxy Server.

### Dự án: Triển khai và Quản lý Squid Proxy Server với Ansible

**Mục tiêu:**
Triển khai và cấu hình máy chủ Squid Proxy Server để cung cấp dịch vụ proxy cho mạng nội bộ bằng Ansible.

**Bước 1: Chuẩn bị môi trường Ansible**

Tạo một thư mục dự án và tạo các tệp sau:

1. `inventory.ini`: Chứa danh sách các máy chủ mục tiêu.
   ```
   [squid_servers]
   your_squid_server_ip ansible_ssh_user=your_ssh_user
   ```

2. `playbook.yml`: Playbook chính để triển khai Squid Proxy Server.
   ```yaml
   ---
   - name: Install and configure Squid Proxy Server
     hosts: squid_servers
     become: yes

     tasks:
       - name: Install Squid
         apt:
           name: squid
           state: present

       - name: Configure squid.conf
         template:
           src: "squid.conf.j2"
           dest: "/etc/squid/squid.conf"
         notify:
           - restart squid

   handlers:
     - name: restart squid
       service:
         name: squid
         state: restarted
   ```

3. `squid.conf.j2`: Mẫu cấu hình Squid Proxy.
   ```squid
   http_port 3128
   acl localnet src 192.168.0.0/24
   http_access allow localnet
   http_access deny all
   ```

**Bước 2: Thực hiện triển khai**

Chạy playbook Ansible để triển khai và cấu hình Squid Proxy Server:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Sau khi triển khai, bạn có thể cấu hình trình duyệt của máy tính trong mạng nội bộ để sử dụng Squid Proxy Server tại địa chỉ IP của máy chủ Squid và cổng 3128.

Dự án này có thể được mở rộng bằng cách thêm tính năng như xác thực người dùng, giới hạn băng thông, và kiểm soát quyền truy cập. Ansible giúp quản lý cấu hình Squid Proxy Server một cách hiệu quả và linh hoạt.