# Proxy caching with varnish

## Install varnish

```bash
apt-get install apt-transport-https
curl https://repo.varnish-cache.org/GPG-key.txt | apt-key add -
echo "deb https://repo.varnish-cache.org/ubuntu/ trusty varnish-4.1" >> /etc/apt/sources.list.d/varnish-cache.list
apt-get update
apt-get install varnish
```

## Setup default options (/etc/default/varnish)

```
DAEMON_OPTS="-a :80 \
             -T localhost:6082 \
             -f /etc/varnish/default.vcl \
             -S /etc/varnish/secret \
             -s malloc,256m"
```

## Setup varnish options (/etc/varnish/default.vcl)

```
vcl 4.0;

backend default{
  .host = "127.0.0.1";
  .port = "8080";
  .connect_timeout = 10s;
  .first_byte_timeout = 15s;
  .between_bytes_timeout = 60s;
  .max_connections = 800;
}
 
sub vcl_recv {
  unset req.http.cookie;
}

# Set 5 min cache if unset
sub vcl_backend_response{
	if (beresp.ttl <= 0s) {
	  set beresp.ttl = 300s;
	  set beresp.uncacheable = false;
	  return (deliver);
	}
	return (deliver);
}

# Set cache helper headers
sub vcl_deliver {
	if (obj.hits > 0) {
    set resp.http.X-Cache = "HIT";
  } else {
    set resp.http.X-Cache = "MISS";
  }
}
```

## Setup nginx (/etc/nginx/sites-enabled/default)

```
upstream node_server { 
  server 127.0.0.1:80; 
}
 
map $http_origin $cors_header {
  default "";
  "~^https?://[^/]+\.oramind\.net(:[0-9]+)?$" "$http_origin";
}

server {
  listen 443 ssl spdy;
  server_name gw2-api.com;
  error_page 404 /404.html;

  location / {
    proxy_pass http://node_server;
  }

  ssl on;
  ssl_certificate /etc/nginx/ssl/www_gw2api_com_bundle.crt;
  ssl_certificate_key /etc/nginx/ssl/www_gw2api_com.key;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
  ssl_prefer_server_ciphers on;
}
```
