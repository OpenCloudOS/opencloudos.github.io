

# <center> OpenCloudOS-系统管理</center>

## 1. 相关技术基础介绍

### 1.1 什么是RPM包

RPM(RedHat Package Manager) 一种通过资料库管理的方式将所需要的软件安装到主机上的管理程序。这一文件格式名称虽然打上了RedHat的标志，但是其原始设计理念是开放式的，包括OpenLinux、SuSe.以及Turbo Linux等Linux的分发版本都有采用，算是公认的行业标准。OpencloudOS同样采用rpm文件格式封装软件。RPM包的命名格式如下
```
bind-9.8.2-0.47.rc1.el6.x86_64.rpm
```
name，如：bind，表示软件名称
version，如：9.8.2- 0 ，是软件的版本号，版本号格式通常为“主版本号.次版本号.修正号”47，是发布版本号，表示这个rpm软件包是第几次编译生成<br />
arch，如i386，是表示包适用的硬件平台<br />
rpm和.src.rpm，是rpm包类型后缀，rpm是编译后的二进制包，src.rpm为源码包特殊名称。<br />
el*：表示发行商的版本，el6表示这个软件包是在rhel6.x/centos6.x下使用。<br />
devel：表示这个rpm包是软件的开发包; noarch：说明这样的软件包可以在任何平台安装和运行，不需要特定的硬件平台;


### 1.2 什么是yum

Yum（Yellow dog Updater, Modified）是众多Linux发行版本的Shell 前端软件包管理器，基于 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。OpencloudOS 中默认指定yum作为rpm软件包管理器，其从腾讯自己维护的镜像仓库下载对应的软件包。

### 1.3 kdump工具

kdump是一项提供内核崩溃时，将内核信息转储的服务。该服务能够保存系统内存的内容以供分析。kdump使用kexec系统调用启动到第二个内核（捕获内核），而无需重新启动；然后捕获崩溃内核内存的内容（崩溃转储或vmcore）并将其保存到文件中。第二个内核位于系统内存的保留部分。
## 2.内核模块配置

内核模块（Linux kernel Modules, LKM）是运行在内核态，用于扩展基本内核功能，并允许在系统运行过程中动态加载或者卸载的一段程序，Linux内核是宏内核，内核作为一个整体，包揽了所有的系统功能，比如，进程调度、内核管理、文件系统、设备管理等等，但若开启所有的功能，会使得内核变得十分的臃肿。可以将内核功能模块化，只保留最为核心的功能，其他功能模块随着后续需求进行动态的加载，达到对内核进行精简的目的。

内核模块主要的功能可以归为以下三类：

- 作为驱动程序支持新硬件设备
- 支持新的文件系统，如GFS或NFS
- 作为系统调用程序为上层应用程序提供支持


### 2.1 查看已加载模块

OpencloudOS系统中默认安装了kmod程序，可以通过kmod程序中的lsmod命令查看当前系统中已经加载的内核模块

```
[root@VM-6-140-opencloudos boot]# lsmod
Module 			Size 		Used 		by
tcp_diag 		16384 		0	
inet_diag 		24576 		1 		tcp_diag
nf_tables 		139264 		0
nfnetlink 		16384 		1 		nf_tables
overlay 		110592 		0
sunrpc 			393216 		1
edac_core 		57344 		0
crc32_pclmul 		16384 		0
ghash_clmulni_intel 	16384 		0
aesni_intel 		372736 		0
crypto_simd 		16384 		1 		aesni_intel
cryptd 			24576 		2 		crypto_simd,ghash_clmulni_intel
glue_helper 		16384 		1 		aesni_intel
virtio_balloon 		20480 		0
...
```
第一列表示当前已加载的模块名；第二列表示对应模块占用的内存大小（单位：KB）；最后一列表示该模块依赖其他模块的数量和模块名。

### 2.2 查看模块信息

