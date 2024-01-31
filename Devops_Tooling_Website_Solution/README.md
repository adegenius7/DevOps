#  Devops Tooling Website Solution
## NFS Server Preparation
AWS provides three popular services — S3, Elastic Block Store (EBS), and Elastic File System (EFS) — which work quite differently and offer different levels of performance, cost, availability, and scalability. 

**Amazon S3**provides simple object storage, useful for hosting website images and videos, data analytics, and both mobile and web applications. Object storage manages data as objects, meaning all data types are stored in their native formatsThere is no hierarchy of relations between files with object storage — data objects can be distributed across several machines. You can access the S3 service from anywhere on the internet.

**AWS EBS** provides persistent block-level data storage. Block storage stores files in multiple volumes called blocks, which act as separate hard drives; block storage devices are more flexible and offer higher performance than regular file storage. You need to mount EBS onto an Amazon EC2 instance. Use cases include business continuity, software testing, and database management. Learn more about EBS volume types and common operations.

**AWS EFS** is a shared, elastic file storage system that grows and shrinks as you add and remove files. It offers a traditional file storage paradigm, with data organized into directories and subdirectories. EFS is useful for SaaS applications and content management systems. You can mount EFS onto several EC2 instances at the same time. 

### Implementation
We are going to be launching 4 (**3 Webservers and one for our NFS**) instances for this project on AWS with RHEL Linux 8 Operating System ![rhel](./img/RHEL_instances.png)

We will be having another instance for our DB which will run on an UBUNTU OS![db](./img/DB-instance.png)

We should end up with ![all](./img/All_instances.png)

Next is to have 3 volume attached to our NFS just like the previous project![vol](./img/NFS_vols.png)![vol](./img/attach_vol.png)
### Connect to instance using ssh
![ssh](./img/ssh.png)

### Inspect the blocks available
![lsblk](./img/lsblk.png)
### create a partition for all three volumes to be used for logical volumes
![gdisk](./img/gdisk.png)

To end upwith a disk like this ![gdisk](./img/gdisk_all.png)
### Install lvm2
![lvm](./img/lvm_installation.png)

### Create a physical volume and a volume group
![pv](./img/pv_vg.png)

### Create 3 logical volumes 
- lv-opt
- lv-apps
- lv-logs
### ![log](./img/logical_volumes.png)

### Format the 3 disks as xfs![pv](./img/xfs.png)
### Make a directory `/mnt/apps` and mount the lv-apps there![mount](./img/mount_lvapps.png) This will be used by the webservers
### Make a directory `/mnt/logs` and mount the lv-logs there![mount](./img/mount_lvlogs.png) This will be used by the webservers logs

### Make a directory `/mnt/opt` and mount the lv-opt there![mount](./img/mount_lvopt.png) This will be used by the jenkins server in the next project

## INSTALL NFS SERVER AND CONFIGURE IT THUS
`sudo yum -y update`![update](./img/nfs_update.png)
`sudo yum install nfs-utils -y`![utils](./img/nfs_utils.png)
`sudo systemctl start nfs-server.service`![start](./img/nfs_start.png)
`sudo systemctl enable nfs-server.service`![enable](./img/nfs_enable.png)
`sudo systemctl status nfs-server.service`![status](./img/nfs_status.png)

### Set up Permission that will allow our Webservers to -rwx files on NFS `sudo chown -R nobody: /mnt/apps``sudo chown -R nobody: /mnt/logs``sudo chown -R nobody: /mnt/opt``sudo chmod -R 777 /mnt/apps``sudo chmod -R 777 /mnt/logs``sudo chmod -R 777 /mnt/opt``sudo systemctl restart nfs-server.service`![perm](./img/perm.png)

### Configure access to NFS for clients within the same subnet by writing this `/mnt/apps <Subnet-CIDR of client webserver>(rw,sync,no_all_squash,no_root_squash)/mnt/logs <Subnet-CIDR of client webserver>(rw,sync,no_all_squash,no_root_squash)/mnt/opt <Subnet-CIDR of client webserver>(rw,sync,no_all_squash,no_root_squash)` into the exports file using `sudo vi /etc/exports`![export](./img/exports.png) substitute the CIDR. Then use the command `sudo exportfs -arv` for export ![export](./img/exportfs.png) of these files to our webserver
### Confirm the Ports being used by NFS`rpcinfo -p | grep nfs` and Open it Using the SG i.e Add new inbound rule![ports](./img/nfs_ports.png)**In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049 ** ![ports](./img/ports.png)
# BACKEND DATABASE CONFIGURATION
### Connect to the database instance using ssh ![ssh](./img/ssh_ubuntu_db.png) Then update and install mysql ![mysql](./img/ypdate_install_mysql.png)![mysql](./img/ypdate_install_mysql1.png) Create a database named `tooling`![mysql](./img/db_tooling.png) Create a Database User and name it `webaccess` and Grant permission to the user only from the webservers `subnet cidr` ![mysql](./img/db_user.png)![mysql](./img/db_priv.png) Confirm if all is intact ![mysql](./img/db_confirm.png)

