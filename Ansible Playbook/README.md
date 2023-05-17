## **How to Configure Ansible to Servers of Different Operating Systems (Ubuntu and CentOs)**

<br>

Welcome to this first series of Ansible. In this blog, I will be showing you how to configure ansible to run a number of tasks on different Operating Systems - systems which serve as interface between the user and the hardware, and manages computing resources. Examples iclude Windows, Linux, MacOS, and so on - at the same time. Intriguing, right? Yes, I guess you might be only used to configuring ansible tasks to different machines of one particular Operating System, prior to the time you are seeing this post. So worry not! Because from this article, you will not go down and out without learning How to CONFIGURE Ansible on Different Machines of Different Operating Systems.
#### But you may ask, what is Ansible that this dude is talking about? 

<br>

## What is Ansible?
Ansible is a management tool that helps you control and handle configurations or architectures across data infrastructures. Let me give you a tiny illustration of how ansible works. Say you want to send an impromptu message to every member in your volleyball team of 200 and you cannot withstand the stress of having to type same message from your cell phone to every member. To save yourself time and energy, you could simply [1] create a list and add the contact of every member on your team, [2] write the message you want to convey as a text, and [3] send to everyone on your team as Broadcast message. Easy-peasy! So, as a cloud engineer, you need to understand how to connect and seamlessly configure various infrastructures. Now let us get our hands dirty.

<br>

## Prerequisites
* Ceaseless Internet Connection
* AWS Account
* A running VM that will serve as our workstation
* Two Target Servers (Ubuntu OS and CentOS)

<br>

## [Step 1] Spin Up Instances/Machines on AWS
We shall start with logging into the AWS Account. In case you are yet to create yours, you can click here to create a free tier account on AWS.
First, navigate to EC2 on AWS console and create an instance. You can name it whatever you want but on this show, let's name it **'ubuntu-master'** as it is going to be our control machine. Follow these steps as shwon in the screenshots below,
<!-- <br> -->
![aws](./images/Screenshot%20(207).png)

![aws](./images/Screenshot%20(208).png)

Select Ubuntu OS image from the options and the free tier volume type.

<br>

![aws](./images/Screenshot%20(209).png)

Next, create a key pair(login) which allows secure connection between your control machine and target machine(s). You should generate a new one(by clicking 'create new key pair') and download/save a copy of it on your computer as you will be needing it later. But in this case, we will be using an existing key pair that has been created on AWS before.

<br>

![aws](./images/Screenshot%20(210).png)

On to the Network settings. Apparently, the whole internet is a Big wide area of network that connects all servers together.
So in this section, we will create a new VPC (although AWS has provided us with a default one but No, we are not using that). A virtual private cloud (VPC) is a secure and isolated private cloud hosted within a public cloud. It is virtual network dedicated to your AWS account where you can specify an IP address range, add subnets, gateways and associate to it security groups.
<br>

Click on 'create a new vpc'. A new tab is opened then name it **'vpc-ansible'** and create an IPv4 CIDR block: **'10.0.0.0/16'** network IP. The '/16' Netmask provides us with about 65,536 available hosts. That's a lot, right? But don't worry how I got that. It is beyond the scope of this article.
<br>

![aws](./images/Screenshot%20(219).png)

Having done that, click 'create VPC'. Then you'll be returned to the instance creation tab. There, you may have to refresh by clicking a small refresh icon by the side of vpc dropdown box and select the **'vpc-ansible'**.
<br>

![aws](./images/Screenshot%20(247).png)

<br>

After VPC is the subnet setting; 'create a new subnet'. A subnet is a range of IP addresses in your VPC. We will name it **'subnet-ansible'** and choose **'US East(N.Virginia)'**.
<br>

![aws](./images/Screenshot%20(211).png)

<br>

![aws](./images/Screenshot%20(212).png)

We'll also associate an IPv4 CIDR(Classless Inter-Domain Routing) block to our subnet - same one from VPC.
<br>

![aws](./images/Screenshot%20(213).png)

Then click on **'Create subnet'**   

<br>

![aws](./images/Screenshot%20(214).png)

Back to our network settings. You may click on the small refresh icon beside 'create new subnet' then on the dropdown to select your just created subnet.

<br>

