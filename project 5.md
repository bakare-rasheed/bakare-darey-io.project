## IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS)

### To demonstrate a basic client-server using MySQL Relational Database Management System (RDBMS)

* Create and configure two Linux-based virtual servers (EC2 instances in AWS).
```
Server A name - Mysql-server
Server B name - mysql-client
```
![Server-Client](https://user-images.githubusercontent.com/114327344/195404537-b5cc0a2d-39fd-4419-a67a-d39517f460fd.PNG)

* Install MySQL Server software on the MySQL server with the secure installation configuration

`sudo apt install mysql_server`

`sudo mysql_secure_installation`

* On mysql client Linux Server install MySQL Client software

`sudo apt install mysql_client`

* MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups.

![inbound](https://user-images.githubusercontent.com/114327344/195405230-91474814-ee66-46c2-aaa2-b55e5a158e93.PNG)


* There is a need to change the bind-address from 127.0.0.1 to 0.0.0.0 , to enable the server to allow connections from remote host

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

![img](https://user-images.githubusercontent.com/114327344/195405460-52df462b-9a83-4f3b-a309-f6c683c4ec07.PNG)


`sudo systemstl restart mysql`

* Create a user on the mysql server with password for authentication 

`  CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY 'insert_password';`

* To extract the  local IP on the mysql server for the security inbound rules

`ip addr show`

* Create a database and grant the remote_user the necessary priviledge

```
CREATE DATABASE bakare_db;
GRANT ALL ON  bakare_db.* TO `remote_user`@`%` WITH GRANT OPTION;
```
![bakare_db](https://user-images.githubusercontent.com/114327344/195406485-e300e4fc-af9e-423a-972d-c8d24b85d8e0.PNG)

![DB](https://user-images.githubusercontent.com/114327344/195406742-1fa70727-7ed7-4c5d-946e-74f5772b567c.PNG)

* From the client, connecting to the server using the remote_user credentials

`sudo mysql -u remote_user -h <private-ip> -p`

* To confirm the client-server architecture, display the database name from the client side

`show databases;`

![Connected](https://user-images.githubusercontent.com/114327344/195406742-1fa70727-7ed7-4c5d-946e-74f5772b567c.PNG)
