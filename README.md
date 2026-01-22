# HowTo_Linux
few commands to setup and secure linux machine

## Update

`sudo apt-get update` </br>
`sudo apt-get upgrade` </br>

## UFW

`sudo apt-get install ufw` <br>
`sudo ufw allow 22` <br>

allow to use port only from entered IP <br>
`sudo ufw allow from 46.151.60.216 to any port 22 proto tcp` <br>
`sudo ufw allow from 178.143.191.171 to any port 22 proto tcp` <br>

`sudo systemctl start ufw` <br>
`sudo systemctl enable ufw` <br>
`sudo ufw enable` <br>
`sudo ufw allow 11200:11300/tcp` <br>
`sudo ufw allow 80/tcp`  port pre http<br> 
`sudo ufw allow 443/tcp`  port pre https<br> 

Zobraziť existujúce pravidlá <br>
`sudo nano /etc/ufw/user.rules` <br>
alebo <br>
`sudo ufw status` <br>

## Fail2Ban

`sudo apt-get install fail2ban` </br>

fail2ban.conf contains the default configuration profile. The default settings give you a reasonable working setup. If you want to make </br>
any changes, it’s best to do it in a separate file, fail2ban.local, which overrides fail2ban.conf. Rename a copy fail2ban.conf to fail2ban.local </br>

`sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local` </br>

`sudo systemctl enable fail2ban` </br>
`sudo systemctl start fail2ban` </br>

test ci sa da spustit </br>
`sudo fail2ban-client -x start` </br>

Ak nevie najst log subor.
[sshd]
enabled = true
port    = ssh
backend = systemd

zobrazí zabanovane ip adresy </br>
`sudo fail2ban-client status sshd` </br>

unban ip </br>
`sudo fail2ban-client unban ipaddress` </br>

pridat ip do whitelistu </br>
`etc/fail2ban/jail.local` </br>

debian rsyslog </br>

```
sudo apt install rsyslog
sudo systemctl start rsyslog
sudo systemctl enable rsyslog
sudo systemctl status rsyslog
```

debian jail.local </br>

```
[sshd]
port    = 22
logpath = /var/log/auth.log
backend = auto
enabled = true
maxretry = 3
bantime = 3000m
```

## nginx <br>

## WASM hosting <br>

`sudo mkdir -p /var/www/blazorapp` <br>
`sudo nano /etc/nginx/sites-available/blazorapp` <br>
`sudo chown -R username:username /var/www/blazorapp` <br>

