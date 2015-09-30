## 编译CloudStack DEB
这里我们主要记载步骤，我是用一个Ubuntu的容器开始编译的，所以附加了有关创建容器的内容在
其中。    

容器的创建:    

```
$ sudo docker pull ubuntu
$ sudo docker run -it ubuntu /bin/bash
root@a1e2fc3f3abe:# 
```

在容器里依次执行以下操作:    

```
# apt-get update
# apt-get install -y vim
# apt-get install -y  python-software-properties 
# apt-get install -y  ant debhelper openjdk-7-jdk tomcat6 libws-commons-util-java \
 genisoimage python-mysqldb libcommons-codec-java libcommons-httpclient-java \
 liblog4j1.2-java maven -y
# mkdir -p ~/Code
# cd ~/Code/
# wget http://xxxxxxxxx/apache-cloudstack-4.5.2-src.tar.bz2 
# tar xjvf apache-cloudstack-4.5.2-src.tar.bz2 
# cd apache-cloudstack-4.5.2-src
# mvn -P deps -Dnonoss -DskipTests=true
# dpkg-buildpackage -uc -us
```

编译完毕后可以在上层找到生成的DEB包：    

```
# ls ../
apache-cloudstack-4.5.2-src          cloudstack-awsapi_4.5.2_all.deb
cloudstack-docs_4.5.2_all.deb        cloudstack_4.5.2.dsc
apache-cloudstack-4.5.2-src.tar.bz2  cloudstack-cli_4.5.2_all.deb
cloudstack-management_4.5.2_all.deb  cloudstack_4.5.2.tar.gz
cloudstack-agent_4.5.2_all.deb       cloudstack-common_4.5.2_all.deb
cloudstack-usage_4.5.2_all.deb       cloudstack_4.5.2_amd64.changes
# du -hs ~/Code/
3.2G    /root/Code/

```

其他版本的CloudStack的生成过程类似。    
