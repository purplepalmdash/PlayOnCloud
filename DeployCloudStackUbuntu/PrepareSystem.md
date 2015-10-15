## 准备系统
我们使用Ubuntu 14.04来部署CloudStack。 以下是系统准备过程。    

### 网络配置
首先把待配置的Ubuntu虚拟机加入到网络里:    

![/images/2015_10_15_10_42_52_602x294.jpg](/images/2015_10_15_10_42_52_602x294.jpg)    

配置IP地址为`10.168.100.10`:   

```
$ sudo vim /etc/network/interfaces
.....
# The primary network interface
auto eth0
iface eth0 inet static
address 10.168.100.10
netmask 255.255.255.0
gateway 10.168.100.1
dns-nameservers 223.5.5.5
```

### 主机名及FQDN配置
步骤如下:    

```
# vim /etc/hostname
ubuntucloudstack
# vim /etc/hosts
....
- 127.0.1.1.	xxxxxx
+ 10.168.100.10	ubuntucloudstack
```

重启后检验:    

```
adminubuntu@ubuntucloudstack:~$ hostname
ubuntucloudstack
adminubuntu@ubuntucloudstack:~$ hostname --fqdn
ubuntucloudstack
```

### 允许ROOT登录
首先sudo 到root用户更改其密码.    

而后更改sshd的配置文件如下:    

```
root@ubuntucloudstack:~# vim /etc/ssh/sshd_config 
+ PermitRootLogin yes
root@ubuntucloudstack:~# service ssh restart
```
现在开始你可以用root用户名登录.    

### 配置桥接网络
安装桥接包:   

```
# apt-get install -y bridge-utils
```
配置网络:   

```
# vim /etc/nework/intefaces
	# The primary network interface
	auto eth0
	iface eth0 inet manual
	
	auto cloudbr0
	iface cloudbr0 inet static
	address 10.168.100.10
	netmask 255.255.255.0
	gateway 10.168.100.1
	dns-nameservers 223.5.5.5
	bridge_ports eth0
	bridge_fd 5
	bridge_stp off
	bridge_maxwait 1
```

重启网络后可以看到cloudbr0被激活。   

### 配置仓库

```
root@ubuntucloudstack:~# vim /etc/apt/sources.list
	### Add cloudstack local repository
	deb http://192.168.0.79/    cloudstackdeb/

root@ubuntucloudstack:~# apt-cache search cloudstack
cloudstack-agent - CloudStack agent
cloudstack-awsapi - CloudStack Amazon EC2 API
cloudstack-cli - The CloudStack CLI called CloudMonkey
cloudstack-common - A common package which contains files which are shared by several CloudStack packages
cloudstack-docs - The CloudStack documentation
cloudstack-management - CloudStack server library
cloudstack-usage - CloudStack usage monitor
```
现在一切准备就绪，接下来将开始安装Cloudstack
