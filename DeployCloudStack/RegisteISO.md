## 注册ISO
除了通过模板创建虚拟机外，我们也可以采用从ISO安装创建虚拟机，以下是详细步骤。   

### 查看ISO
在Template下点击ISO，可以看到已经安装的ISO列表， 如下图:    

![/images/2015_09_26_17_20_06_770x261.jpg](/images/2015_09_26_17_20_06_770x261.jpg)    

其中Win7是我们手工导入的ISO， xs-tools.iso和vmware-tools.iso是Cloudstack自带的
ISO，用于在生成的XenServer的虚拟机和VMware的虚拟机中安装工具。接下来我们将手工
导入一个ISO用于安装虚拟机。    


### 导入ISO
首先在Global Settings里添加internal sites的允许下载IP地址， 如下图所示，更改完
毕以后，重启Coudstack-management服务:    

![/images/2015_09_26_17_25_30_743x190.jpg](/images/2015_09_26_17_25_30_743x190.jpg)    

首先准备好Ubuntu15.04的ISO，放到Web服务器的根目录下, 而后点击Registe ISO,在弹
出的页面中输入以下内容:    

![/images/2015_09_26_17_30_27_391x465.jpg](/images/2015_09_26_17_30_27_391x465.jpg)    

等待一段时间后，点击查看是否被完整导入。安装ISO中的状态如下：    

![/images/2015_09_26_17_32_53_533x264.jpg](/images/2015_09_26_17_32_53_533x264.jpg)    

可用状态如下:    

![/images/2015_09_26_17_33_40_588x197.jpg](/images/2015_09_26_17_33_40_588x197.jpg)       

### 从ISO创建虚拟机    
点击Instance -> Add Instance， 在选择zone时，选择ISO安装:    

![/images/2015_09_26_17_34_51_514x428.jpg](/images/2015_09_26_17_34_51_514x428.jpg)    

在选择模板时，选择对应的ISO：    

![/images/2015_09_26_17_35_46_493x385.jpg](/images/2015_09_26_17_35_46_493x385.jpg)    

剩下的过程和从模板创建的过程相同， 点击View Console后，我们将看到系统安装的页
面:   

![/images/2015_09_26_17_42_58_799x629.jpg](/images/2015_09_26_17_42_58_799x629.jpg)   

安装完后的VM与从模板创建的VM无异。    
