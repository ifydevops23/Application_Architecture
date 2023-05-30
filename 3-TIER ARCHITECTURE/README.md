LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”.
Step 1 — Prepare a Web Server
Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
Open up the Linux terminal to begin configurationUse `lsblk` command to inspect what block devices are attached to the server.Notice names of your newly created devices. All devices in Linux reside in /dev/ directory.
Inspect it with `ls /dev/` and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.<br>
Use `df -h` command to see all mounts and free space on your server.Use gdisk utility to create a single partition on each of the 3 disks `sudo gdisk /dev/xvdf`
Now, your changes has been configured succesfuly, exit out of the gdisk console and do the same for the remaining disks.Use `lsblk` utility to view the newly configured partition on each of the 3 disks. <br>
Install lvm2 package using `sudo yum install lvm2`. <br>
Run `sudo lvmdiskscan` command to check for available partitions.<br>
Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM`sudo pvcreate /dev/xvdf1``sudo pvcreate /dev/xvdg1``sudo pvcreate /dev/xvdh1`<br>
Verify that your Physical volume has been created successfully by running `sudo pvs`<br>
Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
Verify that your VG has been created successfully by running `sudo vgs`<br>
Use `lvcreate` utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. <br>
NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs. `sudo lvcreate -n apps-lv -L 14G webdata-vg``sudo lvcreate -n logs-lv -L 14G webdata-vg`<br>
Verify that your Logical Volume has been created successfully by running `sudo lvs`<br>
Verify the entire setup`sudo vgdisplay -v #view complete setup - VG, PV, and LV``sudo lsblk`<br>
Use mkfs.ext4 to format the logical volumes with ext4 filesystem `sudo mkfs -t ext4 /dev/webdata-vg/apps-lv``sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`
Create /var/www/html directory to store website files `sudo mkdir -p /var/www/html` Create /home/recovery/logs to store backup of log data sudo `mkdir -p /home/recovery/logs`<br>
Mount /var/www/html on apps-lv logical volume `sudo mount /dev/webdata-vg/apps-lv /var/www/html/`<br>
Use rsync utility to back up all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system) <br>
`sudo rsync -av /var/log/. /home/recovery/logs/`<br>
Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is veryimportant)<br>
`sudo mount /dev/webdata-vg/logs-lv /var/log`<br>
Restore log files back into /var/log directory <br> 
`sudo rsync -av /home/recovery/logs/. /var/log`<br>
Update /etc/fstab file so that the mount configuration will persist after restart of the server.Click on the next button To update the /etc/fstab file<br>
-UPDATE THE `/ETC/FSTAB` FILE <br>
The UUID of the device will be used to update the /etc/fstab file;<br>
`sudo blkid`<br>
`sudo vi /etc/fstab`<br>
Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.Test the configuration and reload the daemon <br>
`sudo mount -a`<br>
`sudo systemctl daemon-reload`<br>
Verify your setup by running `df -h` output must look like this:<br>

Step 2 — Prepare the Database ServerLaunch a second RedHat EC2 instance that will have a role – ‘DB Server’<br>
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

C. WEBSERVER<br>
Step 3 — Install WordPress on your Web Server EC2 <br>
Update the repository `sudo yum -y update` <br>
Install wget, Apache and it’s dependencies `sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`<br>
Start Apache`sudo systemctl enable httpd``sudo systemctl start httpd`<br>
To install PHP and its dependencies<br>
`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`<br>
`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`<br>
`sudo yum module list php`<br>
`sudo yum module reset php`<br>
`sudo yum module enable php:remi-7.4`<br>
`sudo yum install php php-opcache php-gd php-curl php-mysqlndsudo systemctl start php-fpm --nobest`<br>
`sudo systemctl enable php-fpm`<br>
`sudo setsebool -P httpd_execmem 1`<br>
Restart Apache`sudo systemctl restart httpd`<br>
- Download wordpress and copy WordPress to var/www/html <br>
`mkdir wordpress`<br>
`cd   wordpress`<br>
`sudo wget http://wordpress.org/latest.tar.gz`<br>
`sudo tar xzvf latest.tar.gz`<br>
`sudo rm -rf latest.tar.gz`<br>
`sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php`<br>
`sudo cp -R wordpress /var/www/html/`

Edit the config file `sudo vi wp-config.php`<br>
Configure SELinux Policies <br>
`sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`<br>
`sudo setsebool -P httpd_can_network_connect=1`<br>
`sudo setsebool -P httpd_can_network_connect_db 1`<br>
Disable apache default page to use WordPress <br>
`sudo mv /etc/httpd/conf.d/welcome.conf  /etc/httpd/conf.d/welcome.conf_backup`<br>

— Configure WordPress to connect to the remote database.
Edit my.conf file, set bind address to allow access from remote host 0.0.0.0 <br>
`sudo vi /etc/my.cnf` 
Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client<br>
`sudo yum install mysql`<br>
`sudo mysql -u <username> -p -h <DB-Server-Private-IP-address>`<br>
Verify if you can successfully execute `SHOW DATABASES;` command and see a list of existing databases.

D. DATABASE <br>
Step 4 — Install MySQL on your DB Server EC2 <br>
`sudo yum update``sudo yum install mysql-server`
Verify that the service is up and running by using <br>
`sudo systemctl status mysqld`
If it is not running, restart the service and enable it so it will be running even after reboot:`sudo systemctl restart mysqld``sudo systemctl enable mysqld`
Step 5 — Configure DB to work with WordPress`sudo mysql``CREATE DATABASE wordpress;``CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';`
`GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';`
`FLUSH PRIVILEGES;``SHOW DATABASES;``exit`
Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32


E. TESTINGChange permissions and configuration so Apache could use WordPress:
`sudo chown -R apache:apache /var/www/html/wordpress`

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)Try to access from your browser the link to your WordPress`http://<Web-Server-Public-IP-Address>/wordpress/`
Fill out your DB credentials:If you see this message – it means your WordPress has successfully connected to your remote MySQL database

