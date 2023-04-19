# Deploying A LEMP Stack Application On AWS Cloud

## Step 1
### Creation of an EC2 Ubuntu Instance

Provisioned an EC2 Ubuntu instance on AWS

![](/02.Images/ubuntu-instance.png)

Login into the instance via ssh

![](/02.Images/ssh-instance.png)

### Installing Nginx Web Server

To install the Nginx web server use the apt package manager to install this package 

Run a sudo apt update command to download package information from all configured sources.

![](/02.Images/apt-update.png)

sudo apt install nginx

![](/02.Images/install-nginx.png)

Spin up the nginx server and ensure it automatically starts on system reboot by running the following commands

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

Run systemctl status nginx to verify if the installation succeeds.

![](/02.Images/status-nginx.png)

Enable TCP port 80 on our EC2 Ubuntu instance so we can receive traffice from our web browser. Edit the inbound rules as shown below

![](/02.Images/inbound-rules.png)

Accessing the default nginx web server block to see if everything works correctly. curl the local IP address of our local machine which in most case is 127.0.0.1 or the DNS name localhost on any web browser on our local machine.
curl http://127.0.0.1:80 or curl http://localhost:80

![](/02.Images/curl-localhost.png)

Test how our Nginx server can respond to requests from the Internet by inserting our public IP address into our web browser. The below result shows nginx has been properly set up and we can deploy our web application via the port 80 which was opened.

![](/02.Images/nginxwebpage.png)

## Step 2
### Installing MySql

After successful completion in installing the nginx web server next step is to install mySQL database which is a relational database management server to help store data and manage content on our web application.

Run sudo apt install mysql-server

![](/02.Images/install-mysql.png)

To provide added security to mySQL server by running sudo apt install mysql-secure_installation. This script will remove some insecure default settings and lock down access to our database system.

![](/02.Images/mysql-secure.png)

On each prompt that comes press Y , add a password and save.

With mysql_server successfully configured, login into the mysql server by running the command sudo mysql.

![](/02.Images/sudo-mysql.png)


## Step 3
### Installing PHP and Its Modules

PHP is used to dynamically display contents of our webpage to users who make requests to the webserver.

Run sudo apt install php-fpm php-mysql

```bash
sudo apt install php-fpm php-mysql
```

![](/02.Images/install-php-sql.png)

## Step 4
### Creating A Web Server Block For Our Web Application

To serve our webcontent on our webserver, we create a directory for our project inside the /var/www/ directory.

```bash
sudo mkdir /var/www/projectlempstack 
``` 
Then we assign ownership of the projectlampstack directory to the current user system

```bash
sudo chown -R $USER:$USER /var/www/projectlempstack
``` 

![](/02.Images/var-mkdir.png)

Create a configuration for our server block

```bash
sudo nano /etc/nginx/sites-available/projectLEMP
``` 

After running the above command the image below represents the configuration required for our web server block to be functional

![](/02.Images/nginx-projectlemp.png)

We then activate our configuration by linking the configuration file to the sites-enabled directory

```bash
sudo ln -s /etc/nginx/sites-available/projectlempstack /etc/nginx/sites-enabled
``` 

To test our configuration for syntax errors we run the command below

```bash
sudo nginx -t
``` 

![](/02.Images/nginx-t.png)

Now our new server block has been created and configured but currently the default server block is the default block that comes with nginx install. To unlink it we run the command below.

```bash
sudo unlink /etc/sites-available/default
``` 

To apply the changes we run the command  

```bash
sudo systemctl reload nginx
``` 

Create an index.html file inside projectlempstack directory and write in contents to be accessed over the internet. Paste public IP address on a browser to see content.

```bash
http://<Public-IP-Address>:80
``` 

![](/02.Images/lemp-ip.png)

Serving PHP using nginx

Create an info.php file inside the /var/www/projectlempstack directory.

On a browser enter 

```bash 
http://<public-ip>/info.php 
```

![](/02.Images/php-version.png)

## Step 5
### Connecting PHP with MySQL and Fetching Content

Login into our mysql-server by running the command

```bash
sudo mysql
```

Create a new database by running the command 

```bash
CREATE DATABASE <db_name>
```

Create a new user and assign user a password by running the command 

```bash 
CREATE USER 'db_user'@'%' IDENTIFIED WITH mysql_native_password BY 'db_password' 
````

Grant the user permission over the created database by running the command 

```bash
GRANT ALL ON 'db_name'.* TO 'db_user'@'%'
```

exit from the mysql-server in which we are currently logged in as root user and then Login into mysql server using the created user.

![](/02.Images/create-db.png)

```bash
mysql -u lemp_user -p
```

![](/02.Images/mysql-lemp.png)

We create a table for the current user inside the lemp_db database and specify content parameters

```bash
CREATE TABLE example_database.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );
```
Push in contents into the table by running the command

```bash
INSERT INTO lemp_db.todo_list(content) VALUES ('enter contents')
```

Create a php file todo_list.php in /var/www/projectlempstack directory and paste the following code

![](/02.Images/phpp.png)

We can then access our webpage via a browser by typing in

```bash 
http://<publicIP>/todo_list.php
```

![](/02.Images/todolist.png)  