可以通过kmod程序中的modinfo命令查看内核模块的信息,这里以加密模块cryptd为例。
```
[root@VM-6-140-opencloudos boot]# modinfo cryptd
filename:       /lib/modules/5.4.119-19.0010.ocrelease.6/kernel/crypto/cryptd.ko.xz
alias:          crypto-cryptd
alias:          cryptd
description:    Software async crypto daemon
license:        GPL
srcversion:     6376601629563723420DCA5
depends:        
retpoline:      Y
intree:         Y
name:           cryptd
vermagic:       5.4.119-19.0010.ocrelease.6 SMP mod_unload modversions 
parm:           cryptd_max_cpu_qlen:Set cryptd Max queue depth (uint)
```
更多了解modinfo相关资料，可查阅：
-  modinfo(8) Manual Page

### 2.3 动态加载/卸载模块

扩展Linux内核功能的最佳方法是加载内核模块。以下过程描述了如何使用modprobe命令查找内核模块并将其加载到当前运行的内核中。值得注意的是，不能加载已经在内核中加载的模块；下面以加载md4模块为例演示动态加载模块的过程

1.在/lib/modules/当前内核版本号/kernel/子系统/路径找到想要加载的模块，以加密子模块md4为例
```
[root@OpencloudOS~]# cd /lib/modules/5.4.119-19.0010.ocrelease.6/kernel/crypto
[root@OpencloudOS~]# ls -l
......
- rwxr--r-- 1 root root 10392 Apr 20 22:44 dh_generic.ko
- rwxr--r-- 1 root root 8304 Apr 20 22:44 md4.ko
...
```
2.检查对应模块是否被加载，执行以下命令后，若该模块被加载则输出对应的模块信息，反之则无任何输出信息
```
[root@OpencloudOS~]# lsmod | grep md
```
3.动态加载模块，使用modprobe命令加载md4模块
```
[root@OpencloudOS~]# modprobe md4
```
同理可知动态卸载模块的操作流程，同样以md4模块为例，在已经加载该模块的前提下，执行下列命令卸载md4模块。
```
[root@OpencloudOS~]# modprobe -r md
```
### 2.4 添加默认启动模块

下面过程描述了如何将内核模块添加到启动列表中，以便它在系统开机引导过程中自动加载。操作过程中需要root权限以及kmod工具，其中在OpencloudOS系统中已集成kmod工具。其操作流程如下

1.在/lib/modules/当前内核版本号/kernel/子系统/路径找到想要加载的模块，以本机内核
5.4.119- 19 - 0009.3中的加密子模块md4为例
```
[root@OpencloudOS~]# cd /lib/modules/5.4.119-19.0010.ocrelease.6/kernel/crypto/
[root@OpencloudOS~]# ls -l
```
2.为需要加载的模块创建一个配置文件，使用命令“echo <模块名> > /etc/modules-load.d/<模块名>.conf”
```
[root@OpencloudOS~]# echo md4 > /etc/modules-load.d/md4.conf
[root@OpencloudOS~]# reboot
```
3.查看当前已加载内核模块
```
[root@OpencloudOS~]#lsmod
```
### 2.5 卸载默认启动模块

以下过程描述了如何将内核模块添加到拒绝列表中，以便它不会在引导过程中自动加载。(以cdrom模块为例)

1.为需要禁用的模块创建黑名单,配置文件路径为：/etc/modprobe.d/blacklist.conf
```
[root@OpencloudOS~]#vim /etc/modprobe.d/blacklist.conf

#blacklist <MODULE_NAME>
blacklist cdrom
install cdrom /bin/false
```
黑名单确保在启动过程中不会自动加载相关的内核模块。然而，黑名单命令并不阻止该模块作为另一个不在拒绝列表中的内核模块的依赖项加载。因此，install行会导致/bin/false运行，而不是安装模块。

2.创建一个当前内核版本的initramfs备份映像，以防配置出现意外
```
[root@OpencloudOS~]#cp /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).bak.$(date+%m-%d-%H%M%S).img
```
3.生成一个新的initramfs镜像
```
[root@OpencloudOS~]#dracut -f -v
```
4.重启机器
```
[root@OpencloudOS~]#reboot
```
5.可以查看到cdrom模块已不再默认加载
```
[root@OpencloudOS~]#lsmod
```
*本节中描述的更改将在重新启动系统后生效。如果不当地将加密内核模块放入拒绝列表中，可能会面临系统不稳定等问题。

