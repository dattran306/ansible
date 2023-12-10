Dưới đây là một dự án Ansible cơ bản để triển khai và cấu hình Zabbix Server, Zabbix Web Interface và Zabbix Agent trên các máy chủ mục tiêu. Để thực hiện dự án này, bạn cần sử dụng Role để tổ chức playbook và thư mục cấu hình.

### Dự Án: Triển Khai Zabbix Server với Ansible

**Bước 1: Chuẩn Bị Môi Trường Ansible**

1. Tạo một thư mục dự án và điều hướng vào đó:

    ```bash
    mkdir zabbix_ansible
    cd zabbix_ansible
    ```

2. Tạo thư mục `roles` để lưu trữ Roles:

    ```bash
    mkdir roles
    ```

**Bước 2: Tạo Roles cho Zabbix Server, Web, và Agent**

1. Tạo Role cho Zabbix Server (`roles/zabbix_server`):

    - Tạo thư mục `roles/zabbix_server/tasks` và tạo file `main.yml`:

    ```yaml
    ---
    - name: Install Zabbix Server
      apt:
        name: zabbix-server-mysql
        state: present
      notify:
        - restart zabbix-server

    - name: Configure Zabbix Server
      template:
        src: "zabbix_server.conf.j2"
        dest: "/etc/zabbix/zabbix_server.conf"
      notify:
        - restart zabbix-server

    - name: Start and enable Zabbix Server
      systemd:
        name: zabbix-server
        state: started
        enabled: yes

    handlers:
      - name: restart zabbix-server
        systemd:
          name: zabbix-server
          state: restarted
    ```

    - Tạo thư mục `roles/zabbix_server/templates` và tạo file `zabbix_server.conf.j2`:

    ```ini
    DBHost=localhost
    DBName=zabbix
    DBUser=zabbix
    DBPassword=zabbix_password
    ```

2. Tạo Role cho Zabbix Web (`roles/zabbix_web`):

    - Tạo thư mục `roles/zabbix_web/tasks` và tạo file `main.yml`:

    ```yaml
    ---
    - name: Install Zabbix Web
      apt:
        name: zabbix-frontend-php
        state: present

    - name: Configure Zabbix Web
      template:
        src: "zabbix_web.conf.php.j2"
        dest: "/etc/zabbix/web/zabbix.conf.php"
      notify:
        - restart apache2

    handlers:
      - name: restart apache2
        service:
          name: apache2
          state: restarted
    ```

    - Tạo thư mục `roles/zabbix_web/templates` và tạo file `zabbix_web.conf.php.j2`:

    ```php
    <?php
    // Zabbix GUI configuration file
    global $DB;

    $DB['TYPE']     = 'MYSQL';
    $DB['SERVER']   = 'localhost';
    $DB['PORT']     = '0';
    $DB['DATABASE'] = 'zabbix';
    $DB['USER']     = 'zabbix';
    $DB['PASSWORD'] = 'zabbix_password';

    // SCHEMA is relevant only for IBM_DB2 database
    $DB['SCHEMA'] = '';

    $ZBX_SERVER      = 'localhost';
    $ZBX_SERVER_PORT = '10051';
    $ZBX_SERVER_NAME = '';

    $IMAGE_FORMAT_DEFAULT = IMAGE_FORMAT_PNG;
    ```

3. Tạo Role cho Zabbix Agent (`roles/zabbix_agent`):

    - Tạo thư mục `roles/zabbix_agent/tasks` và tạo file `main.yml`:

    ```yaml
    ---
    - name: Install Zabbix Agent
      apt:
        name: zabbix-agent
        state: present
      notify:
        - restart zabbix-agent

    - name: Configure Zabbix Agent
      template:
        src: "zabbix_agentd.conf.j2"
        dest: "/etc/zabbix/zabbix_agentd.conf"
      notify:
        - restart zabbix-agent

    - name: Start and enable Zabbix Agent
      systemd:
        name: zabbix-agent
        state: started
        enabled: yes

    handlers:
      - name: restart zabbix-agent
        systemd:
          name: zabbix-agent
          state: restarted
    ```

    - Tạo thư mục `roles/zabbix_agent/templates` và tạo file `zabbix_agentd.conf.j2`:

    ```ini
    Server=your_zabbix_server_ip
    ServerActive=your_zabbix_server_ip
    Hostname={{ ansible_fqdn }}
    ```

**Bước 3: Tạo Playbook chính (`site.yml`):**

```yaml
---
- name: Deploy Zabbix
  hosts: zabbix_servers
  become: yes
  roles:
    - zabbix_server
    - zabbix_web

- name: Deploy Zabbix Agents
  hosts: zabbix_agents
  become: yes
  roles:
    - zabbix_agent
```

**Bước 4: Thực Hiện Triển Khai**

Chạy playbook Ansible để triển khai Zabbix:

```bash
ansible-playbook -i inventory.ini site.yml
```

Lưu ý rằng dự án này chỉ giới thiệu một số bước cơ bản và cần phải được điều chỉnh và mở rộng thêm để đáp 