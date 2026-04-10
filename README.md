# Load-Balancer-Configuration
Configuration and setup of Load Balancer to simulate web server for 1000 concurrent users
<img width="1108" height="511" alt="image" src="https://github.com/user-attachments/assets/3cbeefb6-aa48-4039-a85e-0654cb8e87e4" />

Configuration Summary

Server Information

•	Hostname: edustream.com
•	IP Address: 15.207.247.187
•	Host machine (Operating System): Ubuntu Server 24.04 LTS


Software Stack

Service	Software	Purpose
Backup Server and catalog	Bacula-Server and postgres database (Node 1)	The backup server holding the bacula director, file daemon and storage daemon and postgres database as a catalog
Backup Client	Bacula-Client (Node 2)	The backup client holding the important patient’s data can be up to 500 GB and bacula file daemon to communicate with the backup server.
Host Machine	Ubuntu 24.04	The machine for hosting the backup server and client


 



Table of Contents




Overview of the architecture	3
Architecture 1	3
Architecture 2	3
Configuration Summary	4
Server Information	4
Software Stack	4
Key Configuration Files	4
Objective	4
Key Requirements	5
Web Server Configuration (Node 2 and Node 3)	5
Steps	5
Load Balancer Configuration (Node 1)	5
Steps	5
Security Group Inbound Rules (AWS Cloud)	6
Testing Results	6
Tool: Apache Benchmark (ab)	6
Screenshot of Setups	7
Maintainence Plan	9
Apprentices	9








Overview of the architecture

 




Configuration Summary

Server Information

•	Hostname: edustream.com
•	IP Address: 15.207.247.187
•	Host machine (Operating System): Ubuntu Server 24.04 LTS


Software Stack

Service	Software	Purpose
Backup Server and catalog	Bacula-Server and postgres database (Node 1)	The backup server holding the bacula director, file daemon and storage daemon and postgres database as a catalog
Backup Client	Bacula-Client (Node 2)	The backup client holding the important patient’s data can be up to 500 GB and bacula file daemon to communicate with the backup server.
Host Machine	Ubuntu 24.04	The machine for hosting the backup server and client


Key Configuration Files

Component	File Path
Apache Web Servers	/var/www/html/index.html
Nginx Load Balancer	/etc/nginx/sites-available/loadbalancer.conf

Objective

Design and implement a highly available and scalable web server cluster for EduStream to support 1,000 concurrent users with a target response time of 100ms.

Key Requirements

•	99.9% uptime
•	Use of Ubuntu 20.04 on all nodes
•	NGINX for load balancing
•	Apache2 on two backend nodes
•	Simulated load testing for 1,000 concurrent users
•	Budget: $2,500 for 3 VMs (4GB RAM, 500GB SSD each)
Backup Server Configuration (Node 1)
Bacula Director

•	This is the main file of the bacula. Almost all settings are here
•	Backup jobs, time when to run them, list of clients to backup, storages, and other settings
•	By default, bacula working address is localhost. You can change it or if you want several addresses to listen to



Steps

•	Change the hostname to backup server: sudo hostnamectl set-hostname backup_server
•	Update packages: sudo apt update && sudo apt upgrade -y
•	Install the bacula backup server: sudo apt install bacula-director bacula-sd bacula-client postgresql -y

Configuration of Bacula Director

•	Check the status of bacula director: sudo systemctl status bacula-director
•	Install the net-tools to allow command netstat to work: sudo apt install net-tools
•	To check which port is used by the director: sudo netstat -anp | grep LISTEN | grep
bacula-dir
By default, port 9101 is used by the director
•	If you want to reconfigure the bacula director, you can use the command: sudo dpkg-
reconfigure bacula-director-pgsql 
•	To configure the bacula director: sudo vim /etc/bacula/bacula-dir.conf
•	To first configure the bacula director, define the specific backup job you want to implement

 
•	Add the job you want to run

 
•	Add the client machine which you want to take backup to

 

