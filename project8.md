## LOAD BALANCER SOLUTION WITH APACHE

Capacity planning is an important step to take when architecting any website or web application.
A decision often arises during this process as to whether to scale the server farm vertically or horizontally.
Vertical scaling involves adding more resources to existing servers as demands change to meet these needs.
Horizontal scaling involves the preemptive or dynamic provisioning of a redundant pool of servers along with a load balancer.
When you have just one Web server and load increases – you want to serve more and more customers, you can add more CPU and RAM or 
completely replace the server with a more powerful one – this is called "vertical scaling"

Another approach used to cater for increased traffic is "horizontal scaling" – distributing load across multiple Web servers. 

![Architecture](https://user-images.githubusercontent.com/114327344/196804021-679de8e1-82bf-4edd-a972-5e801332cfc9.PNG)

## This is implemented using the Project 7 

* Two RHEL8 Web Servers
* One MySQL DB Server (based on Ubuntu 20.04)
* One RHEL8 NFS server

![instances](https://user-images.githubusercontent.com/114327344/196804330-5e7de15c-e11c-45b5-9284-eaafe89d14e9.PNG)


* Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:
```
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
```
![Apache running](https://user-images.githubusercontent.com/114327344/196805636-d55e79e9-2929-4d01-970f-570fb8aa25fc.PNG)

* Configure load balancing

`sudo vi /etc/apache2/sites-available/000-default.conf`

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>
```
<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
```

![config](https://user-images.githubusercontent.com/114327344/196805808-88e10028-07eb-4ad7-8755-7b1e258cd479.PNG)


*  Configure local domain name resolution. The easiest way is to use */etc/hosts* file, to configure IP address to domain name mapping for our LB
```
#Open this file on your LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```
![webs](https://user-images.githubusercontent.com/114327344/196806266-4e933339-3180-4f8c-912f-9e390a20b3dd.PNG)

## Sites reachable after changing the hosts file
![site1](https://user-images.githubusercontent.com/114327344/196806466-7c6c34cc-d738-480a-8c50-516c7af97abc.PNG)

![site2](https://user-images.githubusercontent.com/114327344/196806514-fc8b40b0-c9d2-435e-a64a-677680f8e78f.PNG)

![running](https://user-images.githubusercontent.com/114327344/196806572-88676a81-02a7-4515-8d92-2d754824dd11.PNG)

#You can try to curl your Web Servers from LB locally `curl http://Web1` or `curl http://Web2`

![curl1](https://user-images.githubusercontent.com/114327344/196807421-8152db49-1f91-474e-8f4f-66a7609e7e6b.PNG)

![web2](https://user-images.githubusercontent.com/114327344/196806302-09b57a68-837d-4d33-b63d-9c5f67cbc4b1.PNG)




