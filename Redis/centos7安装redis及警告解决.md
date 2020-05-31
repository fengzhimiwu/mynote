1.首先从官网下载redis压缩文件

  wget http://download.redis.io/releases/redis-4.0.9.tar.gz

2.解压文件

  tar -xvf redis-4.0.9.tar.gz

3.创建redis目录

  mkdir /usr/local/redis

4.移动解压后的目录到创建的目录

  mv redis-4.0.9 /usr/local/redis/

5.进入目录

  cd /usr/local/redis/redis-4.0.9/

6.编译文件

  make

7.安装完成，启动redis

  ./src/redis-server ./redis.conf

 

**启动后显示有三个警告：**

14439:M 05 May 19:30:23.090 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
14439:M 05 May 19:30:23.090 # Server initialized
14439:M 05 May 19:30:23.090 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
14439:M 05 May 19:30:23.090 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
14439:M 05 May 19:30:23.090 * DB loaded from disk: 0.000 seconds
14439:M 05 May 19:30:23.090 * Ready to accept connections

 

**第一个警告：**

  临时解决方法：（即下次启动还需要修改此值）

  echo 511 > /proc/sys/net/core/somaxconn

  永久解决方法：（即以后启动还需要修改此值）

  如果想要永久解决，打开ietc/sysctl.conf

  在里面添net.core.somaxconn= 1024 然后执行sysctl -p 就可以永久消除这个warning

  baklog参数实际控制的是已经3次握手成功的还在accept queue的大小。

  参考[linux里的backlog详解](http://blog.csdn.net/raintungli/article/details/37913765)

**第二个警告：**

  意思是：overcommit_memory参数设置为0！在内存不足的情况下，后台程序save可能失败。建议在文件 /etc/sysctl.conf 中将overcommit_memory修改为1。

  临时解决方法：echo "vm.overcommit_memory=1" > /etc/sysctl.conf

  永久解决方法：将其写入/etc/sysctl.conf文件中。vm.overcommit_memory=1

  参考：[有关linux下redis overcommit_memory的问题](http://blog.csdn.net/whycold/article/details/21388455)

**第三次警告：**

  意思是：你使用的是透明大页，可能导致redis延迟和内存使用问题。执行 echo never >   /sys/kernel/mm/transparent_hugepage/enabled 修复该问题。

  临时解决方法：

  echo never > /sys/kernel/mm/transparent_hugepage/enabled

  永久解决方法：

  将其写入/etc/rc.local文件中。