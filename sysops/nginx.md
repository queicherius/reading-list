# Setup ubuntu server with nginx and PHP

```bash
# Update everything!
apt-get update && apt-get upgrade

## TODO ##
# Generate a keyfile for logging in instead of using passwords

# Install nginx and php fpm
apt-get install nginx php5-fpm php5-cli php5-mcrypt git

# Update php config (set "fix_pathinfo=0")
nano /etc/php5/fpm/php.ini

# Enable mcyrypt for php
sudo php5enmod mcrypt
sudo service php5-fpm restart

# Generate the directory for the sourcecode
mkdir -p /var/www/html

# Edit the nginx config (see below)
nano /etc/nginx/sites-enabled/default

# Restart nginx for changes to take effect
sudo service nginx restart

# Change into directory and clone code
cd /var/www/html
git clone http://github.com/...
```

## `/etc/nginx/sites-enabled/default`

```
server {
	listen 443 ssl spdy;

	root /var/www/html/gw2-api/public;
	index index.php;
	server_name gw2-api.com;
	error_page 404 /404.html;

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

	location ~ \.php$ {
		try_files $uri /index.php =404;
		fastcgi_split_path_info ^(.+\.php)(/.+)$;
		fastcgi_pass unix:/var/run/php5-fpm.sock;
		fastcgi_index index.php;
		fastcgi_intercept_errors on;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}

	ssl on;
	ssl_certificate /etc/nginx/ssl/www_gw2api_com_bundle.crt;
	ssl_certificate_key /etc/nginx/ssl/www_gw2api_com.key;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
	ssl_prefer_server_ciphers on;
}
```
