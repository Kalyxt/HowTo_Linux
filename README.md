# HowTo_Linux
few commands to setup or secure linux machine

## Update

`sudo apt-get update` </br>
`sudo apt-get upgrade` </br>

## UFW

`sudo apt-get install ufw` <br>
`sudo ufw allow 22` <br>
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

## nginx <br>

## WASM hosting <br>

`sudo mkdir -p /var/www/blazorapp` <br>
`sudo nano /etc/nginx/sites-available/blazorapp` <br>
`sudo chown -R www-data:www-data /var/www/blazorapp` <br>

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

## Certbot <br>

`sudo apt install certbot python3-certbot-nginx` <br>
`sudo certbot --nginx` <br>

Aktualizovať certifikáty  `sudo certbot renew` <br>
Vymazať certifikát `sudo certbot delete` <br>

## Users <br>

zoznam používateľov  `getent passwd` <br>
vymazanie usera `sudo userdel username` <br>

## MariaDB <br>

`sudo apt install mariadb-server` <br>
`sudo mariadb-secure-installation` <br>

`sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf` <br>

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

## Speedtest <br>

`sudo apt install speedtest-cli` <br>
`speedtest-cli` <br>

## Temp <br>

`sudo apt install lm-sensors` <br>
`sensors` <br>
