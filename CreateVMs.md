Here is the full process from creating the VMs to installing packages:

##### Create the VMs (create_kvm_vms.yml):

```
---
- name: Create KVM virtual servers
  hosts: localhost
  become: yes
  tasks:
    - name: Install required packages
      dnf:
        name:
          - libvirt
          - libvirt-python
          - virt-install
          - qemu-kvm
          - bridge-utils
          - virt-manager
        state: present

    - name: Ensure libvirtd is running and enabled
      service:
        name: libvirtd
        state: started
        enabled: yes

    - name: Define appnode1 VM
      virt:
        name: appnode1
        state: running
        command: define
        memory: 2048
        vcpus: 2
        disk:
          - size: 10
        network:
          - bridge: virbr0
        graphics: vnc
        clock: utc
        os_variant: fedora32

    - name: Define appnode2 VM
      virt:
        name: appnode2
        state: running
        command: define
        memory: 2048
        vcpus: 2
        disk:
          - size: 10
        network:
          - bridge: virbr0
        graphics: vnc
        clock: utc
        os_variant: fedora32

    - name: Start appnode1 VM
      virt:
        name: appnode1
        state: running

    - name: Start appnode2 VM
      virt:
        name: appnode2
        state: running

    - name: Assign static IP to appnode1
      command: >
        virsh net-update default add ip-dhcp-host
        "<host mac='52:54:00:6b:3c:58' name='appnode1' ip='192.168.1.50'/>"
        --live --config

    - name: Assign static IP to appnode2
      command: >
        virsh net-update default add ip-dhcp-host
        "<host mac='52:54:00:6b:3c:59' name='appnode2' ip='192.168.1.51'/>"
        --live --config

```
Ensure SSH Access:

After the VMs are created, ensure they are accessible via SSH. You might need to manually set up SSH keys or use cloud-init if you are automating the setup.
Example of copying an SSH key to the VMs (replace <username> with the appropriate username):

```
ssh-copy-id <username>@192.168.1.50
ssh-copy-id <username>@192.168.1.51
```

Create Inventory File (inventory.ini):
```
[appnodes]
appnode1 ansible_host=192.168.1.50 ansible_user=<username>
appnode2 ansible_host=192.168.1.51 ansible_user=<username>
```
Install Packages Playbook (install_packages.yml):
```
---
- name: Install packages on app nodes
  hosts: appnodes
  become: yes
  tasks:
    - name: Ensure required packages are installed
      ansible.builtin.yum:
        name:
          - httpd
          - vim
          - git
        state: present

```
Running the Playbook
To run the playbook, use the following command:
```
ansible-playbook -i inventory.ini install_packages.yml
```
