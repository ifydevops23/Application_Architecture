## LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”.<br>
*Step 1 — Prepare a Web Server
Launch an EC2 instance that will serve as "Web Server"*. 
Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.<br>
Open up the Linux terminal to begin configurationUse `lsblk` command to inspect what block devices are attached to the server.<br>
Notice names of your newly created devices. All devices in Linux reside in /dev/ directory.<br>
Inspect it with `ls /dev/` and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.<br>
Use `df -h` command to see all mounts and free space on your server.Use gdisk utility to create a single partition on each of the 3 disks `sudo gdisk /dev/xvdf`<br>
Now, your changes has been configured succesfuly, exit out of the gdisk console and do the same for the remaining disks.<br>
Use `lsblk` utility to view the newly configured partition on each of the 3 disks. <br>
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

## PREPARE THE SOFTWARE STACK<br>
**INSTALL APACHE, PHP AND DEPENDENCIES <br>

- Update the repository
`sudo yum -y update`

- Install wget, Apache and it’s dependencies <br>
`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`



- Start Apache <br>
`sudo systemctl start httpd`<br>


- To install PHP and its depemdencies
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1
Restart Apache

 `sudo systemctl restart httpd`

**DOWNLOAD WORDPRESS** <br>

`sudo mkdir -p /var/www/html`
- Download wordpress and copy wordpress to var/www/html

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

**DB SERVER PREP**<br>
- Install Mysqld server
`sudo yum install mysql-server`
- Verify that the service is up and running by using 

`sudo systemctl status mysqld`

- If it is not running, restart the service and enable it so it will be running even after reboot:

`sudo systemctl restart mysqld`
`sudo systemctl enable mysqld`

**Configure WordPress to work with DB** <br>
```
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```


##WEBSERVER PREP <br>

- Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client <br>
`sudo yum install mysql`

- Now edit mysql configuration file by typing <br>
`sudo nano /etc/my.cnf`<br>
Add the following at the end of the file.
```
[mysqld]
bind-address=0.0.0.0
```

- Now, restart mysqld service using 
`sudo systemctl restart mysqld`
- Change permissions and configuration so Apache could use WordPress: <br>

`sudo chown -R apache:apache /var/www/html/wordpress` <br>

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

On the web server, edit wordpress configuration file.<br>

`cd /var/www/html/wordpress`<br>

 `sudo nano wp-config.php`

{Insert webserver private IP as db host}



- Disable the default page of apache so that you can view the wordpress on the internet.
```
sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup
```

- Restart httpd. 

`sudo systemctl restart httpd`



Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases. <br>
`sudo mysql -u admin -p -h <DB-Server-Private-IP-address>



Try to access from your browser the link to your WordPress http://<Webserver-Public-Ip>/wordpress/


Fill in your credentials to setup your account for your wordpress website. If you see this message – it means your WordPress has successfully connected to your remote MySQL database.
Log in with your username and password


 