![aws](./images/Screenshot%20(215).png)

Next is Firewall(security group). Just as the name sounds, firewall - a wall on fire will always prevents an external force from breaking in. Creating a Firewall(security group) in this sense will determine which traffic is allowed into our website in order to ensure security. We'll tick all the 3 boxes for the purpose of case study.
<br>

![aws](./images/Screenshot%20(216).png)

Now, we can create our 'ubuntu-master' instance by clicking "Launch instance".

![aws](./images/Screenshot%20(234).png)

<br>

## [Step 2] Set up Ubuntu and CentOS Target Servers on AWS
Ubuntu and CentOS are different distributions of the Linux Operating Systems. They have different mapplication package management. An excerpt from www.educba.com identifies CentOS as a linux based framework and distribution forked from Red Hat Enterprise Linux while Ubuntu is an open-sourced Linux Distribution based on the architecture of Debian.

Now, we will repeat the previous step and create another Ubuntu OS instance which will server as one of our target machines. Don't forget, our goal is to configure 2 different OS (Ubuntu and CentOS) using ansible. So we will name this instance **'ubuntu'** then select the same key pair, VPC, subnet, and Firewall(security groups) as that of ubuntu-master.  Then 'Launch instance'

![targets](./images/Screenshot%20(217).png)

<br>

Then the last instance to be created is **'CentOS'**. Select same parameters as we did earlier then click on 'Launch instance'.

![targets](./images/Screenshot%20(218).png)

Note: We are using the Amazon Linus image because it features a high level of compatibility with CentOS7.

<br>

### Finally, this is the overview of how our instances look like. Up and Running!
![aws](./images/Screenshot%20(221).png)

<br>

## [Step 3] Create a User (Workstation) on Master-Ubuntu
Recall that we created 3 instances on AWS; master-ubuntu machine + ubuntu and centOS target machines. Before we proceed to writing our ansible configurations, let's quickly note the public IP addresses of our target serversby select each of them to get their public IP. 

![aws](./ansible_images/Screenshot%20(221).png)

Now, Let's begin. Select the **'master-ubuntu'** instance then click on 'connect' at the top of the AWS interface. AWS provides us with a console to help connect to our machine. It is from this master server that we will be configuring our ansible configurations to the target servers. In other words, we will create a workstation user, and grant it sudo privilege, on the 'master-ubuntu'.

First, after being connected to the master-ubuntu, switch to root user by typing on your terminal;

      Run: sudo su

      Run: adduser workstation

      Run: usermod -G sudo workstation 
      (This grants our new user, workstation, a sudo privilege)

![user](./ansible_images/Screenshot%20(181).png)


Next, we will switch to the new user (workstation) from master-ubuntu root user. 

      Run: su - workstation

![user](./ansible_images/Screenshot%20(182).png)

## [Step 4] Install Ansible on Workstation
First, we must install the ansible package repository.
      Run: sudo apt-get-repository ppa:ansible/ansible

      Run: sudo apt update 

![ansible](./ansible_images/Screenshot%20(192).png)

      sudo apt upgrade -y

![ansible](./ansible_images/Screenshot%20(193).png)

Then install ansible

      Run: sudo apt install ansible -y

![ansible](./ansible_images/Screenshot%20(194).png)

<br>

## [Step 5] Write Ansible Configurations
Create a folder in the workstation (using the linux 'mkdir' command). We will name this folder 'ansible' by typing on your terminal;

      mkdir ansible 

Then enter into this new folder by typing;

      cd ansible

Now that we are in ansible folder, we will be creating a number of files to run our configurations. First, we will create an inventory file. This file usually contains the host IP address/Domain name of our target/remote servers. By default, the ansible management tool (which we installed earlier) has provided us with a file where we can save our hosts inventory and that file is located in "/etc/ansible/hosts" (Note: NOT the 'ansible' folder that we just created;the one we are currently in). So now, we will enter into this "/etc/ansible/hosts" and save our target servers' IP adresses (ubuntu and centOS from Step 3).

      sudo nano /etc/ansible/hosts

Then paste this code in our host file 

        [ubuntu_server] ubuntu ansible_host=44.211.153.206 ansible_user=ubuntu
        [linux_server] linux ansible_host=54.160.184.6 ansible_user=ec2-user

