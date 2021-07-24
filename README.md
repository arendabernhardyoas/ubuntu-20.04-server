# Ubuntu 20.04: Server

## Build a server with Linux Ubuntu 20.04

### Inital Config
Configure network<br>
`sudo nano /etc/netplan/50-cloud-init.yaml`<br>
![netplan-config](./images/netplan-config.PNG)
`sudo netplan apply`<br>
![ping-internet](./images/ping-internet.PNG)
Update and upgrade packages<br>
`sudo apt-get update`<br>
`sudo apt-get -y full-upgrade`<br>
```
#DHCP config
network:
ethernets:
      eth0:
        dhcp4: true
        optional: true
version: 2
wifis:
      wlan0:
        dhcp4: true
        optional: true
        access-points:
           "SSID name":
              password: "PASSWORD"
```
```
#Static config
network:
ethernets:
      eth0:
        dhcp4: no
        addresses: [172.17.17.250/25]
        gateway4: 172.17.17.172
        nameservers:
           addresses: [172.17.17.172]
version: 2
wifis:
      wlan0:
        dhcp4: no
        addresses: [172.17.17.250/25]
        gateway4: 172.17.17.172
        nameservers:
           addresses: [172.17.17.172]
        access-points:
           "SSID name":
              password: "PASSWORD"
```

### DNS server
Install BIND<br>
`sudo apt-get -y install bind9 bind9utils`<br>
Configure BIND<br>
`sudo nano /etc/bind/named.conf` [file](./configs/named.conf)<br>
![bind-config-1](./images/bind-config-1.PNG)
`sudo nano /etc/bind/named.conf.options`  [file](./configs/named.conf.options)<br>
![bind-config-2](./images/bind-config-2.PNG)
`sudo nano /etc/default/named`<br>
![bind-config-3](./images/bind-config-3.PNG)
`sudo nano /etc/bind/named.conf.arendabernhardyoas`  [file](./configs/named.conf.arendabernhardyoas)<br>
![bind-config-4](./images/bind-config-4.PNG)
`sudo nano /etc/bind/forward.arendabernhardyoas.com`  [file](./configs/forward.arendabernhardyoas.com)<br>
![bind-config-5](./images/bind-config-5.PNG)
`sudo nano /etc/bind/reverse.17.17.172`  [file](./configs/reverse.17.17.172)<br>
![bind-config-6](./images/bind-config-6.PNG)
`sudo systemctl restart named`<br>
Set name server<br>
`sudo nano /etc/netplan/50-cloud-init.yaml`<br>
![bind-netplan-config](./images/bind-netplan-config.PNG)
`sudo netplan apply`<br>
Not mandatory configuration<br>
`sudo nano /etc/resolv.conf`<br>
![bind-resolv-config](./images/bind-resolv-config.PNG)
Verify Configurations<br>
![bind-1](./images/bind-1.PNG)
`dig arendabernhardyoas.com`<br>
![bind-2](./images/bind-2.PNG)
`dig -x 172.17.17.250`<br>
![bind-3](./images/bind-3.PNG)
![bind-4](./images/bind-4.PNG)

