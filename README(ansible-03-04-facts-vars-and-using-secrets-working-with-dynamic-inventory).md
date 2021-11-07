# Hands-on Ansible-03 : Using facts, vars and secrets

Purpose of the this hands-on training is to give students the knowledge of ansible facts gathering and working with secret values.

## Learning Outcomes

At the end of this hands-on training, students will be able to;

- Explain how and what facts gathering and how to use it in the playbook
- Learn how to deal with secret values with ansible-vault

## Outline

- Part 1 - Install Ansible

- Part 2 - Ansible Facts

- Part 3 - Working with sensitive data


## Part 1 - Install Ansible


- Spin-up 3 Amazon Linux 2 instances and name them as:
    1. control node
    2. node1 ----> (SSH PORT 22, HTTP PORT 80)
    3. node2 ----> (SSH PORT 22, HTTP PORT 80)


- Connect to the control node via SSH and run the following commands.

```bash
sudo yum update -y
sudo amazon-linux-extras install ansible2
```

### Confirm Installation

- To confirm the successful installation of Ansible, run the following command.

```bash
$ ansible --version
```
Stdout:
```
ansible 2.9.12
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/ec2-user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.18 (default, Aug 27 2020, 21:22:52) [GCC 7.3.1 20180712 (Red Hat 7.3.1-9)]
```
- Explain the lines above:
    1. Version Number of Ansible
    2. Path for the Ansible Config File
    3. Modules are searched in this order
    4. Ansible's Python Module path
    5. Ansible's executable file path
    6. Ansible's Python version with GNU Compiler Collection for Red Hat

### Configure Ansible on the Control Node

- Connect to the control node for building a basic inventory.

- Edit ```/etc/ansible/hosts``` file by appending the connection information of the remote systems to be managed.

- Along with the hands-on, public or private IPs can be used.

```bash
$ sudo su
$ cd /etc/ansible
$ ls
$ vim hosts
[webservers]
node1 ansible_host=<node1_ip> ansible_user=ec2-user

[dbservers]
node2 ansible_host=<node2_ip> ansible_user=ec2-user

[all:vars]
ansible_ssh_private_key_file=/home/ec2-user/<pem file>
```

- Explain what ```ansible_host```, ```ansible_user``` and ansible_ssh_key_file parameters are. For this reason visit the Ansible's [inventory parameters web site](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters).

- Explain what an ```alias``` (node1 and node2) is and where we use it.

