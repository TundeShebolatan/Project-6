# Project 6: WEB SOLUTION WITH WORDPRESS

Step 0:  Preparing prerequisites

- Sign in to AWS free tier account and create two new EC2 Instances of t2.nano family with RedHat OS
- Name the new instances
- Server A name - "web-server"
- Server B name - "database-server"
  
Step 1:  Creating and adding EBS Volume to EC2 instances

- Create 3 volumes , each of 10 GiB (web1, web2 and web3) and (db1, db2 and db3) and attach them to the EC2 instances, "web-server" and "database-server" respectively.
- Connect to these instances via SSH on two different windows terminals renamed as;
- Server A name - "web-server"
- Server B name - "database-server"

![Two instances created](images/2-instances.PNG)

Tasks:

1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.
2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.

Open up the Linux terminal to begin configuration

Use `lsblk` command to inspect what block devices are attached to the server

for web-server...

![inspect attached devices](images/lsblk-webServer.PNG)

for database-server...

![inspect attached devices](images/lsblk-dbServer.PNG)

Use `df -h` command to see all mounts and free space on your server

![mount status web-server](images/df-h-webserver.PNG)

Use `gdisk` utility to create a single partition on each of the 3 disks

for web-server...

`sudo gdisk /dev/xvdf`

![partitioning xvdf volume](images/gdisk-xvdf-webServer.PNG)

`sudo gdisk /dev/xvdg`

![partitioning xvdg volume](images/gdisk-xvdg-webServer.PNG)

`sudo gdisk /dev/xvdh`

![partitioning xvdh volume](images/gdisk-xvdh-webServer.PNG)

for database-server...

`sudo gdisk /dev/xvdf`

![partitioning xvdf volume](images/gdisk-xvdf-dbServer.PNG)

`sudo gdisk /dev/xvdg`

![partitioning xvdg volume](images/gdisk-xvdg-dbServer.PNG)

`sudo gdisk /dev/xvdh`

![partitioning xvdh volume](images/gdisk-xvdh-dbServer.PNG)

Use `lsblk` utility to view the newly configured partition on each of the 3 disks

![partitioning confirmed](images/lsblk2-dbServer.PNG)

Install lvm2 package

`sudo yum install lvm2`

for web-server...

![install lvm2](images/install-lvm2.PNG)

for database-server...

![install lvm2](images/install-lvm2-dbServver.PNG)

Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

`sudo pvs`

for web-server...

![physical volumes](images/pvcreate-webServer.PNG)

![pvcreate status](images/pvcreate-successful-webServer.PNG)

for database-server...

![physical volumes](images/pvcreate-dbServer.PNG)

Use vgcreate utility to add all 3 PVs to a volume group (VG).

for web-server...

Name the VG webdata-vg

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![creating volume group](images/webdata-vg-created-webServer.PNG)

![webdata volume group confirmed](images/volumeGroup-confirmed-webServer.PNG)

for database-server...

Name the VG database-vg

`sudo vgcreate database-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![creating volume group](images/database-vg-ceated-dbServer.PNG)

![database volume group confirmed](images/database-vg-created-confirmed-dbServer.PNG)

for web-server...

Use lvcreate utility to create 2 logical volumes, apps-lv and logs-lv, and use half of the PV size for each logical volumes on the web-server.

`sudo lvcreate -n apps-lv -L 14G webdata-vg`

`sudo lvcreate -n logs-lv -L 14G webdata-vg`

for database-server...

`sudo lvcreate -n db-lv -L 20G database-vg`

![creating volume group](images/db-lv-created-dbServer.PNG)

Verify that your Logical Volume has been created successfully by running

`sudo lvs`

![lv creation confirmed](images/lvs-confirmation-webServer.PNG)

![lv creation confirmed](images/db-lv-created-dbServer.PNG)

Verify the entire setup

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

`sudo lsblk`

![setup verified](images/verify-setUp-webServer.PNG)

![setup verified](images/sudo-lsblk-webServer.PNG)

Use "mkfs.ext4" to format the logical volumes with ext4 filesystem

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`

`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![format apps-lv](images/format-apps-lv-webServer.PNG)

![format logs-lv](images/format-logs-lv-webServer.PNG)

Create /var/www/html directory to store website files

`sudo mkdir -p /var/www/html`

![directory created ](images/var-www-html-created-webServer.PNG)

Create /home/recovery/logs to store backup of log data

`sudo mkdir -p /home/recovery/logs`

![directory created](images/home-recovery-logs-webServer.PNG)

Mount /var/www/html on apps-lv logical volume

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

![Mount confirmed](images/Mount-on-apps-lv-confirmed.PNG)

Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

`sudo rsync -av /var/log/. /home/recovery/logs/`

![bak up status](images/resync-log-directory-webServer.PNG)

![back up confirmed](images/rsync-log-directory-confirmed-webServer.PNG)

Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)

`sudo mount /dev/webdata-vg/logs-lv /var/log`

![mount confirmed](images/Mount-on-logs-lv-confirmed.PNG)

Restore log files back into /var/log directory

`sudo rsync -av /home/recovery/logs/. /var/log`

![log files restored](images/restore-log-files.PNG)
![log files restored continued](images/restore-log-files2.PNG)

`sudo ls -l /var/log`

![restored log files confirmed](images/restore-log-files-confirmed-webServer.PNG)

Update "/etc/fstab" file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file;

`sudo blkid`

`sudo vi /etc/fstab`

![open file](images/open-etc-fstab.PNG)

![edit file](images/Update-etc-fstab-webServer.PNG)

Test the configuration and reload the daemon

`sudo mount -a`

`sudo systemctl daemon-reload`

![test the configuration](images/Update-etc-fstab-validation-status-webServer.PNG)

![restart the daemon](images/daemon-reloaded-webServer.PNG)

Verify your setup by running

`df -h`

![setup verified](images/verify-setUp2-webServer.PNG)

Back to the database-server
create the mount point in root directory and add a filesystem

`sudo mkdir /db`

`sudo mkfs -t ext4 /dev/database-vg/db-lv`

![create-db-directory](images/create-db-directory-in-root-dbServer.PNG)

![add a filesystem](images/add-filesystem-dbServer.PNG)

mount db-lv unto /db

`sudo mount /dev/database-vg /db-lv`

![Mount status](images/Mount-on-db-lv-confirmed-dbServer.PNG)

make it persistent, open and edit /etc/fstab using your the UUID

`sudo blkid`

![extract UUID](images/extracting-blockID.PNG)

![open and edit](images/open-etc-fstab.PNG)

![Update](images/Update-etc-fstab-dbServer.PNG)

Test the configuration and reload the daemon

`sudo mount -a`

`sudo systemctl daemon-reload`

![reload daemon](images/daemon-reloaded-dbServer.PNG)

Install WordPress on your Web Server EC2
Update the repository

`sudo yum -y update`

![update status](images/update-repo-webServer.PNG)

![update status](images/update-repo-dbServer.PNG)

Install wget, Apache and it’s dependencies

Install Epel repository first

`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![Install Epel repository](images/install-EPEL-repository-webServer.PNG)

