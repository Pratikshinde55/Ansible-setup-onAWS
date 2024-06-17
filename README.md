# Ansible-setup-onAWS

🌟Ansible-Master and target node configuration on AWS cloud using ssh-key Authentication🌟

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

........Here our instances is successfully connected by "ssh"........



❄️On master node :(Now install Ansible on master node) ❄️

1. install ansible-core  , but in this ansible do not provide config file,
 generally ansible-config file loaction = /etc/ansible/ansible.cfg

        $sudo yum install ansible-core -y
        $ ansible --version

   NOTE:
     If Amazon linux 2 ami use then use follow command for download Ansible (/etc/ansible/ansible.cfg this config file provide)

       # sudo amazon-linux-extras install ansible2

![14](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/1f99f21c-cf32-48b3-a85b-7547308fd0c9)

we can create ansible config file manually.

NOTE: from General user we can't craete config file so we need to go root

        #exit <<-- this helps to exit from General user

On master Root user:
 Go inside /etc/ansible folder and create config file(ansible.cfg) use follw commands:

      #cd /etc/ansible

      # touch anible.cfg

After we created ansible config file it is empty , so we pull file and copy in it for this use following command:

![15](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/c9f71123-d97f-47a8-bab4-da549e18b596)

      # ansible-config init
     
      #ansible-config init --disabled > /etc/ansible/ansible.cfg


⚡In ansible config file we do following changes:
 
open ansible.cfg
        
           #vi /etc/ansible/ansible.cfg

  A. In this file we Add privilege escalation this give become method:

![16](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/90c2ee3b-3e5b-409a-b233-e4093b7af8f0)


 B. In this file we also uncomment ansible Inventory (remove semi colon):

![17](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/6c6377c9-f853-4cbe-91d8-986037f3cb05)


 C. Making host_key_checking is False:

 ![18](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/b6af24e1-3d8a-4e2b-8bef-b1b080c8df14)
          





now  ansible config file set-up


craete ansible inventory:

    #vi /etc/ansible/host


On "psadmin" general user (Master Node):

Check all hosts are connected or not:

       $ ansible all -m ping

![19](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/d7357644-e6d0-40c5-90fb-40e2dd826647)


Create & Run Ansible-Playbook:


      $ vi web.yml
      $ ansible-playbook web.yml

![20](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/0df1a5b2-9049-4242-986d-e22dc662297f)


      


check on target node httpd install or not:--


    # rpm -q httpd
    
![21](https://github.com/Pratikshinde55/Ansible-setup-onAWS/assets/145910708/35e67fef-7fd1-480f-90b9-a1079f81c696)

