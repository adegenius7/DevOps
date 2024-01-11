# Three-Tier Architecture
Managing disk space has always been a significant task for sysadmins. Running out of disk space used to be the start of a long and complex series of tasks to increase the space available to a disk partition. It also required taking the system off-line. This usually involved installing a new hard drive, booting to recovery or single-user mode, creating a partition and a filesystem on the new hard drive, using temporary mount points to move the data from the too-small filesystem to the new, larger one, changing the content of the /etc/fstab file to reflect the correct device name for the new partition, and rebooting to remount the new filesystem on the correct mount point.
![lvm](./img/basic-lvm-volume.png)
LVM allows for very flexible disk space management. It provides features like the ability to add disk space to a logical volume and its filesystem while that filesystem is mounted and active and it allows for the collection of multiple physical hard drives and partitions into a single volume group which can then be divided into logical volumes.
### **<br>In this project** i will be implementing a mobile solution based on a **Three-tier Architecture**
![tier](./img/three-tier-diagram.png)
### I will be spinning up an EC2 instance which will use a `RedHat OS`![OS](./img/Red-Hat-logo_2.original.jpg) and serves as a **Webserver which Wordpress and our clientDB will reside** Launch the EC2 instance![instancecreation](./img/WebServer_instance_creation.png) 
### **_____________________________________________________________**
###  Then ssh into the instance to open up Linux terminal to setup the configuration.![connect](./img/WebServer_instance_connect.png)![connect](./img/WebServer_instance_connect1.png)![connect](./img/WebServer_instance_ssh.png)

 ### will be spinning up another EC2 instances which will use a `RedHat OS` and serves as a **Database server which our mysql serverDB will reside** 
 ![DBserver](./img/DB_server_launch.png)

 ### Check Availabilty Zones of the instances
 - Availability Zones are multiple, isolated locations within each Region. Local Zones provide you the ability to place resources, such as compute and storage, in multiple locations closer to your end users.![zone](./img/Availability_Zones.png)

### Create 3 volumes in the same Availability Zone on AWS
![volume creation](./img/volume_creation0.png) ![volumecreation](./img/volume_creation1.png) ![volume creation](./img/volume_creation2.png) ![volume creation](./img/volume_creation3.png) ![volume creation](./img/volume_creation4.png)

### Attach all three volumes one after the other to the **Webserver** EC2 instance  ![volume Attach](./img/volume_Attach0.png)![volume Attach](./img/volume_Attach1.png)![volume Attach](./img/volume_Attach2.png)

### Inspect that your volumes are succesfully attached
![volume Attach](./img/volume_Attached.png)
This can also be verified from the Terminal using `lsblk` command![volume Attach](./img/lsblk_attached_vol.png)

## CREATION OF PARTITIONS
A partition is a logical division of a hard disk that is treated as a separate unit by operating systems (OSes) and file systems.
Partitioning allows the use of different filesystems to be installed for different kinds of files. Separating user data from system data can prevent the system partition from becoming full and rendering the system unusable. Partitioning can also make backing up easier.
This is done with the use of the `gdisk` command
The gdisk command scans the specified hard disk for existing partitions and prints the result.
`sudo gdisk /dev/xvdf` we input `n` which is "NEW" for a new partition
![gdisk](./img/gdisk_Partition1.png)Partition number ranges from 1 -128, hence we choose any number of our choice. In this project i have chosen
- 11 for xvdf -----> xvdf11
- 12 for xvdg -----> xvdg12
- 13 for xvdh -----> xvdh13
![gdisk](./img/gdisk_Partition2.png)
We can proceed to click `ENTER` if we desire to use the entire space for our partition, else we can specify the required space we desire![gdisk](./img/gdisk_Partition3.png)![gdisk](./img/gdisk_Partition4.png) A GUID will be requested for our desired partition.
**A GUID (globally unique identifier) is a 128-bit text string that represents an identification (ID). Organizations generate GUIDs when a unique reference number is needed to identify information on a computer or network. A GUID can be used to ID hardware, software, accounts, documents and other items. Linux LVM is 8e00** we should end up with this![gdisk](./img/linux_lvm_GUID.png) `p` can be used to inspect our partition before writing which will be done with `w`.
![gdisk](./img/gdisk_p2inspect.png)![gdisk](./img/gdisk_w2write.png) `yes to exit`
![gdisk](./img/gdisk_succesful.png)

