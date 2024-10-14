# Web Application & DB Redundancy

Setting up u1, u2, u3

Set network adapter to LAN

```
sudo hostnamectl set-hostname u1-fatima
sudo adduser fatima
sudo usermod -aG sudo fatima
```

swsss

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption><p>/etc/netplan/00-installer-config.yaml for u1</p></figcaption></figure>

```
sudo netplan apply
```

```
sudo apt update
sudo apt install mariadb-server
```

Go into 50-server.cnf file and make the bind-address 0.0.0.0 for each VM.&#x20;

<figure><img src=".gitbook/assets/image (2) (1).png" alt=""><figcaption><p>/etc/mysql/mariadb.conf.d/60-galera.cnf</p></figcaption></figure>

U1:

<pre><code>sudo mariadb-secure-installation

Enter current password for root (enter for none): *ENTERED*
<strong>           Switch to unix_socket authentication : n
</strong>                      Change the root password? : n
                       Remove anonymouse users? : y
                  Disallow root login remotely? : n
         Remove test database and access to it? : y
                   Reload privilege tables now? : y
</code></pre>

```
CREATING A MARIADB USER:
sudo mysql -u root -p
CREATE USER 'fat'@'10.0.5.%' IDENTIFIED BY 'PASSWORD';
GRANT ALL PRIVILEGES ON *.* to 'fat'@'10.0.5.%' WITH GRANT OPTION;
```

```
Sudo galera_new_cluster

# Log into mariadb now and then do a command that shows the cluster size to see if all 3 joined
```

HA 1+2

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption><p>/etc/haproxy/haproxy.cfg</p></figcaption></figure>

Setting up PHP on webservers (web01 + web02)

```
yum install php php-mysqlnd
setsebool -P httpd_can_network_connect_db on
systemctl restart httpd
```

Create php file

<figure><img src=".gitbook/assets/image (4) (1).png" alt=""><figcaption><p>/var/www/html</p></figcaption></figure>

```
sudo systemctl restart httpd
```

Navigate to the web01 address /info.php

<figure><img src=".gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>



Challenges:

The lab was smooth sailing other than MariaDB. I had multiple issues with being able to start MariaDB on my U1-3. I was able to resolve my issues by rather reverting my box, or going into /var/lib/mysql/grastate.dat file and changing safe\_to\_bootstrap to 1 on U1. I was able to do galera\_new\_cluster after that without an issue.&#x20;
