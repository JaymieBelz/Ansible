## **Docker Installation; Enabling Docker Service and Pulling a Docker Container Image - Using Ansible**

<br>

Docker is a container client tool that makes application shipping to production easy. It works on the container technology, which helps pack together and isolate files to run in a pricate space witin a specified runtime.

In this series we will be installing docker, enable its service and pull a sample ubuntu image to a remote server. Let's go!

<br>

## Install Ansible on Control Machine
In this machine (master-ubuntu instance launched on AWS), we shall install ansible. The ansible package repository first before installation.

      sudo apt-get-repository ppa:ansible/ansible

      sudo apt update           

      sudo apt upgrade -y

Then install ansible

      sudo apt install ansible -y

## Write Ansible Role to Install Docker
First, we shall create a new directory called "docker"

        mkdir docker
        cd docker

Now we are in the "docker" directory. Next step is to create a sub-directory "roles" and then import ansible-galaxy into another directory - "docker_install" under "roles" .        

        mkdir roles
        cd roles
        sudo ansible-galaxy init docker_install

![galaxy](./roles/images/Screenshot%20(350).png)

Still in the "roles" directory. Run;

        ls
        cd docker_install
        ls

Here in the "docker_install" directory, we see several folders and file ansible-galaxy imported. We shall now delete items that we don't need here. Simply run;

        sudo rm -r defaults files handlers meta templates vars tests

But leave the "tasks" folder.

![galaxy](./roles/images/Screenshot%20(352).png)

<br>

Check what's left in the "docker_install" directory, and enter into the "task" folder

        ls
        cd tasks
        ls

We have a default "main.yml" file there. Now, edit this file by;         

        sudo nano main.yml

        (then paste this)
        ---
        - name: Install Docker dependencies
          apt:
            name:
              - apt-transport-https
              - ca-certificates
              - curl
              - software-properties-common
            state: present

        - name: Add Docker GPG key
          apt_key:
            url: https://download.docker.com/linux/ubuntu/gpg
            state: present

        - name: Add Docker repository
          apt_repository:
            repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
            state: present

        - name: Update apt cache
          apt:
            update_cache: yes

        - name: Install Docker
          apt:
            name: docker-ce
            state: present

        - name: Start Docker service
          service:
            name: docker
            state: started

        - name: Pull Ubuntu Docker image
          docker_image:
            name: ubuntu
            source: pull

        - name: Run Ubuntu container
          docker_container:
            name: my_ubuntu_container
            image: ubuntu
            state: started
            detach: yes

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

<br>

We need to return to this path "~/docker/roles/" and create our playbook file, ansible.cfg file and key pair(login) file. But before that, let us edit our host inventory file in a default path ansible installation package produced.

        sudo nano /etc/ansible/hosts

        (then paste this)
        ubuntu ansible_host=54.174.173.107

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

Now let us return to the "~/docker/roles/" path and continue.

Create ansible.cfg and Myapp-key-pair.pem files

        sudo nano Myapp-key-pair

Then paste your SSH key-pair(login) like this;

![ssh](./roles/images/Screenshot%20(185).png)

Press 'Ctrl O', ENTER, and then 'Ctrl X' to save and exit the file.

<br>

        sudo nano ansible.cfg

This is what it should contain;   

![inventory](./roles/images/Screenshot%20(361).png)

<br>

Next, let us create our playbook.

        sudo nano doc-play.yml

        (then paste this)
        ---
        - name: install docker and run ubuntu container
          hosts: all
          become: true
          roles:
            - docker_install

![docplay](./roles/images/Screenshot%20(359).png)  

We can now run ansible playbook from here

        sudo ansible-playbook -i /etc/ansible/hosts doc-play.yml

![playbook](./roles/images/Screenshot%20(362).png)        
![playbook](./roles/images/Screenshot%20(363).png) 
![playbook](./roles/images/Screenshot%20(364).png) 

<br>

And that's it! We have just installed docker and enabled its service and then pull an Ubuntu container image.
























