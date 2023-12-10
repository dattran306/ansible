Dự án dưới đây tập trung vào việc sử dụng Ansible để triển khai và quản lý một hệ thống giám sát với Prometheus và Grafana.

### Dự án: Triển khai Prometheus và Grafana với Ansible

**Mục tiêu:**
Triển khai và cấu hình Prometheus để giám sát hệ thống và ứng dụng, sau đó sử dụng Grafana để hiển thị và quản lý các biểu đồ giám sát trực quan.

**Bước 1: Chuẩn bị môi trường Ansible**

Tạo một thư mục dự án và tạo các tệp sau:

1. `inventory.ini`: Chứa danh sách các máy chủ mục tiêu.
   ```
   [monitoring_servers]
   your_server_ip ansible_ssh_user=your_ssh_user
   ```

2. `playbook.yml`: Playbook chính để triển khai Prometheus và Grafana.
   ```yaml
   ---
   - name: Install and configure Prometheus and Grafana
     hosts: monitoring_servers
     become: yes

     tasks:
       - name: Install Prometheus
         community.general.apt:
           name: prometheus
           state: present

       - name: Install Grafana
         community.general.apt:
           name: grafana
           state: present

       - name: Start and enable Prometheus
         systemd:
           name: prometheus
           state: started
           enabled: yes

       - name: Start and enable Grafana
         systemd:
           name: grafana-server
           state: started
           enabled: yes

       - name: Configure Prometheus
         template:
           src: "prometheus.yml.j2"
           dest: "/etc/prometheus/prometheus.yml"
         notify:
           - restart prometheus

       - name: Configure Grafana datasource
         command: "grafana-cli --homepath /usr/share/grafana --pluginsDir /var/lib/grafana/plugins plugins install grafana-piechart-panel"
         become_user: grafana

   handlers:
     - name: restart prometheus
       systemd:
         name: prometheus
         state: restarted
   ```

3. `prometheus.yml.j2`: Mẫu cấu hình Prometheus.
   ```yaml
   global:
     scrape_interval: 15s

   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']

     - job_name: 'node'
       static_configs:
         - targets: ['your_server_ip:9100']
   ```

**Bước 2: Thực hiện triển khai**

Chạy playbook Ansible để triển khai Prometheus và Grafana:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Sau khi triển khai, bạn có thể truy cập giao diện web của Grafana và kết nối nó với Prometheus để tạo và quản lý các biểu đồ giám sát.

Dự án này có thể được mở rộng bằng cách thêm cấu hình và mô hình giám sát cho các thành phần khác của hệ thống, cũng như việc tối ưu hóa biểu đồ và bảng điều khiển trong Grafana. Ansible giúp đơn giản hóa quá trình triển khai và duy trì các hệ thống giám sát.