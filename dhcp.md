Dưới đây là một dự án sử dụng Ansible để triển khai và quản lý dịch vụ DHCP (Dynamic Host Configuration Protocol) với ISC DHCP Server.

### Dự án: Triển khai và Quản lý ISC DHCP Server với Ansible

**Mục tiêu:**
Triển khai và cấu hình một máy chủ DHCP sử dụng ISC DHCP Server trên một máy chủ Ubuntu bằng Ansible.

**Bước 1: Chuẩn bị môi trường Ansible**

Tạo một thư mục dự án và tạo các tệp sau:

1. `inventory.ini`: Chứa danh sách các máy chủ mục tiêu.
   ```
   [dhcp_servers]
   your_dhcp_server_ip ansible_ssh_user=your_ssh_user
   ```

2. `playbook.yml`: Playbook chính để triển khai ISC DHCP Server.
   ```yaml
   ---
   - name: Install and configure ISC DHCP Server
     hosts: dhcp_servers
     become: yes

     tasks:
       - name: Install ISC DHCP Server
         apt:
           name: isc-dhcp-server
           state: present

       - name: Configure dhcpd.conf
         template:
           src: "dhcpd.conf.j2"
           dest: "/etc/dhcp/dhcpd.conf"
         notify:
           - restart isc-dhcp-server

   handlers:
     - name: restart isc-dhcp-server
       service:
         name: isc-dhcp-server
         state: restarted
   ```

3. `dhcpd.conf.j2`: Mẫu cấu hình DHCP Server.
   ```dhcp
   authoritative;
   default-lease-time 600;
   max-lease-time 7200;
   subnet 192.168.1.0 netmask 255.255.255.0 {
     range 192.168.1.100 192.168.1.200;
     option routers 192.168.1.1;
     option domain-name-servers 8.8.8.8, 8.8.4.4;
     option domain-name "example.com";
   }
   ```

**Bước 2: Thực hiện triển khai**

Chạy playbook Ansible để triển khai và cấu hình ISC DHCP Server:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Sau khi triển khai, máy chủ DHCP sẽ được cấu hình để cung cấp địa chỉ IP động cho các thiết bị trong subnet `192.168.1.0/24`. Bạn có thể tùy chỉnh cấu hình `dhcpd.conf.j2` để phản ánh cấu trúc mạng của bạn.

Dự án này có thể được mở rộng bằng cách thêm tính năng như cấu hình DHCP tùy chỉnh cho các subnet khác, quản lý địa chỉ IP đã cấp phát, và tùy chọn cấu hình bảo mật. Ansible giúp quản lý cấu hình DHCP một cách hiệu quả và linh hoạt.