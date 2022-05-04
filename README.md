# Configure-a-wordpress-site-from-docker-hub-using-ansible

Here we will build a multi-container WordPress installation using docker container with Ansible. Which includes a MySQL database container and WordPress container.

## Prerequisites

- Need 2 Amazon Linux instances, one with ansible installed
- A purcahsed domain name to point the EC2 instance

## Install ansible on an instance

First install the python on the machine.
```bash
sudo amazon-linux-extras install python3.8 -y
```
then install the ansible using pip3.8

```bash
pip3.8 install ansible
```
to check the ansible version

```bash
ansible --version
```
![image](https://user-images.githubusercontent.com/100775027/166636421-e8f09bde-b510-4961-b779-f1d836fa56ad.png)

## Configure Hosts file (Inventory file)

The orginal Hosts file for Ansible is "/etc/ansible/hosts", but am not using this file as my Hosts file, instead am creating new file with name "inventory". 

```bash
# vim inventory

[amazon]
 <your-server-private-ip> ansible_user="ec2-user" ansible_port=22 ansible_ssh_private_key_file="<your-private-key-file-name>"
 ```
 The headings in brackets are group names [amazon], which are used in classifying hosts and deciding what hosts you are controlling at what times and for what purpose. 
 >  Make sure that you add the "your-server-private-ip" and "your-private-key-file-name" of worker instances on the manager server. Change the "your-private-key-file-name" permission to "400" !
  
 For checking SSH connection to worker server/instance:
  
  ```bash
  ansible -i inventory amazon -m ping
  ```
  ![image](https://user-images.githubusercontent.com/100775027/166636594-f379bfd0-9b8a-4d26-b54f-fdecd320ba69.png)

## Create playbook

Playbooks are expressed in YAML format with a minimum of syntax.  A playbook is composed of one or more ‘plays’ in an ordered list. The terms ‘playbook’ and ‘play’ are sports analogies. Each play executes part of the overall goal of the playbook, running one or more tasks. Each task calls an Ansible module.

Here we are going to write the yml file for creating wordpress site using docker container:

```bash
vim docker-container
```
then add

```bash
---

- name: "creating wordpress using ansible"
  hosts: amazon
  become: true
  vars:
    packages:
      - docker
      - python-pip
    users:
      - "ec2-user"
    db_volume: db_data
    wp_volume: wp_data
  tasks:
    
    - name: "installin pip and docker"
      yum:
        name: "{{packages}}"
        state: present

    - name: "installing docker-py"
      pip:
        name: docker-py
        state: present

    - name: " add the {{users}} to docker grp"
      user:
        name: "{{item}}"
        groups: docker
        append: yes
      with_items:
        - "{{users}}"

    - name: "Restarting and enabling docker service"
      service:
        name: docker
        state: restarted
        enabled: true

    - name: "reate a network"
      docker_network:
        name: mynetwrk

    - name: "Launching mysql container"
      docker_container:
        name: mysql
        image: mysql:latest
        state: started
        recreate: yes
        volumes:
          - "{{ db_volume }}:/var/lib/mysql"
        env: 
          MYSQL_ROOT_PASSWORD: root@123
          MYSQL_DATABASE: uber
          MYSQL_USER: uber
          MYSQL_PASSWORD: uber@123
        networks:
          - name: mynetwrk

    - name: "Launching wordpress container"
      docker_container:
        name: wp
        image: wordpress:latest
        state: started
        recreate: yes
        volumes:
          - "{{wp_volume}}:/var/www/html"
        env: 
          WORDPRESS_DB_HOST: mysql
          WORDPRESS_DB_USER: uber
          WORDPRESS_DB_PASSWORD: uber@123
          WORDPRESS_DB_NAME: uber
        networks:
          - name: mynetwrk
        ports:
          - "80:80"

```
 docker-py: python library for the Docker Remote API. It does everything the docker command does, but from within Python – run containers, manage them, pull/push images, etc.
 
 
 **Check syntax**
 
 Once the playbook is created, we need to check the syntax and execute the playbook:
 
 ```bash
 ansible-playbook -i inventory docker-container --syntax-check
 ```
 ![image](https://user-images.githubusercontent.com/100775027/166636654-f9480cdc-1ee3-4ee3-8383-1f758f539cf6.png)

**Execute the playbook**
 
 ```bash
 ansible-playbook -i inventory docker-container
 ```
 Now our setup is complete and you can point your server public IP to the domain name.
 ![image](https://user-images.githubusercontent.com/100775027/166637067-2f2f0b3d-e17b-4626-bdb2-8672fb2ac9cf.png)

 
## Conclusion
This is how we create wordpress site using docker-container with Ansible. Please contact me when you encounter any difficulty error while using this terrform code. Thank you and have a great day!

 ### ⚙️ Connect with Me
<p align="center">
<a href="https://www.linkedin.com/in/radin-lawrence-8b3270102/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a>
<a href="mailto:radin.lawrence@gmail.com"><img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white"/></a>
