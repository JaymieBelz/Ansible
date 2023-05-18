## **How to Install Jenkins; Start its Service and Open a Firewal - Using Ansible**


In this section, we shall be configuring a couple of stuff to a remote server. Which is Jenkins and Firewall Setting.
Jenkins is a CI/CD tools that DevOps engineers use in their constant iteration processes. While Firewall pertains to operation security.

<br>

## Install Ansible on Control Machine
In this machine (master-ubuntu instance launched on AWS), we shall install ansible. The ansible package repository first before installation.

      sudo apt-get-repository ppa:ansible/ansible

      sudo apt update           

      sudo apt upgrade -y

Then install ansible

      sudo apt install ansible -y

<br>

## Write Ansible Role to Install Jenkins
First, we shall create a new directory called "jenkins"

        mkdir jenkins
        cd jenkins

Now we are in the "jenkins" directory. Next step is to create a sub-directory "roles" and then import ansible-galaxy into another directory - "jenkins_install" under "roles" .        

        mkdir roles
        cd roles
        sudo ansible-galaxy init jenkins_install

![jenkins](./roles/images/Screenshot%20(365).png)

Still in the "roles" directory. Run;

        ls
        cd jenkins_install
        
Here in the "jenkins_install" directory, we see several folders and file ansible-galaxy imported. We shall now delete items that we don't need here. Simply run;

        sudo rm -r defaults files handlers meta templates vars tests

But leave the "tasks" folder.

<br>

Check what's left in the "jenkins_install" directory, and enter into the "task" folder

        ls
        cd tasks
        ls

![jenkins](./roles/images/Screenshot%20(366).png)

We have a default "main.yml" file there. Now, edit this file by;

        ---
        - name: Add Jenkins repository key
          apt_key:
            url: https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
            state: present

        - name: Add Jenkins repository
          apt_repository:
            repo: deb https://pkg.jenkins.io/debian-stable binary/
            state: present

        - name: Install Java
          apt:
            name: openjdk-11-jdk
            state: latest

        - name: update apt cache
          apt:
            update_cache: yes

        - name: Install Jenkins
          apt:
            name: jenkins
            state: latest
            update_cache: yes

        - name: Start Jenkins service
          service:
            name: jenkins
            enabled: yes
            state: started

        - name: open port 8080 in firewall
          ufw:
            rule: allow
            port: 8080
            state: enabled

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

<br>

We need to return to this path "~/jenkins/roles/" and create our playbook file, ansible.cfg file and key pair(login) file. But before that, let us edit our host inventory file in a default path ansible installation package produced.

        sudo nano /etc/ansible/hosts

        (then paste this)
        ubuntu ansible_host=54.174.173.107

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

Now let us return to the "~/jenkins/roles/" path and continue.

Create ansible.cfg and Myapp-key-pair.pem files

        sudo nano Myapp-key-pair

Then paste your SSH key-pair(login) like this;

![ssh](./roles/images/Screenshot%20(185).png)

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

<br>

        sudo nano ansible.cfg

        (then paste this)
        [defaults]
        inventory = /etc/ansible/hosts
        remote_user = ubuntu
        private_key_file = /home/ubuntu/jenkins/roles/Myapp-key-pair.pem

This is what it should contain;   

![inventory](./roles/images/Screenshot%20(369).png)            

<br>

Next, let us create our playbook.

        sudo nano jenks-play.yml

        (then paste this)
        ---
        - name: install jenkins and run ubuntu container
          hosts: all
          become: true
          roles:
            - jenkins_install

![jenks-play](./roles/images/Screenshot%20(370).png)

<br> 

We can now run ansible playbook from here

        sudo ansible-playbook -i /etc/ansible/hosts jenks-play.yml

![jenks-play](./roles/images/Screenshot%20(383).png)
![jenks-play](./roles/images/Screenshot%20(384).png)

And there it is! We have just installed Jenkins, started its service and opened a port 8080 firewall on it.