```
server {
    listen 80;
    server_name _;  # or use the Pi's IP if local

    root /var/www/blazorapp/wwwroot;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

`sudo ln -s /etc/nginx/sites-available/blazorapp /etc/nginx/sites-enabled/` <br>

remove default from available sites

`sudo nginx -t` <br>
`sudo systemctl reload nginx` <br>

logs - `sudo tail -n 50 /var/log/nginx/error.log` <br>

Na machine kde bezi reverse proxy -

```
location / {
    proxy_pass http://192.168.1.154:80;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

vytvorit novy subor servers.conf v umiestneni `/etc/nginx`, ktory sa da do include v sekcii http <br>

`touch servers.conf` <br>

refresh konfiguracie pri zmene <br>
`sudo service nginx reload` <br>

nastavenie nginx certifikatu pre 443 <br>

`sudo ufw allow 443/tcp` <br>
`sudo apt install certbot python3-certbot-nginx`  <br>
`sudo certbot --nginx` Manualne vytvorenie certifikatov pre domeny <br>

prikaz na pridanie certifikatu pre subdomenu <br>
`sudo certbot --nginx -d exalogic.server1.oberonlink.sk --agree-tos --force-renewal --non-interactive` <br>

#### nginx - konfiguracia servers.conf <br>

zisti k akej skupine patri subor <br>
`ls -l /etc/nginx/servers.conf` <br>

pridat usera do permission filu (uz netreba robit) <br>
`sudo chgrp kalixtvps /etc/nginx/servers.conf` <br>
`sudo chmod g+w /etc/nginx/servers.conf` <br>

## Wordpress hosting <br>

`sudo apt install nginx php-fpm php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip` <br>

`sudo apt install php-fpm php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip -y` <br>

`sudo systemctl start php8.2-fpm` <br>
`sudo systemctl enable php8.2-fpm` <br>

```
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo mv wordpress /var/www/wordpress
```

`sudo chown -R www-data:www-data /var/www/wordpress` <br>
`sudo chmod -R 755 /var/www/wordpress` <br>

`sudo cp /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php` <br>
`sudo nano /var/www/wordpress/wp-config.php` <br>

```
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpress_user');
define('DB_PASSWORD', 'your_secure_password');
define('DB_HOST', 'localhost');
https://api.wordpress.org/secret-key/1.1/salt/
```

`sudo nano /etc/nginx/sites-available/wordpress` <br>

```
server {
    listen 80;
    server_name your_domain www.your_domain;

    root /var/www/wordpress;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock; # Adjust version if needed
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    }
}
```

`sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/` <br>
`sudo nginx -t` <br>
`sudo systemctl reload nginx` <br>

`sudo apt install certbot python3-certbot-nginx -y` <br>
`sudo certbot --nginx -d your_domain -d www.your_domain` <br>

`sudo find /var/www/wordpress -type d -exec chmod 750 {} \;` <br>
`sudo find /var/www/wordpress -type f -exec chmod 640 {} \;` <br>

## Certbot <br>

`sudo apt install certbot python3-certbot-nginx` <br>
`sudo certbot --nginx` <br>

Aktualizovať certifikáty  `sudo certbot renew` <br>
Vymazať certifikát `sudo certbot delete` <br>

## Users <br>

zoznam používateľov  `getent passwd` <br>
vymazanie usera `sudo userdel username` <br>

## Zaregistrovat API ako service <br>

`sudo nano /etc/systemd/system/myapi.service` <br>

```
[Unit]
Description=baneer online API
After=network.target

[Service]
WorkingDirectory=/var/www/calibos
ExecStart=/var/www/calibos/Calibos
Restart=always
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=calibos
User=raspidev
Environment=ASPNETCORE_URLS=http://0.0.0.0:5000

[Install]
WantedBy=multi-user.target
```

`sudo systemctl daemon-reload` <br>
`sudo systemctl enable myapi` <br>
`sudo systemctl start myapi` <br>
`sudo systemctl status myapi` <br>

remove <br>
`sudo systemctl disable myapi` <br>
`sudo systemctl stop myapi` <br>

## MariaDB <br>

`sudo apt install mariadb-server` <br>
`sudo mariadb-secure-installation` <br>

Treba zmenit IP na lokalnu IP tohto PC <br>
`sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf` <br>

 Start MariaDB
`sudo systemctl start mariadb` <br>
 Enable MariaDB to start on boot
`sudo systemctl enable mariadb` <br>
 Check status
`sudo systemctl status mariadb` <br>

Pridať usera na remote access

`sudo mariadb` <br>
`SELECT host FROM mysql.user WHERE user = 'root';` <br>
malo by vratit len localhost pre roota

Vytvorenie usera

```
CREATE USER 'youruser'@'%' IDENTIFIED BY 'yourpassword';
GRANT ALL PRIVILEGES ON *.* TO 'youruser'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

User list a vymazanie

`SELECT user, host FROM mysql.user;` <br>

```
DROP USER 'testuser'@'%';
DROP USER 'root'@'192.168.1.10';
DROP USER 'admin'@'localhost';
```

Povolenie root remote login len pre vybranu IP <br>
```
CREATE USER 'root'@'46.151.60.216' IDENTIFIED BY 'STRONG_PASSWORD_HERE';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'46.151.60.216' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
## Users permissions <br>

`sudo chmod u+rwx,g+rx,o+rx /var/www/apifolder -R` <br>


## Network <br>

`sudo nano /etc/network/interfaces` <br>

`sudo systemctl restart networking` <br>

## Speedtest <br>

`sudo apt install speedtest-cli` <br>
`speedtest-cli` <br>

## Temp <br>

`sudo apt install lm-sensors` <br>
`sensors` <br>

## Debian PATH fix <br>

```
echo 'export PATH=$PATH:/usr/sbin' >> ~/.bashrc
source ~/.bashrc
```
## sudo <br>

`apt install sudo` <br>

Add pecpc to sudo group <br>
`usermod -aG sudo pecpc` <br>
Verify group membership <br>
`groups pecpc` <br>

## systemctl <br>

`sudo systemctl list-units --type=service` <br>
`sudo systemctl stop name.service` <br>
`sudo systemctl disable name.service` <br>

## Backup <br>

Info <br>
`lsblk` <br>
`df -h` <br>

raspi backup SD card to SD card <br>
`sudo dd if=/dev/mmcblk0 of=/dev/sda bs=4M status=progress` <br>

## Micro <br>

ked otvoris mc, napis select-editor a mozes ho zmenit na micro z nano
sudo apt-get install micro
sudo update-alternatives --config editor takto sa meni editor system wide, nie len v mc, ale nemusi to respektovat mc podla nastavenia


