## 监测Ubuntu

### Graphite

更改主机名:    

```
root@packer-ubuntu-1404-server:~# cat /etc/hostname
monitorserver
root@packer-ubuntu-1404-server:~# cat /etc/hosts
127.0.0.1       localhost
192.168.11.192  monitorserver

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

安装Graphite:    

```
# apt-get -y update
# apt-get install graphite-web graphite-carbon
```

配置postgresql及辅助软件:    

```
#  apt-get install postgresql libpq-dev python-psycopg2
```

创建数据库:    

```
root@monitorserver:~# sudo -u postgres psql
could not change directory to "/root": Permission denied
psql (9.3.9)
Type "help" for help.

postgres=# CREATE USER graphite WITH PASSWORD 'password';
CREATE ROLE
postgres=# CREATE DATABASE graphite WITH OWNER graphite;
CREATE DATABASE
postgres=# \q
root@monitorserv
```

配置Graphite Web应用程序:    

```
$ sudo vim /etc/graphite/local_settings.py
SECRET_KEY = 'a_salty_string'
TIME_ZONE='Asia/Shanghai'
USE_REMOTE_USER_AUTHENTICATION = True
DATABASES = {
    'default': {
        'NAME': 'graphite',
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'USER': 'graphite',
        'PASSWORD': 'password',
        'HOST': '127.0.0.1',
        'PORT': ''
        }
}
```

修改完毕后，我们可以同步数据库以创建正确的网站结构:     

```
# graphite-manage syncdb
```
这里需要输入用户名和设置密码，以便下次我们登录Graphite界面用.    

更改carbon后端的启动方式:    

```
# vim /etc/default/graphite-carbon 
    # Change to true, to enable carbon-cache on boot
    CARBON_CACHE_ENABLED=true
```

打开log rotation的选项:    

```
$ sudo vim /etc/carbon/carbon.conf
ENABLE_LOGROTATION = True
```

更改storage schemas选项, 在default前添加test字段, 如下：    

```
# cat /etc/carbon/storage-schemas.conf
+ [test]
+ pattern = ^test\.
+ retentions = 10s:10m,1m:1h,10m:1d

[default_1min_for_1day]
pattern = .*
retentions = 60s:1d
```

配置storage aggregation选项:    

```
# cp /usr/share/doc/graphite-carbon/examples/storage-aggregation.conf.example \ 
/etc/carbon/storage-aggregation.conf
# service carbon-cache start
```

安装apache2服务器:    

```
# apt-get install apache2 libapache2-mod-wsgi
```
禁止000-default网站, 定义apache2-graphite条目并使能:   

```
# a2dissite 000-default
# cp /usr/share/graphite-web/apache2-graphite.conf \ 
/etc/apache2/sites-available/
# a2ensite apache2-graphite
# service apache2 reload
```

现在访问http://Your_IP， 可以看到界面如下:    

![/images/2015_10_21_11_49_15_430x259.jpg](/images/2015_10_21_11_49_15_430x259.jpg)   

用预设定的用户名/密码登录后，可以看到下面的页面：   

![/images/2015_10_21_11_52_04_337x455.jpg](/images/2015_10_21_11_52_04_337x455.jpg)     

简单测试：    

```
# echo "test.count 4 `date +%s`" | nc -q0 127.0.0.1 2003
# echo "test.count 5 `date +%s`" | nc -q0 127.0.0.1 2003
# echo "test.count 10 `date +%s`" | nc -q0 127.0.0.1 2003
```

![/images/2015_10_21_11_56_09_615x393.jpg](/images/2015_10_21_11_56_09_615x393.jpg)     

### Collected

安装:    

```
# apt-get install collectd collectd-utils
```

配置:   

```
# vim /etc/collectd/collectd.conf
Hostname "localhost"

LoadPlugin apache
LoadPlugin cpu
LoadPlugin df
LoadPlugin entropy
LoadPlugin interface
LoadPlugin load
LoadPlugin memory
LoadPlugin processes
LoadPlugin rrdtool
LoadPlugin users
LoadPlugin write_graphite

<Plugin apache>
    <Instance "Graphite">
        URL "http://domain_name_or_IP/server-status?auto"
        Server "apache"
    </Instance>
</Plugin>

<Plugin interface>
    Interface "eth0"
    IgnoreSelected false
</Plugin>

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
```

添加apache2 状态页:    

```
# vim /etc/apache2/sites-available/apache2-graphite.conf
 +   <Location "/server-status">
 +       SetHandler server-status
 +       Require all granted
 +   </Location>

    ErrorLog ${APACHE_LOG_DIR}/graphite-web_error.log
# service apache2 reload
```
现在访问` http://192.168.10.192/server-status`可以看到有关状态.   


更改：   

```
# vim /etc/carbon/storage-schemas.conf
[collectd]
pattern = ^collectd.*
retentions = 10s:1d,1m:7d,10m:1y
```

重启服务:   

```
# sudo service carbon-cache stop
# sudo service carbon-cache start
# sudo service collectd stop
# sudo service collectd start
```

现在查看collectd收集到的信息:    


### Added CentOS7 Agent
Install, configure, and enable the collectd service:   

```
# yum install -y collectd* 
# vim /etc/collectd.conf
# service collectd start
# systemctl enable collectd.service
```

### Monitor CloudStack
[https://github.com/exoscale/collectd-cloudstack](https://github.com/exoscale/collectd-cloudstack)    

### Monitor Windows
[http://ssc-serv.com/licensing.shtml](http://ssc-serv.com/licensing.shtml)    

### Customization
Add new templates for graphite.   
