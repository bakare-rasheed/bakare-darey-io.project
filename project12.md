## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. 
The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

In this project you will continue working with ansible-config-mgt repository and make some improvements of your code. Now you need to refactor your Ansible code, create assignments, and learn how to use the imports functionality. 
Imports allow to effectively re-use previously created playbooks in a new playbook 
– it allows you to organize your tasks and reuse them when needed.


# Step 1 – Jenkins job enhancement
Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job we will require Copy Artifact plugin.

* Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each build.
```
sudo mkdir /home/ubuntu/ansible-config-artifact
sudo chmod -R 0777 /home/ubuntu/ansible-config-artifact
```

* Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

![p12-artifat](https://user-images.githubusercontent.com/114327344/204156874-083f7ef5-e0f6-4a79-b132-034884b4f4a2.PNG)

* Create a new Freestyle project (you have done it in Project 9) and name it *save_artifacts*

* This project will be triggered by completion of your existing ansible project. Configure it accordingly:

![p12-jenkins1](https://user-images.githubusercontent.com/114327344/204157048-b41fd595-efca-4257-b4cc-6606f0dbca3a.PNG)

![p12-jenkins2](https://user-images.githubusercontent.com/114327344/204157055-a9dc8e74-b1a5-48c2-ac50-b7aa2b554862.PNG)

* The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. 
To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.

![p12-mkdir](https://user-images.githubusercontent.com/114327344/204157713-977593b7-41f0-464b-b930-d92917460d27.PNG)

# Step 2 – Refactor Ansible code by importing other playbooks into site.yaml

Before starting to refactor the codes, ensure that you have pulled down the latest code from master (main) branch, and created a new branch, name it refactor.
Let see code re-use in action by importing other playbooks

* Within playbooks folder, create a new file and name it site.yaml – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously. 

* Create a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.

* Move common.yaml file into the newly created static-assignments folder.

![p12-structureFile](https://user-images.githubusercontent.com/114327344/204157863-e2c86412-38b9-47a1-bce7-3671cb77fe81.PNG)


* Inside site.yaml file, import common.yml playbook.

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yaml
```
* Run ansible-playbook command against the dev environment

`ansible-playbook -i inventory/dev.yaml playbooks/site.yaml`

## Had issue trying to use the site.yaml file 

![p12-delete](https://user-images.githubusercontent.com/114327344/204158054-fdc5fbe2-692c-4f4d-9854-b1bffebeac62.PNG)

![p12-reference](https://user-images.githubusercontent.com/114327344/204158079-38902bd9-d665-4899-8d6b-cb315a423298.PNG)

## Solve renaming the yml with yaml , with necessary chmod permission


* Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under static-assignments and name it common-del.yaml. In this playbook, configure deletion of wireshark utility.

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
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
```

* Update site.yaml with - import_playbook: ../static-assignments/common-del.yaml instead of common.yaml and run it against dev servers:

![p12-ping](https://user-images.githubusercontent.com/114327344/204158358-b5f6b972-6b0f-47d6-8a34-839826cb4142.PNG)

![p12-sucees1](https://user-images.githubusercontent.com/114327344/204158372-c9ec7b85-9ba2-41f0-852e-900f375af8b3.PNG)

![p12-sucAnsible](https://user-images.githubusercontent.com/114327344/204158383-ea2944a4-3f41-4519-b596 1b2b98393fde.PNG)

![p12-success](https://user-images.githubusercontent.com/114327344/204158399-54fb23a1-0b65-4a70-86e8-77cc4211d228.PNG)

* To validate the removal of wiresharks from the target hosts

![p12-wireshark](https://user-images.githubusercontent.com/114327344/204158529-9d0809cd-a809-49fe-8ef9-81107893dab3.PNG)

# Step 3 – Configure UAT Webservers with a role ‘Webserver’

* Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.

![p12-sshinstances](https://user-images.githubusercontent.com/114327344/204158646-cb572193-10cd-42fb-a2e3-c00f3e731aa6.PNG)

* To create a role, you must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.

```
sudo mkdir roles
cd roles
sudo ansible-galaxy init webserver
```

* Update your inventory ansible-config-mgt/inventory/uat.yaml file with IP addresses of your 2 UAT Web servers
```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 
```
![p12-inventory](https://user-images.githubusercontent.com/114327344/205046128-6c0be023-cfb3-4d34-88cd-4064adc6890e.PNG)

* In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory roles_path    = /home/ubuntu/ansible-config-mgt/roles, so Ansible could know where to find configured roles

![p12-structureFile](https://user-images.githubusercontent.com/114327344/205046786-406b3972-2538-4fc8-93b3-d2b058e8295b.PNG)

* It is time to start adding some logic to the webserver role. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

Install and configure Apache (httpd service)
Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
Make sure httpd service is started

  
  
* Your main.yml may consist of following tasks:
  
```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/damdev-95/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```
# Step 4 – Reference ‘Webserver’ role
  
* Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. This is where you will reference the role.

```
---
- hosts: uat-webservers
  roles:
     - webserver
```

* There is need to refer the *uat-webservers.yaml* role inside *site.yaml*.
  
```
  ---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yaml

  ```

![p12-reference](https://user-images.githubusercontent.com/114327344/205047536-84d1cacf-2239-4cda-8972-63d69f56417c.PNG)
  
  # Step 5 – Commit & Test

* Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.

Now run the playbook against your uat inventory :

`ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yaml /home/ubuntu/ansible-config-mgt/playbooks/site.yaml`
 
![p12-ping](https://user-images.githubusercontent.com/114327344/205048877-caaeb7b7-a16f-427a-85c0-997d58d0d248.PNG)

![p12-success](https://user-images.githubusercontent.com/114327344/205049130-40342a24-34e6-46bb-9ecd-01f7006ab700.PNG)

![p12-readmeT](https://user-images.githubusercontent.com/114327344/205049654-bdc2f01b-98a5-4545-b212-24c43a6b4c0b.PNG)

# UAT WEBSERVER 1

![p12-wireshark](https://user-images.githubusercontent.com/114327344/205049603-837cbfc7-aed1-4c83-9cc7-c1084ef9f2b0.PNG)

![p12-apache1](https://user-images.githubusercontent.com/114327344/205050672-8938814c-9a73-4892-acac-3dab38be353c.PNG)