## INSTALL LVM2
In Linux, Logical Volume Manager (LVM) is a device mapper framework that provides logical volume management for the Linux kernel.package using `sudo yum install lvm2`
LVM2, is the default for Red Hat Enterprise Linux, which uses the device mapper driver contained in the 2.6 kernel.
![lvm install](./img/install_lvm2.png) ![lvm install](./img/install_lvm2_.png) 

we can now inspect to see if the kernel has LVM to handle the logical volumes using `which lvm2`![lvm install](./img/install_lvm2_which.png) 
![lvm install](./img/install_lvm2_which.png)

## PHYSICAL VOLUME CREATION
**You cannot create a physical volume directly on a device, it has to be in a partition. Hence we use our created Partitions `xvdf11 xvdg12 and xvdh13`.** The command to use for this is `pvcreate` thus
`sudo pvcreate /dev/xvdf11`
`sudo pvcreate /dev/xvdg12`
`sudo pvcreate /dev/xvdh13` More easily we can create these volumes at once as shown in the project image below
![pv create](./img/physical_vol_create_all.png)
![pv create](./img/physical_vol_create_success.png)
This can be confirmed using `sudo pvs`![pv create](./img/physical_vol_confirmation.png)

## VOLUME GROUP CREATION
This is a concactenation of multiple physical volumes to becomne a **VolumeGroup** VG so that we can then use it logically.
It is the logical volumes that we can give to our servers. You do not give VGs to servers.(Remember we installed LVM for the kernel). For this we use the command `vgcreate` this way `sudo vgcreate webdata-vg /dev/xvdh13 /dev/xvdg12 /dev/xvdf11` and our VG `webdata-vg` can now be verified using `sudo vgs`
![volume group](./img/volumeGroups.png)
! ![volume group](./img/volumeGroups_confirmation.png)

## LOGICAL VOLUME CREATION
LVM2, can be used to gather existing storage devices into groups and allocate logical units from the combined space as needed.

The main advantages of LVM are increased abstraction, flexibility, and control. Logical volumes can have meaningful names like “databases” or "root-backup”. Volumes can also be resized dynamically as space requirements change, and migrated between physical devices within the pool on a running system or exported. LVM also offers advanced features like snapshotting, striping, and mirroring.

In this project we use `lvcreate` command to create two logical volumes. **apps-lv and logs-lv** Here the apps-lv will store data for the website, while the logs-lv will store data for logs.
`sudo lvcreate -n apps-lv -L 14G webdata-vg` ![lvs creation](./img/logical_volume1_apps.png)

`sudo lvcreate -n logs-lv -L 14G webdata-vg` ![lvs creation](./img/logical_volume2_logs.png)
We can confirm by running `sudo lvs`
![lvs creation](./img/logical_volumes_confirmation.png)
we can use `sudo vgdisplay -v` to view our complete set up ![lvs creation](./img/full_partition_vgdisplay.png)

## FORMAT LVS USING EXT4
ext4 enables write barriers by default. It ensures that file system metadata is correctly written and ordered on disk, even when write caches lose power. This goes with a performance cost especially for applications that use fsync heavily or create and delete many small files
`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`
![format](./img/format_applv2ext4.png)![format](./img/format_loglv2ext4.png)

## MAKING WEBSITE DIRECTORIES
To serve our web content, we need to have a directory `sudo mkdir -p /var/www/html` **the flag p is used because the www directory does not exit**
![wbpage dir](./img/webpage_directory.png)![wbpage dir](./img/webpage_directory0.png)

