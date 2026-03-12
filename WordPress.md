## Web Solution Using WordPress

WordPress (WP or WordPress.org) is a free and open-source content management system (CMS) developed in PHP and typically used with a MySQL or MariaDB database. It offers a flexible plugin architecture and a template system known as Themes, which allow users to customize the functionality and design of websites.

Initially, WordPress was designed primarily as a blog publishing platform, but over time it has expanded to support many other types of web applications. These include discussion forums, media galleries, membership platforms, learning management systems (LMS), mailing lists, and e-commerce websites.

For WordPress to operate, it must be installed on a web server. This server can either be part of a hosting platform such as WordPress.com, or it can be installed manually using WordPress.org software on a computer or server that acts as the web host.

### Three-Tier Architecture

Most modern web and mobile applications are built using a design model known as Three-Tier Architecture.

![alt text](/images/Screenshot%202026-03-11%20225315.png)

#### `3-Tier Setup`

In this project, the architecture will consist of the following components:  

`Client Machine`: A laptop or desktop computer that users will use to access the application through a browser.  

`Web Server`: An EC2 Linux instance that will host the WordPress application.  

`Database Server`: Another EC2 Linux instance that will store the WordPress database.  

A key advantage of the three-tier architecture is that each layer can be upgraded, modified, or replaced independently without affecting the others.

### `Structure of Three-Tier Architecture`

The user interface layer can run on various platforms such as desktop computers, smartphones, tablets, or web browsers. It provides a graphical interface that allows users to interact with the application, while the processing tasks are handled by the application server.  

The database layer contains the relational database management system (RDBMS), which stores and manages the application's data.  

The application layer, which sits between the presentation and data layers, handles the application’s processing logic and communication between the user interface and the database.  

Although these layers are logically separated, they can run on different servers either on-premises or in cloud environments such as SaaS platforms.

### Advantages of Three-Tier Architecture

Three-tier architecture offers several benefits:

Development teams can modify or update specific parts of the system without affecting the entire application.  

The system can be scaled easily by separating the application layer from the database layer.  
Additional hardware resources, such as new servers, can be introduced later to support higher workloads or large datasets.  
Organizations gain greater flexibility to adopt emerging technologies without redesigning the whole system.  
Important system components can be isolated and maintained while the rest of the system evolves.  
Upgrade and development cycles become faster, minimizing downtime for users.  
Multiple teams can work on different layers of the system simultaneously, increasing productivity and efficiency.  

### The Three Layers of a Three-Tier Architecture

#### `Presentation Tier`

This is the topmost layer of the architecture and is responsible for displaying information to users through a graphical interface.

It represents the front-end of the application, where users interact directly with the system.

This layer is typically developed using HTML, CSS, and JavaScript, and communicates with the other tiers through API calls or web requests.

#### `Application Tier`

Also referred to as the business logic layer or middle tier, this layer processes data received from the presentation layer.

It contains the core functionality of the application and performs all the necessary computations and processing.

Applications in this layer are commonly written using programming languages such as Python, Java, C++, PHP, or .NET.

#### `Data Tier`

The data tier is where the application’s information is stored and managed.  

It contains the database servers responsible for storing, retrieving, and managing application data.  

Data in this layer is independent from the application logic and is managed using database management systems such as MySQL, PostgreSQL, Oracle, MongoDB, or Microsoft SQL Server.  

### Project Objective

In this project, we will gain practical experience in implementing a three-tier architecture using WordPress. Additionally, we will ensure that storage devices used by the Linux servers are properly partitioned and managed using tools such as gdisk and Logical Volume Manager (LVM).  

To achieve this, the first step will be to:  

## `STEP - 1` -Set Up the Web Server

Launch an RHEL-10 EC2 instance that will function as the Web Server. While creating the instance, also create three additional EBS volumes, each with a size of 10 GiB, making sure they are created in the same Availability Zone (AZ) as the EC2 instance.

Connect to the web server instance via SSH using PuTTY or any other terminal.

Install WordPress on the Web Server EC2 Instance  

### Update the system packages

Before installing any software, update the system repositories and packages to the latest versions.

```bash
sudo dnf update -y
```
### Install Apache, PHP, and required dependencies

Install Apache web server, PHP, and the necessary PHP extensions required to run WordPress.

```bash
sudo dnf install -y wget httpd php php-mysqlnd php-fpm php-json php-gd php-curl
```
### Start and enable Apache

Start the Apache web server and enable it so that it starts automatically whenever the system boots.

```bash
sudo systemctl enable httpd
sudo systemctl start httpd
sudo systemctl status httpd
```
### Install and configure PHP (if additional modules are required)

Install additional PHP packages and start the PHP-FPM service so that Apache can process PHP files.

