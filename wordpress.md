Hãy xem xét một ví dụ về việc sử dụng Ansible trong một dự án nhỏ để quản lý cấu hình hệ thống và triển khai ứng dụng web đơn giản.

### Dự án: Triển khai một trang web WordPress với Ansible

**Mục tiêu:**
Triển khai một trang web WordPress đơn giản trên một máy chủ Ubuntu, cài đặt và cấu hình Apache, MySQL, PHP, và WordPress sử dụng Ansible.

**Bước 1: Chuẩn bị môi trường Ansible**

Tạo một thư mục dự án và tạo các tệp sau:

1. `inventory.ini`: Chứa danh sách các máy chủ mục tiêu.
   ```
   [web_servers]
   your_server_ip ansible_ssh_user=your_ssh_user
   ```

2. `playbook.yml`: Playbook chính để triển khai WordPress.
   ```yaml
   ---
   - name: Install and configure WordPress
     hosts: web_servers
     become: yes

     tasks:
       - name: Update apt cache
         apt:
           update_cache: yes

       - name: Install required packages
         apt:
           name: "{{ item }}"
           state: present
         loop:
           - apache2
           - mysql-server
           - php
           - libapache2-mod-php
           - php-mysql
           - unzip

       - name: Download WordPress
         get_url:
           url: "https://wordpress.org/latest.zip"
           dest: "/tmp/wordpress.zip"

       - name: Extract WordPress
         unarchive:
           src: "/tmp/wordpress.zip"
           dest: "/var/www/html/"
           remote_src: yes

       - name: Configure Apache
         template:
           src: "apache2.conf.j2"
           dest: "/etc/apache2/apache2.conf"
         notify:
           - restart apache

       - name: Configure MySQL
         template:
           src: "my.cnf.j2"
           dest: "/etc/mysql/my.cnf"
         notify:
           - restart mysql

   handlers:
     - name: restart apache
       service:
         name: apache2
         state: restarted

     - name: restart mysql
       service:
         name: mysql
         state: restarted
   ```

3. `apache2.conf.j2`: Mẫu cấu hình Apache.
   ```apache
   <Directory /var/www/html/>
       Options Indexes FollowSymLinks
       AllowOverride All
       Require all granted
   </Directory>
   ```

4. `my.cnf.j2`: Mẫu cấu hình MySQL.
   ```ini
   [mysqld]
   bind-address = 0.0.0.0
   ```

**Bước 2: Thực hiện triển khai**

Chạy playbook Ansible để triển khai WordPress:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Ansible sẽ thực hiện các bước cần thiết để cài đặt và cấu hình hệ thống, sau đó bạn có thể truy cập trang web WordPress qua trình duyệt với địa chỉ IP của máy chủ.

Đây chỉ là một ví dụ đơn giản và thực tế, bạn có thể mở rộng dự án này bằng cách thêm các tác vụ khác như sao lưu, khôi phục, cập nhật nội dung, và quản lý SSL. Ansible giúp tự động hóa các quy trình này và tạo ra một hệ thống dễ duy trì và mở rộng.