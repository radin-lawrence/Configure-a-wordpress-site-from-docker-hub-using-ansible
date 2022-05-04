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
## Configure Hosts file (Inventory file)

The orginal Hosts file for Ansible is "/etc/ansible/hosts", but am not using this file as my Hosts file, instead am creating new file with name "inventory". The headings in brackets are group names [amazon], which are used in classifying hosts and deciding what hosts you are controlling at what times and for what purpose. 

```bash
# vim inventory

[amazon]
 <your-server-private-ip> ansible_user="ec2-user" ansible_port=22 ansible_ssh_private_key_file="<your-private-key-file-name>"
 ```
 > Make sure that you add the <your-server-private-ip> and <your-private-key-file-name> of worker instances on the manager server. Change the <your-private-key-file-name> permission to "400" !
  
 For checking SSH connection to worker server/instance:
  
  ```bash
  ansible -i inventory amazon -m ping
  ```
  
