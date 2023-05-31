# WEB SOLUTION WITH WORDPRESS
## Three-Tier Architecture
Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.
Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

![111_3-tier_picture](https://github.com/ifydevops23/Application_Architecture/assets/126971054/17f2ac59-6f89-40d9-9b8d-14790f55fbf6)

- Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.<br>
- Business Layer (BL): This is the backend program that implements business logic. Application or Webserver <br>
- Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server

Also, Partitions and Volumes were created for persistence of website data.<br>

**STEP 0 - PRE-REQUISITES (REQUIREMENTS)**
- A Laptop or PC to serve as a client<br>
- An EC2 Linux Server as a web server (This is where you will install WordPress)<br>
- An EC2 Linux server as a database (DB) server<br>

## LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”.<br>
**STEP 1 — Prepare volumes for the Web Server** <br>
Launch an EC2 instance that will serve as "Web Server". <br>

![11_created_virtual_machines_with_ip](https://github.com/ifydevops23/Application_Architecture/assets/126971054/d1a2c2f5-2e00-464a-a195-4e33a9eed833)

Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.<br>

![1_attached_volumes](https://github.com/ifydevops23/Application_Architecture/assets/126971054/c7a2261e-c45e-48fb-9d22-7b32899d36d9)

Open up the Linux terminal to begin configurationUse `lsblk` command to inspect what block devices are attached to the server.<br>

![1_lsblk](https://github.com/ifydevops23/Application_Architecture/assets/126971054/de35589d-369d-49da-aa3a-2207e5578c5d)

Notice names of your newly created devices. All devices in Linux reside in /dev/ directory.<br>
Inspect it with `ls /dev/` and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.<br>
![1_ls-dev](https://github.com/ifydevops23/Application_Architecture/assets/126971054/d398e3fd-8899-4d3a-9878-aadc17ca0c3e)

Use `df -h` command to see all mounts and free space on your server. <br>

Use gdisk utility to create a single partition on each of the 3 disks <br>
`sudo gdisk /dev/xvdf`<br>

![1_partionn_single](https://github.com/ifydevops23/Application_Architecture/assets/126971054/91db98d3-c6b6-4527-8934-27de85126365)


Now, your changes has been configured successfuly, exit out of the gdisk console and do the same for the remaining disks.<br>
Use `lsblk` utility to view the newly configured partition on each of the 3 disks. <br>

Install lvm2 package using `sudo yum install lvm2`. <br>


Run `sudo lvmdiskscan` command to check for available partitions.<br>


Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM <br>
`sudo pvcreate /dev/xvdf1`<br>
`sudo pvcreate /dev/xvdg1`<br>
`sudo pvcreate /dev/xvdh1`<br>

![1_pvs_create](https://github.com/ifydevops23/Application_Architecture/assets/126971054/5c87475c-dddc-4fc0-a666-e5f0ae88b483)

Verify that your Physical volume has been created successfully by running <br>
`sudo pvs`<br>
Use vgcreate utility to add all 3 PVs to a volume group (VG). <br>
Name the VG webdata-vg <br> `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`<br>
Verify that your VG has been created successfully by running <br> 
`sudo vgs`<br>

![1_sudo_vgs](https://github.com/ifydevops23/Application_Architecture/assets/126971054/604af261-121d-4433-82e3-637ca41a41da)


Use `lvcreate` utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv <br> Use the remaining space of the PV size. <br>
NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs. <br>
`sudo lvcreate -n apps-lv -L 14G webdata-vg`<br>
`sudo lvcreate -n logs-lv -L 14G webdata-vg`<br>
Verify that your Logical Volume has been created successfully by running `sudo lvs`<br>

![1_logical_volumes](https://github.com/ifydevops23/Application_Architecture/assets/126971054/ae2363aa-6f9a-496e-8a3a-50a2f311caff)

Verify the entire setup <br>
`sudo vgdisplay -v #view complete setup - VG, PV, and LV``sudo lsblk`<br>

![1_verify_setup](https://github.com/ifydevops23/Application_Architecture/assets/126971054/eb0bb189-1d82-4305-83e3-71be2653da4a)


Use mkfs.ext4 to format the logical volumes with ext4 filesystem <br>
`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`<br>
`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`<br>

![1_mks](https://github.com/ifydevops23/Application_Architecture/assets/126971054/f20a0b48-0d4b-4a4f-aab6-4a799f866da9)

Create /var/www/html directory to store website files `sudo mkdir -p /var/www/html` <br>
Create /home/recovery/logs to store backup of log data sudo `mkdir -p /home/recovery/logs`<br>
Mount /var/www/html on apps-lv logical volume `sudo mount /dev/webdata-vg/apps-lv /var/www/html/`<br>

![1_mounts_app](https://github.com/ifydevops23/Application_Architecture/assets/126971054/caa4d801-cad6-4df6-880b-cefb3b97b7fb)

Use rsync utility to back up all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system) <br>
`sudo rsync -av /var/log/. /home/recovery/logs/`<br>
Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is veryimportant)<br>
`sudo mount /dev/webdata-vg/logs-lv /var/log`<br>
Restore log files back into /var/log directory <br> 
`sudo rsync -av /home/recovery/logs/. /var/log`<br>

![1_mounts_log](https://github.com/ifydevops23/Application_Architecture/assets/126971054/d3603050-9ed4-47e1-b198-803136ba4d27)


Update /etc/fstab file so that the mount configuration will persist after restart of the server. <br>
Click on the next button To update the /etc/fstab file<br>
- UPDATE THE `/ETC/FSTAB` FILE <br>
The UUID of the device will be used to update the /etc/fstab file;<br>
`sudo blkid`<br>

![1_blkid](https://github.com/ifydevops23/Application_Architecture/assets/126971054/eb5ba889-54e9-4792-9aee-f32ef0f2e618)

`sudo vi /etc/fstab`<br>

![1_UUID_config](https://github.com/ifydevops23/Application_Architecture/assets/126971054/360d8bdd-3548-49ea-b819-4fb8296ecc40)

Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes. <br>
Test the configuration and reload the daemon <br>
`sudo mount -a`<br>
`sudo systemctl daemon-reload`<br>
Verify your setup by running `df -h` output must look like this:<br>

![1_after_mounts](https://github.com/ifydevops23/Application_Architecture/assets/126971054/2da633e6-7e6c-49c5-8f05-002a7ca1b648)

**STEP 2 - PREPARE VOLUMES FOR THE DB SERVER** <br>
- Launch a second RedHat EC2 instance that will have a role – ‘DB Server’ <br>
- Create single partition in the disks, create physical volumes and logical volumes.

![1_verify_setup_db](https://github.com/ifydevops23/Application_Architecture/assets/126971054/fe60b589-a09d-43b6-94a4-11678ca92229)

- Create db-lv and Mount it to /db directory. <br>

![1_mounts_for_db](https://github.com/ifydevops23/Application_Architecture/assets/126971054/894382ad-9b07-4b5a-87ea-4de2f5477384)


**STEP 3 - PREPARE THE SOFTWARE STACK** <br>
INSTALL APACHE, PHP AND DEPENDENCIES <br>

- Update the repository
`sudo yum -y update`<br>
- Install wget, Apache and it’s dependencies <br>
`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`<br>
- Start Apache <br>
`sudo systemctl start httpd`<br>
- To install PHP and its dependencies
```
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd  --nobest
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1
```
- Restart Apache<br>
`sudo systemctl restart httpd`<br>

**STEP 4 - DOWNLOAD WORDPRESS** <br>

- Download wordpress and copy wordpress to var/www/html<br>

`mkdir wordpress`<br>
`cd   wordpress`<br>

```
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
sudo cp -R wordpress /var/www/html/
```


- Configure SELinux Policies
```
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P httpd_can_network_connect_db 1
```

**STEP 4 - DB SERVER PREP**<br>
Install Mysqld server <br>
`sudo yum install mysql-server`
Verify that the service is up and running by using <br>
`sudo systemctl status mysqld`<br>

![1_sudo_yum_install_mysql](https://github.com/ifydevops23/Application_Architecture/assets/126971054/417a504e-6b9c-4eb9-b469-55ae076c04c5)


If it is not running, restart the service and enable it so it will be running even after reboot:<br>
`sudo systemctl restart mysqld`<br>
`sudo systemctl enable mysqld`<br>

![1_mysql_restsrt](https://github.com/ifydevops23/Application_Architecture/assets/126971054/7975cbdb-fd73-47b8-bb94-ec5081cdbbbf)


**Configure WordPress to work with DB** <br>
```
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

**STEP 5 - WEB SERVER PREP**<br>

Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client <br>
`sudo yum install mysql`

Now edit mysql configuration file by typing <br>
`sudo vi /etc/my.cnf`<br>

Add the following at the end of the file. <br>
```
[mysqld]
bind-address=0.0.0.0
```

![33333-mysqlddd](https://github.com/ifydevops23/Application_Architecture/assets/126971054/a9d52441-60dd-41d1-b39d-900129d1686c)

Change permissions and configuration so Apache could use WordPress: <br>

`sudo chown -R apache:apache /var/www/html/wordpress` <br>

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP) <br>

On the web server, edit wordpress configuration file.<br>

`cd /var/www/html/wordpress`<br>

 `sudo vi wp-config.php` <br>
 
**_Insert Database PRIVATE IP as DATABASE host_**

![333_wp-config](https://github.com/ifydevops23/Application_Architecture/assets/126971054/e68ab1f1-851b-4675-ae5a-f9dbe5026583)


Disable the default page of apache so that you can view the wordpress on the internet.
```
sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup
```

Restart httpd. <br>

`sudo systemctl restart httpd`

Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases. <br>
`sudo mysql -u <user-name> -p -h <DB-Server-Private-IP-address>`


Try to access from your browser the link to your WordPress http://<Webserver-Public-Ip>/wordpress/
 
![1_welcome_to_wordpress](https://github.com/ifydevops23/Application_Architecture/assets/126971054/5758f8be-1d36-4885-be75-9579d68a749b)

Fill in your credentials to setup your account for your wordpress website. If you see this message – it means your WordPress has successfully connected to your remote MySQL database.<br>

Log in with your username and password.<br>
 
![3333_welcome_to_wordpress](https://github.com/ifydevops23/Application_Architecture/assets/126971054/a773985a-b70c-483a-9463-053ff117d08e)

After Succesful Login;<br>

![333_wordpress_latest](https://github.com/ifydevops23/Application_Architecture/assets/126971054/deb18487-9cbc-4617-b680-b3a1abe30df6)

 
