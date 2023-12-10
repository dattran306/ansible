Dưới đây là một dự án sử dụng Ansible để triển khai và quản lý dịch vụ HTTP với Apache HTTP Server.

### Dự án: Triển khai và Quản lý Apache HTTP Server với Ansible

**Mục tiêu:**
Triển khai và cấu hình máy chủ web sử dụng Apache HTTP Server để phục vụ trang web đơn giản bằng Ansible.

**Bước 1: Chuẩn bị môi trường Ansible**

Tạo một thư mục dự án và tạo các tệp sau:

1. `inventory.ini`: Chứa danh sách các máy chủ mục tiêu.
   ```
   [httpd_servers]
   your_httpd_server_ip ansible_ssh_user=your_ssh_user
   ```

2. `playbook.yml`: Playbook chính để triển khai Apache HTTP Server.
   ```yaml
   ---
   - name: Install and configure Apache HTTP Server
     hosts: httpd_servers
     become: yes

     tasks:
       - name: Install Apache
         apt:
           name: apache2
           state: present

       - name: Configure default virtual host
         template:
           src: "000-default.conf.j2"
           dest: "/etc/apache2/sites-available/000-default.conf"
         notify:
           - restart apache2

       - name: Enable mod_rewrite
         apache2_module:
           name: rewrite
           state: present

   handlers:
     - name: restart apache2
       service:
         name: apache2
         state: restarted
   ```

3. `000-default.conf.j2`: Mẫu cấu hình Apache Virtual Host.
   ```apache
   <VirtualHost *:80>
       ServerAdmin webmaster@localhost
       DocumentRoot /var/www/html

       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

**Bước 2: Thực hiện triển khai**

Chạy playbook Ansible để triển khai và cấu hình Apache HTTP Server:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Sau khi triển khai, bạn có thể truy cập trang web mặc định của Apache bằng cách sử dụng địa chỉ IP của máy chủ trong trình duyệt web.

Dự án này có thể được mở rộng bằng cách thêm tính năng như quản lý nhiều trang web, tích hợp với PHP hoặc các ứng dụng web khác, và cấu hình bảo mật. Ansible giúp quản lý cấu hình Apache HTTP Server một cách dễ dàng và linh hoạt.