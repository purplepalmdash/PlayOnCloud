## 编译CloudStack RPM
首先下载第三方依赖包:    

```
# cd deps
# wget http://zooi.widodh.nl/cloudstack/build-dep/cloud-iControl.jar
# wget http://zooi.widodh.nl/cloudstack/build-dep/cloud-manageontap.jar
# wget http://zooi.widodh.nl/cloudstack/build-dep/vmware-vim.jar
# wget http://zooi.widodh.nl/cloudstack/build-dep/vmware-vim25.jar
# wget http://zooi.widodh.nl/cloudstack/build-dep/vmware-apputils.jar
# wget http://zooi.widodh.nl/cloudstack/build-dep/cloud-netscaler-jars.zip
# mv cloud-manageontap.jar manageontap.jar
# mv vmware-apputils.jar apputils.jar
# mv vmware-vim.jar vim.jar
# mv vmware-vim25.jar vim25.jar
# unzip cloud-netscaler-jars.zip
```

从Vmware网站下载到SDK，并提取内容:    

```
VMware-vSphere-SDK-5.1.0-774886.zip
https://my.vmware.com/group/vmware/get-download?downloadGroup=VSP510-WEBSDK-510 

# unzip VMware-vSphere-SDK-5.1.0-774886.zip
# cp -p SDK/vsphere-ws/java/JAXWS/lib/vim25.jar vim25_51.jar
# ./install-non-oss.sh
```

编译依赖关系:    

```
# mvn -P deps -D nonoss -DskipTests=true
```

生成包:    

```
# cd packaging/centos63
#  vi cloud.spec
if [ "%{_ossnoss}" == "NOREDIST" -o "%{_ossnoss}" == "noredist" ] ; then
   echo "Executing mvn packaging with non-redistributable libraries"
   if [ "%{_sim}" == "SIMULATOR" -o "%{_sim}" == "simulator" ] ; then
      echo "Executing mvn noredist packaging with simulator ..."
      mvn -Pawsapi,systemvm -Dnoredist -Dsimulator clean package
   else
      echo "Executing mvn noredist packaging without simulator..."
-      mvn -Pawsapi,systemvm -Dnoredist clean package
+      mvn -Pawsapi,systemvm -Dnoredist -DskipTests=true clean package
   fi
# ./package.sh -p noredist
```

编译完毕后，可以看到内容:    

```
$ ls
cloudstack-agent-4.5.2-1.el6.x86_64.rpm
cloudstack-cli-4.5.2-1.el6.x86_64.rpm
cloudstack-mysql-ha-4.5.2-1.el6.x86_64.rpm
cloudstack-awsapi-4.5.2-1.el6.x86_64.rpm
cloudstack-common-4.5.2-1.el6.x86_64.rpm      cloudstack-usage-4.5.2-1.el6.x86_64.rpm
cloudstack-baremetal-agent-4.5.2-1.el6.x86_64.rpm
cloudstack-management-4.5.2-1.el6.x86_64.rpm
$ pwd
/home/xxxx/Code/apache-cloudstack-4.5.2-src/dist/rpmbuild/RPMS/x86_64
```

