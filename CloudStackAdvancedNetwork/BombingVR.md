## 轰炸VR
### 环境准备
网络拓扑如下:   

![/images/CloudStack.jpg](/images/CloudStack.jpg)   

*Test Machine*:    
**硬件**: I7 2620M 4核8线程， 12G内存.   
**操作系统**: ArchLinux.    
**配置**: 10+ KVM虚拟机，桥接模式，地址范围分别为192.168.1.2 ~ 192.168.1.20.      
**物理机IP**: 192.168.1.11      

*CloudStack Agent*:    
**硬件**: i5-4460 4核4线程，32G 内存.    
**操作系统**: Ubuntu14.04.     
**配置**: CloudStack All-In-One, 版本4.5.2.     
**物理机IP**: 192.168.1.13     

*Clients/Server*:     
**操作系统**: Ubuntu14.04.     
**Server**: KVM虚拟机,双核/4G内存, 通过VR提供NAT获得公网地址.   
**Clients**: KVM虚拟机，单核/768M内存.   
**IP地址**: 均使用192.168.1.1/24段地址.     

*网络*:    
**设备**: OpenWRT路由器，1000M局域网连接.    
**DHCP范围**: 192.168.1.2 ~ 192.168.1.20.   

### 测试原理


### 突破限制
系统默认对

#### 文件句柄限制
Linux默认的单个进程打开的文件限制为1024, 可以通过`ulimit -n`来查看, 永久改变需要更改以
下文件, 在文件最后添加两行:   

```
# vim /etc/security/limits.conf
* soft nofile 1048576
* hard nofile 1048576
```
手动调节:   

```
$ ulimit -HSn 1048576
```
#### 最大文件数限制
当连接条目超过500000的时候有可能出现`[warn] socket: Too many open files in
system`, 我们将修改以下变量, 来确保`  /proc/sys/fs/file-max` 的值突破这个限制:    

```
$ sudo vim /etc/sysctl.conf
.....
fs.file-max=1048576
$ sudo sysctl -p
```

#### 端口限制
Linux系统默认的端口配置为:    

```
# cat /proc/sys/net/ipv4/ip_local_port_range
32768 61000
```
这导致在单机上我们的程序可以使用的地址范围只有三万多个，更改如下，获得六万多个端口范围，从而可
以对外发起六万个连接。

```
# echo "1024 65535"> /proc/sys/net/ipv4/ip_local_port_range
```

永久更改方法:    

````
# vim /etc/sysctl.conf
    net.ipv4.ip_local_port_range= 1024 65535
