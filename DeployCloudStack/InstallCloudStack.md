## 安装CloudStack

基于Packer编译出来的镜像文件我们可以在其上构建出CloudStack运行环境，以下是详细步骤。    

### 节点/网络拓扑
节点:   
* Management节点:  192.168.139.2, CentOS6, 2-Core/3G Mem. 
* Agent节点: 192.168.139.3, CentOS7, 4-Core/8G Mem.
* Gateway: 192.168.139.79    

### Gateway
Gateway本身是192.168.1.0/24网段的机器，需要添加一个192.168.139.79的地址:     

```
# vim /etc/sysconfig/network-scripts/ifcfg-cloudbr0 
    IPADDR=192.168.1.79
    NETMASK=255.255.255.0
    IPADDR1=192.168.0.79
    + IPADDR2=192.168.139.79
    NETMASK1=255.255.255.0
```

同样在Gateway端，需要添加到主机的路由，以使得外部机器都能顺利访问到192.168.139.2和
192.168.139.3两台机器。   

添加iptables:    

```
# iptables -A FORWARD -s 192.168.139.2 -j ACCEPT
# iptables -A FORWARD -s 192.168.139.3 -j ACCEPT
# iptables -t nat -A POSTROUTING -s 192.168.139.2 -j SNAT --to-source \
192.168.1.79
# iptables -t nat -A POSTROUTING -s 192.168.139.3 -j SNAT --to-source \
192.168.1.79
``` 

将添加的iptables规则加入到开机启动文件中:     

```
# iptables-save>1.txt
# cp 1.txt /etc/sysconfig/iptables
```

### Management节点网络
Mangement节点的网络需要桥接到主机节点，如下图, 桥接到了主机的ovsbr0接口:     

![/images/2015_09_23_14_24_47_539x217.jpg](/images/2015_09_23_14_24_47_539x217.jpg)    

配置网络:     

```
# vim /etc/sysconfig/network-scripts/ifcfg-eth0 
    DEVICE="eth0"
    BOOTPROTO="static"
    IPV6INIT="yes"
    NM_CONTROLLED="yes"
    ONBOOT="yes"
    TYPE="Ethernet"
    UUID="a5de6da4-9292-498e-9efd-2139d111dcb6"
    IPADDR=192.168.139.2
    NETMASK=255.255.255.0
    DNS1=223.5.5.5
    GATEWAY=192.168.139.79
```

配置Hostname:     

```
# cat /etc/hosts
192.168.139.2           csmgmt
127.0.0.1               localhost
::1     localhost       ip6-localhost   ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
# cat /etc/hostname 
csmgmt
```

检查hostname：    

```
[root@csmgmt ~]# hostname
csmgmt
[root@csmgmt ~]# hostname --fqdn
csmgmt
```

### Managment节点安装
#### 0. 加载本地仓库:    

```
# cd /etc/yum.respo.d/
# mkdir ./back
# mv *.repo .back/
# wget http://192.168.0.79/cloudstack6.repo
# wget http://192.168.0.79/mrepo6_64.repo
# yum clean all && yum makecache
```

#### 1. 安装和配置 NTP:    

```
# yum install -y ntp
# vim /etc/ntp.conf
    driftfile /var/lib/ntp/drift
    
    restrict default kod nomodify notrap nopeer noquery
    restrict -6 default kod nomodify notrap nopeer noquery
    
    restrict 127.0.0.1 
    restrict -6 ::1
    
    server 0.uk.pool.ntp.org iburst
    server 1.uk.pool.ntp.org iburst
    server 2.uk.pool.ntp.org iburst
    server 3.uk.pool.ntp.org iburst
    
    includefile /etc/ntp/crypto/pw
    
    keys /etc/ntp/keys
    
    disable monitor
# service ntpd restart
# chkconfig ntpd on
``` 

#### 2. SElinux的相关配置:    
安装 libselinux-python:  

```
# yum install -y libselinux-python
```

配置 SELinux:   

```
# vim /etc/selinux/config
SELINUX=disabled
SELINUXTYPE=targeted
```
更改玩SElinux后，最好重启机器以使得规则生效。   
After configurating SELinux, you'd better restart machine to let policy take effects.   

#### 3. MySQL
安装 MySQL:   

```
# yum install -y mysql-server
```

安装 MySQL python模块:    

```
# yum install -y MySQL-python
```

更改MYSQL的my.cnf文件，在'[mysqld_safe]'前添加以下条目:    

```
# vim /etc/my.cnf
    + # CloudStack MySQL settings
    + innodb_rollback_on_timeout=1
    + innodb_lock_wait_timeout=600
    + max_connections=700
    + log-bin=mysql-bin
    + binlog-format = 'ROW'
    + bind-address=0.0.0.0
    
    [mysqld_safe]
``` 

启动服务，并保存到开机启动:    

```
# service mysqld start
# chkconfig mysqld on
```

移除 anonymous 用户:    

