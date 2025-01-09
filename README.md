# Ansible-Master and target node configuration on AWS cloud using ssh-key Authentication
![Ansible-Master and target node configuration on AWS cloud using ssh-key Authentication (1)](https://github.com/user-attachments/assets/4d193546-e2a7-4b5f-afc4-6937c62f0bb6)

- About set-up:

I have installed ansible on the top of AWS Cloud EC2 Insatnce, Setup is like I take Three EC2 amazon linux Instances, One of these make Master-node(Ansible-master) and other remaining Two make 
Target-node.
(Ansible-node-1,Ansible-node-2).

## On three(master & target) Instances Following set-up do as it is:

![Screenshot 2024-03-01 153246](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/a96622a1-f8a6-46f4-a850-23ceb89e8ecb)

### Step-1: [Create New/General User I create "psadmin" general user for Master-node & set password]
Command for create new user:

    useradd psadmin

Command for Set password to user:
    
    passwd psadmin
   
### Step-2: [Give Sudo power to general user "psadmin"]
The general user don't have so much power like root, So I give the root level power/permission to the general user. The file location is **/etc/sudoers**.
    
    vi /etc/sudoers

![Screenshot 2024-03-01 153326](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/3678c115-aa28-406f-827c-3df34969b7e2)


### Step-3: [Allow Authentication in sshd config file]
To access the EC2 by SSH then we need to change some settings in SSH Config file, The Location of SSH Config file is **/etc/ssh/sshd_config**.
    
    vi /etc/ssh/sshd_config

![11](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/b26c3367-0e18-42aa-9031-4adb254a8142)

### Step-4: [Restart sshd service] 
After change in the ssh config file then we need to restart that file to apply the new changes.

    systemctl restart sshd

- NOTE: for Target-node 1 & 2 i use general user is "pratik" after created user above four steps do as it is in each target nodes.


## On Ansible Master-Node:-
After do 1st four steps then create Key in general user in my case psadmin is general user of my Ansible master node

      su - psadmin

Go inside general user (psadmin) and create key for ssh Authentication:
 
      ssh-keygen

- Note: **Create ssh key at general user on which we want run ansible command.**
  

![12](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/c33efb1a-fdff-4d34-8475-e6d4de217dd9)

ssh-key created in **.ssh/** folder:
    
     cd .ssh/
     
Show hidden file:

     ls -l

After Key created then need to copy my "psadmin" key to host nodes , use follw command to copy key to Target node:
Format of ssh key copy to the target:

    ssh-copy-id  <User_name>@<public_ip_of_target_ec2Instance>

Command for Copy ssh key:

    ssh-copy-id pratik@172.31.44.192
     

![13](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/4147ebd7-fdc5-4f0f-bee2-e57754dafafb)
     
- After key add we also check bye using following Command:

To add EC2 1st time with SSH we need to do manual, While adding they ask password.
   
    ssh pratik@172.31.44.192

- NOTE: Do same Key-copy method to all target nodes .

**........Here our instances is successfully connected by "ssh"........**


## On master node: [Install Ansible on master node]

### Method 1st for installing Ansible: (AMI- Amazon-linux-2)
NOTE:

**If Amazon linux 2 ami use then use following command for download Ansible (/etc/ansible/ansible.cfg this config file provide).**

       sudo amazon-linux-extras install ansible2

### Method 2nd for Install Ansible: (AMI- Amazon-linux)
Install ansible-core, but in this ansible do not provide config file, generally ansible-config file loaction = **/etc/ansible/ansible.cfg**

    sudo yum install ansible-core -y

Command for check ansible version:

    ansible --version
![14](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/1f99f21c-cf32-48b3-a85b-7547308fd0c9)

- We can create ansible config file manually:

NOTE: Fom General user we can't create config file so we need to go root or use sudo.

    exit <<-- this helps to exit from General user

On master Root user:
Go inside **/etc/ansible** folder and create config file(ansible.cfg) use follw commands:

    cd /etc/ansible
    
Create ansible.cfg file:

    touch ansible.cfg

After we created ansible config file it is empty, So we pull file and copy in it for this use following command:

    ansible-config init
    
Copy to destination **/etc/ansible/ansible.cfg**:

    ansible-config init --disabled > /etc/ansible/ansible.cfg

![15](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/c9f71123-d97f-47a8-bab4-da549e18b596)

  
### Method 3rd for installing Ansible with download extra package for yum:(AMI- Amazon-linux-2/Amazon-linux)
 
    sudo yum update -y
    
    sudo amazon-linux-extras install epel -y

    sudo yum install ansible -y

Command for check ansible version and /etc/ansible/ansible.cfg location:

    ansible --version

**In this way we give pre-created ansible config file**

![image](https://github.com/user-attachments/assets/115c887d-9b18-4d02-9b33-17dbf6887303)



## Ansible Config file settings: [ansible.cfg] 
    
In Ansible config file we do following changes:
 
Command for open ansible.cfg
        
     vi /etc/ansible/ansible.cfg

**1st In this file we Add privilege escalation this give become method:** 

![16](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/90c2ee3b-3e5b-409a-b233-e4093b7af8f0)

**2nd In this file we also uncomment ansible Inventory (remove semi colon):**

![17](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/6c6377c9-f853-4cbe-91d8-986037f3cb05)

**3rd Making host_key_checking is False:** (This is because while connecting to target by ssh the target node password ask, So deactive Host_key_checking)

![18](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/b6af24e1-3d8a-4e2b-8bef-b1b080c8df14)


become=True: Enables privilege escalation (e.g., running tasks as root).

become_method=sudo: Specifies that the sudo command is used for privilege escalation.

become_user=root: Defines that the tasks will be executed as the root user (or any other user you specify).

become_ask_pass=False: Prevents Ansible from prompting for the password when escalating privileges (assuming passwordless sudo or other configuration).

- Now  ansible config file set-up:

Create ansible inventory, Location is **/etc/ansible/host**:

    vi /etc/ansible/host


On "psadmin" general user (Master Node):

Command for checking all hosts are connected or not:

    ansible all -m ping

![19](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/d7357644-e6d0-40c5-90fb-40e2dd826647)

Create & Run Ansible-Playbook:
    
    vi web.yml
Command for Run ansible-playbook:

    ansible-playbook web.yml

![20](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/0df1a5b2-9049-4242-986d-e22dc662297f)

Check on target node httpd install or not:

    rpm -q httpd
    
![21](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/35e67fef-7fd1-480f-90b9-a1079f81c696)

