## **Working with Ansible Groups; Groups and Group of groups**

<br>

Yet another series we have here on Ansible! Welcome, dear reader.
This article will basically teach you how to run ansible configurations to diferent ansible groups that consists of a number of hosts. We will simply classify host IP addresses into several groups and run ansible tasks on them as a group of hosts.

<br>

I believe you already know the prerequisites to run ansible. If not, see below.
* Adequate Internet Connection
* AWS Account
* A running VM that will serve as our workstation
* Several Target Servers

<br>

## [Step 1] Launch 5 Instances on AWS
In this article, we will be working with five servers. You can choose to run 10 or 20 or even more instances depending on the scope of your project.

So, let's login to our AWS dashboard and spin up some instances. We shall launch

* 3 Ubuntu instances
* 1 CentOS instance, and 
* 1 Ubuntu (master) instance

![aws](./grp_images/Screenshot%20(235).png)

Now, let's connect to the **'master-ubuntu'** instance as we will be using it as 'workstation'. 

<br>

## [Step 2] Create a User on master-ubuntu
we will create a workstation user, and grant it sudo privilege, on the 'master-ubuntu'.

First, after being connected to the master-ubuntu successfully, switch to root user by typing on your terminal;

      sudo su

      adduser workstation

      usermod -G sudo workstation 
      (This grants our new user, workstation, a sudo privilege)

Next, we will switch to the new user (workstation) from master-ubuntu root user. 

      su - workstation

 ## [Step 3] Install Ansible on Workstation
Now, we must install the ansible package repository before installation.

      sudo apt-get-repository ppa:ansible/ansible

      sudo apt update           

      sudo apt upgrade -y

Then install ansible

      sudo apt install ansible -y

## [Step 4] Write Ansible Configurations
Create a folder in the workstation (using the linux 'mkdir' command). We will name this folder 'ansible' by typing on your terminal;

      mkdir ansible 

Then enter into this new folder by typing;

      cd ansible      

Now that we are in ansible folder, we will be creating a list of files to run our configurations. First, we will create an inventory file. This file must contain the host IP address/Domain name of our target/remote servers. By default, the ansible management tool (which we installed earlier) provides us with a file where we can save our hosts inventory and that file is located in "/etc/ansible/hosts" (Note: NOT the 'ansible' folder that we just created;the one we are currently in). So now, we will enter into this "/etc/ansible/hosts" and save our target servers' IP adresses      

      sudo nano /etc/ansible/hosts

Then paste this code in our host file 

        [webserver]
        44.211.153.206
        18.233.154.138
        54.161.207.64

        [centos]
        linux ansible_host=54.160.184.6 ansible_user=ec2-user

        [parent]
        44.211.153.206
        18.233.154.138
        54.161.207.64
        linux ansible_host=54.160.184.6 ansible_user=ec2-user
      
In this inventory, we have 3 groups. The **'webserver'** group comprises 3 ubuntu servers, the **'centos'** group has just 1 linux server, and the  **'parent'** group servers as the "group of groups" (webserver and centos).

This is how it should look like;

![ansible](./grp_images/Screenshot%20(259).png)

Press 'Ctrl O', ENTER, and then 'Ctrl X'. 'Ctrl O' means 'Write Out' this code. Press ENTER to confirm. Then 'Ctrl X' to exit.

<br>

Next, we will create a file where our SSH key pair will be saved. Secure Shell (SSH) allows a secure connection between our control machine and target machines with the use of a key pair. Now in this 'ansible' directory, a new file will be created and named "Myapp-key-pair.pem" then we will paste in an SSH key. Although, while creating your instances on AWS, you'll be prompted to enter an existing key-pair(login) or create a new one. Note that this key-pair(login) must be downloaded/saved on your computer, as it is what you'll use to connect to your target servers via SSH.

      sudo nano Myapp-key-pair.pem

We will paste in our key-pair(login) file like this;

![keypair](./grp_images/Screenshot%20(185).png)

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

<br>

After saving this file, we must confirm that our hosts are accessible from this workstation. So we shall run this command to ping our hosts,

      ansible all -m ping

And our ping output is successful!

![ansible](./grp_images/Screenshot%20(249).png)
![ansible](./grp_images/Screenshot%20(250).png)

Then after our host inventory and key pair, we shall create another file named **'ansible.cfg'**. In this file, we will identify the path that ansible will locate our host inventory file and ssh key pair file. 

      sudo nano ansible.cfg

Then paste this in the file;

      [defaults]
      inventory = /etc/ansible/hosts
      private_key_file = /home/workstation/ansible/Myapp-key-pair.pem            

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

<br>

Lastly, we will create our ansible playbook file. This file consists of the tasks we want to configure to our target servers. In this playbook, we will be writing some tasks to 
* Install Apache application on "centos" group.
* Install NGINX application on "webserver" group.
* Ensure that the Apache and NGINx services are started on both groups and
* Update both groups (webserver and centos) in the parent group

Note: Apache on CentOS distribution is "httpd".

      sudo nano both.yml

Then paste the following code in the file;

    - hosts: webserver
      become: true
      tasks:
      - name: install nginx on webserver
        apt:
          name: nginx
          state: latest
         update_cache: yes
      - name: ensure nginx is started
        service:
         name: nginx
          state: started

    - hosts: centos
      become: true
      tasks:
      - name: install apache(httpd) on centos
        yum:
          name: httpd
         state: latest
         update_cache: yes

      - name: ensure httpd service is started
        ansible.builtin.service:
          name: httpd
          state: started

    - hosts: parent
      become: true
      tasks:
      - name: update webserver
        raw: sudo apt update
        when: ansible_distribution == 'Ubuntu'
        when: inventory_hostname in groups["webserver"]
 
      - name: update centos
        raw: sudo yum update
        when: ansible_os_family == "ec2-user"
        when: inventory_hostname in groups["centos"]     

At the end, this is what your playbook should look like;        

![playbook](./grp_images/Screenshot%20(254).png)
![playbook](./grp_images/Screenshot%20(255).png)

## [Step 6] Run the Ansible Configurations
Having created these files and the hosts file at /etc/ansible/hosts,

![ansible](./grp_images/Screenshot%20(258).png)

 we will run our ansible playbook like this;
 
      sudo ansible-playbook -i /etc/ansible/hosts playbook.yml

* "-i" flag indentifies the inventory file with the path '/etc/ansible/hosts'
* "ansible-playbook" serves as the command that runs our playbook file 'playbook.yml'
 
And this is our output,

![playbook](./grp_images/Screenshot%20(251).png)
![playbook](./grp_images/Screenshot%20(252).png)
![playbook](./grp_images/Screenshot%20(253).png)

<br>

Now let's check our target servers and see if either apache or nginx is installed and started in them. 

![targets](./grp_images/Screenshot%20(256).png)
![targets](./grp_images/Screenshot%20(266).png)
![targets](./grp_images/Screenshot%20(267).png)

![targets](./grp_images/Screenshot%20(257).png)

<br>

### In Conclusion,
We have been able to create ansible groups and group of groups. Also we were able to successfully configure ansible tasks to these groups.

Watch out for the next series on 'Ansible Roles'. Can't wait to share that with you. Thanks for reading, it means alot to me.  