# Prepare the Webservers

### 1. Connect to the webserver instance using ssh ![webserver](./img/webserver_ssh.png) Install NFS client on this server using `sudo yum install nfs-utils nfs4-acl-tools -y`![nfsclient](./img/nfs_client.png)

### 2. Mount `/var/www` and target the NFS's servers export for apps using `sudo mkdir /var/www`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www![mount](./img/mount_webserver.png)

### 3. we can test this mount by creating a file on the webserver to see it on the NFS backend server ![file](./img/webserver_data.png)![file](./img/nfs_test.png)

### 4. Make sure that the changes done on the webserver will persist by running `sudo vi /etc/fstab` ![fstab](./img/fstab_.png) ![fstab](./img/fstab.png)![fstab](./img/fstab_updated.png)on the webserver 

### 5. Install a WEbserver agent like Apache or NGINX and php to serve your webcontent, Apache will be used in this project. This will be installed from Remi Repository thus
### `sudo yum install httpd -y`![install](./img/update_install_httpd.png)![install](./img/update_install_httpd1.png) `sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`![install](./img/update_install_httpd_package_manager.png) `sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm` ![install](./img/update_install_httpd_package_manager1.png) `sudo dnf module reset php`![reset](./img/reset_php.png) `sudo dnf module enable php:remi-7.4`![enable](./img/enable_php.png) `sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`![install](./img/install_php.png) `sudo systemctl start php-fpm``sudo systemctl enable php-fpm``setsebool -P httpd_execmem 1`![start](./img/start_enable_boolean.png)

### check the test page of the webserver ![test](./img/RHELwebserverTestPage.png)

## Repeat steps 1 to 5 for another 2 Webservers

### 6. Verify that Apache files and directories areavailable on Webserver in `/var/www` and also on NFS server in `/mnt/apps`![verify](./img/apache_on_nfs.png) ![verify](./img/apache_on_webserver.png)

### 7. Locate the Log Folder for Apache on the Webserver and mount it to NFS Server's export for logs.![log](./img/apache_log.png)![log](./img/apache_log_mount.png) Repeat step 4 to ensure that your log persist![log](./img/fstab_log.png)![log](./img/fstab_log1.png)

## Tooling
### Install git on your webserver using `sudo yum install git -y`![git](./img/install_git.png) then Fork the tooling source code from `https://github.com/darey-io/tooling`![sc](./img/source_code.png) ![git](./img/git_clone.png) ![git](./img/git_tooling.png) while in the tooling Directory, Copy the html content in the tooling repo into `/var/www/html` thus `sudo cp -R html/. /var/www/html` ![git](./img/html_clone.png) **Open port 80 on the webservers ** ![port](./img/port_80.png) Disable SELinux `sudo setenforce 0``sudo vi /etc/sysconfig/selinux` and set `SELINUX=disabled` then restart httpd ![SELinux](./img/SELinux1.png)![SELinux](./img/SELinux.png) and make this change permanent 
### Update the Website's Configuration to connect to the database thus ![db](./img/db_config0.png) ![db](./img/db_config.png) 
### Apply `tooling-db.sql` script to your database using this command `sudo mysql -h <database_privateIP> -u <db-username> -p <passwordDB> < tooling-db.sql` from the tooling directory ![db](./img/mysqlSG.png)![db](./img/DB_script.png)
1. ###  this will require us to change the bind-address on our DB-Webserver instance`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`![DB](./img/DB-mysqld1.png)![DB](./img/DB-mysqld.png)![DB](./img/DB_restart.png)
2. ### mysql also has to be installed before we use the script ![mysql](./img/webserver_mysql_install.png)

### The script creates new user which can be verified thus ![script](./img/DB_script_UserCreation.png)
### Disable the welcome page on the webserver so it serves our content ![ds](./img/welcomePageDisabled.png) and restart the httpd

### load webserver with publicip/index.php
![web1](./img/web1.png)![web1](./img/admin.png)

















