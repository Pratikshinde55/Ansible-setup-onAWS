# Ansible-setup-onAWS

🌟Ansible-setup on AWS cloud using ssh-key Authentication🌟

About set-up:

I have installed ansible on the top of AWS cloud EC2 ,Setup is i take Three EC2 amazon linux instance , one of these make master node(Ansible-master) and other remaining two make target node (Ansible-node-1,Ansible-node-2).

❄️On three(master and target) instances Following set-up do as it is:

![Screenshot 2024-03-01 153246](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/a96622a1-f8a6-46f4-a850-23ceb89e8ecb)


1. create new user : i created "psadmin" general user for master node & set password :

          #useradd psadmin
          #passwd psadmin
   
3. Give Sudo power to general user "psadmin" : 
    
           #vi /etc/sudoers

![Screenshot 2024-03-01 153326](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/3678c115-aa28-406f-827c-3df34969b7e2)


3.Allow Authentication in sshd config file :
    
           #vi /etc/ssh/sshd_config

![11](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/b26c3367-0e18-42aa-9031-4adb254a8142)


4.restart sshd service 

    
          #systemctl restart sshd

NOTE: for Target-node 1 & 2 i use general user is "pratik" after craeted user above four steps do as it is in each target nodes.


💥On Master-Node 💥:---

Go inside general user (psadmin) and create key for ssh Authentication:
 
      #ssh-keygen
  Note: create ssh key at general user on which we want run ansible command.
  

![12](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/c33efb1a-fdff-4d34-8475-e6d4de217dd9)

ssh-key created in .ssh/ folder


       $ cd .ssh/
       $ ls -l

After Key created then need to copy my "psadmin" key to host nodes , use follw command to copy key to Target node:

 #ssh-copy-id  pratik@<private ip of target ec2 instance>   <<--Format of cmd
    
     $ ssh-copy-id pratik@172.31.44.192
     

![13](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/4147ebd7-fdc5-4f0f-bee2-e57754dafafb)
     

   After key add we also check bye using
   
     #ssh pratik@172.31.44.192

NOTE: Do same Key-copy method to all target nodes .

.....Here our instances is successfully connected by "ssh"



❄️On master node :(Now install Ansible on master node) ❄️

1. install ansible-core  , but in this ansible do not provide config file,
 generally ansible-config file loaction = /etc/ansible/ansible.cfg

        $sudo yum install ansible-core -y
        $ ansible --version

![14](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/1f99f21c-cf32-48b3-a85b-7547308fd0c9)

we can craete ansible config file manually.
NOTE: from General user we can't craete config file so we need to go root
        #exit <<-- this helps to exit from General user

On master Root user:
 Go inside /etc/ansible folder and create config file(ansible.cfg) use follw coomand:

      #cd /etc/ansible

      # touch anible.cfg

After we created ansible config file it is empty , so we pull file and copy in it for this use following command:

      # ansible-config init
     
      #ansible-config init --disabled > /etc/ansible/ansible.cfg


In ansible config file we do following changes:
 
open ansible.cfg
        
           #vi /etc/ansible/ansible.cfg

   In this file we Add privilege escalation this give become method 
        
           //scrensht//

   in this file we also uncomment ansible inventory (remove semi colon)
              //scrensht//

   making host_key_checking is False
           //scrensht//





now  ansible config file set-up


craete ansible inventory:

    #vi /etc/ansible/hosts
    
 [pratik]
3.110.29.198  ansible_user=pratik  ansible_password=ps
13.201.16.171  ansible_user=pratik  ansible_password=ps




on psadmin general user :
check all hosts are connected or not:


   $ ansible all -m ping
   $ vi web.yml
   $ ansible-playbook web.yml
   
play-book:
 
- hosts: pratik
  tasks:
    - name: "installing httpd "
      package:
        name: "httpd"
        state: present


list of pratik group :
         #ansible pratik  --list-hosts



check on target node httpd install or not:--
             # rpm -q httpd
