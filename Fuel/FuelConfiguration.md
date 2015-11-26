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
