## CloudStackAllInOne
本节将建立一个AllInOne的CloudStack环境，在此基础上将引入高级网络模式。    

### 网络准备

![/images/2015_09_28_10_45_20_511x438.jpg](/images/2015_09_28_10_45_20_511x438.jpg)    

![/images/2015_09_28_10_45_40_508x410.jpg](/images/2015_09_28_10_45_40_508x410.jpg)    

### 安装步骤
配置IP地址:    

```
# yum install -y bridge-utils
# cat /etc/sysconfig/network-scripts/ifcfg-eth0 
DEVICE="eth0"
ONBOOT="yes"
BOOTPROTO=none
NM_CONTROLLED="no"
BRIDGE="br0"
TYPE=Ethernet
# cat /etc/sysconfig/network-scripts/ifcfg-br0
DEVICE="br0"
ONBOOT="yes"
NM_CONTROLLED=yes
TYPE="Bridge"
BOOTPROTO=static
IPADDR=10.168.100.2
NETMASK=255.255.255.0
GATEWAY=10.168.100.1
DNS1=223.5.5.5
DEFROUTE=yes
```

接着配置CloudStack Management服务器，类似于上面提到的。   


