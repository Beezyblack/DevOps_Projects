# Deploying A MEAN Stack Application on AWS Cloud

## Step 0

## Prepare Prerequisites

## Create an EC2 Ubuntu Instance

MEAN Stack is a combination of following components:
MongoDB (Document database) – Stores and allows to retrieve data.
Express (Back-end application framework) – Makes requests to Database for Reads and Writes.
Angular (Front-end application framework) – Handles Client and Server Requests
Node.js (JavaScript runtime environment) – Accepts requests and displays results to end user


Provisioned an EC2 Ubuntu Instance on AWS

![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.ec2.png)

After provisioning my EC2 Instance, Log into the instance via ssh

![](/03.Images/3.ssh.png)

## Step 1

## Install NodeJs

Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used in this tutorial to set up the Express routes and AngularJS controllers.

Update and Upgrade dependencies on linux by running the command ``sudo apt update && sudo apt upgrade``


![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.update-upgrade.png)


Applying certificates and Install nodejs


```bash
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```


![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.install-nodejs.png)


## Step 2

## Install MongoDB


MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time. For our example application, we are adding book records to MongoDB that contain book name, isbn number, author, and number of pages.

Next install Mongodb which is a non-relational database which we will use to store our application data


```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```


To install MongoDB run ``sudo apt install -y mongodb``

![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.install-mongodb.png)


Start the server ``sudo service mongodb start``



Verify that the server is up and running ``sudo systemctl status mongodb``


Install npm Node package manager ``sudo apt install -y npm``

![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.install-npm.png)


Install body-parser

![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.install-body.png)


Create a ``Books`` directory and we initialize it as a npm project using ``npm init``. Then create a ``server.js`` file and setup the server.


![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.serverjs.png)



## Step 3

## Install Express and set up routes to the server

Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express in to pass book information to and from our MongoDB database.

We also will use Mongoose package which provides a straight-forward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.

Installing express and mongoose which is a package which provides a straight-forward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.

Install mongoose ``sudo npm install express mongoose``

![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.mongoose.png)



In the books directory create a directory ``apps`` and create a ``routes.js`` file then append the code to it


![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.routesjs.png)


Create a direcotry ``models`` in the books directory and add a file ``books.js`` and append the code which contains the schema model


![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.bookjs.png)


## Step 4

### Access the routes with AngularJS

AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.


In the book directory create a ``public`` directory and create a ``script.js`` file which will contain our angular frontend code


![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.scriptjs.png)


Create a ``index.html`` in the ``public`` directory and append the code


![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.index-html.png)


Move into the books directory and spin up the express server using ``node server.js``


Configure security group inbound rules to allow our application to be accessible via the internet via our server port


![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.inbound-rule.png)


In your browser, paste the public ip address of your instance to view the site


![](/04.Deploying_A_MEAN_STACK_Book-Application/04.Images/4.bookpage.png)
