/* Add a record and cname record at domain name control panel */
;; A Records
@	     IP-address-of-server
;; CNAME Records
www	     example.com

![image](https://user-images.githubusercontent.com/26363918/221420944-7428c61d-a5ab-4995-91b7-4ad37b9ac3b8.png)


```
sudo -s
```

/* Stop restart required prompt for Ubuntu22 */
[Source](https://stackoverflow.com/questions/73397110/how-to-stop-ubuntu-pop-up-daemons-using-outdated-libraries-when-using-apt-to-i)

```
sed -i "s/#\$nrconf{restart} = 'i';/\$nrconf{restart} = 'a';/" /etc/needrestart/needrestart.conf
```

/* update and upgrade */
```
apt update && apt upgrade -y && apt dist-upgrade && apt autoclean && apt autoremove -y && reboot
```

/* Install Nginx */
```
apt install nginx -y && chown -R www-data:www-data /var/www/ && chmod -R 755 /var/www
```

/* updates , firewall and nginx */

```
ufw enable

ufw allow ssh
ufw allow 'Nginx Full'
systemctl restart nginx
```

/* Install and optimize PHP */
```
sudo add-apt-repository ppa:ondrej/php -y && apt update && apt install php8.2-fpm php8.2-common php8.2-mysql php8.2-xml php8.2-xmlrpc php8.2-curl php8.2-gd php8.2-imagick php8.2-cli php8.2-dev php8.2-imap php8.2-mbstring php8.2-soap php8.2-zip php8.2-bcmath -y && sed -i 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' /etc/php/8.2/fpm/php.ini && service php8.2-fpm restart  
```
```
sed -i 's/memory_limit = 128M/memory_limit = 512M/g' /etc/php/8.2/fpm/php.ini
sed -i 's/post_max_size = 8M/post_max_size = 128M/g' /etc/php/8.2/fpm/php.ini
sed -i 's/max_file_uploads = 20/max_file_uploads = 30/g' /etc/php/8.2/fpm/php.ini
sed -i 's/max_execution_time = 30/max_execution_time = 900/g' /etc/php/8.2/fpm/php.ini
sed -i 's/max_input_time = 60/max_input_time = 3000/g' /etc/php/8.2/fpm/php.ini
sed -i 's/upload_max_filesize = 2M/upload_max_filesize = 128M/g' /etc/php/8.2/fpm/php.ini
service php8.2-fpm restart
```

/* Add server block */

```
rm /etc/nginx/sites-available/default
rm /etc/nginx/sites-enabled/default


cd /var/www/
mkdir example.com
cd example.com
mkdir cache
mkdir wordpress
cd wordpress
mkdir public_html

/var/www/example.com/wordpress/public_html

cd /etc/nginx/sites-available/
nano example.com
```


--------------------------------------------------------------------------------------------
```
fastcgi_cache_path /var/www/example.com/cache levels=1:2 keys_zone=example.com:100m inactive=60m;

server {
  listen 80;

  root /var/www/example.com/wordpress/public_html;
  index index.php index.html index.htm;

  server_name  example.com www.example.com;
  client_max_body_size 0;

 error_page 401 403 404 /404.html;
  error_page 500 502 503 504 /50x.html;


###############
set $skip_cache 0;

# POST requests and urls with a query string should always go to PHP
if ($request_method = POST) {
     set $skip_cache 1;
}

#if ($query_string != "")

if ($query_string = "unapproved*") {
     set $skip_cache 1;
}

if ( $cookie_woocommerce_items_in_cart = "1" ){
     set $skip_cache 1;
}

# Don't cache URIs containing the following segments
if ($request_uri ~* "/wp-admin/|/xmlrpc.php|/wp-.*.php|index.php|sitemap(_index)?.xml")  {
     set $skip_cache 1;
}
if ($request_uri ~* "/(cart|checkout|my-account)/*$") {
     set $skip_cache 1;
}

# Don't use the cache for logged-in users or recent commenters
if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_no_cache|wordpress_logged_in") {
     set $skip_cache 1;
}

###############

location / {
    try_files $uri $uri/ /index.php?q=$uri&$args;
  }

  location ~* \.php$ {
    if ($uri !~ "^/uploads/") {
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        }
    include         fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
  
################
fastcgi_cache_bypass $skip_cache;
fastcgi_no_cache $skip_cache;
fastcgi_cache example.com;
fastcgi_cache_valid 200 301 24h;
###############


 }
location = /favicon.ico { 
    log_not_found off;
    access_log off;
  }
  
  location = /robots.txt {
    log_not_found off;
    access_log off;
    allow all; 
  }
  
  location ~* .(css|gif|ico|jpeg|jpg|js|png)$ {
    expires 30d;
    log_not_found off;
  }
}
```

/* Enable Gzip */

```
nano /etc/nginx/nginx.conf
```

Seach for line "gzip on" and replace that by code below:-

```
# Enable Gzip compression.
gzip on;

# Disable Gzip on IE6.
gzip_disable "msie6";

# Allow proxies to cache both compressed and regular version of file.
# Avoids clients that don't support Gzip outputting gibberish.
gzip_vary on;

# Compress data, even when the client connects through a proxy.
gzip_proxied any;

# The level of compression to apply to files. A higher compression level increases
# CPU usage. Level 5 is a happy medium resulting in roughly 75% compression.
gzip_comp_level 5;

# Compress the following MIME types.
gzip_types
	application/atom+xml
	application/javascript
	application/json
	application/ld+json
	application/manifest+json
	application/rss+xml
	application/vnd.geo+json
	application/vnd.ms-fontobject
	application/x-font-ttf
	application/x-web-app-manifest+json
	application/xhtml+xml
	application/xml
application/x-javascript


	font/opentype
font/ttf
font/eot
font/otf


	image/bmp
	image/svg+xml
	image/x-icon
	text/cache-manifest
	text/css
	text/plain
	text/vcard
	text/vnd.rim.location.xloc
	text/vtt
	text/x-component
	text/x-cross-domain-policy;


##
# Cache Settings
##
fastcgi_cache_key "$scheme$request_method$host$request_uri";
add_header Fastcgi-Cache $upstream_cache_status;
fastcgi_cache_use_stale error timeout invalid_header http_500;
fastcgi_ignore_headers Cache-Control Expires Set-Cookie;
```
---------------------------------------------------------------------------

```
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo nginx -t
service php8.2-fpm restart
service nginx restart
```

/*Test nginx and php */
```
echo "<?php phpinfo(); ?>" >> /var/www/example.com/wordpress/public_html/info.php
rm -rf /var/www/example.com/wordpress/public_html/info.php
```


/* Install SSL */
```
apt-get install python3-certbot-nginx
certbot --nginx -d new.example.com
```

```
sudo crontab -e
0 0,12 * * * certbot renew >/dev/null 2>&1
```


/* Enable HTTP2 */
```
sudo nano /etc/nginx/sites-available/example
listen 443 ssl http2 default_server;
listen [::]:443 ssl http2 default_server;
```


/* Install and configure mariadb */
```
sudo apt install mariadb-server -y && sudo mysql_secure_installation && mysql -u root -p
create database myblog1;
CREATE USER myuser1@localhost IDENTIFIED BY 'Secure2018#';
GRANT ALL PRIVILEGES ON myblog1.* TO myuser1@localhost;
FLUSH PRIVILEGES;
exit;
```


/* Download and install WordPress */

```
cd /var/www/example.com/wordpress/public_html

wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
mv -v wordpress/* /var/www/example.com/wordpress/public_html
rmdir wordpress
rm latest.tar.gz

sudo chown -R www-data:www-data /var/www/
sudo find /var/www/ -type d -exec chmod 755 {} \;
sudo find /var/www/ -type f -exec chmod 644 {} \;

apt-get update
service nginx restart
systemctl restart php8.2-fpm.service
systemctl restart mysql

```

Plugin to purge fastcgi cache
```
sed -i "s/define( 'WP_DEBUG', false );/define( 'WP_DEBUG', false );define( 'RT_WP_NGINX_HELPER_CACHE_PATH', '\/var\/www\/example.com\/cache' );/" /var/www/example.com/wordpress/public_html/wp-config.php
```


/* to make sftp edit/upload work */
/* To add new SFTP user */
```
adduser ubuntu                       
usermod -aG sudo ubuntu              
sudo groupadd wwwubuntu                  
sudo usermod -a -G wwwubuntu ubuntu      
sudo usermod -a -G wwwubuntu www-data    


sudo chgrp -R wwwubuntu /var/www/example.com/wordpress/public_html
sudo find /var/www/example.com/wordpress/public_html -type d -exec chmod 775 {} \;
sudo find /var/www/example.com/wordpress/public_html -type f -exec chmod 664 {} \;
```






