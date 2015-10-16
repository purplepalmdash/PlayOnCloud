## Ubuntu主机
Ubuntu14.04上添加Ubuntu服务
### 网络添加
添加一个名为"SingleNode"的网络:    

![/images/2015_10_16_12_23_26_526x390.jpg](/images/2015_10_16_12_23_26_526x390.jpg)    

配置网络IP地址范围:    

![/images/2015_10_16_12_24_51_507x375.jpg](/images/2015_10_16_12_24_51_507x375.jpg)    

在接下来的配置中，去掉`Enable DHCP`的选项， 选择Forward选项:    

![/images/2015_10_16_12_25_45_536x402.jpg](/images/2015_10_16_12_25_45_536x402.jpg)    

### 虚拟机添加
虚拟机添加，安装Management服务。

虚拟机上主要安装cloudstack-management 服务:    

```
# apt-get install -y openntpd
# apt-get install -y mysql-server
# service mysql restart
# cloudstack-setup-databases     cloud:engine@localhost \
--deploy-as=root:xxxx     -e file -m mymskey44 -k mydbkey00
```

### 主机端
主机端开启NFS服务，目录假设在/srv/nfs4上，创建目录并更改其权限:    

```
$ sudo mkdir /srv/nfs4/primary
$ sudo mkdir /srv/nfs4/secondary
$ sudo chmod 777 -R /srv/nfs4
```

安装cloudstack-agent包:    

```
# apt-get install cloudstack-agent
```

配置libvirt:    

```
# vim /etc/libvirt/libvirtd.conf 
listen_tls = 0
listen_tcp=1
tcp_port = "16509"
auth_tcp = "none"
# vim /etc/default/libvirt-bin 
libvirtd_opts="-d -l"
# service libvirt-bin restart
```

配置qemu.conf:    

```
$ sudo vim /etc/libvirt/qemu.conf 
vnc_listen = "0.0.0.0"
$ sudo service libvirt-bin restart
```

打开规则:    

```
# ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
# ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
# apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
# apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
# service libvirt-bin restart
# ufw allow proto tcp from any to any port 22
# ufw allow proto tcp from any to any port 1798
# ufw allow proto tcp from any to any port 16509
# ufw allow proto tcp from any to any port 5900:6100
# ufw allow proto tcp from any to any port 49152:49216
```

### 安装虚拟机模板

```
# /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
    -m /mnt/secondary -u  \
http://192.168.0.79/systemvm64template-4.5-kvm.qcow2.bz2 \ 
    -h kvm -F
```

这里最好重新启动一下机器,继续下一步配置。        

### 配置
重新启动以后，进入到配置界面`http://10.168.100.2:8080/client`, 首先更改global options
里的本地存储:    

![/images/2015_10_16_16_21_00_691x281.jpg](/images/2015_10_16_16_21_00_691x281.jpg)    

重启mangement server:    

```

```
