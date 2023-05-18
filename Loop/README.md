## **Creating Users with Ansible Loop**

<br>

In this section, we will be creating users on a target machine by simply running a few ansible tasks. On linux systems, we create users and groups in order to be more precise when controlling and managing access to files and resources. It,s not always advisable to use the OS or root users so as to avoid tampering with some certain structures of the system.

<br>

In this module, we will create 4 different users on a target server[54.174.173.107] by running the anslible loop module  

On our master-ubuntu; in the workstation user, create a new directory and name it 'user-creation'

        mkdir user-creation
        cd user-creation

Create a new folder, roles, in this folder. Enter into this folder and there we will initialize the ansiblily galaxy 

        mkdir roles
        cd roles

        sudo ansible-galaxy init create-users

![user](./images/Screenshot%20(372).png)  

This successfully creates the roles for us with some ready-made directories. Next we will see what's in the tasks directory in the create-user role that was just created. 

        cd create-users
        ls
        cd tasks
        ls
        sudo nano main.yml

Then input this in the 'main.yml' file

        ---
        - name: Create users
          user:
          name: "{{ item }}"
          state: present
          loop:
            - james
            - amaka
            - joy
            - cloudtopg 

![user](./images/Screenshot%20(380).png)

Now, go back into the roles directory, let's create our playbook.

         cd ~/user-creation/roles/
         sudo nano user-play.yml

Then paste this into the playbook file;

        ---
        - name: create users
          hosts: all
          become: true
          roles:
            - create-users

![user](./images/Screenshot%20(378).png)

In this same '~/user-creation/roles/' directory, we will create our ansible.cfg and Myapp-key-pair.pem files

        sudo nano ansible.cfg

This is what it should contain;

![user](./images/Screenshot%20(376).png)


Now, before running our playbook, let us configure our inventory file as refrenced in the 'ansible.cfg' file.

        sudo nano /etc/ansible/hosts

Then paste this

        ubuntu ansible_host=54.174.173.107

Now let's run our playbook.

        sudo ansible -i /etc/ansible/hosts user-play.yml

![user](./images/Screenshot%20(379).png)

<br>

Gladly, we have successfully created 4 users on our target machine using ansible loop. Cheers! We did it.

      


