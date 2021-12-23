# DEMO VIDEO IS AVAILABLE BELOW  👇
## https://www.youtube.com/watch?v=6nO75HMIAX0

# DATA BACKUP AND SLAVE SQL SERVER
### The database is accessed through webserver and data stored on the system.
### MySQL replicate database (Slave server) for Web Server.
### CentOS 8 Slave SQL server to allow Data Backup via WINDOW SERVER.
### VPN access will be created to facilitate replication and data backups.

![1](https://user-images.githubusercontent.com/71564211/139784343-4cb465c5-1909-42df-b8a9-2fc9ff8abb91.PNG)

## INSTALLING MySQL on WINDOW SERVER machine 

![2](https://user-images.githubusercontent.com/71564211/139784377-2bc8919d-64c3-4598-a340-1a4f7df090ac.PNG)

## CONFIGURING Master Server Replication ON CentOS 8
### BASIC CONFIGURATION
* vi /etc/my.cnf.d 
* bind-address=abc-co ip address
*	server-id=2
*	log_bin=mysql-bin
*	binlog_do_db=project3_test
* systemctl restart mysqld

### GRATING PERMISSION FOR USER
* mysql -u root -p
*	CREATE USER 'project3'@'192.168.10.2' IDENTIFIED BY 'P@ssW0rd';
*	GRANT REPLICATION SLAVE ON *.* TO 'project3'@'WINDOW_SERVER_IPADDRESS';
*	FLUSH PRIVILEGES;

![3](https://user-images.githubusercontent.com/71564211/139784628-612d2979-0fbc-45bc-a127-e42571d36dca.PNG)


### DUMP CREATED DATABASE 
* mysql > USE replication_test;
* FLUSH TABLES WITH READ LOCK;
* SHOW MASTER STATUS;

![4](https://user-images.githubusercontent.com/71564211/139784714-466ddcfd-4403-4d4d-b8dc-95798370e29f.PNG)


### mysqldump 
*	mysqldump -u root -p --opt project3_test > project3_test.sql
*	UNLOCK TABLES;
*	QUIT;

![5](https://user-images.githubusercontent.com/71564211/139784850-cefb7682-2c14-4fbb-a774-c0a763741cac.PNG)

## CONFIGURING DATABASE REPLICATION ON WINDOW SERVER

1. Edit the MySQL server configuration file located in 
### C:\ProgramData\MySQL\MySQL Server 8.0\my.ini

ON [mysqld] section:
*	bind-address= bigguy corp. ip address
*	server-id=1
*	log-bin="mysql-bin"
*	binlog_do_db=project3_test


2. Create a folder called "Data" in C:\Program Files\MySQL\MySQL Server 8.0\ 
* Open cmd.exe
*	cd C:\Program Files\MySQL\MySQL Server 8.0\bin\
*	mysqld --initialize –console

3. Import the replication_test.sql
*	Open cmd.exe
*	cd C:\Program Files\MySQL\MySQL Server 8.0\bin\
*	mysql -u root -p
*	CREATE DATABASE project3_test;
*	mysql -u root -p project3_test < C:/Users/Administrator/Desktop/project3_test.sql

![6](https://user-images.githubusercontent.com/71564211/139785092-232b74b3-b5e8-4e8a-9ece-ab7ecf385e97.PNG)


4. Setup the Slave Server on WINDOW machine
•	CHANGE MASTER TO
•	MASTER_HOST='IP address of ABC Co', 
•	MASTER_PORT=3306,
•	MASTER_USER= 'project3',
•	MASTER_PASSWORD='P@ssW0rd',
•	MASTER_LOG_FILE= 'mysql-bin.00000X',
•	MASTER_LOG_POS=XXX;  
=> XXX is in DUMP DATABASE above

* Then save and restart mysql service
![7](https://user-images.githubusercontent.com/71564211/139785278-e8798ac8-b74a-420e-b957-4c7eadee19e8.PNG)

5. Manual restart sql server
-	Open cmd.exe
-	cd C:\Program Files\MySQL\MySQL Server 8.0\bin\ 
-	To stop the server using the command below:
-	mysqladmin.exe -u root shutdown -p 
-	To start the server…
-	mysqld

![8](https://user-images.githubusercontent.com/71564211/139785282-3f3e675a-fe23-488a-9d16-f6c76df6baba.PNG)

6. Debugging 
* Ensure that, MySQL service is running on proper port and active status

![9](https://user-images.githubusercontent.com/71564211/139785410-95b354a1-5043-43f6-a3fc-51c4128bcf87.PNG)

* Try to use this command when the issue found: mysqld --initialize --console 
* If you get ERROR, you might have to delete all of the content in C:\Program Files\MySQL\MySQL Server 8.0\Data 
* Then run the mysqld --initialize --console command again

## TESTING SLAVE DATABASE SERVER
* ON CentOS 8 machine, login by using root account 
* mysql -u root -p
* Select the proper database backup configured before (project3_test)
* Create the basic database table to test Slave Server 
* CREATE TABLE test(name varchar(30));
* Verify the database automatic replicate on WINDOW SERVER 
-	CREATE database project3_test;
-	USE project3_test;
-	SHOW TABLES;

* Export the mysql dump into db file:
- mysqldump -u root -p --all-databases --master-data > /root/dbdump.db 
* Import mysqldum file again 
-	mysql -u root -p < /root/dbdump.db

## WEB SERVICE WITH MYSQL DATABASE
1. Basic PHP and Apache Web Server installation: 
-	yum install -y httpd
-	yum install -y php php-mysqlnd 
-	systemctl enable httpd
-	systemctl start httpd

2. Simple PHP form to allow add data into the MySQL table
-	cd /var/www/html
-	vim index.html

![10](https://user-images.githubusercontent.com/71564211/139785863-a1240723-0e92-4465-8fb4-1cae6c106347.PNG)

### On CentOS 8 login into MySQL server by using the root Account to verify data collected
-	mysql -u root -p 
-	USE project3_test;
-	CREATE table test3(id int NOT NULL AUTO_INCREMENT, name varchar(255), email varchar(255), issue text, PRIMARY KEY (id)); 
-	DESCRIBE test;

![11](https://user-images.githubusercontent.com/71564211/139785983-736d2e6b-f2a5-46dc-bd23-a1138764aa73.PNG)

-	vim /var/www/html/submit.php
![12](https://user-images.githubusercontent.com/71564211/139786049-eaa21b08-adb0-4844-8542-97de61f74514.PNG)

![13](https://user-images.githubusercontent.com/71564211/139786052-ed000565-a0a2-484d-a69d-9ee08f2f4280.PNG)

-	mysql -u root -p
-	USE project3_test;
-	SELECT * FROM test3;

![14](https://user-images.githubusercontent.com/71564211/139786054-d0882f1e-c872-461c-ac1f-8a65ebe6bd77.PNG)

## DATA BACKUP VIA CRONTAB BASH
-	mkdir /var/lib/mysql/backups
-	mkdir /var/lib/mysql/backup_script
-	vim /var/lib/mysql/backup_script/sql_script.sh
-	chmod +x /var/lib/mysql/backup_script/sql_script.sh

![15](https://user-images.githubusercontent.com/71564211/139786215-b922efd4-3bd9-4eb1-abad-8d4061ab1a33.PNG)

-	crontab -e 
-	0 1 * * * root /var/lib/mysql/backup_script/sql_script.sh
-	systemctl restart crond

![16](https://user-images.githubusercontent.com/71564211/139786230-69783427-00d7-4fbb-8445-41dec3d7d619.PNG)

## Configure VPN IPsec/L2TP (IT IS AN EXTRA PART, SEE MY DOCUMENT)






