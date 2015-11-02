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

```
# sudo qemu-system-x86_64 -net nic,model=virtio,macaddr=52:54:00:12:34:56,vlan=1 -net tap,script=/etc/ovs-ifup,downscript=/etc/ovs-ifdown,vlan=1 -net nic,model=virtio,macaddr=52:54:00:12:34:57,vlan=2 -net tap,script=/etc/ovs-ifup,downscript=/etc/ovs-ifdown,vlan=2 -hda ./ubuntu64perftest.qcow2 -m 1024 --enable-kvm
```