### Web Server Apache
Install apache2<br>
`sudo apt-get -y install apache2`<br>
Install PHP<br>
`sudo apt-get -y install php php-cgi php-pear php-mbstring libapache2-mod-php php-common`<br>
Configure apache2<br>
`sudo chown -R aby:aby /var/www/`<br>
`sudo nano /etc/apache2/apache2.conf`  [file](./configs/apache2.conf)<br>
![apache2-config-1](./images/apache2-config-1.PNG)
`sudo nano /etc/apache2/mods-enabled/status.conf` [file](./configs/status.conf)<br>
![apache2-config-2](./images/apache2-config-2.PNG)
`sudo nano /etc/apache2/conf-enabled/security.conf` [file](./configs/security.conf)<br>
![apache2-config-3](./images/apache2-config-3.PNG)
`sudo nano /etc/apache2/sites-enabled/000-default.conf` [file](./configs/000-default.conf)<br>
![apache2-config-4](./images/apache2-config-4.PNG)
Configure apache2 SSL/TLS<br>
`sudo nano /etc/apache2/sites-available/default-ssl.conf` [file](./configs/default-ssl.conf)<br>
![apache2-config-5](./images/apache2-config-5.PNG)
![apache2-config-6](./images/apache2-config-6.PNG)
![apache2-config-7](./images/apache2-config-7.PNG)
`sudo nano /etc/apache2/sites-enabled/000-default.conf` [file](./configs/000-default.conf)<br>
![apache2-config-8](./images/apache2-config-8.PNG)
`sudo a2enmod ssl`<br>
`sudo a2ensite default-ssl`<br>
`sudo nano /etc/apache2/sites-available/vhost.conf` [file](./configs/vhost.conf)<br>
![apache2-config-9](./images/apache2-config-9.PNG)
`sudo a2ensite vhost`<br>
`sudo a2enmod rewrite`<br>
`sudo systemctl restart apache2`<br>
Verify Configurations<br>
![apache2-1](./images/apache2-1.PNG)
![apache2-2](./images/apache2-2.PNG)
![apache2-3](./images/apache2-3.PNG)

### Database PostgreSQL
Install PostgreSQL and phppgadmin<br>
`sudo apt-get -y install postgresql-12 php-pgsql phppgadmin`<br>
Configure PostgreSQL<br>
`sudo nano /etc/postgresql/12/main/postgresql.conf` [file](./configs/postgresql.conf)<br>
![postgresql-config-1](./images/postgresql-config-1.PNG)
![postgresql-config-2](./images/postgresql-config-2.PNG)
`sudo nano /etc/postgresql/12/main/pg_hba.conf` [file](./configs/pg_hba.conf)<br>
![postgresql-config-3](./images/postgresql-config-3.PNG)
Adding address `0.0.0.0/0` can create postgresql connections from anywhere.<br>
`sudo systemctl restart postgresql`<br>
`sudo su - postgres`<br>
`createuser -s -i -d -r -l -w aby`<br>
`psql -c "ALTER ROLE aby WITH PASSWORD '<<password>>';"`<br>
`exit`<br>
`createdb monitoring_building`<br>
`psql monitoring_building`<br>
`create table public.monitoring_laboratorium ("date" timestamp default current_timestamp not null, "device_name" varchar(30) not null, "device_ipaddr" varchar(30) not null, "temperature" float not null, "humidity" float not null);`<br>
`copy public.monitoring_laboratorium from '<<file location>>';`<br>
`ALTER ROLE aby WITH REPLICATION;`<br>
`ALTER ROLE aby WITH BYPASSRLS;`<br>
`\q`<br>
Configure phppgadmin<br>
`sudo nano /etc/phppgadmin/config.inc.php` [file](./configs/config.inc.php)<br>
![phppgadmin-config-1](./images/phppgadmin-config-1.PNG)
`sudo nano /etc/apache2/conf-enabled/phppgadmin.conf` [file](./configs/phppgadmin.conf)<br>
![phppgadmin-config-2](./images/phppgadmin-config-2.PNG)
`sudo systemctl restart postgresql apache2`<br>
Verify configurations<br>
![postgresql](./images/postgresql.PNG)
![phppgadmin-1](./images/phppgadmin-1.PNG)
![phppgadmin-2](./images/phppgadmin-2.PNG)
![phppgadmin-3](./images/phppgadmin-3.PNG)

