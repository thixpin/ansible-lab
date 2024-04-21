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
1. Access the install_node container `docker exec -it -u ubuntu install_node bash`
2. Install ansible in the install_node container
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
- hosts: web_node1
  become: true
  tasks:
    - name: create a file
      copy:
        dest: /usr/share/nginx/html/file.txt
        content: "This file is created by Ansible!"
```

