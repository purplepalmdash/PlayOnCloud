## Ceph

### 搭建过程
创建两台虚拟机， 分别为ceph-storage和ceph-admin, 每台虚拟机的配置为1G 内存，一核CPU，
100G系统盘， 100G数据盘， 单网卡， 安装系统为CentOS7.    

同步时间:     

```
# yum install -y ntpdate
# ntpdate time.windows.com
```

配置主机名:     

```
# vim /etc/hosts
192.168.177.102         ceph-admin
192.168.177.103         ceph-storage
127.0.0.1               localhost
::1     localhost       ip6-localhost   ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
(ceph-admin) # vim /etc/hostname
ceph-admin
(ceph-storage) # vim /etc/hostname
ceph-storage
```

确保selinux被关闭，确保firewalld.service被关闭。    

```
# systemctl disable firewalld.service
# systemctl stop firewalld.service
```

允许sudo:    

```
# echo "ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph
# sudo chmod 0440 /etc/sudoers.d/ceph
```

ceph-admin节点上允许无密码登录:     

```
# ssh-keygen 
# ssh-copy-id ceph@ceph-storage
# vim  ~/.ssh/config
Host ceph-storage
Hostname ceph-storage
User ceph
```
在ceph-storage节点上，确保requiretty被禁用:    

```
# visudo 
- Defaults    requiretty
+ Defaults:ceph-storage !requiretty
```

调大PID值:    

```
# vim /etc/sysctl.conf
kernel.pid_max=4194303
# sysctl -p 
```

安装ceph库:    

```
# rpm -Uhv http://ceph.com/rpm-giant/el7/noarch/ceph-release-1-0.el7.noarch.rpm
# yum update -y && yum install ceph-deploy -y
```

创建临时目录，用于存放ceph配置中的临时文件:     

```
# mkdir ~/ceph-cluster
# cd ~/ceph-cluster/
# ceph-deploy new ceph-storage
```

添加ceph.conf后的两行:    

```
$ vim ceph.conf
osd pool default size = 1
public network = 192.168.177.0/24
```

配置:    

```
# ceph-deploy new ceph-storage
# ceph-deploy install ceph-admin ceph-storage
# ceph-deploy mon create-initial
# ceph-deploy gatherkeys ceph-storage
# ceph-deploy disk zap ceph-storage:vdb
# ceph-deploy osd prepare ceph-storage:vdb:/dev/vdb
or 
# ceph-deploy osd prepare ceph-storage:vdb
# ceph-deploy osd activate ceph-storage:/dev/vdb1
# ceph-deploy admin ceph-admin ceph-storage
# sudo chmod +r /etc/ceph/ceph.client.admin.keyring
# ceph-deploy mds create ceph-storage
# ceph osd pool create poolname 128
# ceph osd pool create cephfs_data 128
# ceph osd pool create cephfs_metadata 128
# ceph fs new cephfs cephfs_metadata cephfs_data
# ceph mds stat
# ceph osd tree
```

客户端mount命令:     

```
$ sudo mount -t ceph 192.168.177.103:6789:/ /mnt -o  \ 
 name=admin,secret=AQDWvF9W6A/xFhAAMYrTIXwlaBoIYmbOBqC8bw==
```


