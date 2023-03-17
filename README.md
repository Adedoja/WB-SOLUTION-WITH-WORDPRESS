# WEB-SOLUTION-WITH-WORDPRESS
This repository gives a detailed step by step guide on how to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress.

# Three-tier Architecture
Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers:
- Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
- Business Layer (BL): This is the backend program that implements business logic. Application or Webserver.
- Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File       System Server such as FTP server, or NFS Server.

# The Setup:
- A Laptop or PC to serve as a client
- An EC2 Linux Server as a web server (This is where you will install WordPress)
- An EC2 Linux server as a database (DB) server

In previous projects we used ‘Ubuntu’, but for this projects we will use a very popular distribution called ‘RedHat’.
Note: for Ubuntu server we used ubuntu user, but for RedHat you will need to use ec2-user user. Connection string will look like ec2-user@<Public-IP>

# STEP 1 - PREPARE A WEB-SERVER
1. Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/volume%201.PNG)
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/volume%202.PNG)

2. Attach all three volumes one by one to your Web Server EC2 instance.
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/volume%203.PNG)
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/volume%204.PNG)

3. Open up the Linux terminal to begin configuration Use `lsblk` command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with `ls /dev/` and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/lsblk.PNG)

4. Use df -h command to see all mounts and free space on your server
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/df%20-h.PNG)

5. Use fdisk utility to create a single partition on each of the 3 disks `sudo fdisk /dev/xvdf`
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/fdisk.PNG)

6. Use `lsblk` utility to view the newly configured partition on each of the 3 disks.
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/lsblk%20utility.PNG)

7. Install lvm2 package using `sudo yum install lvm2`. Run sudo lvmdiskscan command to check for available partitions.
   Note: yum is the package installer for RedHat/CentOS.

8. Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

   ```
   sudo pvcreate /dev/xvdg1
   sudo pvcreate /dev/xvdh1
   ```
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/pvcreate.PNG)

9. Verify that your Physical volume has been created successfully by running `sudo pvs`
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/sudo%20pvs.PNG)

10. Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
    `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
    
11. Verify that your VG has been created successfully by running `sudo vgs`
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/sudo%20vgs.PNG)

12. Use `lvcreate` utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. ## NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

    ```
    sudo lvcreate -n logs-lv -L 14G webdata-vg
    ```
13. Verify that your Logical Volume has been created successfully by running `sudo lvs`
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/sudo%20lvs.PNG)

14. Verify the entire setup `sudo vgdisplay -v #view complete setup - VG, PV, and LV`
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/vgdisplay.PNG)

`sudo lsblk`
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/sudo%20lsblk.PNG)

15.  Use mkfs.ext4 to format the logical volumes with ext4 filesystem
`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/mkfs.ext4.PNG)

16. Create /var/www/html directory to store website files
`sudo mkdir -p /var/www/html`

17. Create /home/recovery/logs to store backup of log data
`sudo mkdir -p /home/recovery/logs`

18. Mount /var/www/html on apps-lv logical volume
`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

19. Use rsync utility to back up all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

`sudo rsync -av /var/log/. /home/recovery/logs/`

![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/sudo%20rsync.PNG)

20. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)

`sudo mount /dev/webdata-vg/logs-lv /var/log`

21. Restore log files back into /var/log directory

`sudo rsync -av /home/recovery/logs/. /var/log`

![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/sudo%20rsync2.PNG)

22. Update /etc/fstab file so that the mount configuration will persist after restart of the server.

    - The UUID of the device will be used to update the /etc/fstab file; `sudo blkid`
![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/sudo%20blkid.PNG)
    
    - `sudo nano /etc/fstab` *Note: you have to install nano text editor first using `sudo yum install nano`
![](sudo nano /etc/fstab *Note: you have to install nano text editor first using sudo yum install nano)

23. Test the configuration and reload the daemon

`sudo systemctl daemon-reload`

24. Verify your setup by running `df -h`, output must look like this:

![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/vrify%20stup.PNG)

# STEP 2 - PREPARE THE DATABASE SERVER

Launch a second RedHat EC2 instance that will have a role – ‘DB Server’ Repeat the same steps as for the Web Server, but instead of apps-lv and logs-lv, create db-lv and logs-lv, then mount it to /db directory instead of /var/www/html/.

# STEP 3 - INSTALL WORDPRESS ON YOUR WEB SERVER
1. Update the repository

`sudo yum -y update`

2. Install wget, Apache and it’s dependencies

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

3. Start Apache

`sudo systemctl start httpd`

4. To install PHP and its depemdencies

```
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```

5. Restart Apache `sudo systemctl restart httpd`

6. Download wordpress and copy wordpress to var/www/html

```
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
sudo cp -R wordpress /var/www/html/
```
7. Configure SELinux Policies

```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P httpd_can_network_connect_db 1
```

# STEP 4 - INSTALL MYSQL ON YOUR DB SERVER

`sudo yum install mysql-server`

Verify that the service is up and running by using `sudo systemctl status mysqld`, if it is not running, restart the service and enable it so it will be running even after reboot:

`sudo systemctl enable mysqld`

# STEP 5 - CONFIGURE DB TO WORK WITH WORDPRESS

```
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
# STEP 6 - CONFIGURE WORDPRESS TO CONNECT TO THE REMOTE DATABASE

Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32

1. Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

`sudo yum install mysql`

2. Now edit mysql configuration file by typing `sudo vim /etc/my.cnf`. Add the following at the end of the file.

![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/vim%20etc.PNG)

   - Now, restart mysqld service using sudo systemctl restart mysqld
3. Change permissions and configuration so Apache could use WordPress:

4. Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

5. On the web server, edit wordpress configuration file.

cd /var/www/html/wordpress `sudo vim wp-config.php`

![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/cd%20var.PNG)

6. Disable the default page of apache so that you ca view the wordpress on the internet.

   `sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup`
   
   Restart httpd `sudo systemctl restart httpd`
   
 7. Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.
 
`sudo mysql -u admin -p -h <DB-Server-Private-IP-address>`

![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/wordpress%20file.PNG)

 8. Change permissions and configuration so Apache could use WordPress:
 
    ```
    sudo chcon -t httpd_sys_content_t /var/www/html/wordpress -R
    sudo setsebool -P httpd_can_network_connect=1
    sudo setsebool -P httpd_can_network_connect_db 1
    ```
9.  Try to access from your browser the link to your WordPress http://<your server public address>/wordpress/

![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/wordpress%20first.PNG)

    - Fill in your credentials to setup your account for your wordpress website. If you see this message – it means your WordPress has successfully connected to your remote MySQL database.
    
    
    - Log in with your username and password
    

![](https://github.com/Adedoja/WEB-SOLUTION-WITH-WORDPRESS/blob/main/3%20Tier%20Arch/WORDpress%20lastpage.PNG)

# CONGRATULATIONS!

You have learned how to configure Linux storage susbystem and have also deployed a full-scale Web Solution using WordPress CMS and MySQL RDBMS.


































