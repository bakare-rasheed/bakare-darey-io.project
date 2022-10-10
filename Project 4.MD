## MEAN STACK DEPLOYMENT TO UBUNTU IN AWS

MEAN Stack is a combination of the following components:
* MongoDB (Document database) – Stores and allows to retrieve data.
* Express (Back-end application framework) – Makes requests to Database for Reads and Writes.
* Angular (Front-end application framework) – Handles Client and Server Requests
* Node.js (JavaScript runtime environment) – Accepts requests and displays results to end user

## Step 1: Install NodeJs
* Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used to set up the Express routes and AngularJS controllers

`sudo apt update`

`sudo apt upgrade`

* It is advisable to download the latest and stable version of the nodejs to avoid incompatibility errors
```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
```
`sudo apt install -y nodejs`

![Nodejs](https://user-images.githubusercontent.com/114327344/194966413-265a5e75-dbf9-453e-ae99-3b99b4cd6506.PNG)

![Nodejs2](https://user-images.githubusercontent.com/114327344/194966443-76ab3348-280e-48c5-94f7-4b868398e427.PNG)

## Step 2: Install MongoDB

* MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```
```
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
* Installing the mongoDB using  `sudo apt install -y mongodb`
* Starting the server ` sudo service mongodb start ` and checking status `sudo systemctl status mongodb`

![mongodb](https://user-images.githubusercontent.com/114327344/194966601-4f3b33ee-61a6-4a17-bd9a-5f64973b5b58.PNG)

![mongodb](https://user-images.githubusercontent.com/114327344/194966618-d17e96b8-a98b-484b-b78b-653ccc4f1085.PNG)

## Step 3: Install Express and set up routes to the server

* Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express in to pass book information to and from our MongoDB database.

`sudo npm install express mongoose`

![express](https://user-images.githubusercontent.com/114327344/194967894-c4fd211e-4662-426c-8e0e-583de6b5cac2.PNG)

## Step 4 – Access the routes with AngularJS

* AngularJS provides a web framework for creating dynamic views in your web applications.

![Angular js](https://user-images.githubusercontent.com/114327344/194968046-7499b431-c4fd-4d3b-91e7-9d9ac92c988c.PNG)
