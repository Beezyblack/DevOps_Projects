# Deploying a LAMP Stack Web Application on AWS Cloud
## Step 1
### Creation of EC2 Instance
Logged into my AWS account and created an EC2 Ubuntu Instance

![AWS EC2 Instance](/01.Images/ec2-instance.png)

ssh into my EC2 terminal

![Ubuntu EC2 Terminal](/01.Images/Ubuntu-terminal.png)

### Setting Up Apache Web Server And Updating The Firewall

```bash
#update a list of packages in package manager
sudo apt update
```
![Apt update](/01.Images/apt-update.png)

```bash
#run apache2 package installation
sudo apt install apache2
```
![Install Apache](/01.Images/apache-install.png)

Verifying that apache2 is running as a service in our OS
```bash
sudo systemctl status apache2
```

Access it locally in Ubuntu
```bash
 curl http://localhost:80
```

Testing  the Apache HTTP server
```bash
http://<Public-IP-Address>:80
```
![ipcurl](/01.Images/ipcurl.png)



## Step 2

### Installing MySQL

Install a Database Management System (DBMS). MySQL is a relational DBMS we will be installing. To install this software I'll be running the command

```bash
sudo apt install mysql-server
```
![Install Mysql](/01.Images/install-mqsql.png)

After Installation is finished I logged into MySQL by running the command

```bash
sudo mysql
```

![sudo Mysql](/01.Images/sudo-mysql.png)




## Step 3

### Installing PHP and its Modules


PHP is the component of our setup that wil process code to display content to the end user. In addition to the php package I will be installing "php-mysql", which is a PHP module that allows PHP to communicate with our database, I will also be installing "libapache2-mod-php" to enable Apache to handle PHP files. While core PHP files will be installed as dependencies. To install all these i will be running the following command below


```bash
sudo apt install php libapache2-mod-php php-mysql
```

![Install PHP](/01.Images/install-php.png)

To confirm the version of my PHP i ran the following command below

```bash
php -v
```
![PHP Version](/01.Images/php-version.png)

## Step 4

### Deploying Our Site on Apache's VirtualHost


Setting up a domain called **projectlamp** to set up the domain i ran the following command below


```bash
sudo mkdir /var/www/projectlamp
```
img

Next, I assigned ownership of the directory with my current system user by running the command below
```bash
sudo chown -R $USER:$USER /var/www/projectlamp
```

I created and opened a new cofiguration file in Apache's **sites-available** directory by running the command below

```bash
sudo vi /etc/apache2/sites-available/projectlamp.conf
```

Which created a new blank file and I pasted the bare-bones configuration


```bash
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

![projectlamp](/01.Images/projectlamp.png)



I used the **ls** command to show the new file in the **sites-available** directory by running the command below


```bash
sudo ls /etc/apache2/sites-available
```

I enabled the new virtual host by running the command below


```bash
sudo a2ensite projectlamp
```

![a2ensite](/01.Images/a2ensite.png)


To disable Apache's default website I ran the command below


```bash
sudo a2dissite 000-default
```

![a2dissite](/01.Images/a2ensite.png)


To make sure the configuration file does not contain syntax errors I ran the command below

```bash
sudo apache2ctl configtest
```

Reload Apache so the changes take effect

```bash
sudo systemctl reload apache2
```

Create an index.html file to test the virtual host works as expected by running the following command

```bash
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```

Open your website URL using IP address

```bash
http://<Public-IP-Address>:80
```

### Creating Web Domain For Our Site

## Step 5


### Enabling PHP on The Website

```bash
sudo vim /etc/apache2/mods-enabled/dir.conf
```
!
[PHP Landing](/01.Images/php-landing.png)
