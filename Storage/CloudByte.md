## CloudByte

### Virtualbox网络准备
创建一个Host-Only网络:    

![images/2015_12_24_14_41_56_533x357.jpg](images/2015_12_24_14_41_56_533x357.jpg)   

更改网段的IP地址范围:    

![images/2015_12_24_14_41_40_511x229.jpg](images/2015_12_24_14_41_40_511x229.jpg)     

禁止DHCP服务:    

![images/2015_12_24_14_41_48_517x226.jpg](images/2015_12_24_14_41_48_517x226.jpg)    

为了允许此Host-Only网络可以连接到Internet，添加以下规则:    

```
$ sudo iptables -t nat -A POSTROUTING -s 10.58.58.1/24 ! -d 10.58.58.0/24 -j MASQUERADE
```

### Elastistor虚拟机准备
创建一个内存4G， 双核的Virtualbox虚拟机，添加5块硬盘如下:    

![/images/2015_12_24_14_55_50_577x439.jpg](/images/2015_12_24_14_55_50_577x439.jpg)   

加完后的磁盘布局如下:    

![/images/2015_12_24_14_57_20_654x377.jpg](/images/2015_12_24_14_57_20_654x377.jpg)  

选择网络如下:    

![/images/2015_12_24_14_58_27_643x377.jpg](/images/2015_12_24_14_58_27_643x377.jpg)   

点击开始安装系统, 选择:    

![/images/2015_12_24_14_59_37_592x364.jpg](/images/2015_12_24_14_59_37_592x364.jpg)    

选择Standalone ElastiCenter,  选择第一块硬盘作为安装盘，一路进入到设置IP地址的对话框中
。     

![/images/2015_12_24_15_01_50_493x367.jpg](/images/2015_12_24_15_01_50_493x367.jpg)   

开始安装， 安装完毕后，选择时区为Asia/China.    

### CloudStack环境准备
在Virtualbox中创建一个虚拟机，安装CentOS6.7, 部署CloudStack4.6环境，不一一列举了。   


这种其实会失败，因为Virtualbox不支持全虚拟化。so, come back to kvm.   


### Elastistor配置
disable scsi，这样可以继续到下一步Pool的配置:    

```
$ touch /etc/disablescsi
```

添加Pool, 选择mirror模式的话，只需要选择两块盘:    

![images/2015_12_24_17_19_22_425x535.jpg](images/2015_12_24_17_19_22_425x535.jpg)   

继续点击下一步，设定IOPS之类, 我们这里默认不变:    
 
![images/2015_12_24_17_19_38_425x444.jpg](images/2015_12_24_17_19_38_425x444.jpg)    

获得RESTFUL API所需要使用的 API key和password:    

![/images/2015_12_24_17_34_53_985x287.jpg](/images/2015_12_24_17_34_53_985x287.jpg)   

在Cloudstack的Global Setting里，填入api-key和password:    

![/images/2015_12_24_17_39_58_772x220.jpg](/images/2015_12_24_17_39_58_772x220.jpg)    

 
