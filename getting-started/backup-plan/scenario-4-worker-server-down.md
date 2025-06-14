# Scenario 4 - Worker Server Down

#### Domain Name Pointed To

burgerlife.in/asherslife.in -> Bangalore Public IP

#### Node Status - <mark style="color:green;">4 UP</mark> | <mark style="color:red;">1 DOWN</mark>

* Master Server - Up and Running with the attached 4TB&#x20;
* Manager Server (Hosur, TN) - Up and Running with 4TB as active server standby&#x20;
* <mark style="color:red;">Worker Server - DOWN</mark>
* Worker Manager Server - Backup Worker Cluster is active and Keepalived VIP Assigned Here
* Worker Slave Server - Slave Worker Cluster is active and standby

#### Storage Replication

* Web Proxy Hosted By:  Backup Worker Cluster Server
* User Data Written To: Master Server
* Syncthing running to move user and app data uni-directionally to Manager from Master
* All Internet Traffic routed from Internet to the  Backup Worker Cluster Server and then to the Master Server
* All Local traffic router from Router to  Backup Worker Cluster Server and then to the Master Server