- Explain what ```[webservers] and [all:vars]``` expressions are. Elaborate the concepts of [group name](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups), [group variables](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#assigning-a-variable-to-many-machines-group-variables) and [default groups](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#default-groups). 

- Visit the above links for helping to understand the subject. 

- Copy your pem file to the /home/ec2-user directory. First, go to your pem file directory on your local PC and run the following command.

```bash
$ scp -i <pem file> <pem file> ec2-user@<public DNS name of Control Node>:/home/ec2-user
```
- Check if the file is transferred to the remote machine. 

- As an alternative way, create a file on the control node with the same name as the <pem file> in ```/etc/ansible``` directory. 

- Then copy the content of the pem file and paste it in the newly created pem file on the control node.

- To make sure that all our hosts are reachable, we will run various ad-hoc commands that use the ping module.

```bash
$ chmod 400 <pem file>
```

```bash
$ ansible all -m ping -o
```

## Part 2 - Ansible Facts

- Gathering Facts:

```bash
$ ansible node1 -m setup
```
```
ec2-34-201-69-79.compute-1.amazonaws.com | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "172.31.20.246"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::88c:37ff:fe8f:3b71"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "08/24/2006",
        "ansible_bios_vendor": "Xen",
        "ansible_bios_version": "4.2.amazon",
        "ansible_board_asset_tag": "NA",
        "ansible_board_name": "NA",
        "ansible_board_serial": "NA",
```
- create a playbook named "facts.yml"

```yml
- name: show facts
  hosts: all
  tasks:
    - name: print facts
      debug:
        var: ansible_facts
```
- run the play book

```bash
$ ansible-playbook facts.yml
```

- create a playbook named "ipaddress.yml"

```yml
- hosts: all
  tasks:
  - name: show IP address
    debug:
      msg: >
       This host uses IP address {{ ansible_facts.default_ipv4.address }}

```
- run the playbook

```bash
ansible-playbook ipaddress.yml 
```

## Part 3 - Working with sensitive data

- Create encypted variables using "ansible-vault" command

```bash
ansible-vault create secret.yml
```

New Vault password: xxxx
Confirm Nev Vault password: xxxx

```yml
username: tyler
password: 99abcd
```
$ ll
```
- look at the content

```bash
$ cat secret.yml
```
```
33663233353162643530353634323061613431366332373334373066353263353864643630656338
6165373734333563393162333762386132333665353863610a303130346362343038646139613632
62633438623265656330326435646366363137373333613463313138333765366466663934646436
3833386437376337650a636339313535323264626365303031366534363039383935333133306264
61303433636266636331633734626336643466643735623135633361656131316463
```
- how to use it:

- create a file named "create-user"

```bash
$ nano create-user.yml

```

```yml
- name: create a user
  hosts: all
  become: true
  vars_files:
    - secret.yml
  tasks:
    - name: creating user
      user:
        name: "{{ username }}"
        password: "{{ password }}"
```

- run the plaaybook

```bash
ansible-playbook create-user.yml
```
```bash
ERROR! Attempting to decrypt but no vault secrets found
```
- Run the playbook with "--ask-vault-pass" command:

```bash
$ ansible-playbook --ask-vault-pass create-user.yml
```
$ ansible node1 -a "cat /etc/passwd"

```bash (değiştirme)
ansible-vault edit secret.yml
```
--------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------


Vault password: xxxx

```
PLAY RECAP ******************************************************************************************
node1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node2                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- To verrify it

```bash
ansible all -b -m command -a "grep tyler /etc/shadow"
```
```
node1 | CHANGED | rc=0 >>
tyler:99abcd:18569:0:99999:7:::
```

- Create another encypted variables using "ansible-vault" command but this time use SHA (Secure Hash Algorithm) for your password:

```bash
ansible-vault create secret-1.yml
```

New Vault password: xxxx
Confirm Nev Vault password: xxxx

```yml
username: Oliver
pwhash: 14abcd
```

- look at the content

```bash
$ cat secret-1.yml
```
```
33663233353162643530353634323061613431366332373334373066353263353864643630656338
6165373734333563393162333762386132333665353863610a303130346362343038646139613632
62633438623265656330326435646366363137373333613463313138333765366466663934646436
3833386437376337650a636339313535323264626365303031366534363039383935333133306264
61303433636266636331633734626336643466643735623135633361656131316463
```
- how to use it:

---------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------


- create a file named "create-user-1"

```bash
$ nano create-user-1.yml

```

```yml
- name: create a user
  hosts: all
  become: true
  vars_files:
    - secret.yml # secret-1.yml
  tasks:
    - name: creating user
      user:
        name: "{{ username }}"
        password: "{{ password | password_hash ('sha512') }}" # password: "{{ pwhash | password_hash ('sha512') }}"     
``` 

- run the plaaybook


```bash
$ ansible-playbook --ask-vault-pass create-user-1.yml
```
$ ansible node1 -a "cat /etc/passwd"

```bash (ec-2 bağlanma ve kontrol)
$ ssh -i ~/.ssh/2078_ofc.pem  -o IdentitiesOnly=yes ec2-user@52.87.235.255
$ su oliver 
$ Password:11abcd
```
[oliver@node-1 ec2-user]$ sudo cat /etc/shadow

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.
$ oliver@node-1 ec2-user]$ exit
$ ec2-user@node-1 ~]$ sudo su -
$ Password:11abcd
$ [root@node-1 ~]# sudo cat /etc/shadow
tyler hash'li
oliver hash'siz

