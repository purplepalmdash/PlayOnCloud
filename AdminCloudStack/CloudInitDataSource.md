## Cloud-Init数据源
### 数据源介绍
数据源是供cloud-init所使用的数据，通常来自于用户(即userdata)或者来自于创建配置驱动的栈
(即metadata)。通常userdata包括了文件，yaml, 和shell命令等；而典型的metadata则包括服务器
名，instance id, 显示名称以及其他云端需要的细节信息。     

有很多种方法可以用来提供这类数据(每种云解决方案似乎都倾向于提供自己的解决方案)。
Cloud-Init有其内部的数据源抽象类，用一种相同的方式来处理各种不同的云系统所提供的数据源
.       

在API的层面，数据源对象提供了以下的接口:     

```
# returns a mime multipart message that contains
# all the various fully-expanded components that
# were found from processing the raw userdata string
# - when filtering only the mime messages targeting
#   this instance id will be returned (or messages with
#   no instance id)
def get_userdata(self, apply_filter=False)

# returns the raw userdata string (or none)
def get_userdata_raw(self)

# returns a integer (or none) which can be used to identify
# this instance in a group of instances which are typically
# created from a single command, thus allowing programatic
# filtering on this launch index (or other selective actions)
@property
def launch_index(self)

# the data sources' config_obj is a cloud-config formated
# object that came to it from ways other than cloud-config
# because cloud-config content would be handled elsewhere
def get_config_obj(self)

#returns a list of public ssh keys
def get_public_ssh_keys(self)

# translates a device 'short' name into the actual physical device
# fully qualified name (or none if said physical device is not attached
# or does not exist)
def device_name_to_device(self, name)

# gets the locale string this instance should be applying
# which typically used to adjust the instances locale settings files
def get_locale(self)

@property
def availability_zone(self)

# gets the instance id that was assigned to this instance by the
# cloud provider or when said instance id does not exist in the backing
# metadata this will return 'iid-datasource'
def get_instance_id(self)

# gets the fully qualified domain name that this host should  be using
# when configuring network or hostname releated settings, typically
# assigned either by the cloud provider or the user creating the vm
def get_hostname(self, fqdn=False)

def get_package_mirror_info(self)
```
### No Cloud数据源
上一节里所展示的就是用No Cloud来提供数据源的方式。    

数据源`NoCloud`和`NoCloudNet`让用户在无网络服务的情况下提供`user-data`和`meta-data`给虚
拟机的运行实例，甚至它可以在无网络设备的情况下运行。    

我们可以通过在虚拟机启动时加载一个vfat或iso9660文件系统的方式，提供出`user-data`和
`meta-data`。      

这些提供的`user-data`和`meta-data`文件应该具备以下的格式, 即处于该文件系统根目录下:    

```
/user-data
/meta-data
```

下面给出的是ubuntu 12.04 cloud image 可以使用的`user-data`和`meta-data`:    

```
## create user-data and meta-data files that will be used
## to modify image on first boot
$ { echo instance-id: iid-local01; echo local-hostname: cloudimg; } > meta-data

$ printf "#cloud-config\npassword: passw0rd\nchpasswd:  
{ expire: False }\nssh_pwauth: True\n" > user-data

## create a disk to attach with some user-data and meta-data
$ genisoimage  -output seed.iso -volid cidata -joliet -rock user-data meta-data

## alternatively, create a vfat filesystem with same files
## $ truncate --size 2M seed.img
## $ mkfs.vfat -n cidata seed.img
## $ mcopy -oi seed.img user-data meta-data ::

## create a new qcow image to boot, backed by your original image
$ qemu-img create -f qcow2 -b disk.img boot-disk.img

## boot the image and login as 'ubuntu' with password 'passw0rd'
## note, passw0rd was set as password through the user-data above,
## there is no password set on these images.
$ kvm -m 256 \
   -net nic -net user,hostfwd=tcp::2222-:22 \
   -drive file=boot-disk.img,if=virtio \
   -drive file=seed.iso,if=virtio
```
在上一节中使用的img方式类似。    

### CloudStack数据源
Apache CloudStack在Virtual-Router虚拟机上暴露出了user-data, meta-data，用户密码以及账户
的sshkey等数据。更详细的关于meta-data和user-data可以从CloudStack的管理手册上查阅到:    

[CloudStack管理手册](http://docs.cloudstack.apache.org/projects/cloudstack-administration/en/latest/virtual_machines.html#user-data-and-meta-data)    

用于访问user-data和meta-data的URLs如下, 这里10.1.1.1代表Virtual Router的IP：    

```
http://10.1.1.1/latest/user-data
http://10.1.1.1/latest/meta-data
http://10.1.1.1/latest/meta-data/{metadata type}
```


