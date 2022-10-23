## TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS

Deployment automation is what enables you to deploy your software to testing and production environments with the push of a button. Automation is essential to reduce the risk of production deployments. 
It's also essential for providing fast feedback on the quality of your software by allowing teams to do comprehensive testing as soon as possible after changes.

*An automated deployment process has the following inputs:*

* Packages created by the continuous integration (CI) process (these packages should be deployable to any environment, including production).

* Scripts to configure the environment, deploy the packages, and perform a deployment test (sometimes known as a smoke test).

* Environment-specific configuration information.

Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. 
Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

![Architecture](https://user-images.githubusercontent.com/114327344/197402699-43017bc2-fc21-49c0-a269-ea4db9c98757.PNG)


## Step 1 – Install Jenkins server

* Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
* Install JDK (since Jenkins is a Java-based application)

![instance J](https://user-images.githubusercontent.com/114327344/197402932-736e75f2-8d1b-4824-8643-fb5bd3b70a35.PNG)

```
sudo apt update
sudo apt install default-jdk-headless
```
* Install Jenkins
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
![jenkins](https://user-images.githubusercontent.com/114327344/197402963-bfd5c6b9-8770-4ed7-955a-484c4eef3703.PNG)

* By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group

![security-sg](https://user-images.githubusercontent.com/114327344/197402906-1d91e472-f762-422f-8471-2bd1234cb750.PNG)

* Retrieve it from your server: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

![unlock jenkins](https://user-images.githubusercontent.com/114327344/197403152-36f57dac-ba14-4e42-901d-0b1fc0a51d07.PNG)

![welcom-jenk](https://user-images.githubusercontent.com/114327344/197403185-7f7f7234-2e99-4a06-bb37-d7fb700145d3.PNG)

**Then you will be asked which plugings to install – choose suggested plugins.**

![log-in j](https://user-images.githubusercontent.com/114327344/197403121-72c96679-572c-42c9-aa65-0a9fc537c08c.PNG)

## Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks

* Enable webhooks in your GitHub repository settings

![web-hook](https://user-images.githubusercontent.com/114327344/197403318-15edbe2d-320b-44f7-851d-67bfe92ee38a.PNG)

* Go to Jenkins web console, click "New Item" and create a "Freestyle project"

![freestyle](https://user-images.githubusercontent.com/114327344/197403650-8fecd9ad-db24-4bb4-8aed-406bd449e8f7.PNG)

![freestyly2](https://user-images.githubusercontent.com/114327344/197403670-ca1a1120-28c2-4095-b228-07d15f3fb0c2.PNG)

![freestyle2](https://user-images.githubusercontent.com/114327344/197403680-832dbd65-d504-4155-a59d-4dde2b4f6a4f.PNG)

* CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH

Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to `/mnt/apps` directory.

Jenkins is a highly extendable application and there are 1400+ plugins available.
We will need a plugin that is called "Publish Over SSH".

Install `"Publish Over SSH"` plugin.
On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

On "Available" tab search for "Publish Over SSH" plugin and install it

![install ssh](https://user-images.githubusercontent.com/114327344/197406328-6120e420-b17e-41bc-a0d1-764c1bd88804.PNG)

![success](https://user-images.githubusercontent.com/114327344/197406650-103692bd-f9b1-4338-852a-0191abd3cfd8.PNG)

* Having issues sending the files, there is permission issue on the /mnt/apps folder

![permission](https://user-images.githubusercontent.com/114327344/197406408-dadc4a77-36ad-466a-8019-f850eae64ae9.PNG)

* Change the permission  of the */mnt/apps* directory

`sudo chmod 777 -R /mnt/apps`

![chmod](https://user-images.githubusercontent.com/114327344/197406514-6ac85c2e-6519-4105-a7f6-3fb7f1e2fe97.PNG)

* This confirm the changes has been successfully tranfer to the NFS server

![Console output](https://user-images.githubusercontent.com/114327344/197406583-be816843-87b3-480d-ba93-1a2d4c45d5b4.PNG)

![final](https://user-images.githubusercontent.com/114327344/197406596-12c4a753-2be4-4080-9e8a-eee50fb72cd7.PNG)










