# Ansible lab

This is a simple ansible lab to demonstrate how to use ansible to automate tasks.
And this lab is running with docker-compose.

### Requirements
- Docker

### How to run
1. Clone this repository
2. Run `cd ansible-lab`
3. Run `ssh-keygen -t ed25519 -f playbooks/sshkey` to generate a new ssh keypair in the ssh directory
4. Run `touch authorized_keys` to create an authorized_keys file
5. Run `cat playbooks/sshkey.pub >> authorized_keys` to add the public key to the authorized_keys file
6. Run `docker-compose build`
7. Run `docker-compose up -d`

### How to access the containers
1. Run `docker exec -it -u ubuntu control_node bash` to access the control node
2. Run `docker exec -it web_node1 bash` to access the web_node1
3. Run `docker exec -it web_node1 ls -la /usr/share/nginx/html` to list the files in the web_node1's html directory


# Let's start the labs

## Lab 1 - Install ansible on ubuntu
1. Access the control_node container `docker exec -it -u ubuntu control_node bash`
2. Install ansible in the control_node container
    ```bash
    sudo apt-get install software-properties-common -y
    sudo apt-add-repository ppa:ansible/ansible -y 
    sudo apt-get install ansible -y
    ansible --version
    ```

## Lab 2 - Create an inventory file
1. Access the control_node container `docker exec -it -u ubuntu control_node bash`
2. Create playbooks directory
    ```bash
    mkdir -p playbooks
    cd playbooks
    ```
3. Create Ansible configuration file
    ```bash
    vim ansible.cfg
    ```
    #### /home/ubuntu/playbooks/ansible.cfg
    ```ini
    [defaults]
    inventory = inventory
    remote_user = root
    private_key_file = /home/ubuntu/playbooks/key
    host_key_checking = False
    interpreter_python = auto_silent
    ```
2. Create an inventory file 
    | Node Name | IP Address  | Connection | City       |
    | :-------- | :---------- | :--------: | :--------- |
    | web_node1 | 172.20.0.11 | ssh        | Yangon     |
    | web_node2 | 172.20.0.12 | ssh        | Mandalay   |
    | db_node1  | 172.20.0.13 | ssh        | Yangon     |
    ___ 


    | Group Name | Members |
    | :--------- | :------ |
    | web_nodes  | web_node1, web_node2 |
    | db_nodes   | db_node1 |
    | ygn_nodes  | web_node1, db_node1 |
    | mdy_nodes  | web_node2 |
    | myanmar_nodes | ygn_nodes, mdy_nodes |
    ___
    ```bash
    cd ~/playbooks
    vim inventory
    ```

    #### /home/ubuntu/playbooks/inventory
    ```ini
    # Web Servers

    # Located in Yangon
    web_node1 ansible_host=172.20.0.11 ansible_user=root 

    # Located in Mandalay
    web_node2 ansible_host=172.20.0.12 ansible_user=root


    # Database Servers

    # Located in Yangon
    db_node1  ansible_host=172.20.0.13 ansible_user=root



    [web_nodes]
    web_node1
    web_node2

    [db_nodes]
    db_node1

    [all_servers:children]
    web_nodes
    db_nodes

    [ygn_nodes]
    web_node1
    db_node1

    [mdy_nodes]
    web_node2

    [myanmar_nodes:children]
    ygn_nodes
    mdy_nodes

    ```

## Lab 3 - Test the connection
1. Access the control_node container `docker exec -it -u ubuntu control_node bash`
2. Test the connection to the nodes
    ```bash
    ansible all -i inventory -m ping
    ```

## Lab 4 - Creating the ansible playbooks

### 1. Running Commands on Localhost:
Write a playbook to execute a command on localhost and display the contents of /etc/resolv.conf.


```yaml
#command.yml
---
- name: Execute a command on localhost
  hosts: localhost
  connection: local
  tasks:
    - name: Execute a command
      command: cat /etc/resolv.conf
```

### 2. Managing File Permissions on Remote Hosts:
Write a playbook to 
- create a file on web_node1 with a specified group (nginx) and 
- another file on web_node2 with a specified owner (nginx). \
Ensure both tasks require elevated permissions


```yaml
# perm.yml
---
- hosts: web_node1
  become: true
  tasks:
    - name: Creating blog.txt file
      file:
        path: /usr/share/nginx/html/blog.txt
        state: touch
        group: nginx

- hosts: web_node2
  become: true
  tasks:
    - name: Creating story.txt file
      file:
        path: /usr/share/nginx/html/story.txt
        state: touch
        owner: nginx
```

### 3. Creating a File with Specific Content:
Write a playbook to create a file on node01 with a specific text content at /opt/file.txt.

```yaml
# file.yml
---
- hosts: web_nodes
  become: true
  tasks:
    - name: create a file
      copy:
        dest: /usr/share/nginx/html/file.txt
        content: "This file is created with Ansible by KT"
```

### 4. Copying Files to Multiple Hosts:
Write a playbook to copy a file from a source path to a specified destination on web_nodes group. Ensure that the file is sourced from the remote machine (remote_src is set to yes).

```yaml
# copy.yml
---
- hosts: web_nodes
  become: true
  tasks:
    - name: Copying a file to all hosts
      copy:
        src: /usr/share/nginx/html/file.txt
        dest: /usr/share/nginx/html/file-copy.txt
        remote_src: yes
```

### 5. Installing Packages on Remote Hosts:
Write a playbook to install the htop package on all hosts in the web_nodes group.

```yaml
# install.yml
---
- hosts: web_nodes
  become: true
  tasks:
    - name: Install htop package
      apk:
        name: htop
        state: present
```

