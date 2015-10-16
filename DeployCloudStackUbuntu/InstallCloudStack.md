## 安装CloudStack
### openntpd
用于保持本机时间同步。    

```
$ sudo apt-get install -y openntpd
```

### cloudstack-management
因为我们使用了本地的源，所以需要加上`--force-yes`选项

```
# apt-get --yes install cloudstack-management --force-yes
```

### mysql-server
安装my-sql数据库:    

```
# apt-get install -y mysql-server
```
安装时需要制定root的密码，这里我们设置为xxxxxx, 这个值在下面会被用到.    

配置数据库:     

```
root@ubuntucloudstack:~# cat /etc/mysql/conf.d/cloudstack.cnf 
[mysqld]
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=350
log-bin=mysql-bin
binlog-format = 'ROW'
root@ubuntucloudstack:~# service mysql restart
mysql stop/waiting
mysql start/running, process 5812
```
创建数据库:   

```
root@ubuntucloudstack:~# cloudstack-setup-databases \
	cloud:engine@localhost \
	--deploy-as=root:xxxxxx \
	-e file -m mymskey44 -k mydbkey00
```
参数说明:    

```
# mysql root 密码: xxxxxxx
# cloud user 密码: engine
# management_server_key: mymskey44
# database_key: mydbkey00
```


### 准备NFS存储


首先准备NFS共享存储，创建主存储和二级存储:    
```
$ sudo mkdir -p /export/primary /export/secondary
```
安装nfs:    

```
$ sudo apt-get install nfs-kernel-server
```
引出/export目录:    

```
$ cat >>/etc/exports <<EOM
/export  *(rw,async,no_root_squash,no_subtree_check)
EOM
$ exportfs -a

```
配置NFS的statd，在指定端口:    

```
$ apt-get install nfs-common 
$ cp /etc/default/nfs-common /etc/default/nfs-common.orig
$ sed -i '/NEED_STATD=/ a NEED_STATD=yes' /etc/default/nfs-common
$ sed -i '/STATDOPTS=/ a STATDOPTS="--port 662 --outgoing-port 2020"'
/etc/default/nfs-common
$ diff -du /etc/default/nfs-common.orig /etc/default/nfs-common
```
配置lockd:    

```
$ cat >> /etc/modprobe.d/lockd.conf <<EOM
options lockd nlm_udpport=32769 nlm_tcpport=32803
EOM
```
重新启动NFS并测试其导出的目录:    

```
service nfs-kernel-server restart
# test:
showmount -e 127.0.0.1
```

将NFS卷加载到本地. 

```
$ IP=10.168.100.10
$ mkdir -p /mnt/primary /mnt/secondary
$ cat >>/etc/fstab <<EOM
$IP:/export/primary   /mnt/primary    nfs
rsize=8192,wsize=8192,timeo=14,intr,vers=3,noauto  0   2
$IP:/export/secondary /mnt/secondary  nfs
rsize=8192,wsize=8192,timeo=14,intr,vers=3,noauto  0   2
EOM
$ mount /mnt/primary
$ mount /mnt/secondary
```

### 安装和配置libvirt
安装cloudstack-agent，则可以同时安装和配置libvirt:    

```
$ apt-get install cloudstack-agent
```
配置libvirtd:    

```
$ cp /etc/libvirt/libvirtd.conf /etc/libvirt/libvirtd.conf.orig

$ sed -i '/#listen_tls = 0/ a listen_tls = 0' /etc/libvirt/libvirtd.conf
$ sed -i '/#listen_tcp = 1/ a listen_tcp = 1' /etc/libvirt/libvirtd.conf
$ sed -i '/#tcp_port = "16509"/ a tcp_port = "16509"' /etc/libvirt/libvirtd.conf
$ sed -i '/#auth_tcp = "sasl"/ a auth_tcp = "none"' /etc/libvirt/libvirtd.conf
$ diff -du /etc/libvirt/libvirtd.conf.orig /etc/libvirt/libvirtd.conf
```

Patch libvirt-bin.conf:

```
$ cp /etc/default/libvirt-bin /etc/default/libvirt-bin.orig
$ sed -i -e 's/libvirtd_opts="-d"/libvirtd_opts="-d -l"/' /etc/default/libvirt-bin
$ diff -du /etc/default/libvirt-bin.orig /etc/default/libvirt-bin
$ service libvirt-bin restart
```

Patch qemu.conf以监听所有端口:     

```
$ cp /etc/libvirt/qemu.conf /etc/libvirt/qemu.conf.orig
$ sed -i '/# vnc_listen = "0.0.0.0"/ a vnc_listen = "0.0.0.0"' /etc/libvirt/qemu.conf
$ diff -du /etc/libvirt/qemu.conf.orig /etc/libvirt/qemu.conf
$ service libvirt-bin restart
```

关闭 AppArmor:

```
$ ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
$ ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
$ apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
$ apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
$ service libvirt-bin restart
```
在配置防火墙部分，打开以下端口:    

```
$ ufw allow proto tcp from any to any port 22
$ ufw allow proto tcp from any to any port 1798
$ ufw allow proto tcp from any to any port 16509
$ ufw allow proto tcp from any to any port 5900:6100
$ ufw allow proto tcp from any to any port 49152:49216
```
现在重新启动机器，看NFS是否正常:  

```
$ reboot
$ rpcinfo -u 192.168.77.10 mount
$ showmount -e 192.168.77.10
$ mount /mnt/primary
$ mount /mnt/secondary
```

安装系统虚拟机模板：    

```
$ /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
    -m /mnt/secondary -u  \
http://192.168.0.79/systemvm64template-4.5-kvm.qcow2.bz2 \ 
    -h kvm -F
```

访问`http://10.168.100.10:8080/client`, 如果发现
 404s 错误 "The requested resource is not available" :    

```
service cloudstack-management status
service cloudstack-agent status
service tomcat6 status

service cloudstack-management stop
service tomcat6 stop
service cloudstack-agent stop
ps -efl | grep java

service cloudstack-management start
service cloudstack-management status
service cloudstack-agent start
service cloudstack-agent status
```

接下来我们可以开始配置CloudStack了，配置CloudStack的过程和上一章一样。   

### 已知问题
重启机器后可能会碰到404s错误，需要调整service的启动顺序---TBD.   