```bash
sudo dnf install -y php php-opcache php-gd php-curl php-mysqlnd php-fpm
```
### Start and enable the PHP-FPM service.

```bash
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```
Allow Apache to execute memory operations required by PHP.

```bash
sudo setsebool -P httpd_execmem 1
```
### Restart the Apache service

Restart Apache so that all newly installed components and configurations take effect.

```bash
sudo systemctl restart httpd
```
### Download and install WordPress

Download the latest version of WordPress, extract it, and copy the WordPress files to the Apache web root directory.

```bash
mkdir wordpress
cd wordpress
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xvzf latest.tar.gz
sudo rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
sudo cp -R wordpress /var/www/html/
```
### Configure SELinux permissions

Set the correct ownership and SELinux permissions so Apache can properly access and manage the WordPress files.

```bash
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect 1
```
Now we will configure Logical Volume Management (LVM) using the EBS volumes attached to the EC2 instance. Follow the steps below.

To display information about all the available block storage devices (disks and partitions) attached to the system.

```bash
lsblk
```
![alt text](/images/Screenshot%202026-03-11%20215128.png)

To get the disk space usage of mounted filesystems in a human-readable format.

```bash
df -h
```
Partition the three attached disks using the `gdisk` utility to prepare them for use

```bash
sudo gdisk /dev/nvme1n1
sudo gdisk /dev/nvme2n1
sudo gdisk /dev/nvme3n1
```
![alt text](/images/Screenshot%202026-03-11%20215240.png)

Run a command such as `lsblk` or `fdisk -l` to verify the newly attached disks and confirm that they are available for use with Logical Volume Manager (LVM). 

Run `lvmdiskscan` command to view the available storage for LVM.

![alt text](/images/Screenshot%202026-03-11%20215508.png)

Use the pvcreate command to initialize each of the three disks as Physical Volumes (PVs) so that they can be managed by LVM.

```bash
sudo pvcreate /dev/nvme1n1p1
sudo pvcreate /dev/nvme2n1p1
sudo pvcreate /dev/nvme3n1p1
```

View the created `Physical Volumes` using the `pvs` command

Create a Volume Group (VG) by combining all three physical volumes using the vgcreate command. Name the volume group `wordpress-vg`, also confirm that the volume group was created successfully by running a verification command such as vgdisplay or vgs.

```bash
sudo vgcreate wordpress-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1
sudo vgs
```

Use the lvcreate command to create two Logical Volumes (LVs):  

wordpress-lv – allocate half of the total volume group space for storing website data.  
wp-logs-lv – allocate the remaining space for storing system log files.  
wordpress-lv will store the website application files.  
wp-logs-lv will store log data. 

Verify that the logical volumes were created correctly by running lvdisplay or lvs.  

```bash
sudo lvcreate -n wordpress-lv -L 14G wordpress-vg
sudo lvcreate -n wp-logs-lv -L 14G wordpress-vg
sudo lvs
```
![alt text](/images/Screenshot%202026-03-11%20215906.png)

Format both logical volumes with the ext4 filesystem using the mkfs.ext4 command.  

```bash
sudo mkfs -t ext4 /dev/wordpress-vg/wordpress-lv
sudo mkfs -t ext4 /dev/wordpress-vg/wp-logs-lv
```
![alt text](/images/Screenshot%202026-03-11%20220954.png)

## Create Directories for Mounting

### Create a directory to store website files:

```bash
sudo mkdir -p /var/www/html
```
Create a directory to store a backup of log files:

```bash
sudo mkdir -p /home/recovery/logs
```
### Mount the Logical Volumes

Mount the wordpress-lv logical volume to the web directory:

```bash
sudo mount /dev/wordpress-vg/wordpress-lv /var/www/html/
```
### Backup Existing Log Files

Before mounting the new logical volume to /var/log, back up the existing log files using rsync:

```bash
sudo rsync -av /var/log/. /home/recovery/logs/
```
This step is important because mounting a new filesystem on /var/log will hide the existing files.

### Mount Logs Logical Volume

Mount the wp-logs-lv logical volume to the log directory:

```bash
sudo mount /dev/wordpress-vg/wp-logs-lv /var/log
```
### Restore Log Files

Restore the previously backed-up log files to /var/log:

```bash
sudo rsync -av /home/recovery/logs/. /var/log
```
Configure Automatic Mounting

To ensure the volumes are mounted automatically after a system reboot, update the /etc/fstab file.

First, obtain the UUID of the devices:

```bash
sudo blkid
```
Edit the fstab configuration file:

```bash
sudo vi /etc/fstab
```
Add entries using the UUID values of your logical volumes (remove any quotes around the UUID values).

Test the Configuration  
Verify the configuration and reload the system daemon:  

```bash
sudo mount -a
sudo systemctl daemon-reload
```
Verify the Final Setup

Check the mounted filesystems and disk usage:

