## Step 1

## Preparing the Webserver
- Create an EC2 instance server on AWS
- On the EBS console, create 3 storage volumes for the instance. This serves as additional external storage to our EC2 machine

![](image)

- Attach the created volumes to the EC2 instance

![](image)

- SSH into the instance and on the EC2 terminal view the disks attached to the instance. This is achieved using the ``lsblk`` command.

![](image)

- To see all mounts and free spaces on our server we run the command ``df -h``

![](image)

- Create single partitions on each volume on the server using the command ``gdisk``

![](image)

![](image)


- Installing LVM2 package for creating logical volumes on a linux server.

![](image)


- Creating physical volumes on each of the partitioned disk volumes by running the command

```bash
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1

```

![](image)

- Next we add each of the physical volumes into a volume group by running the command ``sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1``

![](image)

- Creating 2 logical volumes ``apps-lv`` and ``logs-lv`` for the volume group by running the command

```bash
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```

![](image)

- The logical volumes are ready to be used as filesystems for storing application and log data
- Create file systems on both logical volumes by running the command below

```bash
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
- The apache webserver uses the html folder in the var directory to store web content by running the command below

``sudo mkdir -p /var/www/html``

- To store backup of log data we run the command below

``sudo mkdir -p /home/recovery/logs``

![](image)

- For our filesystem to be used by the server we mount it on the apache directory. Also we mount the log filesystem to the log directory

![](image)

- Mount logs logical volume to var logs

![](image)

- Restoring back var llogs data into var logs

![](image)

### Persisting Mount Points

- To ensure that all our mounts are not erased on restarting the server we persist the mount points by configuring the ``/etc/fstab`` directory
- We run the command ``sudo blkid`` to get UUID of each mount points

![](image)

- We run the command ``sudo vi /etc/fstab`` to edit the file

![](image)

- Testing the mount point persistence and reload the daemon we run the command below

```bash
sudo mount -a
sudo systemctl daemon-reload
```

![](image)


## Step 2 Preparing the Database Server

- Repeated all the steps taken to configure the webserver on the database server. Also changed the ``apps-lv`` logical volume to ``db-lv``


![](image)


## Step 3 Configuring the Web Server and Installing Wordpress

- Run updates and install httpd on webserver

![](image)

- Start apache web server by running the command below

```bash
sudo systemctl enable httpd
sudo systemctl start httpd
```

![](image)


- Installing php and its dependencies by running the command below

```bash
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```

- Restarting Apache `` sudo systemctl restart httpd``

- Download wordpress and move it into the web content directory ``var/www/html`` by running the command below

```bash
mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
```

- Configure SELinux Policies by running the command below

```bash
sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
```

- Starting database server

```bash
sudo systemctl restart mysqld
sudo systemctl status mysqld
```

![](image)


## Step 4 Installing MySQL on Database Server

```bash
sudo yum update
sudo yum install mysql-server
```

- To ensure that the database server starts automatically on reboot or system startup

```bash
sudo yum update
sudo yum install mysql-server
```

## Step 5 Setting up Database Server

![](image)

- Ensure that we add port ``3306`` on our database server to allow our web server to access the database server.

![](image)

### çonnecting Web Server to Database Server

Installing MySQL client on the webserver so we can connect to the database server

```bash
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```

![](image)

- On the web browser access the web server using the public ip address of the server

![](image)

![](image)