Having pasted that, this is how it should look like;

![ansible/hosts](./ansible_images/Screenshot%20(265).png)

Press 'Ctrl O', ENTER, and then 'Ctrl X'. 'Ctrl O' means 'Write Out' this code. Press ENTER to confirm. Then 'Ctrl X' to exit.

<br>

Next, we will create a file where our SSH key pair will be saved. Secure Shell (SSH) allows a secure connection between our control machine and target machines with the use of a key pair. Recall that, in step 1, when we were creating our instances, they all had one particular key-pair(login) in common - "Myapp-key-pair.pem". Now in this 'ansible' directory, a new file will be created and named with respect to this key-pair and we will paste in the key(content) that was downloaded from AWS.

      sudo nano Myapp-key-pair.pem

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

![keypair](./ansible_images/Screenshot%20(227).png)

<br>

After saving this file, we need to confirm that our hosts are accessible from this workstation. So we will run this command to ping our hosts,

      ansible all -m ping

And our ping output is successful!

![ping](./ansible_images/Screenshot%20(231).png)

<br>

Then after our host inventory and key pair, we shall create another file named **'ansible.cfg'**. In this file, we will identify the path that ansible will locate our host inventory file and ssh key pair file. 

      sudo nano ansible.cfg

Then paste this in the file;

      [defaults]
      inventory = /etc/ansible/hosts
      private_key_file = /home/workstation/ansible/Myapp-key-pair.pem            

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

<br>

Lastly, we will create our ansible playbook file. This file consists of the tasks we want to configure to our target servers. In this playbook, we will be writing some tasks to 
* Install Apache application on both our Ubuntu and CentOS servers. 
* Ensure that the Apache service is started on both servers and 
* Copy a default html page to be displayed on both target servers' IP adresses. 

base off the hosts' IP addresses and key pair file we have provided. Note: Apache on ubuntu OS is "apache2" while it is "httpd" on CentOS distribution.

      sudo nano both.yml

Then paste the following code in the file;

      - hosts: ubuntu_server:linux_server
        become: true
        tasks:
        - name: install apache on both ubuntu and centos
          package:
            name:
             - "{{ 'apache2' if ansible_distribution == 'Ubuntu' else 'httpd' if ansible_distribution == 'ec2-user' }}"
            state: present
            update_cache: yes

      - hosts: ubuntu_server
        tasks:
       - name: start apache service on ubuntu
         service:
           name: apache2
           state: started

      - hosts: linux_server
       tasks:
       - name: start httpd service on centos
         ansible.builtin.service:
           name: httpd
           state: started

      - hosts: ubuntu_server:linux_server
        become: true
       tasks:
       - name: copy a default html page
         copy:
            content: 
                <html>
                <head>
                <title>Welcome to My Website</title>
                </head>
                <body>
                <h1>Hello from Ansible!</h1>
                </body>
                </html>
            dest: /var/www/html/index.html

At the end, this is what your playbook should look like;

![playbook](./ansible_images/Screenshot%20(263).png)
![playbook](./ansible_images/Screenshot%20(264).png)

<br>

And below is our 'ansible' directory with the list of files it contains,

![ansible](./ansible_images/Screenshot%20(230).png)

<br>

## [Step 6] Run the Ansible Configurations
Having created these 4 files, we will run our ansible playbook. Like this;

      sudo ansible-playbook -i /etc/ansible/hosts both.yml

* "-i" flag indentifies the inventory file with the path '/etc/ansible/hosts'
* "ansible-playbook" serves as the command that runs our playbook file 'both.yml' 

![playbook](./ansible_images/Screenshot%20(222).png)
![playbook](./ansible_images/Screenshot%20(223).png)

<br>

Now, let's go and check on our target servers and see if our configuration works fine. Obviously, it will.

![targets](./ansible_images/Screenshot%20(257).png)
![centOS.target](./ansible_images/Screenshot%20(262).png)
![targets](./ansible_images/Screenshot%20(261).png)

And there it is! We have just configured 2 servers with different OS distribution to install apache application, get the app started and display a simple html page for them.

<br>

Thank you for reading this article. Please, like and subscribe to this page to get notified when new updates drop. Stay Jiggy!