$ [root@node-1 ~]# su tyler
$ [tyler@node-1 root]$ exit
exit
$ [root@node-1 ~]# logout
$ [ec2-user@node-1 ~]$ su tyler
Password: 
su: Authentication failure
$ [ec2-user@node-1 ~]$ sudo su
$ [root@node-1 ec2-user]# 

Vault password: xxxx

```
PLAY RECAP ******************************************************************************************
node1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node2                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- to verrify it:

```bash
ansible all -b -m command -a "grep Oliver /etc/shadow"
```
```
node1 | CHANGED | rc=0 >>
tyler:#665fffgkg6&fkg689##2£6466?%^^+&%+:18569:0:99999:7:::
```

# Hands-on Ansible-04: Working with Dynamic Inventory Using EC2 Plugin

The purpose of this hands-on training is to give students the knowledge of using dynamic inventory.

## Learning Outcomes

At the end of this hands-on training, students will be able to;

- Explain what is dynamic inventory
- Explain how to use dynamic inventory with EC2 plugin.

![ho-05](ho-05.png)


## Outline

- Part 1 - Build the Infrastructure

- Part 2 - Install Ansible on the Controller Node

- Part 3 - Pinging the Target Nodes with static inventory

- Part 4 - Working with dynamic inventory


## Part 1 - Build the Infrastructure

- Get to the AWS Console and spin-up 3 EC2 Instances with ```Amazon Linux 2``` AMI.

- Configure the security groups as shown below:

    - Controller Node ----> Port 22 SSH

    - Target Node1 -------> Port 22 SSH, Port 80 HTTP

    - Target Node2 -------> Port 22 SSH, Port 80 HTTP

## Part 2 - Install Ansible on the Controller Node

- Connect to your ```Controller Node```.

- Optionally you can connect to your instances using VS Code.

                    -------------------- OPTIONAL BELOW ----------------------

- You can also use connect to the Controller Node via VS Code's ```Remote SSH``` Extension. 

- Open up your VS Code editor. 

- Click on the ```Extensions``` icon. 

- Write down ```Remote - SSH``` on the search bar. 

- Click on the first option on the list.

- Click on the install button.

- When the extension is installed, restart your editor.

- Click on the green button (```Open a Remote Window``` button) at the most bottom left.

- Hit enter. (```Connect Current Window to Host...```)

- Enter a name for your connection on the input field and click on ```Add New SSH Host``` option.

- Enter your ssh connection command (```ssh -i <YOUR-PEM-FILE> ec2-user@<YOUR SERVER IP>```) on the input field and hit enter.

- Hit enter again.

- Click on the ```connect``` button at the bottom right.

- Click on ```continue``` option.

- Click on the ```Open Folder``` button and then click on the ```Ok``` button.

- Lastly, open up a new terminal on the current window.

                    -------------------- OPTIONAL ABOVE ----------------------


## Part 3 - Pinging the Target Nodes with static inventory


- Make a directory named ```dynamic-inventory``` under the home directory and cd into it.

```bash 
$ mkdir dynamic-inventory
$ cd dynamic-inventory
```

- Create a file named ```inventory.txt``` with the command below.

```bash
$ nano inventory.txt
```

- Paste the content below into the inventory.txt file.

- Along with the hands-on, public or private IPs can be used.

```txt
[servers]
db_server   ansible_host=<YOUR-DB-SERVER-IP>   ansible_user=ec2-user  ansible_ssh_private_key_file=~/<YOUR-PEM-FILE>
web_server  ansible_host=<YOUR-WEB-SERVER-IP>  ansible_user=ec2-user  ansible_ssh_private_key_file=~/<YOUR-PEM-FILE>

- Create file named ```ansible.cfg``` under the the ```dynamic-inventory``` directory.

```bash
$ nano ansible.cfg
```

```cfg
[defaults]
host_key_checking = False
inventory=/etc/ansible/hosts
interpreter_python=auto_silent
private_key_file=~/<pem file>
```


- Create a file named ```ping-playbook.yml``` and paste the content below.

```bash
$ nano ping-playbook.yml
```

```yml
- name: ping them all
  hosts: all
  tasks:
    - name: pinging
      ping:
```

