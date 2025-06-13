---
icon: note
---

# Quick Reference

#### Comprehensive Server Overview

This section provides detailed information about the deployment and specifications of the servers in your network, all running on Debian 12. The data centers are strategically located to balance load and ensure operational continuity.

***

**1. burgermaster**\
A high-performance server equipped to handle intensive processing and storage tasks.

* **IP Address:** 10.10.10.10
* **Processor:** Intel i7 - Equipped for demanding multitasking and processing applications.
* **Memory:** 16GB RAM - Ensures smooth execution of large applications and improved multitasking performance.
* **Storage:**
  * **Primary:** 500GB NVMe SSD - Offers high speed and reliability for frequently accessed data.
  * **Secondary:** 4TB HDD - Provides substantial storage for backups and archival data.
* **Location:** Bangalore – Optimally placed for quick access and reduced latency to local clients.
* **Functionality:** Ideal for hosting critical applications and managing large-scale data operations.

**2. burgermanager**\
A versatile server, serving as a bridge for operations in geographically distinct locations.

* **IP Address:** 10.10.10.20
* **Processor:** Intel i5 - Balanced for everyday computing needs and energy efficiency.
* **Memory:** 16GB RAM - Supports effective multitasking and adequate performance for common applications.
* **Storage:**
  * **Primary:** 500GB NVMe SSD - Fast data retrieval and application loading.
  * **Secondary:** 4TB HDD - Ample space for media, documents, and archiving.
* **Location:** Hosur – Expands operational capacity with geographical redundancy.
* **Functionality:** Suited for general-purpose roles, including hosting services and network management.

**3. burgerworker**\
Designed for lightweight operations, suitable for specific tasks that do not require high throughput.

* **IP Address:** 10.10.10.15
* **Processor:** Raspberry Pi 5 - Energy-efficient and cost-effective for small-scale applications.
* **Memory:** 8GB RAM - Sufficient for running lightweight applications and services.
* **Storage:**
  * **Primary:** 500GB NVMe SSD - Ensures quick access to frequently used applications.
* **Location:** Bangalore – Aiding local operations with reliable, low-power processing.
* **Functionality:** Ideal for test environments, development tasks, or acting as a node in data-centric computations.

