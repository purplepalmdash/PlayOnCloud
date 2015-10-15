## 准备包
我们首先把CloudStack的DEB安装包下载到本地，构建出本地仓库，以便安装和部
署的流程。     

### 批量下载包
可以用Firefox的Downthemall插件，批量从下面地址下载DEB包:    

[http://cloudstack.apt-get.eu/ubuntu/dists/trusty/4.5/pool/](http://cloudstack.apt-get.eu/ubuntu/dists/trusty/4.5/pool/)    

![/images/2015_10_15_10_24_45_680x580.jpg](/images/2015_10_15_10_24_45_680x580.jpg)    

### 构建仓库
在Debian系机器机器上，安装`dpkg-dev`包.   

```
$ ls cloudstackdeb 
cloudstack-agent_4.5.2_all.deb   cloudstack-cli_4.5.2_all.deb
cloudstack-docs_4.5.2_all.deb        cloudstack-usage_4.5.2_all.deb
cloudstack-awsapi_4.5.2_all.deb  cloudstack-common_4.5.2_all.deb
cloudstack-management_4.5.2_all.deb
$ dpkg-scanpackages cloudstackdeb | gzip -9c >
cloudstackdeb/Packages.gz
dpkg-scanpackages: info: Wrote 7 entries to output Packages file.
```  

现在把cloudstackdeb目录上传到Web服务器根目录下.   

以后要使用该仓库时，只需要编辑sources.list文件如下:    

```
$ vim /etc/apt/sources.list
deb http://Your_Server_IP/	cloudstackdeb/
```