### Database MariaDB
Install MariaDB<br>
`sudo apt-get -y install mariadb-server php-mysql`<br>
Configure MariaDB<br>
`sudo mysql_secure_installation`<br>
![mariadb-config-1](./images/mariadb-config-1.PNG)
![mariadb-config-2](./images/mariadb-config-2.PNG)
`sudo mariadb`<br>
`create user 'aby'@'%' identified by '<<password>>';`<br>
`grant all privileges on *.* to 'aby'@'%' with grant option;`<br>
`flush privileges;`<br>
`\q`<br>
`mariadb -p`<br>
`create database monitoring_building;`<br>
`CREATE TABLE monitoring_building.monitoring_laboratorium ("date" timestamp NOT NULL DEFAULT current_timestamp(), "device_name" varchar(30) NOT NULL, "device_ipaddr" varchar(30) NOT NULL, "temperature" float NOT NULL, "humidity" float NOT NULL);`<br>
`LOAD DATA LOCAL INFILE '/home/aby/monitoring_laboratorium.txt' INTO TABLE monitoring_building. monitoring_laboratorium;`<br>
`\q`<br>
Install phpmyadmin<br>
`sudo apt-get -y install phpmyadmin`<br>
![phpmyadmin-install-1](./images/phpmyadmin-install-1.PNG)
![phpmyadmin-install-2](./images/phpmyadmin-install-2.PNG)
![phpmyadmin-install-3](./images/phpmyadmin-install-3.PNG)
See `/etc/dbconfig-common/phpmyadmin.conf` or `/etc/phpmyadmin/config-db.php` for a random password of phpmyadmin.<br>
`sudo systemctl restart mariadb apache2`<br>
Verify Configurations<br>
![mariadb-1](./images/mariadb-1.PNG)
![mariadb-2](./images/mariadb-2.PNG)
![phpmyadmin-1](./images/phpmyadmin-1.PNG)
![phpmyadmin-2](./images/phpmyadmin-2.PNG)

### Web Server NGINX
Install NGINX<br>
`sudo apt-get -y install nginx php7.4-fpm`<br>
Configure NGINX<br>
`sudo nano /etc/nginx/sites-enabled/default`  [file](./configs/default)<br>
![nginx-config-1](./images/nginx-config-1.PNG)
![nginx-config-2](./images/nginx-config-2.PNG)
![nginx-config-3](./images/nginx-config-3.PNG)
Configure NGINX SSL/TLS<br>
`sudo nano /etc/nginx/sites-enabled/default`  [file](./configs/default)<br>
![nginx-config-4](./images/nginx-config-4.PNG)
![nginx-config-5](./images/nginx-config-5.PNG)
![nginx-config-6](./images/nginx-config-6.PNG)
![nginx-config-7](./images/nginx-config-7.PNG)
Configure NGINX phpmyadmin and phppgadmin<br>
`sudo ln -s /usr/share/phppgadmin/ /var/www/html/`<br>
`sudo ln -s /usr/share/phpmyadmin/ /var/www/html/`<br>
![nginx-config-8](./images/nginx-config-8.PNG)
`sudo systemctl restart nginx php7.4-fpm`<br>
Verify Configurations<br>
![nginx-1](./images/nginx-1.PNG)
![nginx-2](./images/nginx-2.PNG)

### Samba
Install samba<br>
`sudo apt-get -y install samba smbclient cifs-utils`<br>
Configure samba<br>
`sudo nano /etc/samba/smb.conf`  [file](./configs/smb.conf)<br>
![samba-config-1](./images/samba-config-1.PNG)
`sudo smbpasswd -a aby`<br>
`sudo mkdir /home/public`<br>
`sudo chmod -R 777 /home/public`<br>
`sudo chown -R nobody:nogroup /home/public`<br>
![samba-config-2](./images/samba-config-2.PNG)
Configure printer share<br>
`sudo nano /etc/samba/smb.conf`  [file](./configs/smb.conf)<br>
![samba-config-3](./images/samba-config-3.PNG)
`sudo systemctl restart smbd nmbd`<br>
Verify configurations<br>
`sudo smbclient '\arendabernhardyoas.com\secure' -U aby`<br>
![samba-1](./images/samba-1.PNG)
`sudo smbclient '\arendabernhardyoas.com\public'`<br>
![samba-2](./images/samba-2.PNG)
![samba-3](./images/samba-3.PNG)
![samba-4](./images/samba-4.PNG)

### CUPS (Common UNIX Printing System)
Install CUPS<br>
`sudo apt-get -y install cups`<br>
Configure CUPS<br>
`sudo nano /etc/cups/cupsd.conf` [file](./configs/cupsd.conf)<br>
`sudo systemctl restart cups`<br>
Add local printer<br>
![cups-config-1](./images/cups-config-1.PNG)
![cups-config-2](./images/cups-config-2.PNG)
![cups-config-3](./images/cups-config-3.PNG)
![cups-config-4](./images/cups-config-4.PNG)
![cups-config-5](./images/cups-config-5.PNG)
![cups-config-6](./images/cups-config-6.PNG)
![cups-config-7](./images/cups-config-7.PNG)
![cups-config-8](./images/cups-config-8.PNG)
![cups-config-9](./images/cups-config-9.PNG)

