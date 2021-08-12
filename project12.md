<p>In this project you will continue working with ansible-config-mgt repository and make some improvements of your code. Now you need to refactor your Ansible code, create assignments, and learn how to use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook – it allows you to organize your tasks and reuse them when needed.</p1>

<h2>Step 1 – Jenkins job enhancement</h2>

<h3>Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change.</h3>

<p>Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each build.</p>

    sudo mkdir /home/ubuntu/ansible-config-artifact
    
<p>Change permissions to this directory, so Jenkins could save files there:</p>

    chmod -R 0777 /home/ubuntu/ansible-config-artifact
    
<p>Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins</p>

![1 c](https://user-images.githubusercontent.com/10243139/129186304-6e4e01fe-fbd0-4599-b7ff-0133b2ec4814.jpg)

<p>Create a new Freestyle project (you have done it in Project 9) and name it save_artifacts</p>

<p>This project will be triggered by completion of your existing ansible project. Configure it accordingly:</p>

![1 e](https://user-images.githubusercontent.com/10243139/129186419-c3800121-e7c4-4d29-9549-5b17842795ad.png)

<p>The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.</p>

![1 f](https://user-images.githubusercontent.com/10243139/129186482-f226f974-a9cf-4705-8058-4f69ac4cc217.png)

<p>Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside master branch).</p>

![1 g](https://user-images.githubusercontent.com/10243139/129186536-7622e0f4-f713-42c0-af71-df418e39476b.jpg)

<h2>Step 2 – Refactor Ansible code by importing other playbooks into site.yml</h2>

<h3>Most Ansible users learn the one-file approach first. However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

Let see code re-use in action by importing other playbooks.</h3>

<p>Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point into the entire infrastructure configuration</p>

<p>Create a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work.</p>

<p>Move common.yml file into the newly created static-assignments folder</p>

<p>Inside site.yml file, import common.yml playbook:</p>

    ---
    - hosts: all
    - import_playbook: ../static-assignments/common.yml

<p>The code above uses built in import_playbook Ansible module.</p>

<p>Your folder structure should look like this:</p>

    ├── static-assignments
    │   └── common.yml
    ├── inventory
        └── dev
        └── stage
        └── uat
        └── prod
    └── playbooks
        └── site.yml
        
<p>Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under static-assignments and name it common-del.yml. In this playbook, configure deletion of wireshark utility.</p>

    ---
    - name: update web and nfs servers
      hosts: webservers, nfs
      remote_user: ec2-user
      become: yes
      become_user: root
      tasks:
      - name: delete wireshark
        yum:
          name: wireshark
          state: removed

    - name: update LB and db server
      hosts: lb, db
      remote_user: ubuntu
      become: yes
      become_user: root
      tasks:
      - name: delete wireshark
        apt:
          name: wireshark-qt
          state: absent
          autoremove: yes
          purge: yes
          autoclean: yes

<p>update site.yml with - import_playbook: ../static-assignments/common-del.yml instead of common.yml</p>

<p>Run ansible-playbook command against the dev environment</p>

    sudo ansible-playbook -i /home/ubuntu/ansible-config-artifact/inventory/dev.yml /home/ubuntu/ansible-config-artifact/playbooks/site.yml
    
![2 f](https://user-images.githubusercontent.com/10243139/129188350-ed40ffd6-ef9b-48ed-9454-463884478f70.jpg)

<p>Make sure that wireshark is deleted on all the servers by running wireshark --version</p>

![2 g](https://user-images.githubusercontent.com/10243139/129188431-0565376e-e9fa-4861-a061-95b9881c4200.jpg)

<h2>Step 3 – Configure UAT Webservers with a role ‘Webserver’</h2>

<h3>We have our nice and clean dev environment, so let us put it aside and configure 2 new Web Servers as uat. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.</h3>

<p>Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.</p>

<p>To create a role, you must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.</p>

<p>Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)</p>

    mkdir roles
    cd roles
    ansible-galaxy init webserver
    
![3 b](https://user-images.githubusercontent.com/10243139/129189422-5ae0e8d3-7603-4f8e-ba8b-0d3f943ff91c.jpg)


<p>Update your inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of your 2 UAT Web servers</p>

    [uat-webservers]
    <Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
    <Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>

<p>In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory, so Ansible could know where to find configured roles.</p>

    roles_path    = /home/ubuntu/ansible-config-mgt/roles

![3 d](https://user-images.githubusercontent.com/10243139/129189485-59b93a07-f2c6-4ed2-9ecb-f7efca96e95e.jpg)


