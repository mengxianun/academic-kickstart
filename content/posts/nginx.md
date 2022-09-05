---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Nginx"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2022-09-05T10:20:54+08:00
lastmod: 2022-09-05T10:20:54+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

SSL

1. 生成证书

   - openssl

     ```
     openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout 23.key -out 23.crt -config localhost.conf
     ```

   - 查看证书

     ```
     openssl x509 -in localhost.crt -noout -text
     ```

   - localhost.conf

     ```
     [req]
     default_bits       = 2048
     default_keyfile    = localhost.key
     distinguished_name = req_distinguished_name
     req_extensions     = req_ext
     x509_extensions    = v3_ca
     
     [req_distinguished_name]
     countryName                 = Country Name (2 letter code)
     countryName_default         = CN
     stateOrProvinceName         = State or Province Name (full name)
     stateOrProvinceName_default = Beijing
     localityName                = Locality Name (eg, city)
     localityName_default        = Beijing
     organizationName            = Organization Name (eg, company)
     organizationName_default    = localhost
     organizationalUnitName      = organizationalunit
     organizationalUnitName_default = Development
     commonName                  = Common Name (e.g. server FQDN or YOUR name)
     commonName_default          = localhost
     commonName_max              = 64
     
     [req_ext]
     subjectAltName = @alt_names
     
     [v3_ca]
     subjectAltName = @alt_names
     
     [alt_names]
     DNS.1   = localhost
     DNS.2   = 127.0.0.1
     IP.1 = 192.168.1.1
     ```

2. 配置nginx.conf

   ```
   
   #user  nobody;
   worker_processes  1;
   
   #error_log  logs/error.log;
   #error_log  logs/error.log  notice;
   #error_log  logs/error.log  info;
   
   #pid        logs/nginx.pid;
   
   
   events {
       worker_connections  1024;
   }
   
   
   http {
       include       mime.types;
       default_type  application/octet-stream;
   
       #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
       #                  '$status $body_bytes_sent "$http_referer" '
       #                  '"$http_user_agent" "$http_x_forwarded_for"';
   
       #access_log  logs/access.log  main;
   
       sendfile        on;
       #tcp_nopush     on;
   
       #keepalive_timeout  0;
       keepalive_timeout  65;
   
       #gzip  on;
   	
   	map $http_upgrade $connection_upgrade {
   		default upgrade;
   		'' close;
   	}
   
       server {
           listen       80;
           server_name  localhost;
   
           #charset koi8-r;
   
           #access_log  logs/host.access.log  main;
   
           location / {
               root   html;
               index  index.html index.htm;
           }
   
           #error_page  404              /404.html;
   
           # redirect server error pages to the static page /50x.html
           #
           error_page   500 502 503 504  /50x.html;
           location = /50x.html {
               root   html;
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
   
   
       # another virtual host using mix of IP-, name-, and port-based configuration
       #
       #server {
       #    listen       8000;
       #    listen       somename:8080;
       #    server_name  somename  alias  another.alias;
   
       #    location / {
       #        root   html;
       #        index  index.html index.htm;
       #    }
       #}
   
   
       # HTTPS server
       
       server {
           listen       9443 ssl;
           server_name  localhost;
   
           ssl_certificate      localhost.crt;
           ssl_certificate_key  localhost.key;
   
           ssl_session_cache    shared:SSL:1m;
           ssl_session_timeout  5m;
   
           ssl_ciphers  HIGH:!aNULL:!MD5;
           ssl_prefer_server_ciphers  on;
   
           location / {
               root   html;
               index  index.html index.htm;
           }
   		
   		location /api/ {
   			#proxy_pass http://api-server;
   			proxy_pass http://localhost:8080;
   			#proxy_connect_timeout 300s;
   			#client_max_body_size 100m;
   			#proxy_send_timeout 300s;
   			#proxy_read_timeout 300s;
   			proxy_http_version 1.1;
   			#proxy_set_header X-Real-IP           $remote_addr;
   			proxy_set_header Upgrade             $http_upgrade;
   			proxy_set_header Connection          $connection_upgrade;
   			#proxy_set_header Connection          "upgrade";
   			#proxy_set_header Origin              "";
   			#proxy_set_header Host                $host:$server_port;
   			#proxy_set_header X-Forwarded-For     $proxy_add_x_forwarded_for;
   			#proxy_set_header X-Forwarded-Proto   $scheme;
   			#proxy_set_header X-Forwarded-Port    $server_port; 
   		}
   
   }
   
   ```

   

3. 
