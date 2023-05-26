
# <center> OpenCloudOS-Kernel update</center>


## 1. Upgrade and configure the kernel

Kernel is the core of linux operating system. It manages system resources as a whole, provides system call interfaces upward for developers to develop programs, and uses drivers downward to manage hardware devices.
```
View the kernel version
[root@OpencloudOS~]# uname -r
5.4.119-19.0010.ocrelease.6
```

View the system version information
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

### 1.1 Use yum to upgrade the kernel

```
[root@VM-6-140-opencloudos /]#yum update kernel
```
### 1.2 Manually install the RPM package to update the kernel

Download the kernel RPM package you need to install at the following link
```
http://mirrors.tencent.com/opencloudos/Version-number/BaseOS/x86_64/os/Packages/kernel-xxx.rpm
```
Go to the directory, download the kernel rpm package, and install the RPM package. xxx.xxx.xx indicates the kernel version number
```
[root@VM-6-140-opencloudos /]#rpm -ivh kernel-xxx-xxx-xxx.rpm
```
*The new kernel installed in this way will take effect after the next boot as the default kernel


### 1.3 View the installed kernel

Multiple kernels can be installed in the same system. The grubby program is installed in the OpencloudOS system by default. You can run the grubby command as user root to view the installed kernels in the current system.

```
# grubby --info=ALL | grep ^kernel
[root@VM-6-140-opencloudos boot]# grubby --info=ALL | grep ^kernel
kernel="/boot/vmlinuz-5.4.119-19.0010.ocrelease.6"
kernel="/boot/vmlinuz-0-rescue-990d0c1266464c1a9aeb3740e9b64217"
```
### 1.4 Specifies the default boot kernel

1.View the kernel installed in the current system
```
[root@VM-6-140-opencloudos ~]# grubby --info=ALL | grep ^kernel
kernel="/boot/vmlinuz-5.4.119-19.0010.ocrelease.6"
kernel="/boot/vmlinuz-0-rescue-990d0c1266464c1a9aeb3740e9b64217"
```
2.Select the kernel that you want to start by default and set it to default
```
#grubby --set-default = $kernel_path
Such as:
[root@VM-6-140-opencloudos ~]# grubby --set-default=/boot/vmlinuz-5.4.119-19.0010.ocrelease.6
The default is /boot/loader/entries/990d0c1266464c1a9aeb3740e9b64217-5.4.119-19.0010.ocrelease.6.conf with index 0 and kernel /boot/vmlinuz-5.4.119-19.0010.ocrelease.6
```
3.Restart the machine to switch the kernel
```
[root@VM-6-140-opencloudos /]#reboot
```
4.After the restart, check whether the kernel has been changed
```
[root@VM-6-140-opencloudos /]#uname -r
```