### Mail Server
Install Postfix<br>
`sudo apt-get -y install postfix sasl2-bin`<br>
![postfix-install](./images/postfix-install.PNG)
Configure Postfix<br>
`sudo nano /etc/postfix/main.cf`  [file](./configs/main.cf)<br>
![postfix-config-1](./images/postfix-config-1.PNG)
![postfix-config-2](./images/postfix-config-2.PNG)
![postfix-config-3](./images/postfix-config-3.PNG)
![postfix-config-4](./images/postfix-config-4.PNG)
![postfix-config-5](./images/postfix-config-5.PNG)
![postfix-config-6](./images/postfix-config-6.PNG)
![postfix-config-7](./images/postfix-config-7.PNG)
![postfix-config-8](./images/postfix-config-8.PNG)
![postfix-config-9](./images/postfix-config-9.PNG)
![postfix-config-10](./images/postfix-config-10.PNG)
Postfix run SMTP server on tcp port 25<br>
Install Dovecot<br>
`sudo apt-get -y install dovecot-core dovecot-pop3d dovecot-imapd`<br>
Configure Dovecot<br>
`sudo nano /etc/dovecot/dovecot.conf`  [file](./configs/dovecot.conf)<br>
![dovecot-config-1](./images/dovecot-config-1.PNG)
`sudo nano /etc/dovecot/conf.d/10-auth.conf`  [file](./configs/10-auth.conf)<br>
![dovecot-config-2](./images/dovecot-config-2.PNG)
![dovecot-config-3](./images/dovecot-config-3.PNG)
`sudo nano /etc/dovecot/conf.d/10-mail.conf`  [file](./configs/10-mail.conf)<br>
![dovecot-config-4](./images/dovecot-config-4.PNG)
`sudo nano /etc/dovecot/conf.d/10-master.conf`  [file](./configs/10-master.conf)<br>
![dovecot-config-5](./images/dovecot-config-5.PNG)
`sudo nano /etc/dovecot/conf.d/10-ssl.conf`  [file](./configs/10-ssl.conf)<br>
![dovecot-config-6](./images/dovecot-config-6.PNG)
Dovecot run POP service on tcp port 110 and IMAP on tcp port 143<br>
Configure mail account<br>
`sudo apt-get -y install mailutils`<br>
`sudo nano /etc/profile.d/mail.sh`<br>
![mailutils-config](./images/mailutils-config.PNG)
`sudo systemctl restart postfix dovecot`<br>
Verify configurations<br>
![mail-1](./images/mail-1.PNG)
![mail-2](./images/mail-2.PNG)
![mail-3](./images/mail-3.PNG)
![mail-4](./images/mail-4.PNG)
![mail-5](./images/mail-5.PNG)
![mail-6](./images/mail-6.PNG)

### Other Config
Create SSL certificates self-sign<br>
`sudo openssl genrsa -aes128 -out /etc/ssl/private/server.key 2048`<br>
`sudo openssl rsa -in /etc/ssl/private/server.key -out /etc/ssl/private/server.key`<br>
`sudo openssl req -new -days 365 -key /etc/ssl/private/server.key -out /etc/ssl/private/server.csr`<br>
`sudo openssl x509 -in /etc/ssl/private/server.csr -out /etc/ssl/private/server.crt -req -signkey /etc/ssl/private/server.key -days 365`<br>
![ssl-1](./images/ssl-1.PNG)
![ssl-2](./images/ssl-2.PNG)

** **

**NOTE:**<br>
Download Ubuntu Server ARM Raspberry Pi [here](https://ubuntu.com/download/raspberry-pi).<br>
Raspberry Pi 4 ARM 64-bit 4GB RAM<br>
MikroTik RB941-2nD
