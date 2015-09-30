## Build For CentOS 7

```
http://m.oschina.net/blog/290886

```

命令:    

```
$ yum update
$ yum install -y net-tools vim wget
$ yum groupinstall "Development Tools"
$ wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
$ scp dash@192.168.0.119:/home/dash/cloudstack.tar.gz ./
$ yum install -y java-1.7.0-openjdk-devel
$ yum install genisoimage
$ yum install -y ws-commons-util
$ yum install -y MySQL-python
$ yum install -y createrepo
$ yum install -y tomcat
$ yum install -y mariadb mariadb-server
$ scp dash@192.168.0.119:/home/dash/apache-maven-3.0.5-bin.tar.gz ./
$ tar xzvf apache-maven-3.0.5-bin.tar.gz 
$ export PATH=~/apache-maven-3.0.5/bin:$PATH
$ export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk
$ mvn --version
$ mvn -P deps -D nonoss -DskipTests=true
$ yum install -y maven
$ ./install-non-oss.sh 
$ cd packaging/
$ ./package.sh -p noredist -d centos7

```
