Dưới đây là một dự án sử dụng Ansible để triển khai và quản lý dịch vụ HAProxy để làm Load Balancer.

### Dự án: Triển khai và Quản lý HAProxy Load Balancer với Ansible

**Mục tiêu:**
Triển khai và cấu hình máy chủ HAProxy để làm Load Balancer cho một nhóm máy chủ web cùng một ứng dụng.

**Bước 1: Chuẩn bị môi trường Ansible**

Tạo một thư mục dự án và tạo các tệp sau:

1. `inventory.ini`: Chứa danh sách các máy chủ mục tiêu.
   ```
   [haproxy_servers]
   your_haproxy_server_ip ansible_ssh_user=your_ssh_user
   
   [web_servers]
   web_server1 ansible_ssh_user=your_ssh_user
   web_server2 ansible_ssh_user=your_ssh_user
   ```

2. `playbook.yml`: Playbook chính để triển khai HAProxy.
   ```yaml
   ---
   - name: Install and configure HAProxy
     hosts: haproxy_servers
     become: yes

     tasks:
       - name: Install HAProxy
         apt:
           name: haproxy
           state: present

       - name: Configure haproxy.cfg
         template:
           src: "haproxy.cfg.j2"
           dest: "/etc/haproxy/haproxy.cfg"
         notify:
           - restart haproxy

   handlers:
     - name: restart haproxy
       service:
         name: haproxy
         state: restarted
   ```

3. `haproxy.cfg.j2`: Mẫu cấu hình HAProxy.
   ```haproxy
   global
       log /dev/log    local0
       log /dev/log    local1 notice
       chroot /var/lib/haproxy
       stats socket /run/haproxy/admin.sock mode 660 level admin
       stats timeout 30s
       user haproxy
       group haproxy
       daemon

   defaults
       log     global
       mode    http
       option  httplog
       option  dontlognull
       timeout connect 5000
       timeout client  50000
       timeout server  50000

   frontend web
       bind *:80
       default_backend servers

   backend servers
       balance roundrobin
       server web_server1 web_server1_ip:80 check
       server web_server2 web_server2_ip:80 check
   ```
   
**Bước 2: Thực hiện triển khai**

Chạy playbook Ansible để triển khai và cấu hình HAProxy:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Sau khi triển khai, HAProxy sẽ làm Load Balancer cho các máy chủ web `web_server1` và `web_server2`.

Dự án này có thể được mở rộng bằng cách thêm tính năng như cấu hình SSL, chuyển hướng dựa trên URL, và quản lý cân bằng tải dựa trên thuật toán khác nhau. Ansible giúp quản lý cấu hình HAProxy một cách linh hoạt và dễ dàng.