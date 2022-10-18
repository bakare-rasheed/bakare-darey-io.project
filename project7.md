## DEVOPS TOOLING WEBSITE SOLUTION

- DevOps is the combination of cultural philosophies, practices, and tools that increases an organization’s ability to deliver applications and services at high velocity:
evolving and improving products at a faster pace than organizations using traditional software development and infrastructure management processes.

- This speed enables organizations to better serve their customers and compete more effectively in the market.

- The DevOps model relies on effective tooling to help teams rapidly and reliably deploy and innovate for their customers.
- These tools automate manual tasks, help teams manage complex environments at scale, and keep engineers in control of the high velocity that is enabled by DevOps.

- DevOps tools facilitate ways for effective sharing and exchange of resources, information and technical know-how of the tasks between the development, operations 
and security teams for effective product output.

## STEP 1 – PREPARE NFS SERVER

* Spin up a new EC2 instance with RHEL Linux 8 Operating System
* Configure LVM on the server
* Format the Disks as xfs
* Ensure there are 3 Logical Volumes. `lv-opt`, `lv-apps` and `lv-logs`

![instances](https://user-images.githubusercontent.com/114327344/196402138-534c5942-e186-4701-aab2-167ac8bd9f9e.PNG)

![volumes](https://user-images.githubusercontent.com/114327344/196407012-29d32fbd-4742-4dd9-a630-60980b514976.PNG)

![All-partitions](https://user-images.githubusercontent.com/114327344/196407832-53ff76c4-2f26-40c6-b24d-29cc880f36c7.PNG)

![volume-grp](https://user-images.githubusercontent.com/114327344/196407521-503e324c-4ced-49e5-81ab-0f5e44b94fcc.PNG)

![available-partions](https://user-images.githubusercontent.com/114327344/196408195-00f38995-f9eb-4eaa-bcad-99f3769829e4.PNG)

```
sudo mkfs.xfs /dev/vg-nfs/lv-apps
sudo mkfs.xfs /dev/vg-nfs/lv-logs
sudo mkfs.xfs /dev/vg-nfs/lv-opt
```
![mounting](https://user-images.githubusercontent.com/114327344/196408725-4afa8d4a-c904-4096-b267-5a6dfa8365c5.PNG)

* Install NFS server, configure it to start on reboot and make sure it is running

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```

![NFS instal](https://user-images.githubusercontent.com/114327344/196409376-d7c9e798-cd9e-4098-bbfc-8fb3d4e6cbfb.PNG)

* Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```
* Configure access to NFS for clients within the same subnet

```
sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
```
* Then export the filesystem to the clients

`sudo exportfs -arv`

![exportfs](https://user-images.githubusercontent.com/114327344/196411803-069e3e41-705b-42c1-a0d4-86da43b79374.PNG)

* Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

`rpcinfo -p | grep nfs`

![NFS ports](https://user-images.githubusercontent.com/114327344/196412396-bb3d3f14-a2e9-493b-afcd-03a6e24ae1a4.PNG)

## STEP 2 — CONFIGURE THE DATABASE SERVER

`create database tooling`

`CREATE USER 'webaccess'@'172.31.16.0/20' IDENTIFIED BY 'password';`

```
 grant all privileges on tooling.* to 'webaccess'@'172.31.16.0/20';
 flush privileges;
 ```
 
##  Step 3 — Prepare the Web Servers
```
#!/bin/bash

sudo yum install nfs-utils nfs4-acl-tools -y
sudo mkdir -p /var/www
sudo mount -t nfs -o rw,nosuid 172.31.19.201:/mnt/apps /var/www
sudo chmod 766 /etc/fstab
sudo echo "172.31.19.201:/mnt/apps /var/www     xfs defaults 0 0" >> /etc/fstab

sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm -y

sudo dnf module reset php -y

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd -y

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1

```

* Clone the repo from the github account 

```
    sudo yum install git
    git int
    clone https://github.com/darey-io/tooling.git
```
![git install](https://user-images.githubusercontent.com/114327344/196417173-5019993f-542e-4a35-9018-7b583ddf0f4b.PNG)

![clone](https://user-images.githubusercontent.com/114327344/196415986-a915ff8b-8b9b-4337-9a57-9a2ac6cbc651.PNG)

* Move contents of the html folder to the present working directoty
`sudo mv ./html/* .`

![html copied](https://user-images.githubusercontent.com/114327344/196415831-ad3eaf60-3591-4acc-892c-26c9d1063ae4.PNG)

* Connect the database to the webservers.
* Edit the bind address in the mysqld.conf

` sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

`sudo systemctl restart mysql`

* Install MySQL client on the WebServer and connect to the MySQL server(DB).
* Ensure to run the connection config in the */var/www/html* directory

```
sudo yum install mysql -y
mysql -u webaccess  -h  172.31.85.252  -p password tooling < tooling-db.sql
mysql -u webaccess  -h  172.31.85.252  -p pasword tooling 
```

![tooling](https://user-images.githubusercontent.com/114327344/196419366-4bacfbca-bbb6-470f-b0cc-8a99b5ccc667.PNG)

![toolingdb](https://user-images.githubusercontent.com/114327344/196419507-7577d960-6f77-48c3-9cd2-1f3af0a59dd7.PNG)

* Edit the *functions.php* file in the html directory on the web server

`sudo nano functions.php`

* I had no permission  Error – check permissions to your /var/www/html folder and also disable SELinux 
`sudo setenforce 0`

* Edit the file `sudo vim /etc/etc/sysconfig/selinux` and  set SELINUX=disabled

![selinux disabled](https://user-images.githubusercontent.com/114327344/196419845-91a8d988-4b0c-4e1c-9fda-f60f655fbd01.PNG)

* Initial login with the following credentials
```
username: admin
password: admin
```

![admin-login](https://user-images.githubusercontent.com/114327344/196421208-d31d07a9-0b78-4a00-b4f2-a6cb81db1e40.PNG)

![admin-login2](https://user-images.githubusercontent.com/114327344/196421363-35c8b24f-b879-4030-97fc-b512ff291ee1.PNG)

* Create another user

![create-user](https://user-images.githubusercontent.com/114327344/196421066-a59f7769-1391-48a5-9afc-eb42b8dc4474.PNG)

![user-created](https://user-images.githubusercontent.com/114327344/196420490-1a1298a0-44fa-4edb-ab42-24c02fd7035a.PNG)


