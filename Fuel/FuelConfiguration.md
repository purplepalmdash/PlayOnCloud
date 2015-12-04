## Fuel配置
### Fuel本地部署库
同步本地用于部署Ubuntu的命令很简单:    

```
# fuel-createmirror
```

考虑到网络环境的烂，可以用以下的脚本来自动完成:      

```
[root@fuel ~]# cat resume.sh 
#!/bin/bash
touch createmirrorappend.txt
until fuel-createmirror 
do
        date>>createmirrorappend.txt
        echo "Yes you have to resume again===================">>createmirrorappend.txt
done
```
这个命令将一直执行，直到fuel-createmirror命令执行成功为止。    

### 网络配置
如果使用Fuel部署，则需要至少两个网卡，一个单独的用于public网络，另外一个用于
PXE/Storage/Mgmt/Neutron L3.   

![/images/2015_11_30_09_51_40_663x479.jpg](/images/2015_11_30_09_51_40_663x479.jpg)    

上图里采用了三块网卡用于承载。事实上，两个物理网卡就足够了。     

### 部署OpenStack环境
首先在Fuel后台界面里创建一个新的OpenStack环境，然后启动两台从PXE启动的物理机器。启动完
毕后，分配Fuel里的角色(Compute/Controller), 部署完毕后，就可以在Fuel后台中访问到
OpenStack环境了。    

![/images/2015_11_30_10_16_16_613x376.jpg](/images/2015_11_30_10_16_16_613x376.jpg)    