- Run the command below for pinging the servers.

```bash
$ ansible-playbook ping-playbook.yml
```

- Explain the output of the above command.

- Change the inventory's value in ansible.cfg file to inventory.txt. 'inventory=/home/ec2-user/dynamic-inventory/inventory.txt'

- Run the command below for pinging the servers.

```bash
$ ansible-playbook ping-playbook.yml
```
## Part4 - Working with dynamic inventory

- go to AWS Management Consol and select the IAM roles:

- click the  "create role" then create a role with "AmazonEC2FullAccess"

- go to EC2 instance Dashboard, and select the control-node instance

- select actions -> security -> modify IAM role

- select the role thay you have jsut created for EC2 full access and save it.

- install "boto3 and botocore"

```bash
$ sudo yum install pip
$ pip install --user boto3 botocore
$ pip3 install --user boto3 botocore
```
$ mkdir dynamic-inventory && cd $_

- Create another file named ```inventory_aws_ec2.yml``` in the project directory.

```bash
$ nano inventory_aws_ec2.yml # ismi bu olmalı.
```

```yml
plugin: aws_ec2
regions:
  - "us-east-1"
keyed_groups:
  - key: tags.Name
  - key: tags.role # eklendi
compose:
  ansible_host: public_ip_address

```
- see the inventory

```bash
$ ansible-inventory -i inventory_aws_ec2.yml --graph
```

$ vim ansible.cfg
[defaults]
host_key_checking=False # host_key_checking = False
inventory=/home/ec2-user/dynamic-inventory/inventory_aws_ec2.yml
interpreter_python=auto_silent
private_key_file=~/.ssh/2078_ofc.pem

[inventory]
enable_plugins=aws_ec2
```

**** sorun yaşandığındaki kodlar:
pip3 install --user awscli boto
pip3 install --user awscli botocore


@all:
  |--@aws_ec2:
  |  |--ec2-34-201-69-79.compute-1.amazonaws.com
  |  |--ec2-54-234-17-41.compute-1.amazonaws.com
  |--@ungrouped:
```

$ nano inventory_aws_ec2.yml # ismi bu olmalı.
```

```yml
plugin: aws_ec2
regions:
  - "us-east-1"
keyed_groups:
  - key: tags.Name
    prefix: "Name"
  - key: tags.role # eklendi
    prefix: "role"
compose:
  ansible_host: public_ip_address




- Change the inventory's value in ansible.cfg file to inventory.txt. 'inventory=/home/ec2-user/dynamic-inventory/inventory_aws_ec2.yml'


tag
node1 
role   webserver

node2 
role   api

- To make sure that all our hosts are reachable with dynamic inventory, we will run various ad-hoc commands that use the ping module.

```bash
$ ansible all -m ping --key-file "~/.ssh/2078_ofc.pem"

$ vim ansible.cfg
[defaults]
host_key_checking=False # host_key_checking = False
inventory=/home/ec2-user/dynamic-inventory/inventory_aws_ec2.yml
interpreter_python=auto_silent
private_key_file=~/.ssh/2078_ofc.pem

```

- create a playbook name "user.yml"

```yml
---
- name: create a user using a variable
  hosts: all
  become: true
  vars:
    user: lisa
    ansible_ssh_private_key_file: "/home/ec2-user/.ssh/2078_ofc.pem"
  tasks:
    - name: create a user {{ user }}
      user:
        name: "{{ user }}"
```
- run the playbook

```bash
$ ansible-playbook user.yml
$ ansible-playbook user.yml -i inventory_aws_ec2.yml
```
```bash
step 5
Name node-3
role webserver


önceki makinelere eklenmesin
$ ansible-playbook user.yml --limit 3.92.77.44
$ ansible-playbook user.yml --limit ec2-3-92-77-44.compute-1.amazonaws.com

node3 stop ettik. güncelledi.
ansible playbook user.yml


```bash
$ ansible all -a "tail -2 /etc/passwd"
```

19 .... count = 4 # 2

22 ....role = "webserver"

control node için role tanı....