```bash
df -h
```
The output should show wordpress-lv mounted on /var/www/html and wp-logs-lv mounted on /var/log.

## `Step - 2`

Launch an RHEL-10 EC2 instance that will function as the Web Server. While creating the instance, also create three additional EBS volumes, each with a size of 10 GiB, making sure they are created in the same Availability Zone (AZ) as the EC2 instance.

## Install MySQL (MariaDB) on the Database Server EC2 Instance

First, update the system packages to ensure the latest versions are installed.

```bash
sudo dnf update -y
```
Install the MariaDB server, which is the default MySQL-compatible database server used in RHEL systems.

```bash
sudo dnf install -y mariadb-server
```
Start the database service and enable it so that it automatically starts whenever the server boots.

```bash
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo systemctl status mariadb
```
### Configure the Database for WordPress

Log in to the MariaDB database shell.

```bash
sudo mysql
```
Create a new database that will be used by WordPress.

```bash
CREATE DATABASE wordpress;
```
Create a database user that will connect from the Web Server's private IP address, and assign a password.

```bash
CREATE USER 'myuser'@'<Web-Server-Private-IP-Address>' IDENTIFIED BY 'wordpress';
```
Grant the user full privileges on the WordPress database.

```bash
GRANT ALL PRIVILEGES ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
```
Apply the changes.

```bash
FLUSH PRIVILEGES;
```
Verify that the database has been created successfully.

```bash
SHOW DATABASES;
```
Exit the MariaDB shell.

```bash
EXIT;
```

`Note`: Repeat the LVM creation steps that were performed on the WordPress web server. However, use different names for the Physical Volumes (PVs), Volume Groups (VGs), and Logical Volumes (LVs) as required.

## Configure Mount Points on the Database Server

After creating the LVM on the Database Server, the next step is to create directories that will be used as mount points for the logical volumes and configure the system to store database and log files properly.

First, create a directory called /db which will store the database data files. Then create another directory /home/recovery/logs to temporarily store a backup of the existing log files before mounting the new filesystem.

```bash
sudo mkdir /db
sudo mkdir -p /home/recovery/logs
```

Next, mount the logical volume <lv-name> to the /db directory so it can be used for database storage. After mounting, verify the mount point using the `df -h` command.

```bash
sudo mount /dev/<vg-name>/<lv-name> /db
```

Before mounting a new filesystem on /var/log, back up the existing log files to /home/recovery/logs using the rsync utility. This step is important because mounting a new filesystem on /var/log will hide the existing log files.

```bash
sudo rsync -av /var/log/. /home/recovery/logs/
```

After completing the backup, mount the logical volume <lv-name> to the /var/log directory. Then restore the previously backed-up log files back into /var/log.

```bash
sudo mount /dev/<vg-name>/<lv-name> /var/log
sudo rsync -av /home/recovery/logs/. /var/log
```

Finally, update the /etc/fstab configuration file by following the same steps used on the WordPress web server to ensure that the mount points are automatically mounted after every system reboot.

```bash
sudo vi /etc/fstab
```

## STEP - 3 

Before configuring WordPress, ensure that the database server allows connections on port 3306. For security reasons, access to the database server should be restricted so that only the Web Server’s private IP address can connect. In the DB Server EC2 security group, create an inbound rule allowing port 3306 with the source set to the Web Server private IP address using /32.

![alt text](/images/image.png)

### Install the MariaDB client on the Web Server

Install the database client so the web server can connect to the remote database server.

```bash
sudo dnf install -y mariadb
```
Test the connection from the Web Server to the Database Server.

```bash
sudo mysql -h <DB-Server-Private-IP-address> -u myuser -p
```
Enter the database password when prompted.

### Verify database connectivity

After successfully logging in, run the following command to confirm that the database server is reachable and that the WordPress database exists.

```bash
SHOW DATABASES;
```
You should see the wordpress database in the list.

### Update WordPress configuration

Modify the WordPress configuration file so that it connects to the remote database server.
Edit the file:

```bash
sudo vi /var/www/html/wordpress/wp-config.php
```
Update the database parameters:

```bash
DB_NAME
DB_USER
DB_PASSWORD
DB_HOST
```
Set DB_HOST to the private IP address of the database server.

### Allow HTTP access to the Web Server

In the Web Server EC2 security group, allow inbound traffic on port 80 (HTTP) so that the WordPress site can be accessed from a browser.

For testing purposes you can allow:

```bash
0.0.0.0/0
```
or restrict access to your workstation IP address for better security.

Access the WordPress installation page

Open a browser and navigate to the following URL:

```bash
http://<Web-Server-Public-IP>/wordpress
```
This will launch the WordPress setup page where you can complete the installation.

![alt text](/images/Screenshot%202026-03-12%20001745.png)