## 3.系统日志

日志文件是包含有关系统消息的文件，包括内核、服务和在上面运行的应用程序。opencloudos的日志系统基于内置的syslog协议。各种实用程序使用此系统来记录事件并将其组织成日志文件。这些文件在升级操作系统或故障排除问题时非常有用。在启动过程中，控制台提供了许多关于系统启动初始阶段的重要信息。为了避免丢失早期消息，内核使用所谓的环形缓冲区。此缓冲区存储内核代码中printk()函数生成的所有消息，包括引导消息。然后，来自内核环缓冲区的消息被读取并存储在永久存储的日志文件中，例如syslog服务。

上面提到的缓冲区是一个具有固定大小的循环数据结构，并硬编码到内核中。用户可以通过dmesg命令或/var/log/boot.log文件查看存储在内核环缓冲区中的数据。当环缓冲区已满时，新数据将覆盖旧数据。

内核报告的每条消息都有一个与之关联的日志级别，该日志级别定义了消息的重要性。内核环缓冲区收集所有日志级别的内核消息，kernel.printk参数定义了从缓冲区打印到控制台的消息。

```
0 - 内核紧急情况，系统无法正常启动
1 - 内核警报，故障必须立即采取措施
2 - 内核目前情况较严峻
3 - 一般内核错误
4 - 一般内核警告
5 - 正常但是较重要的情况
6 - 普通内核信息
7 - 内核调试信息
```
查看opencloud系统的kernel.prink值

```
[root@VM-6-140-opencloudos crypto]# cat /proc/sys/kernel/printk
5       4       1       7
```
这四个值定义如下（数值越小，优先级越高）：<br />
第一个值：控制台日志级别；优先级高于该值的消息将被打印至控制台<br />
第二个值：默认的消息日志级别；将用该优先级来打印没有优先级的消息<br />
第三个值：最低的控制台日志级别；控制台日志级别可被设置的最小值(最高优先级)<br />
第四个值：默认的控制台日志级别：控制台日志级别的缺省值<br />

内核及系统日志由系统服务rsyslog统一管理，根据其主配置文件/etc/rsyslog.conf中的设置决定将内核消息及各种系统程序消息记录到什么位置。
常用的系统日志文件如下表所示：<br />
```
/var/log/messages 记录内核及各种应用程序的公共日志信息
/var/log/cron     记录crond计划任务产生的事件信息
/var/log/dmesg    记录操作系统在引导过程中的各种事件信息
/var/log/mailog   记录进入或发出系统的电子邮件活动
/var/log/boot.log 系统启动时日志，包括自启动服务
/var/log/yum.log  包含yum安装软件包的信息
/var/log/audit/   包含audit daemon的审计日志
/var/log/sa/      包含每日由sysstat软件包收集的sar文件
```
查看日志方法,如：

```
[root@OpencloudOS~]#cat /var/log/boot.log
```

## 4. kdump-分析内核崩溃

kdump是一项提供内核崩溃时，将内核信息转储的服务。该服务能够保存系统内存的内容以供分析。kdump使用kexec系统调用启动到第二个内核（捕获内核），而无需重新启动；然后捕获崩溃内核内存的内容（崩溃转储或vmcore）并将其保存到文件中。第二个内核位于系统内存的保留部分。

安装kdump:
```
[root@OpencloudOS~]#yum install kexec-tools
```

### 4.1 配置kdump

#### 4.1.1 预估kdump内存占用

