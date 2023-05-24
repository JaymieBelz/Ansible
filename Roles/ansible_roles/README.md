## **Ansible Roles to Install Apache Server and Copy a Simple HTML Page**

<br>

Welcome to another series of Ansible. In this series, I will be sharing with you how to write ansible roles to install apache and copy a simple Html page to a target server.

Ansible Role is a primary mechanism for breaking/splitting an ansible playbook into multiple files.

<br>

## Create 2 Instances on AWS page
Name one "ubu" ( which will serve as master/control machine) and the other "ubuntu". Now connect to the "ubu" instance.

## Install Ansible on Control Machine
On the "ubu" machine (instance launched on AWS), we shall install ansible. The ansible package repository first before installation.

      sudo apt-get-repository ppa:ansible/ansible

      sudo apt update           

      sudo apt upgrade -y

Then install ansible

      sudo apt install ansible -y

<br>

## Write Ansible Roles To Install Apache
Having installed ansible and its PPA package manager, on the control machine, create a new directory (using linux "mkdir" command)

        mkdir ansible-roles
        cd ansible-roles

        mkdir ranjay
        cd ranjay

Import ansible-galaxy resource into a new folder under "ranjay" directory where we will write our installation configurations.

        sudo ansible-galaxy init apache_install
        cd apache_install
        ls

Here in the "apache_install" folder, we see several folders and file ansible-galaxy imported. We shall now delete items that we don't need here. Simply run;

        sudo rm -r defaults files README.md meta templates vars tests

        ls

Then you see "handlers" and "tasks" folders  left

![apache-inst](./ranjay/images/Screenshot%20(298).png)

Now run;

        ls
        cd tasks
        ls

![tasks](./ranjay/images/Screenshot%20(302).png)

There's also a "main.yml" file in here (/apache_install/tasks/)

        sudo nano main.yml

        (then paste this)
        ---
        - name: Install Apache
          apt:
            name: apache2
            state: present
          notify: Restart Apache

[This play helps install the apache2 application]
Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.     

![tasks](./ranjay/images/Screenshot%20(303).png)

<br>

Next, go a step back to the "apache_install" directory by typing

        cd ..

Then run;

        cd hanlers
        ls
        sudo nano main.yml

![handlers](./ranjay/images/Screenshot%20(299).png)  

In the "main.yml", paste this;

        ---
        - name: Restart Apache
          service:
            name: apache2
            state: restarted

![main.yml](./ranjay/images/Screenshot%20(301).png)

[This play helps install the apache2 application]
Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

<br>

## Copy a Simple Html Page the the Installed Apache
Now go back to the "~/ansble-roles/ranjay" and create another folder which will be called apache_copy

        cd ~/ansble-roles/ranjay
        mkdir ansible_copy
        cd ansible_copy

![copy](./ranjay/images/Screenshot%20(291).png)      

Then create a sample html file here

        sudo nano default.html

        (then paste)
        <!DOCTYPE html>
        <html>
        <head>
          <title>Workstation</title>
        </head>
        <body>
          <h1>Welcome to My HTML Page</h1>
          <p>This is a sample HTML file.</p>

        </body>
        </html>

![default](./ranjay/images/Screenshot%20(293).png)

Still in the "~/ansible-roles/ranjay/apache_copy" directory. Create a "tasks" file

        mkdir tasks
        cd tasks

![tasks](./ranjay/images/Screenshot%20(294).png)

Now create a .yml file

        sudo nano main.yml

        (then paste this)
        ---
        - name: Copy default HTML page
          copy:
            src: /home/ubuntu/ansible-roles/ranjay/apache_copy/default.html
            dest: /var/www/html/index.html

![copy-main](./ranjay/images/Screenshot%20(307).png)            

<br>

When we now go back into "~/ansble-roles/ranjay" directory, we will see both "apache_install" "apache_copy" folders.

        cd "~/ansble-roles/ranjay"
        ls

        (then create our playbook here)
        sudo nano playbook.yml

![ranjay](./ranjay/images/Screenshot%20(308).png)   

        (then paste into playbook.yml)
        ---
        - name: Install and configure Apache
          hosts: all
          become: true
          roles:
            - apache_install
            - apache_copy

![playbook](./ranjay/images/Screenshot%20(320).png)

<br>

## Configure Host Inventory
Right in this directory "~/ansble-roles/ranjay", we can edit our inventory file (at /etc/ansible/hosts) which by default ansible pcakage has provided from when we ran ansible installion. Now run;

        sudo nano /etc/ansible/hosts

        (then paste this)
        18.212.252.162

![hosts](./ranjay/images/Screenshot%20(312).png)  

<br>

## Write SSH key pair file
SSH Key pair ensures there's a secure shell connection between our infrastructures (control and target machines). Note that when creating instances on AWS, you will be required to create/choose an existing a key pair (which must be downloaded to your computer after creation for future use) before you can launch an instance. And for the purpose this practice, I used the same key pair(login) - named "Myapp-key-pair" on AWS - to launch 2 instances. This key pair, I have downloaded and saved somewhere in my Private Computer because I will be needing it at this very stage. 

        sudo nano Myapp-key-pair.pem

Then paste your own SSH key-pair(login) like I just did;

![ssh](./ranjay/images/Screenshot%20(185).png)

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.        

<br>

## Write Ansible.cfg file
It is from this file that ansible will locate where our host inventory and key pair(login) files are. Don't forget that we are still in the "~/ansble-roles/ranjay" directory. 

        sudo nano ansible.cfg

        (then paste this below)
        [defaults]
        inventory = /etc/ansible/hosts
        remote_user = ubuntu
        private_key_file = /home/ubuntu/ansible-roles/ranjay/Myapp-key-pair.pem

![ansible.cfg](./ranjay/images/Screenshot%20(318).png)  

<br>

Now let's ping to the host and check if the conncetion is a success.

![ping](./ranjay/images/Screenshot%20(319).png)

<br>

## Run Ansible Playbook to Run Roles

        ansible-playbook -i /etc/ansible/hosts playbook.yml

![play](./ranjay/images/Screenshot%20(321).png)             

<br>

## Switch to the Target Server and View 
Now that ansible playbook has successfully configured our roles, we can now check what our target server looks like.

Connect to the "ubuntu" instance on AWS via the EC2 Console option. And this is our target server;

![target](./ranjay/images/Screenshot%20(322).png)

<br>

Ensure that apacahe is installed and currently running/active by typing;

        sudo systemctl status apache2

![apache2](./ranjay/images/Screenshot%20(323).png)   

<br>

Finally, we shall view the Public IP address of our target server to see the sample HTML Page we copied.

![htmlpage](./ranjay/images/Screenshot%20(324).png)

<br>

## In Conclusion
We have successfully installed apache, ensured apache service is started and copied a sample html page. By now, you should be able to configure ansible roles to any remote server(s) of your choosing. 

Kindly rate this article by liking and expect to see more of our Ansible series on this space. Thank you.
        



          
