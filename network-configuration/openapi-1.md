---
icon: network-wired
---

# Cloudflared Configuration

#### Visit CloudFlare Zero Trust Dashboard

{% embed url="https://one.dash.cloudflare.com/" %}

**Under Networks > Tunnels**

Jotting down thoughts

* Master Node has WG IP
* Manager Node has WG IP
* WOrker Node has WG IP
* Master Node has Priority for WG KeepAlived VIP
* On Master Node Fail
  * Manager Node must get a signal that Master is down, it must become the Master
  * All sensitive applications must start up
* On Master Node back&#x20;
  * it must not sync anything from synthing
  * it must not get VIP back (until ready)
  * Manager node must be notified and all manager application (sensitive ones) must be stopped
  * rsync must run to back copy all changes from manager&#x20;
  * Master node must regain VIP once sync completes and data is ready
  * Manager node stays as is for backup
* Now the actual Domain if we use Cloudflare Tunnels can point to the WG VIP
* but WG VIP must not move back from manager to master until master is ready
*