在规划和构建kdump环境时，了解崩溃转储文件需要多少空间很重要。makedumpfile --mem-usage命令能够估计崩溃转储文件需要多少空间。它生成内存使用情况报告。该报告可帮助确定转储级别以及哪些页面可以安全地被排除在外。
```
[root@VM-6-140-opencloudos crypto]# makedumpfile --mem-usage /proc/kcore

TYPE            PAGES                   EXCLUDABLE      DESCRIPTION
----------------------------------------------------------------------
ZERO            20764                   yes             Pages filled with zero
NON_PRI_CACHE   86248                   yes             Cache pages without private flag
PRI_CACHE       11756                   yes             Cache pages with private flag
USER            38352                   yes             User process pages
FREE            747034                  yes             Free pages
KERN_DATA       73160                   no              Dumpable kernel data 

page size:              4096            
Total pages on system:  977314          
Total size on system:   4003078144       Byte
```
#### 4.1.2 配置捕获内核内存占用

dump的内存在系统启动期间保留。内存大小在系统Grand Unified Bootloader（GRUB） 2配置文件中配置。内存大小取决于配置文件中指定的crashkernel=选项的值和系统物理内存的大小。crashkernel=选项可以通过多种方式定义。用户可以指定crashkernel=值或配置自动选项。crashkernel=auto参数根据系统中的物理内存总量自动保留内存。配置后，内核将自动为捕获内核保留适当数量的所需内存。这有助于防止内存溢出（OOM）错误。其中kdump的自动内存分配因系统硬件架构和可用内存大小而异。例如，在AMD64和Intel 64上，crashkernel=auto参数仅在可用内存超过1GB时生效。 64 位ARM架构和IBM Power Systems需要物理内存超过2GB时生效。如果系统低于自动分配的最低内存阈值，需要用户手动配置保留内存量。配置kdump的内存占用需要root权限。配置过程如下

1.打开配置文件“/etc/default/grub/”, 找到字段“crashkernel=”,并根据以下参数配置
```
[root@VM-6-140-opencloudos /]# vim /etc/default/grub 
......
GRUB_CMDLINE_LINUX="quiet elevator=
noop console=ttyS0,115200 console=tty0 vconsole.keymap=us crashkernel=1800M-64G:256M,64G-
128G:512M,128G-:768M
......
```
用户可以根据已安装的内存总量将保留内存量设置为变量。将内存保留到变量的语法是crashkernel=<range1>:<size1>，<range2>:<size2>。如上图所示。如果系统内存总量在1800MB至64GB之间，如上述示例会保留256MB的内存。
**需要注意的是首次配置，需要重启预留内存，才能正常使用kdump**

2.更新grub2配置文件
```
[root@VM-6-140-opencloudos crypto]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
done
```
#### 4.1.3 捕获内核存放方式

崩溃转储通常作为文件存储在本地文件系统中，直接写入设备。或者，用户可以设置使用NFS或SSH协议通过网络发送崩溃转储,一次只能设置其中一个选项来保存崩溃转储文件。默认行为是将其存储在本地文件系统的/var/crash/目录中。配置流程如下：

1.如要将崩溃转储文件存储在本地文件系统的/var/crash/目录中，请编辑/etc/kdump.conf文件并指定路径
```
path /var/crash
core_collector makedumpfile -l --message-level 7 -d 31
#core_collector scp
#kdump_post /var/crash/scripts/kdump-post.sh
#kdump_pre /var/crash/scripts/kdump-pre.sh
```
2.使用NFS协议将崩溃转储存储到远程计算机，需要编辑/etc/kdump.conf配置文件，打开配置文件/etc/kdump.conf将nfs my.server.com:/export/tmp前的#号删除，将值替换为正确的主机名和目的地址
```
#ext4 LABEL=/boot
#ext4 UUID=03 138356 - 5e61-4ab3-b58e-27507ac
nfs my.server.com:/export/tmp
#ssh user@my.server.com
```
3.若使用SSH协议将崩溃转储存储到远程计算机，同样需要编辑/etc/kdump.conf配置文件，打开配置文件/etc/kdump.conf将ssh my.server.com:/export/tmp前的#号删除，将值替换为正确的主机名和目的地址，同样将ssh-key替换为正确的key
```
#nfs my.server.com:/export/tmp
#nfs [2001:db8::1:2:3:4]:/export/tmp
ssh user@my.server.com
sshkey /root/.ssh/kdump_id_rsa
path /var/crash
```
#### 4.1.4 查看kdump配置文件

