---
description: 8/27/2024
---

# Network Redundancy

### Risk Analysis

* Redundancy is expensive
* Disaster Recovery Site Options&#x20;
  * Types of Failover like sites (Hot \$$$, Warm \$$, Cold $)&#x20;
    * Cold is cyber.local going down
    * Warm is a redundant cyber.local
    * Hot is a critical system that will impact people, jobs, and money.&#x20;
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

set interfaces ethernet eth0 address '10.0.17.16/24'

set interfaces ethernet eth0 description 'WAN'



Setting up my vms

```
set nat source rule 10 description 'NAT FROM LAN to WAN'
set nat source rule 10 outband-interface eth0
set nat source rule 10 source address 10.0.5.0/24
set nat source rule 10 translation address masquerade
```

vyos01 to get xubuntu-lan to work

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

web01

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption><p>gateway is 10.0.5.1** due to virtual</p></figcaption></figure>



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
set interfaces ethernet eth0 hw-id '00:50:56:a1:7d:9e'
set interfaces ethernet eth1 address '10.0.5.2/24'
set interfaces ethernet eth1 description 'SEC440-LAN'
set interfaces ethernet eth1 hw-id '00:50:56:a1:50:28'
set interfaces ethernet eth2 address '10.0.6.2/24'
set interfaces ethernet eth2 description 'SEC440-OPT'
set interfaces ethernet eth2 hw-id '00:50:56:a1:f3:af'
set interfaces loopback lo
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
set system config-management commit-revisions '100'
set system conntrack modules ftp
set system conntrack modules h323
set system conntrack modules nfs
set system conntrack modules pptp
set system conntrack modules sip
set system conntrack modules sqlnet
set system conntrack modules tftp
set system console device ttyS0 speed '115200'
set system host-name 'vyos01-fatima'
set system login user vyos authentication encrypted-password '$6$YUTCBnIl7XuxPfv7$UQXsMiDLSJsDs9mPJ2PQ.9IjjMks5MrKu6IlQRJsS.VIvkYeQXFvupJVrZMTQFYjkbTkRshVAYECJS337kHAS/'
set system login user vyos authentication plaintext-password ''
set system name-server '10.0.17.2'
set system ntp server time1.vyos.net
set system ntp server time2.vyos.net
set system ntp server time3.vyos.net
set system syslog global facility all level 'info'
set system syslog global facility protocols level 'debug'

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
set interfaces ethernet eth0 hw-id '00:50:56:a1:8a:01'
set interfaces ethernet eth1 address '10.0.5.3/24'
set interfaces ethernet eth1 description 'SEC440-LAN'
set interfaces ethernet eth1 hw-id '00:50:56:a1:2c:00'
set interfaces ethernet eth2 address '10.0.6.3/24'
set interfaces ethernet eth2 description 'SEC440-OPT'
set interfaces ethernet eth2 hw-id '00:50:56:a1:93:78'
set interfaces loopback lo
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
set system config-management commit-revisions '100'
set system conntrack modules ftp
set system conntrack modules h323
set system conntrack modules nfs
set system conntrack modules pptp
set system conntrack modules sip
set system conntrack modules sqlnet
set system conntrack modules tftp
set system console device ttyS0 speed '115200'
set system host-name 'vyos02-fatima'
set system login user vyos authentication encrypted-password '$6$YUTCBnIl7XuxPfv7$UQXsMiDLSJsDs9mPJ2PQ.9IjjMks5MrKu6IlQRJsS.VIvkYeQXFvupJVrZMTQFYjkbTkRshVAYECJS337kHAS/'
set system login user vyos authentication plaintext-password ''
set system name-server '10.0.17.2'
set system ntp server time1.vyos.net
set system ntp server time2.vyos.net
set system ntp server time3.vyos.net
set system syslog global facility all level 'info'
set system syslog global facility protocols level 'debug'


```



Review:

I had a couple of troubles during this lab. This was my first experience setting up VRRP and a Google Authenticator. I first did VRRP all under one group for LAN, WAN, & OPT and I didn't do the 156 VRID for my Eth0 (WAN). Once I figured that out, I fixed it and then understanding that SSHing with my Virtual IP helped too
