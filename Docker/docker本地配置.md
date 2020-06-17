目录结构

```
mytest  // 根目录
|-- mysql-conf           // mysql配置文件目录
|-- |-- my.cnf           // mysql配置文件
|-- mysql-data           // mysql数据库本地映射目录
|-- nginx-conf           // nginx配置文件目录
|-- |-- nginx.conf       // nginx主配置文件
|-- |-- conf.d           // nginx主配置文件加载目录
|-- |-- |-- default.conf // 默认配置文件
|-- nginx-log            // nginx log目录
|-- php-conf             // php配置文件目录
|-- www                  // 项目文件目录
docker-compose.yml
Dockerfile
```

docker-compose.yml

```
version: "3"
services: 
  nginx: 
    image: nginx
    restart: always
    depends_on: 
      - php
      - mysql
    ports: 
      - "80:80"
    volumes: 
     - "./nginx-conf/nginx/nginx.conf:/etc/nginx/nginx.conf"
     - "./nginx-conf/nginx/conf.d:/etc/nginx/conf.d"
     - "./www:/usr/share/nginx/html"
  php: 
    build: 
      context: .
    restart: always
    links:
      - mysql:mysql
    ports: 
      - "9000:9000"
    volumes: 
     - "./www:/var/www/html"
     - "./php-conf/etc:/usr/local/etc"
  mysql: 
    image: mysql
    restart: always
    ports: 
      - "3306:3306"
    volumes: 
      - "./mysql-conf/mysql/my.cnf:/etc/mysql/my.cnf"
      - "./mysql-data:/var/lib/mysql"
    environment: 
      - MYSQL_ROOT_PASSWORD=123456
```

Dockerfile

```
FROM php:7.2-fpm

RUN docker-php-ext-install mysqli \
    && docker-php-ext-install pdo_mysql

RUN pecl install -o -f redis \
&&  rm -rf /tmp/pear \
&&  docker-php-ext-enable redis
```

ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

FLUSH PRIVILEGES;