•	Add as many client machines as you can here, which you want to provide backups for the file in the bacula-dir.conf in the server
•	To check the configuration for the director: sudo bacula-dir -t -c /etc/bacula/bacula-dir.conf
•	After correct syntax and configuration, restart the bacula director: sudo systemctl restart bacula-dir or service bacula-director restart


Configuration of Postgres database

•	If want to go into the postgres database: sudo su postgres
•	Then you can use \l to check the list of databases
•	To go into the database directory, use \c database_name command
•	To see if the database is filled with tables, use \d command
•	To go out of the database, use \q command



Configuration of Bacula Storage Daemon

•	Check the status of bacula Storage Daemon: sudo systemctl status bacula-sd
•	To check which port is used by the Storage Daemon: sudo netstat -anp | grep LISTEN | grep bacula-sd
By default, port 9103 is used by the storage daemon
•	To configure the bacula storage daemon: sudo vim /etc/bacula/bacula-sd.conf
•	Edit the storage section in the bacula storage daemon





 

 

•	Change the SDAddress from localhost to your private address 
•	Change the name of the archive device to your desired archive device	 in the device section



Backup Client Configuration (Node 2)
Bacula Client

•	This is the client side of the bacula. There is a file called file daemon where all the changes related to backup are located
•	The client username, passwords, ports, IPs and all the information related to backup and connection are in the file daemon 
•	By default, bacula working address is localhost. You can change it or if you want several addresses to listen to
•	You can add as many client nodes as you can if your data backup is in multiple nodes

Steps

•	Change the hostname to backup client: sudo hostnamectl set-hostname backup_client
•	Update packages: sudo apt update && sudo apt upgrade -y
•	Install the bacula backup client: sudo apt install bacula-client -y

Configuration of Bacula File Daemon

•	Check the status of bacula File Daemon director: sudo systemctl status bacula-fd.service
•	To check which port is used by the File Daemon: sudo netstat -anp | grep LISTEN | grep
•	bacula-fd
By default, port 9102 is used by the file daemon
•	To configure the bacula file daemon: sudo vim /etc/bacula/bacula-fd.conf
 
•	Keep the FDAddress to 0.0.0.0 or client’s Private IP, not to localhost or 127.0.0.1. This way, the client machine will be open for communication with the backup server.

Security Group Inbound Rules (AWS Cloud)
Node 1 (Backup Server):
•	TCP 9103: Node 2 Private IP
•	TCP 22: Admin IP only
•	ICMP: Open for all ports
Node 2 (Backup Client):
•	TCP 9102: Node 1 Private IP
•	TCP 22: Admin IP only
•	ICMP: Open for all ports

Administer the Bacula Backup Server

•	To administer bacula, we will use bconsole
•	To install bconsole, the command is: sudo apt-get install bacula-console
•	Run sudo bconsole

 

•	Bacula brings all known file daemons, but it doesn’t mean the connection with them is established. It only means you have written them in a config file.
•	To check the connection, we need to use command status
 
•	You will see the server is connected to file daemon but there is no job running
•	

Testing Results 
Tool: Apache Benchmark (ab)

ab -n 1000 -c 100 http://<load_balancer_ip>/
or
sudo tail -f /var/log/nginx/access.log


Results:
•	1000 concurrent users simulated
•	Target response time: 100ms











Screenshot of Setups



 


 







Result of command: ab -n 1000 -c 100 http://<load_balancer_ip>/

 
 

Result of command: sudo tail -f /var/log/nginx/access.log

 

Maintainence Plan

Weekly Tasks:
•	Check load balancer health (sudo systemctl status nginx)
•	Check Apache logs (/var/log/apache2/)
•	Monitor disk usage and memory

Monthly:
•	Apply security patches
•	Rotate logs
•	Run a full performance test

Backup Strategy:

•	Use rsync for daily snapshots of web content
•	Backup NGINX and Apache config files to secure storage


High Availability Consideration (Future Work):

•	Add a second NGINX node with keepalived for failover
•	Implement auto-scaling rules if moved to cloud



Apprentices
•	Apache and NGINX config files
•	Load test raw output
•	Screenshots of setup


Prepared by: Ali Mansur
Date: 26-06-2025
Contact: +92 331-5229434

