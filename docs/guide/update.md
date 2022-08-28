
# <center> OpenCloudOS-内核更新</center>


## 1. 内核升级与配置

内核（kernel）是linux操作系统的核心，对系统资源进行统筹管理，向上提供系统调用接口便于开发人员进行程序开发，向下利用驱动程序对硬件设备进行管理。
```
查看内核版本号
[root@OpencloudOS~]# uname -r
5.4.119-19.0010.ocrelease.6
```

查看系统版本信息
```
[root@VM-6-140-opencloudos boot]# cat /etc/os-release
NAME="OpenCloudOS"
VERSION="8.5"
ID="opencloudos"
ID_LIKE="rhel fedora centos"
VERSION_ID="8.5"
PLATFORM_ID="platform:oc8"
PRETTY_NAME="OpenCloudOS 8.5"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:opencloudos:opencloudos:8"

CENTOS_MANTISBT_PROJECT="CentOS-8"
CENTOS_MANTISBT_PROJECT_VERSION="8"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="8"
NAME_ORIG="CentOS Linux"
......
```

### 1.1 使用yum升级内核

```
[root@VM-6-140-opencloudos /]#yum update kernel
```
### 1.2 手动安装RPM包更新内核

在下列链接中下载需要安装的内核RPM包
```
http://mirrors.tencent.com/opencloudos/版本号/BaseOS/x86_64/os/Packages/kernel-xxx.rpm
```
进入该目录，下载内核rpm包,安装RPM包，xxx.xxx.xx表示对应内核版本号
```
[root@VM-6-140-opencloudos /]#rpm -ivh kernel-xxx-xxx-xxx.rpm
```
*通过该方式安装的新内核会作为默认内核在下次启动后生效


### 1.3 查看已安装内核

同一个系统中可以安装多个内核，OpencloudOS系统中默认安装了grubby程序，在具有root权限的账户下，可以通过grubby命令查看当前系统中已经安装的内核。

```
# grubby --info=ALL | grep ^kernel
[root@VM-6-140-opencloudos boot]# grubby --info=ALL | grep ^kernel
kernel="/boot/vmlinuz-5.4.119-19.0010.ocrelease.6"
kernel="/boot/vmlinuz-0-rescue-990d0c1266464c1a9aeb3740e9b64217"
```
### 1.4 指定默认启动内核

1.查看当前系统中已安装的内核
```
[root@VM-6-140-opencloudos ~]# grubby --info=ALL | grep ^kernel
kernel="/boot/vmlinuz-5.4.119-19.0010.ocrelease.6"
kernel="/boot/vmlinuz-0-rescue-990d0c1266464c1a9aeb3740e9b64217"
```
2.选定需要默认启动的内核并将其设置为默认启动
```
#grubby --set-default = $kernel_path
如：
[root@VM-6-140-opencloudos ~]# grubby --set-default=/boot/vmlinuz-5.4.119-19.0010.ocrelease.6
The default is /boot/loader/entries/990d0c1266464c1a9aeb3740e9b64217-5.4.119-19.0010.ocrelease.6.conf with index 0 and kernel /boot/vmlinuz-5.4.119-19.0010.ocrelease.6
```
3.重启机器切换内核
```
[root@VM-6-140-opencloudos /]#reboot
```
4.重启后查看内核是否已变更
```
[root@VM-6-140-opencloudos /]#uname -r
```