Enable remi repo by running

`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![remiL-repository](images/install-remiL-repository-webServer.PNG)

`sudo yum module list php`

`sudo yum module reset php`

![module list](images/module_list.PNG)

![module reset](images/reset-webServer.PNG)

`sudo yum module enable php:remi-8.0`

`sudo yum install php php-opcache php-gd php-curl php-mysqlnd`

![install php](images/Installing-PHP-webServer.PNG)

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`sudo systemctl status php-fpm`

![restart and enable PHP fpm](images/restart-and-reboot-webServer.PNG)

![PHP fpm active](images/PHP-status-webServer.PNG)

`sudo setsebool -P httpd_execmem 1`

![setsebool](images/setsebool-command.PNG)

Restart Apache

`sudo systemctl restart httpd`

![restart Apache](images/starting-httpd-webServer.PNG)

If you search the web with the public IP address of the web-server, the page below should appear

![Apache test page](images/Apache-status-confirmed-webServer.PNG)

Download wordpress and copy wordpress to var/www/html

`mkdir wordpress`

`cd wordpress`

![mkdir and cd into wordpress](images/mkdir-cd-wordpress-webServer.PNG)

Now download wordpress

`sudo wget http://wordpress.org/latest.tar.gz`

![sudo wget](images/install-wget.PNG)

Extract zip file

`sudo tar xzvf latest.tar.gz`

![extract wordpress zip](images/extract-tgz-webServer.PNG)

sudo into wordpress/

`sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php`

![download wordpress](images/downloaded-wordpress-files.PNG)

copy content in wordpress into var-www-html

`cp -R wordpress /var/www/html/`

![copy content successful](images/copy-content-in-wordpress-into-var-www-html.PNG)

Install MySQL Server on both the web-server and the database-server

`sudo yum install mysql-server`

![Install MySQL Server](images/install-mysql-server-as-client-webServer.PNG)

![Install MySQL Server](images/install-mysql-server-dbServer.PNG)

`sudo systemctl restart mysqld`

`sudo systemctl enable mysqld`

![restart mysqld](images/systemctl-restart-mysqld-dbServer.PNG)

![enable mysqld](images/mysql-enabled-webserver.PNG)

![enable mysqld](images/mysql-enabled-dbserver.PNG)

Connect to MySQL

![connecting to MySQL](images/connect-to-mysql-dbServer.PNG)

![connection successful](images/connected-to-msql-dbServer.PNG)

Create and show databases

![Create and show databases](images/create-show-databases-dbServer.PNG)

Create a Mysql user to connect to granting all privileges. Then flush privileges and exit

![Create a Mysql user](images/create-mysql-user-and-password-dbServer.PNG)

Next will be to set the bind address of the database

open the database configuration file

`sudo vi /etc/my.cnf

![open the config file](images/open-db-config-file-dbServer.PNG)

update the bind address

![set the bind address of the database](images/edit-db-config-file-dbServer.PNG)

Also edit the PHP configuration file

`sudo vi wp-config.php`

![edit PHP configuration file](images/edit-wp-config-php-file.PNG)

 Test that you can connect from your Web Server to your DB server

 Disable Apache homepage first

 `sudo mv /etc/httpd/conf.d/welcome.conf/ /etc/httpd/conf.d/welcome.conf_backup

![Disable Apache](images/disable-apache-homePage-webServer.PNG)

`sudo mysql -u admin -p -h 172.31.38.193 -u wordpress -p`

![test status](images/communication-btw-webserver-and-database-established.PNG)

Configure SELinux Policies so that Apache can access wordpress files

`sudo chown -R apache:apache /var/www/html/`

![access permissions granted](images/Grant-apache-access-to-root.PNG)

`sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`

![chcon](images/chcon-command.PNG)

`sudo setsebool -P httpd_can_network_connect=1`

![setsebool](images/setsebool-command.PNG)

Now refresh your browser to see the landing page for WordPress

![WordPress landing page](images/wordpress-webpage-loaded-1.PNG)

Click continue

![form page](images/wordpress-webpage-loaded-2.PNG)

fill the form and click on "install WordPress"

![acknowledgement page](images/wordpress-webpage-loaded-3.PNG)

Then Log-in to WordPress

![welcome page](images/wordpress-webpage-loaded-4.PNG)
