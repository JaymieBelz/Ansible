## **How to Install Terraform on a Server Using Ansible**

<br>

Holla! Welcome to this new series of Ansible Roles. In this section, we shall install Terraform on a target server using ansible roles. Terraform is a type of Infrastructure as a Code (IaaC) tool, like Ansible configuration tool. As an IaaC tool, it is agentless. It is an open surce tool used to define and provision a datacenter infrastructure using a high level configuration language, known as Hashicorp Configuration Language (HCL), written in golang. 

<br>

### Prerequisites
* A target/remote Ubuntu server, and 
* A control Ubuntu machine (master-ubuntu) 

<br>

## Install Ansible on Control Machine
In this machine (master-ubuntu instance launched on AWS), we shall install ansible. The ansible package repository first before installation.

      sudo apt-get-repository ppa:ansible/ansible

      sudo apt update           

      sudo apt upgrade -y

Then install ansible

      sudo apt install ansible -y

<br>

## Write Ansible Role to Install Terraform.
First, we will create a user on this machine (master-ubuntu) and grant it sudo access.

        adduser workstation
        usermod -G sudo workstation

        
        (Then switch to this new user)
        su - workstation

Now, in the workstation user, create a new directory and name it 'terraform_role'

        mkdir terraform_role
        cd terraform_role

Create a new folder, roles, in this folder. Enter into this folder and there we will initialize the ansiblily galaxy package

        mkdir roles
        cd roles

        sudo ansible-galaxy init terra

This successfully creates the roles for us with some ready-made directories. Next we will see what's in this new directory, where ansible-galaxy is imported, "terra". Then make use of the tasks sub-folder in "terra". 

        cd terra
        ls
        cd tasks

In this tasks subfolder, there is a default "main.yml" file. We shall edit this file by running; 

        ls
        sudo nano main.yml

Then input this in the 'main.yml' file

        ---
        - name: Install unzip
          apt:
            name: unzip
            state: present

        - name: Download Terraform binary
          get_url:
            url: "https://releases.hashicorp.com/terraform/0.15.5/terraform_0.15.5_linux_amd64.zip"
            dest: "/tmp/terraform.zip"
            mode: '0755'

        - name: Unzip Terraform binary
          unarchive:
            src: "/tmp/terraform.zip"
            dest: "/usr/local/bin"
            remote_src: yes
            mode: '0755'

        - name: Add Terraform to PATH
          lineinfile:
            path: "~/.bashrc"
            line: 'export PATH=$PATH:/usr/local/bin'
            create: yes
            state: present
          become: true
          become_user: "{{ ansible_user }}"

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

<br>

Next, return to the ~/terraform_role/roles directory. There we shall create an "ansible.cfg" file, key pair file and ansible playbook - "play.yml". 
<br>

But before that, let us quickly edit our host inventory file:

        sudo nano /etc/ansible/hosts

        (Then paste this. Note: you should edit the ip address to yours) 
        ubuntu ansbile_hosts=54.174.173.107 ansible_user=ubuntu

![hosts](./roles/images/Screenshot%20(340).png)

<br>

Now back to the where we were

        cd ~/terraform_role/roles
        sudo nano ansible.cfg

The paste in this code;

        [defaults]
        inventory = /etc/ansible/hosts
        remote_user = ubuntu
        private_key_file = /home/ubuntu/terraform-role/roles/Myapp-key-pair.pem

![ansiblecfg](./roles/images/Screenshot%20(342).png)

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

This 'ansible.cfg' file helps ansible locate where the inventory file where the host iP address is (we will write that shortly), which remote/target user we are working on and where the key pair(login) file is.

<br>

Next is to create our key-pair file

        sudo nano Myapp-key-pair

Then paste your SSH key-pair(login) like this;

![ssh](./roles/images/Screenshot%20(185).png)

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

<br>

Now let us create our playbook

        sudo nano play.yml

Then paste this in the file;

        ---
        - name: install and configure terraform
          hosts: all
          become: true
          roles:
             - terra
![play](./roles/images/Screenshot%20(347).png)

<br>

Now let us ping our hosts and be sure the connection is successful before running ansible playbook.

        ansible all -m ping

![hosts](./roles/images/Screenshot%20(343).png)        

        ansible-playbook -i /etc/ansible/hosts play.yml

And it works!

![play](./roles/images/Screenshot%20(346).png)

<br>

### In Conclusion

When we check our remote server to see whether Terraform has been installed, we shall see that ansible Roles configuration is successful.

Simply run;

        terraform -version

![output](./roles/images/Screenshot%20(348).png)