Step 1: Start the first container to act as the HTTP Proxy for the others

```
docker run -it -p80:80 -p443:443 --mount type=bind,source="/Users/Shared",target=/shared ubuntu /bin/bash  
```
Step 2: Install apache / certbot / nano for working in this HTTP-proxy container
```
apt install apache2 nano python3-certbot-apache
```
Step 3: Enable the required Apache modules
```
a2enmod proxy rewrite proxy_http
```

Step 4: Sample Virtualhost file to replace /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
        ServerName mimecast-1.bsidespgh-ctf.com
        ProxyPreserveHost off
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined


        #Include conf-available/serve-cgi-bin.conf
        RewriteEngine On
        RewriteCond %{HTTP:Upgrade} =websocket [NC]
        RewriteRule ^/(.*)           ws://YOURHOST:6901/$1 [P,L]
        RewriteCond %{HTTP:Upgrade} !=websocket [NC]
        RewriteRule ^/(.*)           http://YOURHOST:6901/$1 [P,L]
</VirtualHost>


certbot --apache


root@cf6d00ce8ae6:/# cat /etc/apache2/sites-available/000-default-le-ssl.conf
<IfModule mod_ssl.c>
<VirtualHost *:443>
        ServerName mimecast-1.bsidespgh-ctf.com
        ProxyPreserveHost off
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined


        #Include conf-available/serve-cgi-bin.conf
        RewriteEngine On
        RewriteCond %{HTTP:Upgrade} =websocket [NC]
        RewriteRule ^/(.*)           ws://10.0.2.16:6901/$1 [P,L]
        RewriteCond %{HTTP:Upgrade} !=websocket [NC]
        RewriteRule ^/(.*)           http://10.0.2.16:6901/$1 [P,L]


SSLCertificateFile /etc/letsencrypt/live/mimecast-1.bsidespgh-ctf.com/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/mimecast-1.bsidespgh-ctf.com/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</IfModule>

