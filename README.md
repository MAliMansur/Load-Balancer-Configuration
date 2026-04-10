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

