---
title: Nginx 設定檔筆記
date: 2020-03-23 17:06:30
tags:
- Nginx
- Web
- DevOps
---

## Nginx相關設定檔的位置

- 主要設定檔在`/etc/nginx/nginx.conf`
-  `/etc/nginx/conf.d/` 、 `/etc/nginx/site-enable/`或 `/etc/nginx/site-available/`下面會有更細的設定檔，我們需要從`/etc/nginx/nginx.conf`裡面include的路徑來判斷
-

查看設定檔
```sh
cat /etc/nginx/nginx.conf
```

### 開源版的設定

```sh
user www-data;
# 使用者名稱，使用 ps aux 查看 process會顯示這個名稱

worker_processes auto;
# Nginx的執行緒數量（建議為機器CPU數量x2）

pid /run/nginx.pid;
# Nginx的執行緒pid

include /etc/nginx/modules-enabled/*.conf;
# 讀取 ./modules-enabled/ 內的設定檔


# events 區塊，負責處理每個連結
events {
	worker_connections 768;
  # 每個執行緒同一時間允許的連線總數量

	# multi_accept on;
}

# http 區塊，負責處理透過http協定的連結
http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
  # Access log 檔案存放位置
	error_log /var/log/nginx/error.log;
  #  log 檔案存放位置

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}

# http 區塊，負責處理透過mail協定的連結，目前註解掉了
#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
#
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}
```


### 商業版的設定

``` sh
user  nginx;
# 使用者名稱，使用 ps aux 查看 process會顯示這個名稱
worker_processes  1;
# Nginx的執行緒數量（建議為機器CPU數量x2）

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
# Nginx的執行緒pid


events {
    worker_connections  1024;
    # 每個執行緒同一時間允許的連線總數量
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
    #這行會讀取到前面那個default.conf，表示/conf.d/default.conf是專門給http使用

    ---------------------------------------------
    #設定load balancer，首先要設定server的group，放在upstream這個指標裡面，group名稱為'backend'

    upstream backend {
        server backend1.example.com weight=5;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }
    ---------------------------------------------
    #slow_start參數：server重啟可能一瞬間收到大量的流量又掛掉一次，設定slow_start可以在設定時間內讓流量逐漸達到預設值。

    upstream backend {
        server backend1.example.com slow_start=30s;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }
    ---------------------------------------------
}
```

用`cat`查閱一下設定檔
```sh
cat /etc/nginx/conf.d/default.conf
```
會跑出

```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```





## Load Balancing 的幾種方式

- Round Robin(default): 可以在IP/domain設定weight權重，依照權重分配
- Least Connections: 流量會導到連結最少的那個server
- IP Hash: 會將IPv4的第一組八位元或是IPv6整串經過hash table對應到一台server，只要使用者從同一個IP來，流量就會固定對應到同一台server。
- Generic Hash:
- Least Time (NGINX Plus only)
- Random:

## Session Persistence 會話持續

- Sticky cookie
- Sticky route
- Sticky learn


## References

[HTTP Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)