kdump内核的配置文件是/etc/sysconfig/kdump。此文件控制kdump内核命令行参数。对于大多数配置，使用默认选项即可。但是，在某些情况下需要修改某些参数来控制kdump内核行为。例如，添加 kdump 内核命令行以获得更详细的调试信息。本节针对参数KDUMP_COMMANDLINE_REMOVE以及KDUMP_COMMANDLINE_APPEND进行说明，更
多的信息请参考/etc/sysconfig/kdump 文件。


- KDUMP_COMMANDLINE_REMOVE<br />
此变量用于从当前kdump 命令行中删除参数。错误的设置该变量的某些参数可能导致
kdump错误或kdump内核引导失败。这些参数可能是从之前的KDUMP_COMMANDLINE进程解析的，也可能是从/proc/cmdline文件中继承的。当未配置此变量时，它将从/proc/cmdline文件中继承所有值。配置此变量有助于调试问题。例如在该变量中添加参数“hugepages hugepagesz slub_debug quiet log_buf_len swiotlb”
```
# This variable lets us remove arguments from the current kdump commandline
# as taken from either KDUMP_COMMANDLINE above, or from /proc/cmdline
# NOTE: some arguments such as crashkernel will always be removed
KDUMP_COMMANDLINE_REMOVE="hugepages hugepagesz slub_debug quiet log_buf_len swiotlb cma
hugetlb_cma"
```
- KDUMP_COMMANDLINE_APPEND<br />
此 选 项 将 参 数 附 加 到 当 前 命 令 行 。 这 些 参 数 可 能 已 被 之 前 的KDUMP_COMMANDLINE_REMOVE变量解析。对于kdump而言，禁用一些模块例如mce，cgroup，numa，hest_disable可以防止内核错误。这些模块会消耗很大一部分为kdump保留的内核内存，或导致kdump内核启动失败。例如将cgroup模块在kdump内核中禁用。
```
# This variable lets us append arguments to the current kdump commandline
# after processed by KDUMP_COMMANDLINE_REMOVE
KDUMP_COMMANDLINE_APPEND="irqpoll nr_cpus=1 reset_devices cgroup_disable=memory mce=off
numa=off udev.children-max=2 panic=10 acpi_no_memhotplug transparent_hugepage=never nokaslr
hest_disable novmcoredd cma=0 hugetlb_cma=0 noefi brd.rd_nr=0 nbd.nbds_max=0 pty.legacy_count=
swiotlb=noforce loop.max_loop=4 audit=0"
```
#### 4.1.5 启用kdump配置文件

本节将测试kdump崩溃转储过程是否有效。*以下命令会导致内核崩溃。遵循这些步骤时要小心，切勿在生产环境进行测试。

1.确保kdump在测试机器上已经启动
```
[root@VM-6-140-opencloudos crypto]# systemctl is-active kdump
active
```
2.强制崩溃内核，核心转储
```
#root@VM-6-140-opencloudos]#echo c > /proc/sysrq-trigger
```
再次启动后，格式为address-YYYY-MM-DD-HH:MM:SS/vmcore文件将在/etc/kdump.conf文
件中指定的位置创建（默认为/var/crash/）。
```
[root@VM-6-140-opencloudos /]# ls /var/crash/127.0.0.1-2022-08-19-02\:34\:00/
kexec-dmesg.log  vmcore  vmcore-dmesg.txt
```
   
### 4.2 启动kdump

本节介绍如何为所有已安装的内核或特定内核启用和启动kdump服务。

#### 4.2.1 内核启用kdump

用户可以为机器上安装的所有内核启用并启动kdump服务。该操作步骤需具有root权限，其操作流程如下

