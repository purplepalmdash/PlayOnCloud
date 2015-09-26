## 二级存储
这里我们配置NFS服务作为CloudStack中用到的二级存储, 并在其上引入系统虚拟机模板。    

### NFS服务器
在Management节点上，默认的nfs-utils已经被安装，所以我们只需要按照以下步骤配置：    

```
# vim /etc/exports 
/home/exports *(rw,async,no_root_squash,no_subtree_check)
# mkdir -p /home/exports
# chmod 777 -R /home/exports/
# chkconfig nfs on
# chkconfig rpcbind on
# service nfs restart
# service rpcbind restart
# iptables -D INPUT -j REJECT --reject-with icmp-host-prohibited
# vim /etc/sysconfig/iptables
-D INPUT -j REJECT --reject-with icmp-host-prohibited
```

### 引入系统虚拟机模板
在Management机器上做如下操作:     

```
# mount -t nfs 192.168.139.2:/home/exports/ /mnt
# /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
 -m /mnt/ -u http://192.168.0.79/systemvm64template-4.5-kvm.qcow2.bz2 -h kvm -F
```

安装完模板后我们可以到/home/exports目录下检查：    

```
[root@csmgmt ~]# ls /home/exports/
template
[root@csmgmt ~]# du -hs /home/exports/template/
290M    /home/exports/template/
```
