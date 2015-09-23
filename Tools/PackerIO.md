## Packer.io
Packer是一个用于从预定义好的配置文件中创建出虚拟机和容器镜像的工具，用Packer创
建出来的镜像可以运行于多种平台，也可以直接部署到云端。Packer可以运行在
Linux/Unix/MacOS以及Windows上。   

官方网站:    
[https://www.packer.io/](https://www.packer.io/)   
 
### 安装
首先从
[https://www.packer.io/downloads.html](https://www.packer.io/downloads.html)下
载相应版本的Packer.io,我们这里下载Linux 64-bit的安装包，下载完毕后的大小是
127M:     

```
$ wget https://dl.bintray.com/mitchellh/packer/packer_0.8.6_linux_amd64.zip
```

直接解压压缩包到某个目录而后把该目录加入到系统路径:    

```
$ mkdir ~/bin
$ mv packer_0.8.6_linux_amd64.zip ~/bin
$ cd ~/bin
$ unzip *.zip
$ vim ~/.bashrc
export PATH=/home/XXXXX/bin:$PATH
$ source ~/.bashc
$ which packer
/home/XXXXX/bin/packer
```

### 
