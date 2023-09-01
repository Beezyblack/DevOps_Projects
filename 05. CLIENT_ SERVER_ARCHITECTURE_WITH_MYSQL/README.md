# Implementing a Client Server Architecture Using MySQL server and MySQL client

## Step 1
### Provisioned two Linux-based virtual servers (EC2 instances in AWS)

image

## Step 2 IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS)
### Install MySQL Server software on On mysql server Linux Server

image


### install MySQL Client software On mysql client Linux Server

image


Open port 3306 on Mysql-server allow for connection. Both server can communicate using private IPs since they belong to the same subnet

image

Change bind-address on Mysql-server to allow for connection from any IP address. Set the bind-address to 0.0.0.0 using the command below:
``sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf``

Configure MysQL server and create database and user

- Set up password with sudo mysql_secure_installation and create a user

- Create database

image

- Grant all permission on database

image

From mysql client Linux Server connect remotely to mysql server Database Engine without using SSH. You must use the mysql utility to perform this action.

image

show databases
``show databases``

Check that you have successfully connected to a remote MySQL server and can perform SQL queries. You should something similar to the screenshot above


