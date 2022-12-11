## ANSIBLE CONFIGURATION MANAGEMENT – PROJECT 7 TO 10 AUTOMATION

- Ansible is an open source, command-line IT automation software application written in Python. It can configure systems, deploy software, and orchestrate advanced workflows to support application deployment, system updates, and more.

- Ansible’s main strengths are simplicity and ease of use. It also has a strong focus on security and reliability, featuring minimal moving parts. 
- It uses OpenSSH for transport (with other transports and pull modes as alternatives), and uses a human-readable language that is designed for getting started quickly without a lot of training.

In Projects 7 to 10 you had to perform a lot of manual operations to set up virtual servers, install and configure required software, deploy your web application.

This Project will make you appreciate DevOps tools even more by making most of the routine tasks automated with Ansible Configuration Management, at the same time you will become confident at writing code using declarative language such as YAML

## Ansible Client as a Jump Server (Bastion Host)
A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provide better security and reduces attack surface.

# INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

* Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
* In your GitHub account create a new repository and name it ansible-config-mgt.
* Instal Ansible

![p11-archy](https://user-images.githubusercontent.com/114327344/203422022-03e1d1a7-7e41-4b27-bcd7-fa5e937a1360.PNG)

```
sudo apt update
sudo apt install ansible
```
![jenkins-ansible](https://user-images.githubusercontent.com/114327344/202836981-86f15b8b-5875-4502-b7fe-44953928e4cc.PNG)

![ansible-version](https://user-images.githubusercontent.com/114327344/202836995-250faffd-f129-4222-9238-b9ad2c08a880.PNG)


* Configure Jenkins build job to save your repository content every time you change it.
* Create a new Freestyle project *ansible* in Jenkins and point it to your ‘ansible-config-mgt’ repository.
* Configure Webhook in GitHub and set webhook to trigger ansible build.
* Configure a Post-build job to save all (**) files, like you did it in Project 9.
* Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

![new-item](https://user-images.githubusercontent.com/114327344/202837165-c1b2bb21-c029-4ad6-ace2-909659bcc8f1.PNG)

![jenkins-built](https://user-images.githubusercontent.com/114327344/202837176-f6f5b2cb-e0ce-4a33-8eb3-ee42561f98f4.PNG)

![project11](https://user-images.githubusercontent.com/114327344/202837243-d18bafc0-480e-419e-8b42-6cb5c3633996.PNG)

![commit](https://user-images.githubusercontent.com/114327344/202837310-20629b30-40ad-41b4-aa01-8f2e17fe3651.PNG)


```
sudo apt update
sudo apt install default-jdk-headless

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
* To connect with ssh from the ansible server to the taget hosts using SSH
```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```

* Copy the pem files for the web,db,nfs and lb servers to the jump server.
* Assign necessary writes to the pem files
```
sudo chomod 400 bakare.pem
sudo chomod recovery.pem
```
* ssh into your Jenkins-Ansible server using ssh-agent

```
ssh -A ubuntu@private-ip
```
* Update your inventory/dev.yml file with this snippet of code

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```

* Update your playbooks/common.yml file with following code
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```
* To execute ansible-playbook command and verify if your playbook actually works:
```
cd ansible-config-mgt
ansible all -m ping -v
ansible-playbook -i inventory/dev.yaml playbooks/common.yaml
```

![p11=cloning](https://user-images.githubusercontent.com/114327344/203419235-af4365bf-a020-48e4-8c79-28b735d4be4f.PNG)

![p11-cloning](https://user-images.githubusercontent.com/114327344/203419388-41927878-25f4-4e3d-b719-0ddfcf0aac29.PNG)

![p11-files](https://user-images.githubusercontent.com/114327344/203419651-c5deaddd-8d08-451d-bf94-c021715a1db1.PNG)

![p11-commitGit](https://user-images.githubusercontent.com/114327344/203419484-3b669a9f-74ae-44a5-803c-36e02ad216d3.PNG)

![p11-inventory](https://user-images.githubusercontent.com/114327344/203419853-118f7da0-d52a-494b-90f6-df4742919f3c.PNG)

![p11-gitpush](https://user-images.githubusercontent.com/114327344/203419964-738f7b08-4b58-4d73-a5fd-051fbea4bf0a.PNG)

![p11-jenkins](https://user-images.githubusercontent.com/114327344/203420051-0b7233cd-4551-4e1c-8465-e790de529e86.PNG)

![p11-newbranch](https://user-images.githubusercontent.com/114327344/203420135-844b2715-3e46-4481-a15f-c896591a98cc.PNG)

![p11-playbokOK](https://user-images.githubusercontent.com/114327344/203420421-d39aefdf-c096-44b2-8d58-24c1ac002ee4.PNG)

![p11-servers](https://user-images.githubusercontent.com/114327344/203420594-f9465e4d-4583-4c02-b94b-23ba5a0b33c6.PNG)

![p11-sshmysql](https://user-images.githubusercontent.com/114327344/203420794-766d0330-d409-4d0c-8e5a-96ea5dd1c1e4.PNG)

![p11-sshNFS](https://user-images.githubusercontent.com/114327344/203420836-ca4cec27-3c99-482b-b686-9d2773436422.PNG)

![p11-wireshakLB](https://user-images.githubusercontent.com/114327344/203421037-5c587f92-665f-4ea4-b277-fe74c47b57b7.PNG)

![pem-vscode](https://user-images.githubusercontent.com/114327344/203421132-c809ca3f-3f99-46f2-b6c3-a8cf0b84ffb1.PNG)

![README](https://user-images.githubusercontent.com/114327344/203421219-c6b49be9-51ac-43a7-809c-500d796bcdc5.PNG)

![p11-apache2](https://user-images.githubusercontent.com/114327344/203421525-30c11e42-fdd0-4cc9-adfd-e1253a8713be.PNG)

![p11-apachpage](https://user-images.githubusercontent.com/114327344/203421757-33a46417-6ed8-4a0e-964d-a054fcfaac97.PNG)