### 6. Replacing Text in Files:
Create a playbook to replace occurrences of "KT" with "thixpin" in a specific file on node01. Similarly, replace "KT" with "DevOps" in another file on node02.
  
```yaml
# replace.yml
---
- hosts: web_node1
  become: true
  tasks:
    - name: Replace text in file
      replace:
        path: /usr/share/nginx/html/file.txt
        regexp: 'KT'
        replace: 'thixpin'

- hosts: web_node2
  become: true
  tasks:
    - name: Replace text in file
      replace:
        path: /usr/share/nginx/html/file.txt
        regexp: 'KT'
        replace: 'DevOps'
```


### 7. Checking for Conditions:
Write a playbook to check if a user is a DevOps or not. If the user is a DevOps, display "I am a DevOps". Otherwise, display "I am not a DevOps".

```yaml
# condition.yml
---
- name: 'Am I a DevOps or not?'
  hosts: localhost
  connection: local
  vars:
    user: devops
  tasks:
    - name: 'Check if I am a DevOps'
      debug:
        msg: 'I am a DevOps'
      when: user == "devops"
    - name: 'Check if I am not a DevOps'
      debug:
        msg: 'I am not a DevOps'
      when: user != "devops"
```

### 8. Looping Through Variables to Execute Commands:
Write a playbook that uses a list of DevOps tools and loops through them to execute a command that prints each tool name.

```yaml
# loop.yml
---
- name: Loop through DevOps tools
  hosts: localhost
  connection: local
  vars:
    devops_tools:
      - Ansible
      - Docker
      - Kubernetes
      - Jenkins
      - Terraform
  tasks:
    - name: Print DevOps tools
      debug:
        msg: 'DevOps tool: {{ item }}'
      loop: "{{ devops_tools }}"
```

### 9. Conditional File Copy with Owner and Permissions:
Create a playbook to copy a file with specified owner, group, and permissions on different hosts. Use conditions to differentiate tasks based on the host
  
```yaml
# file_conditional.yml
---
- hosts: web_nodes
  become: true
  tasks:
    - name: Copy file with owner, group, and permissions on web_node1
      copy:
        src: /usr/share/nginx/html/file.txt
        dest: /usr/share/nginx/html/new-file.txt
        remote_src: yes
        owner: nginx
        group: nginx
        mode: '0755'
      when: inventory_hostname == 'web_node1'
      
    - name: Copy file with owner, group, and permissions on web_node2
      copy:
        src: /usr/share/nginx/html/file.txt
        dest: /usr/share/nginx/html/new-file.txt
        remote_src: yes
        owner: nginx
        group: nginx
        mode: '0400'
      when: inventory_hostname == 'web_node2'
```

### 10. Inserting Lines into a File:
Write a playbook to insert a specific line into a given file on web_node1. Ensure the line is inserted at the beginning of the file.

```yaml
# insert.yml
---
- hosts: web_node1
  become: true
  tasks:
    - name: Insert a line into a file
      lineinfile:
        path: /usr/share/nginx/html/file.txt
        line: 'This line is inserted by Ansible'
        insertbefore: BOF
```

### 11. Managing Services with Ansible:
Create a playbook that installs a package (nginx) on all web servers. And also update nginx configuration file (change keepalive_timeout  to 120) and restart the service by using handlers.

```yaml
# service.yml
---
- hosts: web_nodes
  become: true
  tasks:
    - name: Install nginx package
      apk:
        name: nginx
        state: present

    - name: Update nginx configuration file
      lineinfile:
        path: /etc/nginx/nginx.conf
        regexp: 'keepalive_timeout'
        line: '    keepalive_timeout 120;'
      notify: restart nginx

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

### 12. Creating a User with Specific UID :
Write a playbook to create a user with a specific UID on localhost and assign the user to the sudo group. And also, update the sudoers file to allow the user to run sudo commands without a password.

```yaml
# user.yml
---
- hosts: localhost
  become: true
  tasks:
    - name: Create a user with specific UID
      user:
        name: devops
        uid: 2000
        group: sudo
        state: present

    - name: Allow devops user to run sudo commands without password
      lineinfile:
        path: /etc/sudoers
        line: 'devops ALL=(ALL) NOPASSWD: ALL'

```

### 13. Creating an Archive:
Write a playbook to create a compressed archive (demo.tar.gz) from a specific path and save it to a designated destination on all hosts.

```yaml
# archive.yml
---
- hosts: web_nodes
  become: true
  tasks:
    - name: Create an archive
      archive:
        path: /usr/share/nginx/html
        dest: /usr/share/nginx/demo.tar.gz
```

### 14. Installing a Role:
Install the `geerlingguy.nginx` role from Ansible Galaxy under ~/playbooks/roles directory. Use the role to install Node.js on localhost.
Further consume this role in /home/bob/playbooks/role.yml playbook so that this role can be applied on localhost.
```bash
ansible-galaxy install geerlingguy.nginx -p ./roles
```

```yaml
# role.yml
---
- hosts: localhost
  become: true
  roles:
    - geerlingguy.nginx
```


### 15. Using templates:
Create a playbook to create a text file with a specific content on web_node1 using a template file located in the templates directory.

```yaml
# template.yml
---
- hosts: web_node1
  become: true
  tasks:
    - name: Create a file using a template
      template:
        src: templates/hello.txt.j2
        dest: /usr/share/nginx/html/hello.txt
        remote_src: no
```

```jinja2
# templates/hello.txt.j2
Hello, this is a template file created by {{ ansible_user_id }} on date {{ ansible_date_time.date }} at {{ ansible_hostname }}.
```


