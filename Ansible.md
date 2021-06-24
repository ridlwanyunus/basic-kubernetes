# Ansible

## Installing, Configure Host, Writting Playbook

1. Install ansible
   ```bash
   $ sudo yum install ansible2
   ```

2. Configure SSH between Ansible Host and Ansible Node
   Setting up `ssh private key` in Ansible host machine
   ```bash
   $ ssh-keygen
   $ cat /home/ec2-user/.ssh/id_rsa.pub
   ```
   Copy public key to `authorized_keys` Ansible node machine
   ```bash
   $ nano /home/ec2-user/.ssh/authorized_keys
   ```

3. Writting Ansible playbook
   Configuring `hosts` in `Ansible host` machine
   ```bash
   $ vim /home/ec2-user/hosts
   ```
   And then write
   ```yaml
   [production]
   slave1 ansible_ssh_host=3.0.57.102
   ```
   creating playbook
   ```bash
   $ vim /home/ec2-user/test-playbook.yml
   ```
   And then write
   ```yaml
   ---
   - name: write command and install nginx and start its services
     hosts: all
     become: true

     tasks:
      - name: how to install nginx
        yum:
         name: nginx
         state: present
         update_cache: true
        become: true

      - name: start nginx
        service:
         name: nginx
         state: started

      - name: example command
        command: touch file1 file2 file3
        register: myoutput
   ```

   Run the playbook
   ```bash
   $ ansible-playbook -i hosts test-playbook.yml
   ```