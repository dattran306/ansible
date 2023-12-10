Triển khai và quản lý Microsoft Hyper-V (Hypervisor) bằng Ansible có thể giúp tự động hóa quá trình triển khai và cấu hình máy ảo trong môi trường Hyper-V. Dưới đây là một dự án Ansible đơn giản để tạo và quản lý máy ảo trên Hyper-V.

### Dự Án: Triển Khai Máy Ảo trên Hyper-V với Ansible

**Bước 1: Chuẩn Bị Môi Trường Ansible**

1. Tạo một thư mục dự án và điều hướng vào đó:

    ```bash
    mkdir hyperv_ansible
    cd hyperv_ansible
    ```

2. Tạo thư mục `roles` để lưu trữ Roles:

    ```bash
    mkdir roles
    ```

**Bước 2: Tạo Role cho Hyper-V**

1. Tạo Role cho Hyper-V (`roles/hyperv`):

    - Tạo thư mục `roles/hyperv/tasks` và tạo file `main.yml`:

    ```yaml
    ---
    - name: Install Hyper-V module for PowerShell
      win_shell: Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart

    - name: Create Virtual Switch
      win_shell: New-VMSwitch -Name "ExternalSwitch" -NetAdapterName "Ethernet" -AllowManagementOS $true

    - name: Create Virtual Machine
      win_shell: New-VM -Name "TestVM" -MemoryStartupBytes 1GB -BootDevice VHD -NewVHDPath "C:\VMs\TestVM.vhdx" -NewVHDSizeBytes 20GB -SwitchName "ExternalSwitch"

    - name: Install Windows OS
      win_shell: Install-WindowsFeature -Name Web-Server -IncludeAllSubFeature -IncludeManagementTools
    ```

**Bước 3: Tạo Playbook Chính (`site.yml`):**

```yaml
---
- name: Deploy Virtual Machine on Hyper-V
  hosts: localhost
  gather_facts: no
  tasks:
    - include_role:
        name: hyperv
```

**Bước 4: Thực Hiện Triển Khai**

Chạy playbook Ansible để triển khai máy ảo trên Hyper-V:

```bash
ansible-playbook -i localhost, site.yml -e ansible_connection=winrm -e ansible_winrm_transport=ntlm -e ansible_winrm_server_cert_validation=ignore
```

Lưu ý rằng dự án này chỉ giới thiệu một số bước cơ bản và có thể cần được tùy chỉnh thêm để đáp ứng yêu cầu cụ thể của bạn. Bạn có thể thêm các nhiệm vụ cấu hình và quản lý các thuộc tính máy ảo Hyper-V khác nhau, chẳng hạn như cấu hình mạng, kích thước bộ nhớ, và cấu hình CPU. Ansible giúp quản lý quá trình triển khai và duy trì máy ảo trên Hyper-V một cách tự động và hiệu quả.