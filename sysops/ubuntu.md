# Setup ubuntu server

> This assumes a raw Ubuntu 16.04.1 LTS system

## General setup

### Switch to the root login

I want to login as root, and the raw install does not have a password for it set, so you can't log in.

> It's true that this is less secure than giving superuser priviledges to the original user, but having to prepend `sudo` before every command defeats the purpose as well. This also makes port-binding easier.

```bash
# Set a password for root
sudo passwd root

# Update the SSH file to allow login with passwords (this will get changed below)
# -> PermitRootLogin yes
sudo nano /etc/ssh/sshd_config
sudo service ssh restart

# Logout and log back in as "root" with the password
logout

# Remove the (now obsolete) user
userdel -r david
```

### Setting up networking

Since Ubuntu 15, the network interfaces have different names than `eth0` etc. 
In my opinion that's confusing, so I am changing it back.

```bash
nano /etc/default/grub
# -> change GRUB_CMDLINE_LINUX="net.ifnames=0"
update-grub
reboot
```

### Setup the static IP

```bash
nano /etc/network/interfaces
```

```
auto eth0
iface eth0 inet static
address x.x.x.x
netmask 255.255.255.255
gateway x.x.x.254
dns-nameservers 8.8.8.8 8.8.4.4
```

```bash
ping google.com
```

### Update the system

```bash
# Update everything!
apt-get update && apt-get upgrade
apt-get install build-essential 
```

> **Note:** Since we changed the "grub" file, it might ask you if you want to keep your version or update it. Just check the diff to see if everything is in order.

### SSH-Key only login

```
# [CLIENT] Generate a keyfile or use an existing keyfile (skip this)
ssh-keygen -t rsa -C "my@email.com"

# [CLIENT] Copy the keyfile onto the server
ssh-copy-id -i ~/.ssh/id_rsa root@serverip

# [CLIENT] Login without the SSH key (should not promt for user password!)
ssh root@serverip

# ---------- Server now ----------

# Don't continue if we can't log in with the key!
# Disable password login
# -> PermitRootLogin without-password
# -> PasswordAuthentication no
nano /etc/ssh/sshd_config
service ssh restart

# Install fail2ban for blocking too many ssh tries in a row
apt-get install fail2ban
```

### Update your timezone

The timezone should be set to UTC

```
rm /etc/localtime
ln -s /usr/share/zoneinfo/UTC /etc/localtime

# Now these two commands should say the same
date
date -u
```

## nginx and PHP

```bash
# Install nginx, php fpm, git and htop
apt-get install nginx php5-fpm php5-cli php5-mcrypt php5-mysqlnd git htop php5-curl

# Update php config (change fix_pathinfo=0)
nano /etc/php5/fpm/php.ini

# Enable mcyrypt for php
sudo php5enmod mcrypt
sudo service php5-fpm restart

# Install composer
wget https://getcomposer.org/composer.phar
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer

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

### `/etc/nginx/sites-enabled/default`

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

### Restarting

```bash
sudo service php5-fpm restart && sudo service nginx restart
```

**Note** Sometimes the php5-fpm process doesn't want to restart, in which case you should try and kill the process by process ids.

## Mysql

```bash
apt-get install mysql-server
sudo mysql_install_db
sudo /usr/bin/mysql_secure_installation # "yes" to all questions
```

Create a user:

```sql
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost';
CREATE USER 'username'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'username'@'%';
FLUSH PRIVILEGES;
```

## Redis

```bash
sudo apt-get install redis-server
sudo nano /etc/redis/redis.conf # set "requirepass <password>" & "bind 0.0.0.0"
```

## Node.js

```bash
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```

> **Note**: Confirm with `node -v` that it actually installed the right version, if the installer fails it may install the version of the official repository (which is 4.x)

## Python

```bash
apt-get install python
```

## MongoDB

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo service mongod start
```

## Elasticsearch

```bash
apt-get install default-jre
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/deb/elasticsearch/2.3.3/elasticsearch-2.3.3.deb
sudo dpkg -i elasticsearch-2.3.3.deb
sudo systemctl enable elasticsearch.service
service elasticsearch start
```

## Ruby & Compass

```bash
# Install all dependencies
sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties
sudo apt-get install libgdbm-dev libncurses5-dev automake libtool bison libffi-dev

# Install rvm
curl -L https://get.rvm.io | bash -s stable
source ~/.rvm/scripts/rvm
echo "source ~/.rvm/scripts/rvm" >> ~/.bashrc

# Install ruby
rvm install 2.1.2
rvm use 2.1.2 --default
ruby -v

# Install compass
gem install compass
```