```
# mysql
mysql>  SELECT User, Host, Password FROM mysql.user;
+------+-----------+----------+
| User | Host      | Password |
+------+-----------+----------+
| root | localhost |          |
| root | csmgmt    |          |
| root | 127.0.0.1 |          |
|      | localhost |          |
|      | csmgmt    |          |
+------+-----------+----------+
mysql> DROP USER ''@'csmgmt'; 
mysql> DROP USER ''@'localhost'; 
mysql>  SELECT User, Host, Password FROM mysql.user;
+------+-----------+----------+
| User | Host      | Password |
+------+-----------+----------+
| root | localhost |          |
| root | csmgmt    |          |
| root | 127.0.0.1 |          |
+------+-----------+----------+
3 rows in set (0.00 sec)
```
Remove the testdb:   

```
mysql> select * from mysql.db;
........
mysql> DELETE FROM mysql.db WHERE Db LIKE 'test%';
Query OK, 2 rows affected (0.00 sec)

mysql> select * from mysql.db;
Empty set (0.00 sec)
mysql> DROP DATABASE test;
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
mysql> \q
Bye
```

加密MYSQL安装并更改root用户密码，可以通过以下命令完成:     

```
# mysql_secure_installation
```
开启 iptables:    

```
# iptables -A INPUT -p tcp -m tcp --dport 3306 -j ACCEPT
# vim /etc/sysconfig/iptables
+       -A INPUT -p tcp -m tcp --dport 3306 -j ACCEPT
        -A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
# service ntpd restart
```

#### 4. 安装 CloudStack-Management
安装 cloudstack management packages via:   

```
# yum install -y cloudstack-management
```

#### 5. Install Cloud-Monkey    
Cloud-Monkey是用来方便的配置CloudStack的组件，用下列命令安装:    

```
# yum install -y python-pip
# pip install cloudmonkey
```

#### 6. 配置CloudStack数据库

```
# cloudstack-setup-databases cloud:engine@localhost \ 
--deploy-as=root:engine123 -i 192.168.139.2>>/root/cs_dbinstall.out 2>&1
```

#### 7.  配置 Management 服务器

```
# cloudstack-setup-management >> /root/cs_mgmtinstall.out 2>&1
```

到现在你可以通过浏览器访问Management节点:    

```
# firefox http://192.168.139.2:8080/client/
```

### Agent网络配置
Agent节点的网络需要配置Cloudbr0桥接，以下是详细的配置。     

配置主机名:   

```
# vim /etc/hostname 
csagent
# vim /etc/hosts
192.168.139.3           csagent
127.0.0.1               localhost
::1     localhost       ip6-localhost   ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

检查hostname配置:    

```
# hostname
csagent
# hostname --fqdn
csagent
```

配置桥接和固定IP地址:    

```
# cat /etc/sysconfig/network-scripts/ifcfg-cloudbr0 
DEVICE=cloudbr0
TYPE=Bridge
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.139.3
NETMASK=255.255.255.0
DNS1=223.5.5.5
GATEWAY=192.168.139.79
# cat /etc/sysconfig/network-scripts/ifcfg-eth0 
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=none
BRIDGE=cloudbr0
```

配置完毕后重启网络:    

```
# systemctl restart network.service
```

### Agent节点安装
#### 0. 仓库配置

```
# cd /etc/yum.repos.d/
# mkdir ./back
# mv *.repo ./back
# wget http://192.168.0.79/mrepo7.repo
# wget http://192.168.0.79/cloudstack7.repo
# yum clean all && yum makecache
```

#### 1. 安装包
安装CloudStack Agent包比较简单:     

```
# yum install -y cloud-agent
```

#### 2. 配置Agent
需要更改掉以下配置中的选项后重启libvirtd:    

```
# sed -i 's/#vnc_listen = "0.0.0.0"/vnc_listen = "0.0.0.0"/g' \ 
 /etc/libvirt/qemu.conf && sed -i 's/cgroup_ \ 
 controllers=["cpu"]/#cgroup_controllers=["cpu"]/g' /etc/libvirt/qemu.conf
# sed -i 's/#listen_tls = 0/listen_tls = 0/g' \ 
  /etc/libvirt/libvirtd.conf && sed -i 's/#listen_tcp = 1/listen_tcp = 1/g' \
  /etc/libvirt/libvirtd.conf && sed -i \ 
 's/#tcp_port = "16509"/tcp_port = "16509"/g' \
 /etc/libvirt/libvirtd.conf && sed -i 's/#auth_tcp = "sasl"/auth_\
 tcp = "none"/g' /etc/libvirt/libvirtd.conf && \
 sed -i 's/#mdns_adv = 1/mdns_adv = 0/g' /etc/libvirt/libvirtd.conf
# sed -i 's/#LIBVIRTD_ARGS="--listen"/LIBVIRTD_ARGS="--listen"/g' \
 /etc/sysconfig/libvirtd 
# sed -i '/cgroup_controllers/d' \
  /usr/lib64/python2.7/site-packages/cloudutils/serviceConfig.py
```

重启libvirtd:    

```
# service libvirtd restart
```