we then need to make a directory for log which we'll later use for back up`sudo mkdir -p /home/recovery/logs`
![log](./img/recovery_log_directory.png)
## MOUNTING ON OUR LVs
Mounting is synonymous to overwriting what is in the destination drive. Hence it is always a good practice to check the destination drive if empty so as to avoid overwrite.
`ls -l /var/log`![mount check](./img/mount_check_directory_log.png)
This directory is not empty, hence we have to use the `rsync` command to transfer the content to our recovery directory before mounting **/var/log to logs-lv**
`sudo rsync -av /var/log/. /home/recovery/logs/`![rsync](./img/rsync_backup.png) we then mount on the drive
![mount](./img/mount_confirmation_log.png)
Restore log files back to **/var/log** `sudo rsync -av /home/recovery/logs/log/. /var/log`![restore](./img/rsync_backup_restore.png)]

we do same for our html directory
`ls -l /var/www/html`![mount check](./img/mount_check_directory.png)

![mount](./img/mount_directory.png)

We can now confirm our mounts by using `df -h` command ![mount](./img/mount_df-h.png)
**However this is a temporary mount, such that if this instance is restarted, the mount will be discarded. We have to make it persist by 
-   we need to first check the block ID using `sudo blkid` command which will reveal the UUID ![UUID](./img/UUID.png) this ID is same as our logical volume.
<br>Next is to update our **/etc/fstab** file so that the mount configuration will persist after restart of the server, this will be done with the UUID generated<br>
`sudo vi /etc/fstab`
![fstab](./img/fstab_edit.png)
**Save && Exit**
`sudo mount -a` can now be used to test if there's no error in the configuration.
![fstab_test](./img/error_test.png)
**Return value must be 0**
we reload the daemon now using`sudo systemctl daemon-reload`![reload](./img/daemon-reload.png)


## INSTALLING WORDPRESS AND CONFIGURING IT TO USE MYSQL DATABASE

### PREAPARING THE DBSERVER
Remember we have a second RedHat OS instance named DBserver![db](./img/DB_server_launch.png)
![db](./img/DB_server_launch_.png)

We will need to repaet the same steps for this server too.
![vol](./img/volumes.png)

**So that we can end up with**![vol](./img/db-lv.png)

### INSTALL WORDPRESS ON WEBSERVER EC2

1. we start by updating our server using `sudo yum update -y` command 
2. install wget, Apache and it's dependencies using `sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json` ![apache](./img/apache_php_dependencies.png)![apache](./img/apache_php_dependencies_.png)
3. start Apache using `sudo systemctl enable httpd`![httpd](./img/enable_httpd.png)
`sudo systemctl start httpd`![httpd](./img/start_httpd.png)



First, install the EPEL repository.
`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`
![php](./img/php_version01.png)

Then check the available version of php using `sudo dnf module list php`![php](./img/php_version.png)



Next, install yum utils and enable remi-repository using the command below. `sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.1rpm`

![php](./img/php_8_1.png)
We are going to install the latest version of PHP ( PHP 8.1 by the time of penning down this guide) using the Remi repository.
![php](./img/enable_php.png)

Finally, install PHP, PHP-FPM (FastCGI Process Manager) and associated PHP modules using the command.`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`![php](./img/install_php_.png)

Check the Version installed using `php -v`![php](./img/version_installed_php.png)

Start the php using the `sudo systemctl start php-fpm`
enable the php using`sudo systemctl enable php-fpm`
and you can also check the status using `sudo systemctl status php-fpm`![php](./img/php_start_enable_status.png)

**To instruct SELinux to allow Apache to execute the PHP code via PHP-FPM run.** `sudo setsebool -P httpd_execmem 1`![security](./img/boolean.png)
**This is important in RedHat**

### WordPress

