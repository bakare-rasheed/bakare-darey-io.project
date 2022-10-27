## LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

- Load balancing across multiple application instances is a commonly used technique for optimizing resource utilization, 
maximizing throughput, reducing latency, and ensuring fault-tolerant configurations.

- It is possible to use nginx as a very efficient HTTP load balancer to distribute traffic to several application servers and to improve performance, 
scalability and reliability of web applications with nginx.

*The following load balancing mechanisms (or methods) are supported in nginx:*

* round-robin — requests to the application servers are distributed in a round-robin fashion,
* least-connected — next request is assigned to the server with the least number of active connections,
* ip-hash — a hash-function is used to determine what server should be selected for the next request (based on the client’s IP address).

SSL and its newer version, TSL – is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS – even though SSL was replaced by TLS, the term is still being widely used.

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

## CONFIGURE NGINX AS A LOAD BALANCER
* Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)
* Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
* Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

![instances](https://user-images.githubusercontent.com/114327344/198314754-035fbad5-ca17-4d3b-9fe0-73fe8f34fbb4.PNG)

```
sudo apt update
sudo apt install nginx
```

![nginx running](https://user-images.githubusercontent.com/114327344/198318546-2d9f54a3-9d09-48e3-b5b5-cadfde4af0f4.PNG)

* Open the default nginx configuration file

`sudo vi /etc/nginx/nginx.conf`

```
#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```
![nginx conf 2](https://user-images.githubusercontent.com/114327344/198316575-81578901-d444-4a2f-b22f-96cead1cd78a.PNG)

* To check the state of nginx , if succesfully configured
 
 `sudo nginx -t`
 
* I made a mistake, I edit the */etc/hosts* in a wrong way
* This really take my time to figure out

![nginx config](https://user-images.githubusercontent.com/114327344/198317683-833b6302-06b6-4775-987d-9b2e205b2a23.PNG)

## REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

Register a new domain name with any registrar of your choice in any domain zone (e.g. ``.com, .net, .org, .edu, .info, .xyz`` or any other)

* Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP 
* I used both Route 53 and Elastic IP 

![freenom](https://user-images.githubusercontent.com/114327344/198323779-cd5c669e-9ca9-4ee9-8e8b-70db36e464ce.PNG)

![name servers](https://user-images.githubusercontent.com/114327344/198324042-edb00631-3a4a-411b-87ad-0dd8d3cea4d3.PNG)

![elastic-ip](https://user-images.githubusercontent.com/114327344/198324095-a383328b-c379-4522-b1d7-a3076f25740f.PNG)


## My Load balancer is working !!!!!!!!!!!!

![route53](https://user-images.githubusercontent.com/114327344/198318977-c0e4f3bf-d89e-4dcc-baef-f91b9ca4748d.PNG)

* Install certbot and request for an SSL/TLS certificate, Make sure snapd service is active and running

`sudo systemctl status snapd`

![snap](https://user-images.githubusercontent.com/114327344/198325211-5b428684-9179-4bb9-9104-070a5f0383bb.PNG)

* Install certbot

`sudo snap install --classic certbot`
* To activate the ssl certificate

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```
![cartbot](https://user-images.githubusercontent.com/114327344/198326207-808beb1a-fd5f-436e-85f5-d6d4f1f892c0.PNG)

![padlock](https://user-images.githubusercontent.com/114327344/198326899-edd7ef00-eb7d-4ddb-98e7-b5565cc583ac.PNG)

![Certificate](https://user-images.githubusercontent.com/114327344/198326703-0761cf08-a06c-4871-a770-f991638bd593.PNG)


* Set up periodical renewal of your SSL/TLS certificate
* By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.
* You can test renewal command in dry-run mode

`sudo certbot renew --dry-run`

* To do so, lets edit the crontab file with the following command:

`crontab -e`
* Add following line:

`* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`

![crontab](https://user-images.githubusercontent.com/114327344/198327288-75f3a047-9d54-4bb0-9faf-8400e115cf9a.PNG)


