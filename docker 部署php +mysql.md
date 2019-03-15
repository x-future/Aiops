docker 部署php +mysql

```
yum remove docker  docker-common docker-selinux docker-engine
```

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

```
yum list docker-ce --showduplicates | sort -r
```

```
yum install docker-ce
```

```
systemctl start docker
```

```
systemctl enable docker
```

```
docker version
```

## 配置镜像加速器

阿里docker镜像加速器

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://r6ao1ipx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

安装docker-compose

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

```
chmod +x /usr/local/bin/docker-compose
```

安装php7.

rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm   

rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

yum -y install php72w php72w-cli php72w-common php72w-devel php72w-embedded php72w-fpm php72w-gd php72w-mbstring php72w-mysqlnd php72w-opcache php72w-pdo php72w-xml



```
rpm -Uvhhttp://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```

```
yum install mysql-community-server
```

```
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

```s
yum -y install nginx
```

```
yum -y install redis
```

部署lnmp环境

lnmp
├── conf
│   ├── mysql
│   ├── nginx
│   │   ├── conf.d
│   │   │   ├── default.conf
│   │   │   └── default.conf.new
│   │   ├── nginx.conf
│   │   └── nginx.conf.0315
│   └── php
│       ├── php-fpm.conf
│       ├── php-fpm.conf.new
│       ├── php.ini
│       └── php.ini.new
├── docker-compose.yml
├── docker-compose.yml.bak
├── log
│   ├── nginx
│   │   ├── access.log
│   │   └── error.log
│   └── php
├── php
│   └── php56
│       └── Dockerfile
└── www
    ├── test.html
    └── test.php

cd php/php56

[root@docker-01 php56]# cat Dockerfile 

```shell
FROM php:5.6.30-fpm
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
        libmemcached-dev \
        zlib1g-dev \
        libcurl4-openssl-dev \
        libxml2-dev \
        --no-install-recommends && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-install -j$(nproc) \
        iconv mcrypt gettext curl mysqli pdo pdo_mysql zip \
        mbstring bcmath opcache xml simplexml sockets hash soap \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd
RUN pecl install redis-3.1.0 \
    && pecl install memcached-2.2.0 \
    && pecl install xdebug-2.5.0 \
    && docker-php-ext-enable redis memcached xdebug

RUN mkdir -p /var/www/ \
  && chmod -R 755 /var/www/

CMD ["php-fpm", "-F"]
```

```shell
docker build -t project1_php .
```

cd ../..

docker-compose version 查看docker-compose 版本

cat docker-compose.yml

```yaml
[root@docker-01 lnmp]# cat docker-compose.yml
version: "3"
services:
  nginx:
    restart: always
    image: docker.io/nginx
    container_name: project1_nginx
    ports:
      - "80:80"
      - "444:443"
    links:
      - "php56"
    volumes:
      - ./conf/nginx/conf.d/:/etc/nginx/conf.d/:ro
      - ./conf/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./log/nginx/:/var/log/nginx/:rw
      - ./www/:/var/www/html/:rw
    networks:
      - net-php
  php56:
    restart: always
    image: project1_php
    container_name: project1_php
    expose:
      - "9000"
    volumes:
      - ./conf/php/php.ini:/etc/php/php.ini:ro
      - ./conf/php/php-fpm.conf:/etc/php/php-fpm.conf:ro
      - ./log/php/:/var/log/php/:rw
      - ./www/:/var/www/html/:rw
    networks:
      - net-php
 
networks:
  net-php:
```

cat conf/nginx/conf.d/default.conf

```shell
server {
        listen       80;
        server_name  localhost;
 
        charset UTF-8;
        #access_log  /var/log/nginx/host.access.log  main;
 
        root /var/www/html;
 
        location / {
            root   /var/www/html;
            index  index.html index.htm index.php;
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
        location ~ \.php$ {
            fastcgi_pass   php56:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
 
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        location ~ /\.ht {
            deny  all;
        }
    }
 
```

https://blog.csdn.net/u011781521/article/details/80464826 继续搭建mysql





