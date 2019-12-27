# 推荐目录结构
```
/root/docker/mysql
/root/docker/nginx
/root/docker/scripts
/root/docker/svn
/root/docker/www
```
## 容器自启动脚本
  /root/docker/scripts/startup.sh
```
#!/bin/bash
# start docker container
sudo systemctl start docker
docker start $(docker ps -aq -f status=exited)
```
  
# 拉取 SVN 镜像
```
docker pull garethflowers/svn-server
docker run \
    --name svn \
    --detach \
    --volume /root/docker/svn:/var/opt/svn \
    --volume /root/docker/www:/www \
    --publish 3690:3690 \
    garethflowers/svn-server
```
## 创建仓库
```
docker exec -it svn svnadmin create test
```
## 设置钩子
```
/usr/bin/svn update --username your-name --password your-password /www/test/
```
## 初始化部署目录
```
docker exec -it svn svn co svn://your-ip/test /www/test --username your-name --password your-password
```

# 拉取 MySQL 镜像
```
docker pull mysql:5.6
docker run --name mysql -p 3306:3306 \
-v /root/docker/mysql:/etc/mysql/sqlinit \
-e MYSQL_ROOT_PASSWORD=your-password \
-d mysql:5.6
```

# 拉取 NGINX-PHP-FPM 仓库
```
git clone https://github.com/toctrash/nginx-php-fpm.git
docker build -t nginx-php-fpm .
docker run --name nginx-php-fpm -p 80:80 \
-v /root/docker/www:/var/www/html \
-v /root/docker/nginx/conf/conf.d:/etc/nginx/conf.d \
--link=mysql:mysql \
-d nginx-php-fpm:latest
```
## 服务重载
```
docker exec -it nginx-php-fpm nginx -t -c /etc/nginx/nginx.conf
docker exec -it nginx-php-fpm nginx -s reload
```
## 安装PHP扩展
```
docker-php-ext-install sockets
```
## 推荐的NGINX配置
/root/docker/nginx/conf/nginx.conf
```
#user  nobody;
worker_processes auto;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        run/nginx.pid;


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
    keepalive_timeout 2;
        client_max_body_size 100m;

    server_tokens off;
    #gzip  on;

    include /etc/nginx/sites-enabled/*;
    include /etc/nginx/conf.d/*.conf;
}
#daemon off;
```
/root/docker/nginx/conf/conf.d/test.conf
```
server {
    listen      80;
    server_name your-server-name;
    root        /var/www/html/test/public;
    index       index.php index.html index.htm index.phtml;
    charset     utf-8;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        fastcgi_index index.php;
        include fastcgi_params;
    }

    location / {
        if (!-e $request_filename) {
        	rewrite ^/index.php(.*)$ /index.php?s=$1 last; 
        	rewrite ^(.*)$ /index.php?s=$1 last; 
        	break;
        }
    }

    location ~* ^/(css|img|js|flv|swf|download)/(.+)$ {
        root /var/www/html/test/public;
    }
    location ~* \.(eot|ttf|woff|svg|otf)$ {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Headers X-Requested-With;
        add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
    }
    location ~ /\.ht {
        deny all; 
    }
}
```
