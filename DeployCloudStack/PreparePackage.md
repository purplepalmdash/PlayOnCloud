## 准备包
两个用于部署的仓库，其repo配置文件如下:     

```
# cat mrepo7.repo 
[0-base]
name=CentOS-local-base
baseurl=http://192.168.0.79/mrepo/centos7-x86_64/RPMS.os/
gpgcheck=0
 
[0-updates]
name=CentOS-local-updates
baseurl=http://192.168.0.79/mrepo/centos7-x86_64/RPMS.updates/
gpgcheck=0

[0-contrib]
name=CentOS-local-contrib
baseurl=http://192.168.0.79/mrepo/centos7-x86_64/RPMS.contrib/
gpgcheck=0

[0-extras]
name=CentOS-local-extras
baseurl=http://192.168.0.79/mrepo/centos7-x86_64/RPMS.extras/
gpgcheck=0

[0-fasttrack]
name=CentOS-local-fasttrack
baseurl=http://192.168.0.79/mrepo/centos7-x86_64/RPMS.fasttrack/
gpgcheck=0

[0-centosplus]
name=CentOS-local-centosplus
baseurl=http://192.168.0.79/mrepo/centos7-x86_64/RPMS.centosplus/
gpgcheck=0

[0-repo]
name=epel-7
baseurl=http://192.168.0.79/epelRepo/7/epel/
gpgcheck=0
```


```
# cat cloudstack7.repo 
[cloudstack]
name=cloudstack
baseurl=http://192.168.0.79/4.5CentOS7
enabled=1
gpgcheck=0
```
