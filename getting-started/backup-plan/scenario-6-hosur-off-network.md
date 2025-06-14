# Scenario 6 - Hosur Off Network

#### Domain Name Pointed To

burgerlife.in/asherslife.in -> Bangalore Public IP

#### Node Status - <mark style="color:green;">4 UP</mark> | <mark style="color:red;">1 DOWN</mark>

* Master Server - Up and Running with the attached 4TB&#x20;
* Manager Server (Hosur, TN) - Up and Running with 4TB BUT OFF INTERNET&#x20;
* Worker Server - Worker Cluster Keepalived VIP Assigned Here
* Worker Manager Server - Backup Worker Cluster is active and ready
* Worker Slave Server - Slave Worker Cluster is active and standby

#### Storage Replication

* Web Proxy Hosted By: Worker Server
* User Data Written To: Master Server
* Syncthing running to move user and app data uni-directionally to Manager from Master <mark style="color:red;">BUT MANAGER NOT RECEIVING</mark>
* All Internet Traffic routed from Internet to the Worker Server and then to the Master Server
* All Local traffic router from Router to Worker Server and then to the Master Server



