# 推荐目录结构
  /root/docker/mysql
  /root/docker/nginx
  /root/docker/scripts
  /root/docker/svn
  /root/docker/www
## 容器自启动脚本
  /root/docker/scripts/startup.sh
```
#!/bin/bash
# start docker container
sudo systemctl start docker
docker start $(docker ps -aq -f status=exited)
```
  
# 拉取 SVN 镜像
	docker pull garethflowers/svn-server
	docker run \
	    --name svn \
	    --detach \
	    --volume /root/docker/svn:/var/opt/svn \
	    --volume /root/docker/www:/www \
	    --publish 3690:3690 \
	    garethflowers/svn-server
## 创建仓库
    docker exec -it svn svnadmin create test
## 设置钩子
    /usr/bin/svn update --username your-name --password your-password /www/test/
## 初始化部署目录
    docker exec -it svn svn co svn://your-ip/test /www/test --username your-name --password your-password

# 拉取 MySQL 镜像
	docker pull mysql:5.6
	docker run -itd --name mysql -p 3306:3306 -v /root/docker/mysql:/etc/mysql/sqlinit -e MYSQL_ROOT_PASSWORD=your-password mysql:5.6

# 拉取 NGINX-PHP-FPM 仓库
  git clone https://github.com/toctrash/nginx-php-fpm.git
  docker build -t nginx-php-fpm .
  docker run --name nginx-php-fpm -p 80:80 \
	-v /root/docker/www:/var/www/html \
	-v /root/docker/nginx/conf/conf.d:/etc/nginx/conf.d \
	--link=mysql:mysql \
	-d nginx-php-fpm:latest
## 服务重载
  docker exec -it nginx-php-fpm nginx -t -c /etc/nginx/nginx.conf
  docker exec -it nginx-php-fpm nginx -s reload
## 安装PHP扩展
  docker-php-ext-install sockets
## 推荐的NGINX配置
