# Web Application & DB Redundancy

Setting up u1, u2, u3

```
sudo hostnamectl set-hostname u1-fatima
sudo adduser fatima
sudo usermod -aG sudo fatima
```

swsss

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption><p>/etc/netplan/00-installer-config.yaml for u1</p></figcaption></figure>

```
sudo netplan apply
```



U1:

```
sudo apt update
sudo apt install mariadb-server
```

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption><p>/etc/mysql/mariadb.conf.d/60-galera.cnf</p></figcaption></figure>

