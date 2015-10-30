## 监控CloudStack
CentOS6 的CloudStack Management节点上:   


### 编译collectd RPM

#### 准备编译环境
安装编译RPM所需要的包:   

```
# yum groupinstall -y 'Development Tools'
# yum install -y rpmdevtools yum-utils
```

创建一个专门用来编译rpm的用户，并创建编译树:     

```
# su - makerpm
$ rpmdev-setuptree 
$ ls rpmbuild/
BUILD  RPMS  SOURCES  SPECS  SRPMS
```

首先准备源代码：   

```
$  wget https://collectd.org/files/collectd-5.5.0.tar.gz
$  tar xzvf collectd-5.5.0.tar.gz
$  cd collectd-5.5.0
```

编译过程:    

```
$ cp ~/Code/collectd-5.5.0/contrib/redhat/collectd.spec rpmbuild/SPECS/
$ cd ~/rpmbuild/SPECS
$ spectool -g -R collectd.spec
$ rpm --eval "%{_sourcedir}"
$ rpmbuild --bb collectd.spec
$ sudo yum-builddep collectd.spec
$ sudo yum install perl-Heap
$ rpmbuild --bb collectd.spec
```

编译完毕后，可以看到rpm包：    

```
$ pwd
/home/makerpm/rpmbuild/RPMS
$ ls x86_64/
collectd-5.5.0-1.el6.x86_64.rpm              collectd-hddtemp-5.5.0-1.el6.x86_64.rpm         collectd-pinba-5.5.0-1.el6.x86_64.rpm
collectd-amqp-5.5.0-1.el6.x86_64.rpm         collectd-ipmi-5.5.0-1.el6.x86_64.rpm            collectd-ping-5.5.0-1.el6.x86_64.rpm
collectd-apache-5.5.0-1.el6.x86_64.rpm       collectd-iptables-5.5.0-1.el6.x86_64.rpm        collectd-postgresql-5.5.0-1.el6.x86_64.rpm
collectd-ascent-5.5.0-1.el6.x86_64.rpm       collectd-java-5.5.0-1.el6.x86_64.rpm            collectd-python-5.5.0-1.el6.x86_64.rpm
```

#### 本地安装库的创建
注意要移除掉epel里的定义, CentOS 6 epel仓库中含有的collectd的版本太旧:   

```
# cat /etc/yum.repos.d/epel.repo 
[epel]
name=Extra Packages for Enterprise Linux 6 - $basearch
#baseurl=http://download.fedoraproject.org/pub/epel/6/$basearch
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-6&arch=$basearch
failovermethod=priority
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6

# Blacklist for collectd
exclude=collectd*
```

### CloudStack添加
首先确保`collectd-python`包被安装:    

```
yum install -y collectd-python
```

取得cloudstack-python的文件:   

```
# git clone https://github.com/exoscale/collectd-cloudstack.git
# cd /usr/lib/collectd/
# cp ~/collectd-cloudstack/*.py ./
# cd /usr/lib64/collectd/
# cp ~/collectd-cloudstack/*.py ./
```

配置文件:    

```
Hostname "192.168.139.2"
FQDNLookup true
Interval 10
Timeout 2
ReadThreads 5
LoadPlugin syslog
<Plugin syslog>
        LogLevel info
</Plugin>
<LoadPlugin python>
    Globals true
</LoadPlugin>
<Plugin python>
    # cloudstack.py is at /usr/lib/collectd/cloudstack.py
    ModulePath "/usr/lib64/collectd/"
    #ModulePath "/usr/lib/collectd/"

    Import "cloudstack"

<Module cloudstack>
  Api "http://192.168.139.2:8080/client/api"
  Auth "True"
  ApiKey
"MZdZARfnQ5pvRLbWC4ORQvo5WXWioTVfz7f2q2Lf5R1Q8QGi0HxrxzkkMi4Gg0nMrqSBBYLy-Ooy1T1CKDwLzA"
  Secret
"82uaYtqj-G4dR4GbTWDagMXyPxOkcJWlZc6UV5SObA4V7-V5a-2cRvyC_aaKxzNC6UdqXuwFC3WbSU48y11BYw"
</Module>
</Plugin>


LoadPlugin battery
#LoadPlugin cloudstack
LoadPlugin cpu
LoadPlugin df
LoadPlugin disk
LoadPlugin entropy
LoadPlugin interface
LoadPlugin irq
LoadPlugin load
LoadPlugin memory
LoadPlugin network
LoadPlugin processes
LoadPlugin rrdtool
LoadPlugin swap
LoadPlugin users
LoadPlugin write_graphite
# <Plugin network>
#    Listen "10.211.55.46" "25826"
# </Plugin>
<Plugin write_graphite>
    <Node "graphing">
        Host "192.168.10.192"
        Port "2003"
        Protocol "tcp"
        LogSendErrors true
        Prefix "collectd."
        StoreRates true
        AlwaysAppendDS false
        EscapeCharacter "_"
    </Node>
</Plugin>


<Plugin rrdtool>
    DataDir "/var/lib/collectd/rrd"
</Plugin>
```

启动并设置为开机自启动:   

```
# service collectd start
# chkconfig collectd on
```