1.对系统中所有安装的内核添加crashkernel=auto命令行参数
```
[root@OpencloudOS~]#grubby --update-kernel=ALL --args="crashkernel=auto"
```
2.开启kdump服务
```
[root@OpencloudOS~]#systemctl enable --now kdump.service
```
3.验证kdump服务是否激活
```
[root@VM-6-140-opencloudos crypto]# systemctl status kdump.service
● kdump.service - Crash recovery kernel arming
   Loaded: loaded (/usr/lib/systemd/system/kdump.service; enabled; vendor preset: enabled)
   Active: active (exited) since Fri 2022-08-19 02:34:21 UTC; 45min ago
  Process: 1222 ExecStart=/usr/bin/kdumpctl start (code=exited, status=0/SUCCESS)
 Main PID: 1222 (code=exited, status=0/SUCCESS)
```
#### 4.2.2 指定内核启动kdump

用户可以为机器上特定的内核启用并启动kdump服务。该操作步骤需具有root权限，其操作流程如下

1.添加kdump内核到特定内核的Grand Unified Bootloader（GRUB） 2 配置文件中
```
[root@OpencloudOS~]#grubby --update-kernel=vmlinuz-xxx-330.el8.x86_64 --args="crashkernel=auto"
```
2.启动kdump服务
```
[root@OpencloudOS~]#systemctl enable --now kdump.service
```
3.验证kdump服务是否已经激活
```
[root@OpencloudOS~]#systemctl status kdump.service
```
#### 4.2.3 kexec切换内核

kexec系统调用允许从当前运行的内核加载和引导到另一个内核，充当一个当前内核下的
bootloader程序。kexec程序加载内核和initramfs映像，以便kexec系统调用启动到另一个内核
中。下面描述了如何手动调用kexec系统调用重新启动到另一个内核。

查看当前已安装内核

```
[root@VM-6-140-opencloudos boot]# grubby --info=ALL | grep ^kernel
kernel="/boot/vmlinuz-5.4.119-19.0010.ocrelease.6"
kernel="/boot/vmlinuz-0-rescue-990d0c1266464c1a9aeb3740e9b64217"
```
查看当前内核
```
[root@VM-6-140-opencloudos crypto]# uname -r
5.4.119-19.0010.ocrelease.6
```
手动加载kexec系统调用的内核和initramfs映像。
```
[root@OpencloudOS~]#kexec -l /boot/vmlinuz-5.4.119-xx-xx --initrd=/boot/initramfs-5.4.119-xx-xx.img --reuse-cmdline
```
重启机器
```
[root@VM-6-140-opencloudos boot]# reboot
```
查看当前内核版本
```
[root@OpencloudOS~]# uname -r
```
成功切换内核<br />
**当使用kexec -e命令将机器重新启动到另一个内核时，系统在启动下一个内核之前不会经过标准关机顺序。这可能会导致数据丢失或系统无响应。**

### 4.3 分析crash内核

要确定系统崩溃的原因，用户可以使用崩溃分析程序，该分析程序提供了一个与GNU调试器（GDB）非常相似的交互式提示。此实用程序允许交互式分析由kdump、netdump、diskdump或xendump以及正在运行的Linux系统创建的核心转储。以下过程描述了如何启动崩溃分析程序来分析系统崩溃的原因。OpencloudOS需要从下面的链接中下载kernel-debuginfo-common软件包安装（以5.14.119为例）

```
http://mirrors.tencent.com/opencloudos/8.5/BaseOS/x86_64/debug/tree/Packages/
```
安装下载的kernel-debuginfo包
```
[root@OpencloudOS~]#rpm -ivh kernel-debuginfo-5.4.119-19.0010.ocrelease.6.oc8.x86_64.rpm
```
运行crash分析程序需要确保以下两个文件的存在
- 调试镜像（vmlinux映像）
- vmcore文件(参考4.1.5节)
使用crash工具开始分析vmcore。crash接受两个参数，首先需要指定用于测试的调试镜像，
其次需要指定被分析的vmcore文件

启动crash
```
[root@OpencloudOS~]#crash /boot/vmlinux-xxx-tlinux-xxx /var/crash/xxx/vmcore
[root@VM-6-140-opencloudos /]# crash /boot/vmlinux-5.4.119-19.0010.ocrelease.6 /var/crash/127.xxx.xx.xx/vmcore
crash> q //退出crash
crash> log //显示消息缓存区
crash> bt //显示内核崩溃前的函数调用栈
crash> ps //显示内核崩溃前的进程状态
crash> vm //显示内核崩溃前的虚拟内存信息
crash> files //显示内核崩溃前的文件句柄信息
```
   