#### WordPress (also known as WP or WordPress.org) is a web content management system. It was originally created as a tool to publish blogs but has evolved to support publishing other web content, including more traditional websites, mailing lists and Internet forum, media galleries, membership sites, learning management systems and online stores.
In this project, WordPress will be Downloaded and copied to **our var/www/html** on our Webserver EC2 instance
make a directory called wordpress `mkdir wordpress` and then `cd wordpress`![wp](./img/wordpress_directory.png)In this directory we will download wordpress and configure it.
### Download wordPress
`sudo wget http://wordpress.org/latest.tar.gz` ![wp](./img/wordpress_zip.png)This will be download as a Zip folder. ![wp](./img/wordpress_zip_ls.png)
`sudo tar xzvf latest.tar.gz` to unzip the folder ![wp](./img/wordpress_unzip.png) ![wp](./img/wordpress_unzip_folder.png)
`cd wordpress` and `ls -l` to view the installation files ![wp](./img/wordpress_unzip_folder_cd_ls.png)
Next is to copy the configuration file into a newly created file for convenience using `sudo cp -R wp-config-sample.php wp-config.php`![wp_config](./img/wordpress_config_file.png) **Ensure to go back a step in the directory to copy your content Thus**`cd .. && ls`![wp_config](./img/wordpress_cd.png) Then Copy using `sudo cp -R wordpress/ /var/www/html/`![wp_config](./img/wordpress_content.png) and then change the directory and list the content using `cd /var/www/html && ls`![wp_config](./img/wordpress_content_.png)  
#### Install mysql server on both webserver and DB instances
`sudo yum -y install mysql-server`![mysql](./img/mysql_on_both_server.png)
#### Establish a Connection between the mysql databases where that of the Webserver will act as the Client

start and enable the mysql database
`sudo systemctl start mysqld` `sudo systemctl enable mysqld`
![mysqld](./img/mysql_both_server_status.png)

##### Configure the mysql DBServer
`sudo mysql_secure_installation`![secure](./img/mysql_DBserver_secure_installation.png) ![secure](./img/mysql_DBserver_secure_installation1.png) 
We can now login to it  ![secure](./img/mysql_DBserver_login.png)

Create a Database called `WordPress`![secure](./img/mysql_DBserver_wordpressDB.png)
Create a User for this DB `CREATE USER 'wordpressUSER'@'%' IDENTIFIED WITH mysql_native_password BY Adewole@1;` **The `%` here can be replaced with the private ip address of our client DB instance which will make our Db more secure rather than % which makes it connect from anywhere**
Grant all priviledges to the user `GRANT ALL PRIVILEGES ON *.* TO 'wordpressUSER'@'%' WITH GRANT OPTION;` The first asterick means DB and second is access to tables 
Then Flush
`FLUSH PRIVILEGES`
![secure](./img/mysql_DBserver_USER.png)
Confirm the user
`select user, host from mysql.user`![users](./img/mysql_USERs.png) **Take note of the % sign which will make us connect from any machine as against localhosts**
##### EXIT EXIT EXIT
#### Open configuration file
`sudo vi /etc/my.cnf`![cnf](./img/cnf.png)
include the following content to connect from any server using the 0.0.0.0![cnf](./img/cnf_bind_address.png)
**Ensure to always restart after changing a configuration** `sudo systemctl restart mysqld`![mysql_restart](./img/mysql_restart.png)

#### Configure Your Webserver
Let's go back to our wp-config.php file ![config](./img/webserver_config_php.png)
we configure our file thus![config](./img/wordpress_config_file_php.png)

Restart the httpd![restart](./img/restart_httpd.png)

##### Disable/Rename Appache default page so as to see your configuration served
`sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup`![disable](./img/disable.png)

##### Check if you can connect to your DB from the webserver by first enabling mysql from inbound on your DBinstance ![rules](./img/msql_port_3306.png) using `sudo mysql -h 172.31.34.228 -u wordpressUSER -p`
![connected](./img/mysql_connected.png)

##### Verify the privileges
![verify](./img/verify_privileges.png) then exit

To Grant Apache the access to use the configuration file we need to change the ownership, the initial owner is root as shown below![own](./img/chown01.png) Use `sudo chown -R apache:apache /var/www/html`![own](./img/chown02.png)
We also have to change the configuration so Apache can use wordPress `sudo chcon -t httpd_sys_rw_content_t /var/www/html -R` and `sudo setsebool -P httpd_can_network_connect=1` and this `sudo setsebool -P httpd_can_network_connect_db 1` to connect it to db![ch](./img/chcon.png)
#### PASTE THE PUBLIC IP OF YOUR WEBSERVER![WP](./img/WORDPRESS.png)![WP](./img/WORDPRESS2.png)![WP](./img/WORDPRESS3.png)![WP](./img/WORDPRESS4.png)![WP](./img/WORDPRESS5.png)
![Thank you](./img/Thank_you.jpg)