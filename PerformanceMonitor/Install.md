### 安装步骤
主要参考了
[http://www.unixmen.com/install-graphite-centos-7/](http://www.unixmen.com/install-graphite-centos-7/)     

````
# yum -y update
# yum install -y httpd net-snmp perl pycairo mod_wsgi python-devel git gcc-c++
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# yum install -y python-pip node npm
# pip install 'django<1.6'
# pip install 'Twisted<12'
# pip install django-tagging==0.3.6
# pip install whisper
# pip install graphite-web
# pip install carbon
# pip install fields
# yum install collectd collectd-snmp
# git clone https://github.com/etsy/statsd.git /usr/local/src/statsd/
# cp /opt/graphite/examples/example-graphite-vhost.conf /etc/httpd/conf.d/graphite.conf
# cp /opt/graphite/conf/storage-schemas.conf.example /opt/graphite/conf/storage-schemas.conf
# cp /opt/graphite/conf/storage-aggregation.conf.example /opt/graphite/conf/storage-aggregation.conf
# cp /opt/graphite/conf/graphite.wsgi.example /opt/graphite/conf/graphite.wsgi
# cp /opt/graphite/conf/graphTemplates.conf.example /opt/graphite/conf/graphTemplates.conf
# cp /opt/graphite/conf/carbon.conf.example /opt/graphite/conf/carbon.conf
# chown -R apache:apache /opt/graphite/storage/
# vi /opt/graphite/conf/storage-schemas.conf
    [default]
    pattern = .*
    retentions = 10s:4h, 1m:3d, 5m:8d, 15m:32d, 1h:1y
# cd /opt/graphite/webapp/graphite
# sudo python manage.py syncdb
# systemctl enable httpd
# systemctl start httpd
# /opt/graphite/bin/carbon-cache.py start
# /opt/graphite/bin/run-graphite-devel-server.py /opt/graphite/
```

这时候打开Graphite，是什么都看不到的，因为没有往里面喂数据。    

### 注入数据
使用statsd注入数据，注入的数据会被在graphite被显示出来。    

前面我们已经clone了statsd的代码到本地，直接使用它:   

```
$ cd /usr/local/src/statsd
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
$ cnpm install nodeunit
$ cnpm install temp
$ cnpm install underscore
$ vim exampleConfig.js
    {
      graphitePort: 2003
    , graphiteHost: "192.168.10.191"
    , port: 8125
    , backends: [ "./backends/graphite" ]
    }
$ node stats.js ./exampleConfig.js 
```

statsd将监听8125端口.     

```
# cnpm install statsd-client --save
# mkdir ~/Code/statsD
# vim app.js
    var sdc_init = require('statsd-client')  
    var sdc = new sdc_init({ host: 'localhost' });  
    sdc.increment('my.test.1', 50);  
    sdc.close(); 
# node ./app.js
# vim /opt/graphite/conf/storage-schemas.conf
    [default_1min_for_1day]
    pattern = .*
    retentions = 60s:1d
    
    [statsD]  
    pattern = ^stats\.  
    retentions = 10s:1d,30s:7d,1m:21d,15m:5y  
      
    [statsD-2]  
    pattern = ^stats_counts\.  
    retentions = 10s:1d,30s:7d,1m:21d,15m:5y 
# cd /opt/graphite/bin
# python carbon-cache.py stop 
# python carbon-cache.py start  
```

现在重启可以看到结果:    

![/images/2015_10_21_11_03_10_602x399.jpg](/images/2015_10_21_11_03_10_602x399.jpg)    