## 5.cgroup-系统资源配置

用户可以使用控制组（cgroups）内核功能来设置限制、优先排序或隔离进程的硬件资源。这允许用户更精细地控制应用程序的资源使用，以更有效地利用。


### 5.1 cgroup简介

cgroup是Linux内核的一个功能，可以将进程组织成分层有序的组。cgroup层次结构（cgroup
树）通过cgroups虚拟文件系统定义，默认情况下挂载在/sys/fs/cgroup/目录上。这是通过在
/sys/fs/cgroup/中创建和删除子目录手动完成的。或者，使用systemd系统和服务管理器。资源控
制器（内核组件）通过限制、优先排序或分配这些进程的系统资源（如CPU时间、内存、网络
带宽或各种组合）来修改cgroup中进程的行为。

- cgroup-v1
cgroups-v1提供了每个资源控制器层次结构。这意味着每个资源，如CPU、内存、I/O等，
都有自己的控制组层次结构,可以组合不同的控制组使一个控制器可以在管理各自的资源时与
另一个控制器协调。然而，这两个控制器可能属于不同的流程层次结构，不允许它们进行适当
的协调。Cgroups-v1控制器是在很长的时间跨度内开发的，因此，其控制文件的行为和命名并
不统一。

- cgroup-v2
cgroup v1的协调问题源直接导致了cgroup v2的开发。cgroups-v2提供了单个控制组层次结
构，所有资源控制器都安装在该层次结构上。

### 5.2 内核资源控制器

cgroup的功能由内核资源控制器启用。opencloudos支持cgroups-v1和
cgroups-v2的各种控制器。资源控制器，也称为控制组子系统，是表示单个资源的内核子系统，
例如CPU时间、内存、网络带宽或磁盘I/O。Linux内核提供一系列由systemd系统和服务管理
器自动安装的资源控制器。在/proc/cgroups文件中查找当前挂载的资源控制器列表。

cgroup v1主要支持以下控制器

- blkio-限制块设备的输入输出访问
- cpu-可以为cgroup的任务调整完全公平调度器（CFS）调度器的参数
- cpuacct-创建cgroup任务使用的CPU资源报告。它与cpu控制器一起安装
- cpuset-可用于限制cgroup任务仅在指定的CPU子集上运行，并指导任务仅在指定的内存节点上使用内存
- devices-控制cgroup任务对设备的访问
- freezer-能够挂起和恢复cgroup任务
- memory-可用于设置cgroup任务的内存使用限制，并生成有关这些任务使用的内存资源的自动报告
- net_cls-使用类标识符（classid）标记网络数据包，使Linux流量控制器（tc命令）能够识别来自特定控制组任务的数据包。Net_cls的子系统net_filter（iptables）也可以使用此标签对此类数据包执行操作。net_filter使用防火墙标识符（fwid）标记网络套接字，该标识符允许Linux防火墙（通过iptables命令）识别来自特定控制组任务的数据包
- net_prio-设置网络流量的优先级
- pids-可以在对照组中为多个进程及其子进程设置限制
- perf_event-可以通过perf性能监控和报告实用程序对要监控的任务进行分组
- rdma-可以设置控制组中远程直接内存访问特定资源的限制
- hugetlb-可用于限制控制组中任务对大尺寸虚拟内存页面的使用

cgroup v2 主要支持以下控制器：

- io-Cgroups-v1的blkio的升级
- memory- Cgroups-v1的memory的升级
- pids- Cgroups-v1的pids的升级。
- rdma- Cgroups-v1的rdma的升级。
- cpu- Cgroups-v1的cpu的升级。
- cpuset-仅支持具有新分区功能的核心功能（cpus{,.effective}、mems{,.effective}）。
- perf_event-内部支持且没有单独控制文件。
*资源控制器可以在cgroups-v1层次结构或cgroups-v2层次结构中使用，但不能同时在两者
中使用。

