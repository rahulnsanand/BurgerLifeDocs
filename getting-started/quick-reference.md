---
icon: note
---

# Quick Reference

#### Comprehensive Server Overview

This section provides detailed information about the deployment and specifications of the servers in your network, all running on Debian 12. The data centers are strategically located to balance load and ensure operational continuity.

***

#### System Overview

<table><thead><tr><th width="167.5999755859375">Host Name</th><th width="113">LAN IP</th><th width="141.800048828125">WireGuard IP</th></tr></thead><tbody><tr><td>burgermaster</td><td>10.10.10.10</td><td>10.21.22.10</td></tr><tr><td>burgermanager</td><td>10.10.10.20</td><td>10.21.22.20</td></tr><tr><td>burgercloud</td><td>-</td><td>10.21.22.15</td></tr></tbody></table>

***

**1. burgermaster**

* **IP Address:** 10.10.10.10
* **Processor:** Intel i7
* **Memory:** 16GB RAM
* **Storage:**
  * **Primary:** 500GB NVMe SSD
  * **Secondary:** 4TB HDD
* **Location:** Bangalore, KA

**2. burgermanager**

* **IP Address:** 10.10.10.20
* **Processor:** Intel i5
* **Memory:** 16GB RAM
* **Storage:**
  * **Primary:** 500GB NVMe SSD
  * **Secondary:** 4TB HDD
* **Location:** Hosur, TN

**3. burgercloud**

* **IP Address:** 10.10.10.15 | **VIP:** 10.10.10.5
* **Processor:** Raspberry Pi 5
* **Memory:** 8GB RAM
* **Storage:**
  * **Primary:** 500GB NVMe SSD
* **Location:** Bangalore