# sysctl -p
```
#### nf_conntrack限制
Netfilter模块对系统能track的connection做了限制，默认为65535, 查看如下:   

```
$ sudo sysctl -a | grep -i conntrack_max
net.netfilter.nf_conntrack_max = 65536
net.nf_conntrack_max = 65536
```

Ubuntu 14.04/ArchLinux下更改如下:    

```
# vim /etc/sysctl.d/99-sysctl.conf
net.netfilter.nf_conntrack_max = 1048576
# sysctl --system
```

或者(对Ubuntu14.04生效):    

```
# vim /etc/sysctl.conf
net.netfilter.nf_conntrack_max = 6553500
# sysctl -p /etc/sysctl.conf
# sysctl -a | grep -i conntrack_max
net.netfilter.nf_conntrack_max = 6553500
net.nf_conntrack_max = 6553500
```

### CloudStack Agent机器配置


### Test Machine配置

#### 单机多网卡
由下面的命令启动qemu实例,则可以得到8块网卡的虚拟机,每个上面6万多个连接,可以超
过50万.    

```
$ sudo qemu-system-x86_64 -net nic,model=virtio,macaddr=52:54:00:12:34:56,vlan=1
-net tap,vlan=1 -net nic,model=virtio,macaddr=52:54:00:12:34:57,vlan=2 -net
tap,vlan=2 -net nic,model=virtio,macaddr=52:54:00:12:34:58,vlan=3 -net
tap,vlan=3 -net nic,model=virtio,macaddr=52:54:00:12:34:59,vlan=4 -net
tap,vlan=4 -net nic,model=virtio,macaddr=52:54:00:12:34:60,vlan=5 -net
tap,vlan=5 -net nic,model=virtio,macaddr=52:54:00:12:34:61,vlan=6 -net
tap,vlan=6 -net nic,model=virtio,macaddr=52:54:00:12:34:62,vlan=7 -net
tap,vlan=7 -net nic,model=virtio,macaddr=52:54:00:12:34:63,vlan=8 -net
tap,vlan=8 -hda ./ubuntu64perftest.qcow2 -m 5120 --enable-kvm
```

启动虚拟机后,设置地址:    

```
# cat ./ethernet.sh
ifconfig eth1 up
ifconfig eth1 192.168.1.254
ifconfig eth2 up
ifconfig eth2 192.168.1.253
ifconfig eth3 up
ifconfig eth3 192.168.1.252
ifconfig eth4 up
ifconfig eth4 192.168.1.251
ifconfig eth5 up
ifconfig eth5 192.168.1.250
ifconfig eth6 up
ifconfig eth6 192.168.1.249
ifconfig eth7 up
ifconfig eth7 192.168.1.248
#@ifconfig eth8 up
#@ifconfig eth8 192.168.1.247
```
调用客户端大量连接的脚本:    

```
 cat startbomb.sh 
#!/bin/sh
./client2 -h 192.168.1.109 -p 8000 -m 64000 -o  192.168.1.16,192.168.1.254,192.168.1.253,192.168.1.252,192.168.1.251,192.168.1.250,192.168.1.249,192.168.1.248
```

`client2.c`下载地址为    

[https://gist.github.com/yongboy/5324779/raw/f29c964fcd67fefc3ce66e487a44298ced611cdc/client2.c](https://gist.github.com/yongboy/5324779/raw/f29c964fcd67fefc3ce66e487a44298ced611cdc/client2.c)   

`client2.c`中需要修改的行:   

```
static char ip_array[300] =
"192.168.1.16,192.168.1.254,192.168.1.253,192.168.1.252,192.168.1.251,192.168.1.250,192.168.1.249,192.168.1.248";
//"192.168.190.134,192.168.190.143,192.168.190.144,192.168.190.145,192.168.190.146,192.168.190.147,192.168.190.148,192.168.190.149,192.168.190.151,192.168.190.152";
static char server_ip[16] =  "192.168.1.109";

```

### 轰炸开始
在CloudStack的实例上运行server端程序, 此时的输出如下:    

VR的`top`输出:    

![/images/2015_11_02_21_11_06_810x293.jpg](/images/2015_11_02_21_11_06_810x293.jpg)    

Server输出:   

![/images/2015_11_02_21_13_37_788x205.jpg](/images/2015_11_02_21_13_37_788x205.jpg)   

客户端开始发包连接的过程中:    

10W:   

![/images/2015_11_02_21_15_11_794x589.jpg](/images/2015_11_02_21_15_11_794x589.jpg)    

20W:   

![/images/2015_11_02_21_15_53_841x563.jpg](/images/2015_11_02_21_15_53_841x563.jpg)   

30W:   

![/images/2015_11_02_21_16_35_902x622.jpg](/images/2015_11_02_21_16_35_902x622.jpg)   

40W:  

![/images/2015_11_02_21_17_19_869x654.jpg](/images/2015_11_02_21_17_19_869x654.jpg)   

44W, VR轰挂:   

![/images/2015_11_02_21_18_06_813x689.jpg](/images/2015_11_02_21_18_06_813x689.jpg)   

此时查看VR的重启时间:    

![/images/2015_11_02_21_18_56_665x149.jpg](/images/2015_11_02_21_18_56_665x149.jpg)   

 
