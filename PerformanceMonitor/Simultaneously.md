## 并发测试
这里介绍一个并发10W~100W的解决方案.    
主要参考了:    
[http://www.blogjava.net/yongboy/archive/2015/10/13/397559.html#427723](http://www.blogjava.net/yongboy/archive/2015/10/13/397559.html#427723)   

### 服务端/客户端准备
服务端: i5 4核4线程, 32G 内存, 1000M网口. Ubuntu 15.04 x86_64.       
客户端: i7 4核8线程, 12G 内存, 1000M网口. ArchLinux x86_64.    

均打开security的limits:    

```
# vim /etc/security/limits.conf

* soft nofile 104857
* hard nofile 104857
```

而后重启动机器.   

### 服务端准备
服务器端依赖于libev, 首先编译:    

```
$ wget http://dist.schmorp.de/libev/libev-4.20.tar.gz
$ tar xzvf libev-4.20.tar.gz
$ cd libev-4.20/
$ ./configure --enable-static
$ make -j4
$ cp .libs/libev.a ./
$ mkdir include
$ cp ./libev-4.20/*.h include/
```

获取源文件:   

```
$ wget
https://gist.github.com/yongboy/5318994/raw/f0cc8650c3350df1578dd986a38f4700152cd976/client1.c
```

```
$ wget
https://gist.githubusercontent.com/yongboy/5318930/raw/ccf8dc236da30fcf4f89567d567eaf295b363d47/server.c
```

编译:    

```
$ vim server.c
- #include "../include/ev.h"
+ #include "./include/ev.h"
$ gcc -o server server.c  ./libev-4.20/libev.a -lm
$ ls server*
server  server.c
```

运行`./server`将打印出当前的内存数,并监听8000端口.    

### 服务端调优
需要注意的是conntrack的最大数需要修改:   

```
# sysctl -a | grep -i conntrack_max
net.netfilter.nf_conntrack_max = 65536
net.nf_conntrack_max = 65536
```
这意味着如果连接数超过65536时,将无法进行其他连接操作,需要手动调大其值:    

```
# vim /etc/sysctl.conf
net.netfilter.nf_conntrack_max = 6553500
# sysctl -p /etc/sysctl.conf
# sysctl -a | grep -i conntrack_max
net.netfilter.nf_conntrack_max = 6553500
net.nf_conntrack_max = 6553500
```
ulimit的限制数目:    

```
$ ulimit -HSn 1048576
```

### 客户端准备
客户端依赖于libevent, 所以先下载libevent并编译:    

```
$ wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
$ tar xzvf libevent-2.0.21-stable.tar.gz
$ cd libevent-2.0.21-stable
$ ./configure --prefix=/usr
$ make -j5
$ sudo make install
```

修改服务器地址:    

```
# vim client1.c
+ #define SERVERADDR "192.168.1.13"
```

编译:    

```
$ gcc -o client client1.c -levent
$ sudo ln -s /usr/lib/libevent-2.0.so.5 /lib64/libevent-2.0.so.5
```

客户端突破28200限制:    

```
# cat /proc/sys/net/ipv4/ip_local_port_range
32768 61000
# echo "1024 65535"> /proc/sys/net/ipv4/ip_local_port_range
```

通过sysctl写入:    

```
$ sudo vim /etc/sysctl.conf
net.ipv4.ip_local_port_range= 1024 65535
```

### 测试调优
接下来我们将在CloudStack环境里开始轰炸VR, 看到多少连接时它将挂掉.    
