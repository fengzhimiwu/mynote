```
yum -y install make zlib zlib-devel gcc gcc-c++ libtool  openssl openssl-devel automake

wget http://nginx.org/download/nginx-1.18.0.tar.gz
tar -zxvf nginx-1.18.0.tar.gz
cd nginx-1.18.0
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
make && make install
```

