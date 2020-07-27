yum install libjpeg\* libpng\* freetype\* gd\*



error：Please reinstall the BZip2 distribution

yum install bzip2 bzip2-devel



在编译php的时候，报错：
checking for cURL 7.15.5 or greater… configure: error: cURL version 7.15.5。

yum -y install curl-devel



Linux下安装php报错：libxml2 not found. Please check your libxml2 installation

ubuntu/debian:

```bash
apt-get install libxml2-dev
```

centos/redhat:

```bash
yum install libxml2-devel
```



./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --enable-fpm --with-fpm-user=www --with-fpm-group=www --enable-mbstring --with-curl --with-gd --with-zlib --with-bz2 --enable-sockets --enable-sysvsem --enable-sysvshm --enable-pcntl --enable-mbregex --enable-exif --enable-bcmath --with-mhash --enable-zip --with-pcre-regex --with-pdo-mysql --with-mysqli --with-jpeg-dir --with-png-dir --with-openssl --with-libdir --enable-ftp --with-gettext --with-xmlrpc --enable-opcache --with-iconv --enable-mysqlnd --with-mysqli=mysqlnd --with-iconv-dir --with-kerberos --with-pdo-sqlite --with-pear --enable-libxml --enable-shmop --enable-xml --enable-opcache



