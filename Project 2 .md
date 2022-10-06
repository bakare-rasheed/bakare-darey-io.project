## WEB STACK IMPLEMENTATION (LEMP STACK)
LEMP STACK is a combination of four technologies
- Linux, which is the Operating system
- Nginx, which is the server
- MySQL, the database
- Python, PHP or Perl

This project is similar to the previous stack, but with an alternative Web Server – *NGINX*, which is also very popular and widely used by many websites in the Internet.

## Step 1 – Installing the Nginx Web Server

* Nginx serves as a high performnace web server which is installed using this commands;
```
sudo apt update
sudo apt install nginx
```
* To ensure the succesful installation of the nginx web sever `sudo systemctl status nginx`
* Regarding AWS, the Public IP of the nginx web server can be retrieved from the instance using 
``` 
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```
* This shows the default page of the nginx web server

![Nginx](https://user-images.githubusercontent.com/114327344/194341880-6b1287b7-f80d-4dc2-88e2-6d584ad13dad.PNG)


## Step 2 — Installing MySQL

* MySQL is a popular relational database management system used within PHP environments, which is used in this project
`sudo apt install mysql-server`
* To enable security measures on the mysql requires an inital secuirty configuration `sudo mysql_secure_installation`
* This shows the mysql display page on the server

![mysql](https://user-images.githubusercontent.com/114327344/194342846-5f25a14b-1931-4994-82da-e030a314c2dc.PNG)

## Step 3 – Installing PHP
* In order to instal the php to serve dynamic web contents on the nginx server `sudo apt install php-fpm php-mysql`

## Step 4 — Configuring Nginx to Use PHP Processor

* Create a root web direectory for my projectLEMP, which will host the contents of the webiste `sudo mkdir /var/www/projectLEMP`
* Create a new configuration file in Nginx’s sites-available directory
```
sudo nano /etc/nginx/sites-available/projectLEMP
```
* Then imput the configuration on the file, to craete a site for the projectLEMP.
* Link the config file to sites-enabled directory and reload the nginx server 
* Also unlink the default enabled site on the nginx server
```
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`
sudo unlink /etc/nginx/sites-enabled/default
sudo nginx -t
```
![Nginx and PHP](https://user-images.githubusercontent.com/114327344/194343466-bc6e7ccc-2f7b-4654-be90-63123eb8024a.PNG)

* Create a test PHP file called info.php in the document root using nano editor
```
sudo nano /var/www/projectLEMP/info.php
```
* Then, put the below commands in the test file 
```
<?php
phpinfo();
```
* The web browser displays the contents of the pubilc_ip/info.php

![info.php](https://user-images.githubusercontent.com/114327344/194344116-2ea658ad-13c1-4f6a-8608-0116be322df0.PNG)

## Step 5 – Testing PHP with Nginx

![info.php](https://user-images.githubusercontent.com/114327344/194344116-2ea658ad-13c1-4f6a-8608-0116be322df0.PNG)

## Step 6 — Retrieving data from MySQL database with PHP

* In this step you will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it

![todo_list](https://user-images.githubusercontent.com/114327344/194347643-82424d51-63d9-47a9-a198-60da184d0696.PNG)

* The data insert in the todo_list table is sucessfully saved on the databse

* The PHP script has been connected with the MySQL database when reaching the browser with the todo_list.php, it displays the page below

![Data](https://user-images.githubusercontent.com/114327344/194348049-1adc468f-a7ee-454a-bc37-81b058ea92bc.PNG)

