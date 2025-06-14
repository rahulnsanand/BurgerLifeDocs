# Scenario 7 - Master NAS HDD Down

#### Domain Name Pointed To

burgerlife.in/asherslife.in -> Bangalore Public IP

#### Node Status - <mark style="color:green;">5 UP</mark> | <mark style="color:orange;">0 DOWN</mark>

* Master Server - Up and Running with the attached 4TB&#x20;
* Manager Server (Hosur, TN) - Up and Running with 4TB as active server standby&#x20;
* Worker Server - Worker Cluster Keepalived VIP Assigned Here
* Worker Manager Server - Backup Worker Cluster is active and ready
* Worker Slave Server - Slave Worker Cluster is active and standby

#### Storage Replication

* Web Proxy Hosted By: Worker Server
* User Data Written To: Manager Server
* Syncthing <mark style="color:red;">NOT</mark> running to move user and app data uni-directionally to Manager from Master
* All Internet Traffic routed from Internet to the Worker Server and then to the Manager Server
* All Local traffic router from Router to Worker Server and then to the Manager Server