更多资料请参考：
- cgroup(7):manual page
- /usr/share/doc/kernel-doc<kernel_version>/Documentation/cgroups-v

### 5.3 命名空间

命名空间是组织和识别软件对象的最重要方法之一。命名空间将全局系统资源（例如挂载点、网络设备或主机名）包装在一个抽象中，使命名空间中的进程似乎拥有全局资源隔离实例。使用命名空间的最常见技术之一是容器。对特定全局资源的更改仅对该命名空间中的进程可见，不会影响系统的其余部分或其他命名空间。用户可以检查/proc/<PID>/ns/目录中的符号
链接从而得知进程是哪些命名空间的成员。

更多资料参考：

- namespace(7)以及cgroups_namespaces(7) manual pages

### 5.4 系统资源限制实例

某些情况下，应用程序会消耗大量的CPU时间，这可能会对环境的整体稳定性产生负面影响。使用/sys/fs/虚拟文件系统使用cgroups-v1为应用程序配置CPU限制。本节通过一个实例演示cgroup v1进行资源限制的流程。


CPU资源的配置的操作需要
- root权限
- 需要CPU资源限制的应用程序

确认cgroup v1已加载

```
[root@VM-6-140-opencloudos /]# mount -l | grep cgroup
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup
(rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-
agent,name=systemd)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
......
```
确定需要进行CPU限制的进程PID,(以stress压测软件为例),当前占用率为100%
```
[root@VM-6-140-opencloudos /]#top
PID   USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
39984 root 20 0  7968 100 0   R 100.0 0.0 0:07.56 stress
```
在CPU控制器目录下创建子目录，下面的目录代表一个控制组，用户可以在其中放置特定进
程，并对进程应用某些CPU限制。与此同时，将在目录中创建一些cgroups-v1接口文件和cpu
控制器特定文件。
```
[root@VM-6-140-opencloudos /]# mkdir -p /sys/fs/cgroup/cpu/stress
[root@VM-6-140-opencloudos /]# ls -l /sys/fs/cgroup/cpu/stress
total 0
- rw-r--r-- 1 root root 0 Aug 9 17:33 cgroup.clone_children
- rw-r--r-- 1 root root 0 Aug 9 17:33 cgroup.procs
- r--r--r-- 1 root root 0 Aug 9 17:33 cpuacct.mbuf
- r--r--r-- 1 root root 0 Aug 9 17:33 cpuacct.sli
- r--r--r-- 1 root root 0 Aug 9 17:33 cpuacct.sli_max
- r--r--r-- 1 root root 0 Aug 9 17:33 cpuacct.stat
- r--r--r-- 1 root root 0 Aug 9 17:33 cpuacct.uptime
......
```
示例输出显示了代表特定配置或限制的文件，如cpuacct.usage、cpu.cfs.period_us，这些文
件可以为stress控制组中的进程设置。请注意，各自的文件名前缀着它们所属的控制组控制器
的名称。默认情况下，新创建的控制组不受限制地继承对系统整个CPU资源的访问。

配置具体CPU参数,将对应的进程CPU使用率限制到50%
```
[root@OpencloudOS~]# echo “50000” > /sys/fs/cgroup/cpu/stress/cpu.cfs_quata_us
[root@OpencloudOS~]# echo “100000” > /sys/fs/cgroup/cpu/stress/cpu.cfs_period_us
```
指定 100 毫秒为一个时钟周期，stress进程运行 50 毫秒，即CPU使用率为50%
cpu.cfs_quota_us文件表示控制组中所有进程可以在一段时间内运行的总时间（以微秒为单
位）（由cpu.cfs_period_us定义）。一旦对照组中的进程在单个期间用完配额规定的所有时间，
它们就会在剩余时间内节流，并且不允许运行到下一个周期。下限是 1000 微秒。

确认参数配置
```
[root@OpencloudOS~]#cat /sys/fs/cgroup/cpu/stress/cpu.cfs_period_us /sys/fs/cgroup/cpu/stress
/cpu.cfs_quota_us
```
- 更多资料请参考：cgroups(7), sysfs(5) manual pages


