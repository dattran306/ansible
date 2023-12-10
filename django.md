Hãy xem xét một dự án khác sử dụng Ansible để triển khai và quản lý một ứng dụng web Python Django trên một máy chủ.

### Dự án: Triển khai ứng dụng web Django với Ansible

**Mục tiêu:**
Triển khai một ứng dụng web Django đơn giản trên máy chủ Ubuntu, cài đặt và cấu hình PostgreSQL, Nginx, Gunicorn, và triển khai ứng dụng Django sử dụng Ansible.

**Bước 1: Chuẩn bị môi trường Ansible**

Tạo một thư mục dự án và tạo các tệp sau:

1. `inventory.ini`: Chứa danh sách các máy chủ mục tiêu.
   ```
   [web_servers]
   your_server_ip ansible_ssh_user=your_ssh_user
   ```

2. `playbook.yml`: Playbook chính để triển khai ứng dụng Django.
   ```yaml
   ---
   - name: Install and configure Django app
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
           - python3
           - python3-pip
           - postgresql
           - libpq-dev
           - nginx
           - supervisor

       - name: Install virtualenv
         pip:
           name: virtualenv

       - name: Set up PostgreSQL database
         postgresql_db:
           name: mydatabase
           state: present

       - name: Clone Django app from Git
         git:
           repo: https://github.com/yourusername/your-django-app.git
           dest: /opt/your-django-app

       - name: Set up virtual environment
         command: virtualenv /opt/your-django-app/env

       - name: Install Python dependencies
         pip:
           requirements: /opt/your-django-app/requirements.txt
           virtualenv: /opt/your-django-app/env

       - name: Configure Gunicorn
         template:
           src: "gunicorn.service.j2"
           dest: "/etc/systemd/system/gunicorn.service"
         notify:
           - restart gunicorn

       - name: Configure Nginx
         template:
           src: "nginx.conf.j2"
           dest: "/etc/nginx/sites-available/your-django-app"
         notify:
           - reload nginx

   handlers:
     - name: restart gunicorn
       systemd:
         name: gunicorn
         state: restarted

     - name: reload nginx
       systemd:
         name: nginx
         state: reloaded
   ```

3. `gunicorn.service.j2`: Mẫu systemd service cho Gunicorn.
   ```ini
   [Unit]
   Description=gunicorn daemon for your-django-app
   After=network.target

   [Service]
   User=your_user
   Group=your_group
   WorkingDirectory=/opt/your-django-app
   ExecStart=/opt/your-django-app/env/bin/gunicorn --workers=3 --bind unix:/opt/your-django-app/your-django-app.sock your_django_app.wsgi:application

   [Install]
   WantedBy=multi-user.target
   ```

4. `nginx.conf.j2`: Mẫu cấu hình Nginx.
   ```nginx
   server {
       listen 80;
       server_name your_server_ip;

       location = /favicon.ico { access_log off; log_not_found off; }
       location /static/ {
           root /opt/your-django-app;
       }

       location / {
           include proxy_params;
           proxy_pass http://unix:/opt/your-django-app/your-django-app.sock;
       }
   }
   ```

**Bước 2: Thực hiện triển khai**

Chạy playbook Ansible để triển khai ứng dụng Django:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Ansible sẽ thực hiện các bước cần thiết để cài đặt và cấu hình hệ thống, sau đó bạn có thể truy cập ứng dụng Django qua trình duyệt với địa chỉ IP của máy chủ.

Dự án này có thể được mở rộng bằng cách thêm các nhiệm vụ như sao lưu cơ sở dữ liệu, cập nhật mã nguồn từ kho Git, và quản lý biến môi trường. Ansible giúp tự động hóa việc triển khai và quản lý ứng dụng web Django, tạo ra một quá trình phát triển linh hoạt và hiệu quả.