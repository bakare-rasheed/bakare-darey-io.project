## SIMPLE TO-DO APPLICATION ON MERN WEB STACK

# This project is based on inplementation of web solution using MERN stack in AWS Cloud.

MERN Web stack is a compartmentalization of four components:
* MongoDB: A document-based, No-SQL database used to store application data in a form of documents.
* ExpressJS: A server side Web Application framework for Node.js.
* ReactJS: A frontend framework developed by Facebook. It is based on JavaScript, used to build User Interface (UI) components.
* Node.js: A JavaScript runtime environment. It is used to run JavaScript on a machine rather than in a browser.

## Step 1 – Backend configurations

![config](https://user-images.githubusercontent.com/114327344/194880117-3d69f5f0-8d2b-4732-9ab4-631d5a4dbac6.PNG)

* Creating Todo Directory
![Todo](https://user-images.githubusercontent.com/114327344/194880814-5c8856c0-d7f4-4a2b-90cd-a2212e698dbc.PNG)

* Install ExpressJS 
* Express is a framework for Node.js, it simplifies development, and abstracts a lot of low level details.

``npm install express``

![Express](https://user-images.githubusercontent.com/114327344/194881223-5a2d26e5-3db4-4d09-b6d7-dd47858f5951.PNG)

![Express](https://user-images.githubusercontent.com/114327344/194883153-874907cd-0f94-4a11-ab08-96e7686cb27d.PNG)

* *Models* A model is at the heart of JavaScript based applications, and it is what makes it interactive.

`npm install mongoose`

* MongoDB Database


![database](https://user-images.githubusercontent.com/114327344/194884379-d06a73eb-e1b3-4d65-a286-f7c0e9f6e7d5.PNG)

![MongoDB](https://user-images.githubusercontent.com/114327344/194885215-3b67832d-014a-4f31-92ce-39e742445635.PNG)

* Testing Backend Code without Frontend using RESTful API using *Postman*
![post/get](https://user-images.githubusercontent.com/114327344/194887855-fc72538b-83a5-4d9b-bd40-c7ed0baa1a28.PNG)

## Step 2 – Frontend creation

* Update the node version 
 ```
 npx create-react-app client
 npm install concurrently --save-dev
 npm install nodemon --save-dev
```

* Running a React App
![react](https://user-images.githubusercontent.com/114327344/194888400-5e9aff0b-ebaa-4fe3-83f8-9b592ff9da0e.PNG)

* The frontend application using React.js that communicates with a backend application written using Expressjs.
![react](https://user-images.githubusercontent.com/114327344/194889479-27867dab-4e1d-4a47-928d-ca0630903ec3.PNG)
