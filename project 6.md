# WEB SOLUTION WITH WORDPRESS

This project is based on preparation of storage infrastructure on two Linux servers and implementation of basic web solution using WordPress.
WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

## Project 6 consists of two parts;
- Configure storage subsystem for Web and Database servers based on Linux OS. This involves working with disks, partitions and volumes in Linux.
- Install WordPress and connect it to a remote MySQL database server. This part involves deploying Web and DB tiers of Web solution.

## Three-tier Architecture
Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.
* Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
* Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
* Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. 
  Database Server or File System Server such as FTP server, or NFS Server
  
  ## LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”.
 
![Servers](https://user-images.githubusercontent.com/114327344/196040140-3ccfd2c2-f6e2-4a57-ba26-d90be841c6d2.PNG)

* Login to the web server and use `lsblk` command to inspect what block devices are attached to the server

* Creating partition on the disk and  gdisk utility to create a single partition on each of the 3 disks
```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```
![f-partition](https://user-images.githubusercontent.com/114327344/196041089-ebe45be6-f2db-4144-81a0-caf9f7af75f0.PNG)

![g-partition](https://user-images.githubusercontent.com/114327344/196041122-3b39c152-a067-4a5a-af02-b28db7f5ae7e.PNG)

![h-partition](https://user-images.githubusercontent.com/114327344/196041138-fcaa2d3e-2b51-45c1-9be2-cfad1f7668c7.PNG)

![partitions](https://user-images.githubusercontent.com/114327344/196041153-06750a65-7d6c-4d63-9e38-ea3962346acc.PNG)

* Install lvm2 on the web server using `sudo yum instal lvm2 -y`
* 
* physical volume can only be created on the 3 partition disks, which is done using the `pvcreate` utlity to be used by lvm

```
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
```
* Verify that the Physical volume has been created successfully by running `sudo pvs`

![physical volume](https://user-images.githubusercontent.com/114327344/196041296-8afe0ab6-d1b4-4b41-8776-90da845ff06a.PNG)

* Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg and verify the sucessfully created volume group using `sudo vgs`
```
sudo vgcreate vg-webdata /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
![vgs](https://user-images.githubusercontent.com/114327344/196041400-ec02ed22-dd88-433e-87c4-80e6018bf416.PNG)

* Use `lvcreate` utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

```
sudo lvcreate -n apps-lv -L 14G vg-webdata
sudo lvcreate -n logs-lv -L 14G vg-webdata
```

![volume details](https://user-images.githubusercontent.com/114327344/196041579-ff589239-624b-453f-a116-b54e47c6c6f7.PNG)

* Attaching another */dev/xvdi*, create a physical volume, add to vg-webdata , then
`sudo lvx -n apps-lv -L 25G vg-webdata`

* Use mkfs.ext4 to format the logical volumes with ext4 filesystem

```
 sudo mkfs.ext4 /dev/vg-webdata/apps-lv
 sudo mkfs.ext4 /dev/vg-webdata/logs-lv

```

* Create /var/www/html directory to store website files `sudo mkdir -p /var/www/html`

* Create /home/recovery/logs to store backup of log data
`sudo mkdir -p /home/recovery/logs`

* Mount /var/www/html on apps-lv logical volume 
`sudo mount /dev/vg-webdata/apps-lv /var/www/html/`

* Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
`sudo rsync -av /var/log/. /home/recovery/logs/`

* Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is the previous step is very
important)
`sudo mount /dev/vg-webdata/logs-lv /var/log`

* Restore log files back into /var/log directory 
`sudo rsync -av /home/recovery/logs/. /var/log`

* Update /etc/fstab file so that the mount configuration will persist after restart of the server
```
sudo vim /etc/fstab
```
* Test the configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
```
## Step 2 — Prepare the Database Server

* Use the RedHat Linux for the database server.
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

`sudo vgcreate vg-database /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

`sudo lvcreate -n db-lv -L 20G vg-database`

`sudo mkfs.ext4 /dev/vg-database/db-lv`

## Step 3 — Install WordPress on your Web Server EC2

* Update the repository 
`sudo yum -y update`
* Install wget, Apache and it’s dependencies
`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

* Start Apache
```
sudo systemctl enable httpd
sudo systemctl start httpd
```
* To install PHP and it’s depemdencies
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
* Restart Apache `sudo systemctl restart httpd`

* Download wordpress and copy wordpress to *var/www/html*
```
mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
  ```
* Configure SELinux Policies
```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
```
![wordpress](https://user-images.githubusercontent.com/114327344/196042178-c632d4a5-9efb-4cbc-b5ec-294b088f8aa8.PNG)

## Step 4 — Install MySQL on your DB Server EC2
* This step include the installation of mysql-server on the Database server
```
sudo yum update
sudo yum install mysql-server
```
* Verify that the service is up and running by using
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```

![mysql](https://user-images.githubusercontent.com/114327344/196042251-29fc0112-aa2f-4174-a1de-df53583eafa9.PNG)

## Step 5 — Configure DB to work with WordPress
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`172.31.5.140` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'172.31.94.34';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
![mysql2](https://user-images.githubusercontent.com/114327344/196042420-b1267983-2bd4-4ea7-8201-aa277d8923f8.PNG)

Using `ip addr show` to check the private ip on the web server.

## Step 6 — Configure WordPress to connect to remote database.

![connection](https://user-images.githubusercontent.com/114327344/196042463-26c5b8bb-c6c8-494c-8b98-3e1806fa4a74.PNG)

## ENCOUNTER THIS ERROR
![error](https://user-images.githubusercontent.com/114327344/196042550-a9cd7eff-c2a4-4e82-891c-861f485da97c.PNG)

* This requires to debug and troubleshoot
* The issue is from the WordPress database credentials are stored in the wp-config.php file
* adding the bind-address (0.0.0.0) in the */etc/my.cnf*
`sudo vim /etc/my.cnf`
`sudo systemctl restart mysqld`

![Apache working](https://user-images.githubusercontent.com/114327344/196042610-9b2ede06-ffd1-4d23-be72-0f085c994690.PNG)

![mysql-running](https://user-images.githubusercontent.com/114327344/196042627-bfaad132-9020-400c-bfee-67201492717b.PNG)

![wordpress](https://user-images.githubusercontent.com/114327344/196042649-21bd20ef-5241-49d6-9991-ad6b4e7aa586.PNG)

![wordpress2](https://user-images.githubusercontent.com/114327344/196042675-ffe127e4-6e0e-43bc-aaa3-606b853555d7.PNG)
