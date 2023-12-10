Triển khai và quản lý Docker containers bằng Ansible là một nhiệm vụ phổ biến. Dưới đây là một dự án Ansible cơ bản để cài đặt Docker và triển khai một container đơn giản.

### Dự Án: Triển Khai Docker Container với Ansible

**Bước 1: Chuẩn Bị Môi Trường Ansible**

1. Tạo một thư mục dự án và điều hướng vào đó:

    ```bash
    mkdir docker_ansible
    cd docker_ansible
    ```

2. Tạo thư mục `roles` để lưu trữ Roles:

    ```bash
    mkdir roles
    ```

**Bước 2: Tạo Role cho Docker**

1. Tạo Role cho Docker (`roles/docker`):

    - Tạo thư mục `roles/docker/tasks` và tạo file `main.yml`:

    ```yaml
    ---
    - name: Install Docker
      apt:
        name: docker.io
        state: present
      notify:
        - restart docker

    - name: Start and enable Docker
      systemd:
        name: docker
        state: started
        enabled: yes

    - name: Install docker-py (Python library for Docker)
      pip:
        name: docker
    ```

2. Tạo thư mục `roles/docker/handlers` và tạo file `main.yml`:

    ```yaml
    ---
    - name: restart docker
      systemd:
        name: docker
        state: restarted
    ```

**Bước 3: Tạo Role cho Docker Container**

1. Tạo Role cho Docker Container (`roles/docker_container`):

    - Tạo thư mục `roles/docker_container/tasks` và tạo file `main.yml`:

    ```yaml
    ---
    - name: Pull Docker Image
      docker_image:
        name: "nginx:latest"
      notify:
        - restart docker_container

    - name: Run Docker Container
      docker_container:
        name: "nginx_container"
        image: "nginx:latest"
        ports:
          - "80:80"
      notify:
        - restart docker_container

    handlers:
      - name: restart docker_container
        systemd:
          name: docker_container
          state: restarted
    ```

**Bước 4: Tạo Playbook Chính (`site.yml`):**

```yaml
---
- name: Deploy Docker Container
  hosts: localhost
  become: yes
  roles:
    - docker
    - docker_container
```

**Bước 5: Thực Hiện Triển Khai**

Chạy playbook Ansible để triển khai Docker và container:

```bash
ansible-playbook -i localhost, site.yml
```

Lưu ý rằng dự án này chỉ giới thiệu một số bước cơ bản. Bạn có thể mở rộng nó bằng cách thêm các tác vụ như cài đặt và cấu hình Docker Compose, quản lý networks và volumes, và triển khai nhiều containers hơn với nhiều hình ảnh khác nhau. Ansible cung cấp sự linh hoạt để quản lý Docker một cách tự động.