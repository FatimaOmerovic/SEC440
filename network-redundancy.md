---
description: 8/27/2024
---

# Network Redundancy

### Risk Analysis

* Redundancy is expensive
* Disaster Recovery Site Options
  * Types of Failover like sites (Hot \$$$, Warm \$$, Cold $)
    * Cold is cyber.local going down
    * Warm is a redundant cyber.local
    * Hot is a critical system that will impact people, jobs, and money.
* Disaster Recovery Means
  * Data - Replication, Offsite Backup -- NOT Cloud environment
  * Networks - High availability options, double
  * People
    * Complex issues
      * Who goes to the backup site?
      * In a disaster, who actually goes to the backup site?
      * Who is actually critical?

Project 1: Network Redundancy

* Redundant VyOS Routers
  * The 2 routers can "share" the same IPs so that if one fails, not reconfiguration or convergence is required
  * VRRP (Virtual Router Redundancy Protocol) will be used to make that happen
  * Port Forwarding will also help meet the redundancy requirements, setting up 2FA for SSH on the web server

Lab commands:



vyos 1+2

```
set nat source rule 10 description 'NAT FROM LAN to WAN'
set nat source rule 10 outband-interface eth0
set nat source rule 10 source address 10.0.5.0/24
set nat source rule 10 translation address masquerade
```



web01

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

setting up VRRP

VYOS 1 + 2

<pre><code><strong>set high-availability vrrp group WAN vrid 156
</strong>set high-availability vrrp group WAN interface eth0
set high-availability vrrp group WAN address 10.0.17.106/24

set high-availability vrrp group LAN vrid 20
set high-availability vrrp group LAN interface eth1
set high-availability vrrp group LAN address 10.0.5.1/24

set high-availability vrrp group OPT vrid 30
set high-availability vrrp group OPT interface eth2
set high-availability vrrp group OPT address 10.0.6.1/24
</code></pre>

nat destination vyos01 + 2

```
nat destination rule 10 description "HTTP - > WEB01"
set nat destination rule 10 destination port '80'
set nat destination rule 10 inbound-interface eth0
set nat destination rule 10 protocol 'tcp'
set nat destination rule 10 translation address 10.0.5.100
set nat destination rule 10 translation port '80'
commit
save

nat destination rule 20 description "SSH - > WEB01"
set nat destination rule 20 destination port '22'
set nat destination rule 20 inbound-interface eth0
set nat destination rule 20 protocol 'tcp'
set nat destination rule 20 translation address 10.0.5.100
set nat destination rule 20 translation port '22'
commit
save

set service dns forwarding listen-address 10.0.5.1
set service dns forward allow-from 10.0.5.0/24
set service dns forwarding system
commit
save
```

Google Authentication:

[https://www.digitalocean.com/community/tutorials/how-to-set-up-multi-factor-authentication-for-ssh-on-centos-8](https://www.digitalocean.com/community/tutorials/how-to-set-up-multi-factor-authentication-for-ssh-on-centos-8)

{% embed url="https://www.untrustedconnection.com/2013/06/dual-factor-ssh-google-authenticator.html" %}

vy0s1

```
set high-availability vrrp group LAN address 10.0.5.1/24
set high-availability vrrp group LAN interface 'eth1'
set high-availability vrrp group LAN priority '200'
set high-availability vrrp group LAN vrid '20'
set high-availability vrrp group OPT address 10.0.6.1/24
set high-availability vrrp group OPT interface 'eth2'
set high-availability vrrp group OPT priority '200'
set high-availability vrrp group OPT vrid '30'
set high-availability vrrp group WAN address 10.0.17.106/24
set high-availability vrrp group WAN interface 'eth0'
set high-availability vrrp group WAN priority '200'
set high-availability vrrp group WAN vrid '156'
set interfaces ethernet eth0 address '10.0.17.16/24'
set interfaces ethernet eth0 description 'SEC440-WAN'
set interfaces ethernet eth1 address '10.0.5.2/24'
set interfaces ethernet eth1 description 'SEC440-LAN'
set interfaces ethernet eth2 address '10.0.6.2/24'
set interfaces ethernet eth2 description 'SEC440-OPT'
set nat destination rule 10 description 'HTTP -> WEB01'
set nat destination rule 10 destination port '80'
set nat destination rule 10 inbound-interface 'eth0'
set nat destination rule 10 protocol 'tcp'
set nat destination rule 10 translation address '10.0.5.100'
set nat destination rule 20 description 'SSH -> WEB01'
set nat destination rule 20 destination port '22'
set nat destination rule 20 inbound-interface 'eth0'
set nat destination rule 20 protocol 'tcp'
set nat destination rule 20 translation address '10.0.5.100'
set nat destination rule 20 translation port '22'
set nat source rule 10 description 'LAN to WAN'
set nat source rule 10 outbound-interface 'eth0'
set nat source rule 10 source address '10.0.5.0/24'
set nat source rule 10 translation address 'masquerade'
set nat source rule 20 description 'NAT from opt-wan'
set nat source rule 20 outbound-interface 'eth0'
set nat source rule 20 source address '10.0.6.0/24'
set nat source rule 20 translation address 'masquerade'
set protocols static route 0.0.0.0/0 next-hop 10.0.17.2
set service dns forwarding allow-from '10.0.5.0/24'
set service dns forwarding listen-address '10.0.5.1'
set service dns forwarding system
set service ssh listen-address '0.0.0.0'
set system host-name 'vyos01-fatima'
set system name-server '10.0.17.2'


```

vyos2

```
set high-availability vrrp group LAN address 10.0.5.1/24
set high-availability vrrp group LAN interface 'eth1'
set high-availability vrrp group LAN priority '100'
set high-availability vrrp group LAN vrid '20'
set high-availability vrrp group OPT address 10.0.6.1/24
set high-availability vrrp group OPT interface 'eth2'
set high-availability vrrp group OPT priority '100'
set high-availability vrrp group OPT vrid '30'
set high-availability vrrp group WAN address 10.0.17.106/24
set high-availability vrrp group WAN interface 'eth0'
set high-availability vrrp group WAN priority '100'
set high-availability vrrp group WAN vrid '156'
set interfaces ethernet eth0 address '10.0.17.76/24'
set interfaces ethernet eth0 description 'SEC440-WAN'
set interfaces ethernet eth1 address '10.0.5.3/24'
set interfaces ethernet eth1 description 'SEC440-LAN'
set interfaces ethernet eth2 address '10.0.6.3/24'
set interfaces ethernet eth2 description 'SEC440-OPT'
set nat destination rule 10 description 'HTTP -> WEB01'
set nat destination rule 10 destination port '80'
set nat destination rule 10 inbound-interface 'eth0'
set nat destination rule 10 protocol 'tcp'
set nat destination rule 10 translation address '10.0.5.100'
set nat destination rule 10 translation port '80'
set nat destination rule 20 description 'SSH -> WEB01'
set nat destination rule 20 destination port '22'
set nat destination rule 20 inbound-interface 'eth0'
set nat destination rule 20 protocol 'tcp'
set nat destination rule 20 translation address '10.0.5.100'
set nat destination rule 20 translation port '22'
set nat source rule 10 description 'NAT FROM LAN to WAN'
set nat source rule 10 outbound-interface 'eth0'
set nat source rule 10 source address '10.0.5.0/24'
set nat source rule 10 translation address 'masquerade'
set nat source rule 20 description 'nat from opt-wan'
set nat source rule 20 outbound-interface 'eth0'
set nat source rule 20 source address '10.0.6.0/24'
set nat source rule 20 translation address 'masquerade'
set protocols static route 0.0.0.0/0 next-hop 10.0.17.2
set service dns forwarding allow-from '10.0.5.0/24'
set service dns forwarding listen-address '10.0.5.1'
set service dns forwarding system
set service ssh listen-address '0.0.0.0'
set system host-name 'vyos02-fatima'
set system name-server '10.0.17.2'



```

Review:

I had a couple of troubles during this lab. This was my first experience setting up VRRP and a Google Authenticator. I first did VRRP all under one group for LAN, WAN, & OPT and I didn't do the 156 VRID for my Eth0 (WAN). Once I figured that out, I fixed it and then understanding that SSHing with my Virtual IP helped too
