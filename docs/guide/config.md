# <center> OpencloudOS基础配置

## 第一章 初始环境设置

基础环境配置是安装过程的一部分，下面介绍下如何在安装系统后修改OS层面的基础配置。 基础环境配置包括：

- 时间和日期
- 系统locale

### 1.1 配置时间与日期

精确的时间在生产系统中至关重要。opencloudos下，通过NTP协议保证时间的准确性。NTP协议是通过用户态一个守护进程实现的，其会更新内核中的系统时钟，而系统时钟可以采用不同的时钟源去维护时间。


```
[root@VM-6-130-opencloudos /]# date

Tue Aug 2 19:44:17 CST 2022
```

如需查看更多信息，请使用timedatectl命令：
```
[root@VM-6-130-opencloudos /]# timedatectl

Local time: Tue 2022-08-02 19:46:01 CST

Universal time: Tue 2022-08-02 11:46:01 UTC

RTC time: Tue 2022-08-02 11:46:00

Time zone: Asia/Shanghai (CST, +0800)

System clock synchronized: yes

NTP service: active

RTC in local TZ: no
```

**更多帮助：** man date(1) / man timedatectl(1)

### 1.2 配置系统locale

系统范围内的locale配置于/etc/local/conf文件，在systemd初始启动时会读取该文件。每个服务或者用户都会默认继承这里的配置，当然一些程序和用户也可自定义修改配置。

查看可用的locale设置： localectl list-locales
```
[root@VM-6-130-opencloudos /]# localectl list-locales

C.utf8

en\_AG

en\_AU

en\_AU.utf8

en\_BW

en\_BW.utf8

...
```

查看当前的系统locale设置
```
[root@VM-6-130-opencloudos /]# localectl status

System Locale: LANG=en\_US.UTF-8

VC Keymap: us

X11 Layout: us
```

- 更多帮助：man localectl(1) man locale(7) man locale.conf(5)

## 第二章 配置管理网络环境

本章介绍如何在opencloudos 上配置网络连接，opencloudos默认管理方式为NetworkManager.service，支持NetworkManager的所有配置方式。但是server端管理通过传统ifcfg配置的方式较为方便，这里介绍下ifcfg的配置方式。

### 2.1 静态网络配置

1. 确定配置网卡名字

/proc/net/dev 文件可查看网络设备

2. 为eth1配置ip

这里直接采用传统ifcfg配置方式，我们默认仅仅配置ip，不配置dns和gateway

直接编辑文件/etc/sysconfig/network-scripts/ifcfg-eth1
```
[root@VM-6-140-opencloudos /]# vim /etc/sysconfig/network-scripts/ifcfg-eth1

BOOTPROTO=static 
DEVICE=eth1

ONBOOT=yes

HWADDR=20:90:6F:EC:F4:52 # 网卡mac地址

TYPE=Ethernet

USERCTL=no # 是否允许非root用户控制设备

IPADDR=172.27.16.39 # 网卡ip地址

NETMASK=255.255.255.0 # 子网掩码
```

3) 重启网络
```
[root@OpencloudOS~]#systemctl restart NetworkManager.service
```

### 2.2 DHCP网络配置

1. 确定配置网卡名字

/proc/net/dev 文件可查看网络设备

2. 为eth1配置ip

这里直接采用传统ifcfg配置方式，方式为dhcp
```

BOOTPROTO=dhcp

DEVICE=eth1

HWADDR=20:90:6F:EC:F4:52

ONBOOT=yes

TYPE=Ethernet

USERCTL=no
```

3. 重启网络

```
[root@OpencloudOS~]#systemctl restart NetworkManager.service
```

- 更多帮助： man ifcfg

### 2.3 自定义DNS

编辑/etc/resolv.conf
```
[root@VM-6-140-opencloudos /]# vim /etc/resolv.conf

nameserver 183.60.83.19

nameserver 183.60.82.98
```

- 更多帮助 man(5) resolve.conf

## 第三章 systemd管理

通过systemd服务，用户可以使服务以一种优雅的方式实现自动启动。

### 3.1 服务启动设置

1. 服务自动启动
```
[root@OpencloudOS~]#systemctl enable service\name
```
2. 服务禁用自动启动
```
[root@OpencloudOS~]#systemctl disable service\name
```
注意：被mask的服务无法enable或者disable，需要先unmask，参考3.2 节。

### 3.2 服务屏蔽
```
[root@OpencloudOS~]#systemctl mask service\_name
[root@OpencloudOS~]#systemctl unmask service\_name
```
### 3.3 服务日常操作

1. 启动服务
```
[root@OpencloudOS~]#systemctl start service_name
```
2. 重启服务
```
[root@OpencloudOS~]#systemctl restart service_name
```
3. 查看服务状态
```
[root@OpencloudOS~]#systemctl status service_name
```
- 更多帮助： man systemctl

## 第四章 账户管理

opencloudos支持多账户管理，允许多个账户同时登陆一台系统进行操作。账户一般分为普通账户和系统账户。

- 普通账户

普通账户一般为特定的系统用户创建，在系统管理过程中可以被增加、删除、修改。主要用作用户登录操作。

- 系统账户

系统账户一般用作特定的应用，在应用程序安装时被创建，之后便不在修改

用户ID低于1000 为系统账户预留。高于1000的ID可以用作普通账户，但是比较推荐从5000开始分配普通账户。

- 组

一个组可以和多个账户绑定，进行统一的授权操作。

### 4.1 管理账户和组

- 查看用户和组ID
```
[root@VM-16-5-opencloudos ~]# id

uid=0(root) gid=0(root) groups=0(root)
```
- 创建用户
```
[root@OpencloudOS~]#adduser user_name
```
- 为用户设置密码
```
[root@OpencloudOS~]#passwd user_name
```
- 把用户加到用户组（如root用户）
```
[root@OpencloudOS~]#usermod -g root user_name
```
## 第五章 dump crash内核

### 5.1 kdump安装

kdump是在系统崩溃、死锁或死机时用来转储内存运行参数的一个工具和服务，是一种新的crash dump捕获机制，用来捕获kernel crash（内核崩溃）的时候产生的crash dump。

1. 安装kexec-tools
```
[root@OpencloudOS~]#yum install kexec-tools
```
2. 更新kexec-tools
```
[root@OpencloudOS~]#yum update kexec-tools
```
### 5.2 配置捕获内核大小

1. 编辑配置文件"/etc/default/grub",找到字段"crashkernel=",并根据以下参数配置
```
crashkernel=1800M-64G:256M，64G-128G:512M,128G-:768M
```
用户可以根据已安装的内存总量将保留内存量设置为变量。将内存保留到变量的语法是crashkernel=\<range1\>:\<size1\>，\<range2\>:\<size2\>。如上图所示。如果系统内存总量在1800MB至64GB之间，则上述示例会保留256MB的内存。

*需要注意的是，首次配置，需要重启预留内存，才能正常使用kdump

2. 更新grub2配置文件
```
[root@OpencloudOS~]#grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 5.3 配置捕获内核的存放位置

打开配置文件"/etc/kdump.conf"
```
[root@OpencloudOS~]#vim /etc/kdump.conf
```
默认存放地址为"/var/crash/"

- 使用NFS协议将捕获内核转储到远程计算机

将"nfs test.example.com:/export/tmp"前的"#"号删除，然后将地址信息正确配置

- 使用SSH协议将捕获内核转储到远处计算机

与上述NFS协议同理，将"ssh test.example.com"前的"#"号删除，然后将地址信息正确配置

### 5.4 触发内核崩溃

强制触发内核崩溃，测试kdump是否正常工作
```
[root@OpencloudOS~]#echo c \> /proc/sysrq-trigger
```
### 5.5 分析vmcore

在进行捕获内核分析前，需要准备:

- vmlinux调试镜像

该文件一般位于"/boot/"目录下对应的内核版本，文件命名格式如：

vmlinux-5.4.109-x-xx-xxx

若该目录下没有对应的调试镜像，则可在以下链接中下载对应内核版本的kernel-debuginfo工具包

http://mirrors.tencent.com/opencloudos/8.5/BaseOS/x86_64/debug/tree/Packages/
并使用如下命令进行安装，成功安装后在/boot/会生成对应的vmlinux镜像

并使用如下命令进行安装，成功安装后在/boot/会生成对应的vmlinux镜像
```
#rpm -ivh kernel-xxx-debuginfo-common-xxxx.rpm#rpm -ivh kernel-xxx-debuginfo-common-xxxx.rpm
```
- vmcore文件

在对应的目录（默认为/var/crash/）中找到vmcore文件

开始运行crash程序分析vmcore

```
[root@VM-6-140-opencloudos /]# ls /var/crash/127.0.0.1-2022-08-19-02\:34\:00/
kexec-dmesg.log  vmcore  vmcore-dmesg.txt

crash\> q //退出crash

crash\> log //显示消息缓存区

crash\> bt //显示内核崩溃前的函数调用栈

crash\> ps //显示内核崩溃前的进程状态

crash\> vm //显示内核崩溃前的虚拟内存信息

crash\> files //显示内核崩溃前的文件句柄信息
```

## 第六章 日志分析

内核及系统日志由系统服务rsyslog统一管理，根据其主配置文件/etc/rsyslog.conf中的设置决定将内核消息及各种系统程序消息记录到什么位置。

常用的系统日志文件如下表所示：

| 路径 | 说明 |
| --- | --- |
| /var/log/messages | 记录内核及各种应用程序的公共日志信息 |
| /var/log/cron | 记录crond计划任务产生的事件信息 |
| /var/log/dmesg | 记录操作系统在引导过程中的各种事件信息 |
| /var/log/mailog | 记录进入或发出系统的电子邮件活动 |
| /var/log/boot.log | 系统启动时日志，包括自启动服务 |
| /var/log/yum.log | 包含yum安装软件包的信息 |
| /var/log/audit/ | 包含audit daemon的审计日志 |
| /var/log/sa/ | 包含每日由sysstat软件包收集的sar文件 |

- 用户日志

用户日志用于记录操作系统用户登录及退出系统的相关信息，包括用户名、登录的终端、登录时间、来源主机、正在使用的进程操作等。

常用的用户日志目录：

| 路径 | 说明 |
| --- | --- |
| /var/log/lastlog | 记录每个用户最近的登录事件 |
| /var/log/secure | 记录用户认证相关的安全事件信息 |
| /var/log/wtmp | 记录每个用户登录、注销及系统启动和停机时间 |
| /var/log/btmp | 记录失败的、错误的登录尝试及验证事件 |

查看日志方法：
```
[root@VM-6-140-opencloudos /]#cat /var/log/xxx
```
其中xxx表示日志类型，如cat /var/log/lastlog

- 程序日志

通常应用程序会自己维护一份日志文件，用于记录本程序运行过程中的各种事件信息，由于各程序的日志管理和规则独立，导致不同程序所使用的日志记录格式存在较大差异。

- 服务日志

可以通过Journal处理由内核、initrd以及服务等产生的信息。通过journalctl工具，访问并操作journal内部的数据。如查看服务最新信息
```
[root@VM-6-140-opencloudos /]#journalctl -xe -u "service"
```


## 第七章 systemd机制

systemd是opencloudos系统上主要的系统守护进程管理工具，由于init一方面对于进程的管理是串行化的，容易出现阻塞情况，另一方面init也仅仅是执行启动脚本，并不能对服务本身进行更多的管理,所以由systemd取代了init作为默认的系统进程管理工具。

systemd所管理的所有系统资源都称作Unit，通过systemd命令集可以方便的对这些Unit进行管理。比如systemctl、hostnamectl、timedatectl、localctl等命令。

systemd的语法如下所示：

| systemctl [command] | [unit]（配置的应用名称,以nginx为例） |
| --- | --- |
| command可选项： |
| start：启动指定的unit | systemctl start nginx· |
| stop：关闭指定的unit | systemctl stop nginx |
| restart：重启指定unit | systemctl restart nginx |
| reload：重载指定unit | systemctl reload nginx |
| enable：系统开机时自动启动指定unit，前提是配置文件中有相关配置 | systemctl enable nginx |
| disable：开机时不自动运行指定unit | systemctl disable nginx |
| status：查看指定unit当前运行状态 | systemctl status nginx |

**systemd**  **配置文件说明**

每一个 Unit 都需要有一个配置文件用于告知 systemd 对于服务的管理方式

1. 配置文件存放于 /usr/lib/systemd/system/，设置开机启动后会在 /etc/systemd/system 目录建立软链接文件
2. 每个Unit的配置文件配置默认后缀名为.service
3. 在 /usr/lib/systemd/system/ 目录中分为 system 和 user 两个目录，一般将开机不登陆就能运行的程序存在系统服务里，也就是 /usr/lib/systemd/system
4. 配置文件使用方括号分成了多个部分，并且区分大小写
<br />

## 第八章 chrony机制

chrony 是网络时间协议 (NTP) 的通用实现。它可以将系统时钟与 NTP 服务器、参考时钟（例如 GPS 接收器）同步。它还可以作为 NTPv4 (RFC 5905) 服务器和对等点运行，为网络中的其他计算机提供时间服务。chronyd 是一个可以在引导时启动的守护程序。

### 8.1 安装配置

1. 安装chrony
```
[root@VM-6-140-opencloudos /]#yum install chrony
```
2. 启动chrony
```
[root@VM-6-140-opencloudos /]#systemctl start chronyd

[root@VM-6-140-opencloudos /]#systemctl enable chronyd
```

3. 查看chrony状态

```
[root@VM-6-140-opencloudos /]#systemctl status chronyd
```

### 8.2 添加时间源

1. 打开配置文件
```
[root@VM-6-140-opencloudos /]#vim /etc/chrony.conf
```
2. 添加时间源参数

在/etc/chrony.conf配置文件中添加以下参数：
```
[root@VM-6-140-opencloudos /]#server time.tencentyun.com iburst
```
3. 查看时间同步源状态
```
[root@VM-6-130-opencloudos ~]# chronyc sources -v

^+ 169.254.0.79 2 10 377 688 +149us[+281us] +/- 24ms

^\* 169.254.0.80 2 10 377 29 -380us[-245us] +/- 39ms

^+ 169.254.0.81 2 10 377 691 +31us[+162us] +/- 27ms

^+ 169.254.0.82 2 10 377 613 +158us[+290us] +/- 28ms

^+ 169.254.0.83 2 10 377 767 +168us[+299us] +/- 37ms
```


### 8.3 配置生效

重启chrony，使配置生效
```
[root@VM-6-140-opencloudos /]#systemctl restart chronyd
```

查看时间同步源状态
```
[root@VM-6-130-opencloudos ~]# chronyc sourcestats -v

169.254.0.79 6 3 86m +0.081 0.289 +191us 178us

169.254.0.80 24 14 396m -0.015 0.017 -235us 156us

169.254.0.81 6 3 86m +0.044 0.257 +62us 140us

169.254.0.82 6 3 86m -0.062 0.420 -118us 142us

169.254.0.83 9 8 137m +0.031 0.035 +165us 47us
```

## 第九章 bond配置

bond技术是将多块物理网卡绑定同一IP地址对外提供服务，通过不同的模式配置，从而达到高可用、负载均衡及链路冗余等效果。

### 9.1 bond的模式

bond具有7种模式，如下表所示

| 模式 | 说明 |
| --- | --- |
| mode 0 | load balancing (round-robin)模式，多张网卡同时工作 |
| mode 1 | fault-tolerance (active-backup)提供冗余功能，工作方式是主备的工作方式 |
| mode 2 | balance-x，提供负载均衡和冗余功能 |
| mode 3 | 表示broadcast，该模式提供容错性 |
| mode 4 | 表示802.3ad，表示支持802.3ad协议，和交换机的聚合LACP方式配合(需要xmit\_hash\_policy) |
| mode 5 | 表示balance-tlb，自动适应负载均衡，自动切换故障。在此基础上Ethtool支持驱动 |
| mode 6 | 表示在5模式的基础上优化了arp的广播信息 |

其中以mode4为常用模式

### 9.2 配置bond

1. 选择2块需要绑定的网卡进行配置。例：eth0/eth1，则eth0在/etc/sysconfig/network-scripts/目录下的配置文件ifcfg-eth0的参数表示如下：
```
[root@VM-6-130-opencloudos ~]# /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0 #网口名：eth0

NAME=eth0

TYPE=Ethernet #网口类型：以太网接口

ONBOOT=yes #系统启动时网口状态为激活

BOOTPROTO=none #网口激活协议：nono不适用任何协议 static：设置静态ip dhcp：设置动态获取ip

MASTER=bond\_test #指定虚拟网口的名字

SLAVE=yes #备用（从设备）
```

2. eth1在/etc/sysconfig/network-scripts/目录下的配置文件ifcfg-eth1的参数表示如下：
```
[root@VM-6-130-opencloudos ~]# /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1 #网口名：eth1

NAME=eth1

TYPE=Ethernet #网口类型：以太网接口

ONBOOT=yes #系统启动时网口状态为激活

BOOTPROTO=none #网口激活协议：nono不适用任何协议 static：设置静态ip dhcp：设置动态获取ip

MASTER=bond\_test #指定虚拟网口的名字

SLAVE=yes #备用（从设备）
```

3. 在/etc/sysconfig/network-scripts/ifcfg-bond\_test目录下配置bond\_test网卡
```
[root@VM-6-130-opencloudos ~]# /etc/sysconfig/network-scripts/ifcfg-bond_test

DEVICE=bond\_test #网口名：bond\_test

NAME=bond\_test

TYPE=Ethernet #网口类型：以太网接口

ONBOOT=yes #系统启动时网口状态为激活

DELAY=0

NM\_CONTROLLED=yes

BOOTPROTO=static #网口激活协议：nono不适用任何协议 static：设置静态ip dhcp：设置动态获取ip

IPADDR='9.27.148.124' #ip地址

NETMASK='255.255.255.192' #子网掩码

GATEWAY='9.27.148.65' #网关

BONDING\_OPTS='mode=4 miimon=100 lacp\_rate=fast xmit\_hash\_policy=1'
```

其中miimon表示链路监测：miimon=100表示系统每100ms监测一次链路状态，如果有一条线路不通就转入另一条线路

4. 配置bonding
```
[root@VM-6-140-opencloudos /]#vim /etc/modprobe.d/dist.conf
alias bond\_test bonding
```
5. 查看当前使用网口
```
[root@VM-6-140-opencloudos /]#cat /proc/net/bonding/bond_test
```
<br />


## 第十章 LVM管理

　　LVM（Logical Volume Manager）逻辑卷管理，是在硬盘分区和文件系统之间添加的一个逻辑层，为文件系统屏蔽下层硬盘分区布局，并提供一个抽象的盘卷，在盘卷上建立文件系统。系统管理员利用LVM可以在硬盘不用重新分区的情况下动态调整文件系统的大小，并且利用LVM管理的文件系统可以跨越物理硬盘。当服务器添加了新的硬盘后，管理员不必将原有的文件移动到新的硬盘上，而是通过LVM直接扩展文件系统来跨越物理硬盘。

### 10.1 基础概念

物理卷(PV)：把常规的块设备（硬盘，分区等可以读写数据的设备）通过pvcreate命令对其进行初始化，形成物理卷

卷组(VG):把多个物理卷的容量组成一个逻辑整体，可以从里面灵活分配容量

逻辑卷(LV):从卷组中划分部分空间成为一个可以读写数据的逻辑单元，需要对其格式化然后挂载使用

### 10.2 部署lvm

1. 添加物理磁盘，创建物理卷
```

[root@VM-16-5-opencloudos ~]# lsblk | grep "vd[bcd]"

vdb 253:0 0 50G 0 disk

vdc 253:16 0 50G 0 disk

vdd 253:32 0 50G 0 disk
```
2. 将磁盘加入pv

```

[root@VM-16-5-opencloudos ~]# pvcreate /dev/vdb

Physical volume "/dev/sdb" successfully created.//检查pv创建情况

[root@VM-16-5-opencloudos ~]# pvs

PV VG Fmt Attr PSize PFree

/dev/vdb lvm2 --- 50.00g 50.00g
```

3. 创建名为datavg的卷组

```

[root@VM-16-5-opencloudos ~]# vgcreate datavg /dev/vdb

Volume group "datavg" successfully created

[root@VM-16-5-opencloudos ~]# vgs

VG #PV #LV #SN Attr VSize VFree

datavg 1 0 0 wz--n- \<50.00g \<50.00g
```

4. 创建逻辑卷，分配名称，以及大小，指定卷组

```
[root@VM-16-5-opencloudos ~]# lvcreate -L 100M -n lv1 datavg

Wiping ext4 signature on /dev/datavg/lv1.

Logical volume "lv1" created.

[root@VM-16-5-opencloudos ~]# lvscan

ACTIVE '/dev/datavg/lv1' [100.00 MiB] inherit
```

5. 格式化文件系统

```
[root@VM-16-5-opencloudos ~]# mkfs.ext4 /dev/datavg/lv1

Allocating group tables: done

Writing inode tables: done

Creating journal (4096 blocks): done

Writing superblocks and filesystem accounting information: done
```

6. 挂载并使用

```
[root@VM-16-5-opencloudos ~]#mkdir /lv1

[root@VM-16-5-opencloudos ~]#mount /dev/datavg/lv1 /lv1/

[root@VM-16-5-opencloudos ~]#df -hl

Filesystem Size Used Avail Use% Mounted on

/dev/mapper/datavg-lv1 93M 1.6M 85M 2% /lv1
```

### 10.3 卷组管理

- 扩展卷组，将新磁盘加入卷组

1. 新磁盘加入pv
```
[root@VM-6-140-opencloudos /]#pvcreate /dev/vdc
```
2. 使用vgextend扩展
```
[root@VM-6-140-opencloudos /]#vgextend datavg /dev/vdc
```

- 缩减卷组
```
[root@VM-6-140-opencloudos /]#vgreduce datavg /dev/vdb
```
- 数据迁移卷组

只有同一卷组的磁盘才能够进行在线迁移

1. 检查当前逻辑卷VG中PV使用情况
```
[root@VM-16-5-opencloudos ~]#pvs

PV VG Fmt Attr PSize PFree

/dev/vdb vg1 lvm2 a -- 2.00g 1.76g

/dev/vdc vg1 lvm2 a -- 2.00g 2.00g
```

2. pvmove在线将/dev/vdb数据迁移至/dev/vdc

```
[root@VM-16-5-opencloudos ~]#pvmove /dev/vdb /dev/vdc
```

3) 查看是否迁移成功
```
[root@VM-16-5-opencloudos ~]#pvs

/dev/vdb vg1 lvm2 a -- 2.00g 2.00g

/dev/vdc vg1 lvm2 a -- 2.00g 1.76g
```
- 删除卷组
```
[root@VM-16-5-opencloudos ~]#vgremove vgname
```
删除卷组之前请确保卷组内的逻辑卷未被挂载

### 10.4 逻辑卷管理

- 逻辑卷扩展

逻辑卷的扩展取决于卷组中的容量，逻辑卷扩展的容量不能超过卷组的容量

1. 为逻辑卷增加1G容量
````
[root@VM-16-5-opencloudos ~]#lvextend -L +1G /dev/datavg/lv1 (注意：1G与+1G意义不同)
````
2. 扩展文件系统

xfs\_growfs /dev/datavg/lv1 //xfs文件系统扩容

resize2fs /dev/datavg/lv1//ext文件系统扩容

- 对ext格式的逻辑卷裁剪容量

以裁剪逻辑卷512M容量为例，操作步骤如下：

1. 首先创建一个1G的逻辑卷作为被裁剪的对象
```
[root@VM-16-5-opencloudos ~]#lvcreate -n rm_test -L 1G datavg //datavg为卷组

[root@VM-16-5-opencloudos ~]#mkfs.ext4 /dev/datavg/rm_test

[root@VM-16-5-opencloudos ~]#mkdir -p /rm_test

[root@VM-16-5-opencloudos ~]#mount /dev/datavg/rm_test /rm_test/
```
2. 如果已经挂载，必须先卸载
```
[root@VM-16-5-opencloudos ~]#umount /dev/datavg/rm_test
```
3. 裁剪容量，必须先检测文件系统
```
[root@VM-16-5-opencloudos ~]#e2fsck -f /dev/datavg/rm_test

[root@VM-16-5-opencloudos ~]#resize2fs /dev/datavg/rm_test 512M
```
4. 调整完毕后，裁剪逻辑卷容量
```
[root@VM-16-5-opencloudos ~]#lvreduce -L 512M /dev/datavg/rm_test
```
5. 挂载测试，若能成功挂载，则文件系统未被损坏
```
[root@VM-16-5-opencloudos ~]#mount /dev/datavg/rm_test
```
- 删除逻辑卷

1. 确保被删除的逻辑卷未使用
#umount /dev/卷组名/逻辑卷名
```
[root@VM-16-5-opencloudos ~]#umount /dev/datavg/rm_test
```
2. 删除逻辑卷

- lvremove <volume_group>/<logical_volume>
```
[root@VM-16-5-opencloudos ~]#lvremove /dev/datavg/rm_test
```
#

## 第十一章 软RAID管理

RAID(redundant array of independent disks)磁盘冗余阵列，通过把多个硬盘设备组合成一个容量更大、安全性更好的磁盘阵列，并把数据切割成多个段后分别存放在各个不同的物理硬盘设备上，然后利用分散读写技术来提升磁盘阵列整体的性能，同时把多个重要数据的副本同步到不同的物理硬盘设备上，从而起到了非常好的数据冗余备份效果和容错能力。

RAID根据硬件的使用与否分为软RAID和硬RAID

软RAID：无独立的RAID控制卡，由操作系统和CPU来实现所有的RAID功能

### 11.1 RAID逻辑分类

- RAID0

数据条带化，无校验，不提供数据保护。数据并发写入多个硬盘。

优点：1.所有RAID中读写性能最高。2.100%的磁盘空间利用率

缺点：不提供数据冗余保护，一旦数据损坏，将无法恢复。

使用场景：RAID 0适用于迅速读写，但对数据安全性和可靠性要求不高的场景，如视频、打印等。

- RAID1

数据镜像，无校验。一半的空间存储冗余数据，所有RAID中数据安全性最高。

优点：1.所有的RAID中安全性最高，即使有一半的磁盘发生故障，仍能正常运转。2.镜像磁盘没有全部故障，数据就不会丢失。

缺点：1.磁盘空间利用率为50%，一半的空间用于存储冗余数据。2.成本高。

- RAID5

数据条带化，校验数据（1组）均匀分布在每个物理磁盘上。当某个物理磁盘发生故障时，可根据同一条带的其他数据块和对应的校验数据来重建损坏的数据。

优点：1.允许1个物理磁盘发生故障，而不丢失数据。2.读取性能相对高，磁盘空间利用率大于RAID 10。

缺点：1.写入性能相对低。2.重建数据时，性能会受到较大的影响。

- RAID10

RAID 1与RAID 0的结合，从磁盘的角度看，先将四块磁盘两两分组形成两组镜像对，组内先使用RAID1模式用于数据备份，组间使用RAID0进行数据拆分，提高读写速率。相对于RAID0，RAID1使用更多的磁盘。

优点：1.读取性能仅次于RAID 0。2.镜像对中的磁盘没有全部故障，数据就不会丢失。3.一半的物理磁盘发生故障时，仍可正常运转。

缺点：1.成本高。2.磁盘空间利用率50%，一半的空间用于存储冗余数据。

- RAID6

数据条带化，校验数据（2组）均匀分布在每个物理磁盘上。即使有两个磁盘同时故障，也可通过2组校验数据来重建两个磁盘上损坏的数据。

优点：1.允许2个物理磁盘发生故障，而不丢失数据。2.读取性能较高，磁盘空间利用率大于RAID 10。

缺点：成本高于RAID 5，写入性能较低（低于RAID 5）。

- RAID50

RAID 5与RAID 0的结合，先创建RAID 5，再创建RAID 0。有效提升了RAID 5的性能。将作为组成部分的磁盘划分为若干完全相同的RAID 5。配置RAID 50至少需要6个磁盘，划分为2个RAID 5，每组有3个磁盘。

优点：1.读写性能高于RAID 5。2.容错能力高于RAID 0或RAID 5。3.发生故障的磁盘在不同的RAID 5中，最多允许n个物理磁盘发生故障（n为RAID 5的数量）而不丢失数据。

缺点：1.重建故障磁盘时，如果同一RAID 5中又有磁盘发生故障，则会丢失所有数据。2.磁盘中需要更多的空间存储校验数据。

- RAID60

RAID 6与RAID 0的结合，先创建RAID 6，再创建RAID 0。有效提升了RAID6的性能。将作为组成部分的磁盘划分为若干完全相同的RAID 6。配置RAID 60 至少需要8个磁盘，划分为两个RAID 6，每组有4个磁盘。

优点：1.读写性能高于RAID 6。2.容错能力高于RAID 0或RAID 6。3.同一RAID 6中发生故障的磁盘不超过两个，最多可允许2n个物理磁盘发生故障（n为RAID 6的数量）而不丢失数据。

缺点：1.重建故障磁盘时，如果同一RAID 6中又有第三个磁盘发生故障，则会丢失所有数据。2.磁盘中需要更多的空间存储校验数据。

RAID模式的选择如下表所示：

| RAID级别 | 容错性 | 硬盘数 | 可用容量 | 允许故障硬盘数 | 使用场景 |
| --- | --- | --- | --- | --- | --- |
| RAID 0 | 无 | N\>=2 | 全部 | 0 | 高读写，无保护 |
| RAID 1 | 有 | N\>=2且N%2=0 | 一半 | 一半 | 读写速率不变，高数据保护 |
| RAID 5 | 有 | N\>=3 | （N-1）\*单块磁盘容量 | 1 | 随机数据传输，安全要求高 |
| RAID 6 | 有 | N\>=4 | （N-2）\*单块磁盘容量 | 2 | 使用较少 |
| RAID 10 | 有 | N\>=4且N%2=0 | 一半 | 一半 | 高读写，高保护 |
| RAID 50 | 有 | N\>=6 | 每个RAID5中有1块磁盘用于存储校验数据 | 每个RAID5允许1块磁盘故障 | 使用较少 |
| RAID 60 | 有 | N\>=8 | 每个RAID6中有2块磁盘用于存储校验数据 | 每个RAID6允许2块磁盘故障 | 使用较少 |

### 11.2 软RAID10部署

Linux内核中有一个md(multiple devices)模块在底层管理RAID设备，它会在应用层为用户提供一个应用程序的工具mdadm ，mdadm是linux下用于创建和管理软件RAID的命令。

本文档将磁盘/dev/vdb/分四个分区作为四个磁盘搭建软RAID10；

1. 创建raid10
```
[root@VM-16-5-opencloudos ~]#mdadm -C -v /dev/md10 -l 10 -n 4 /dev/vdb[1-4]

mdadm: layout defaults to n2

mdadm: layout defaults to n2

mdadm: chunk size defaults to 512K

mdadm: size set to 5237760K

mdadm: Defaulting to version 1.2 metadata

mdadm: array /dev/md10 started.
```

参数说明：

-C 创建

-v 可视化

-l 指定RAID级别

-n 指定设备数量


2. 查看保存配置
```
[root@VM-16-5-opencloudos ~]#mdadm -D /dev/md10

[root@VM-16-5-opencloudos ~]#mdadm -Dsv \> /etc/mdadm.conf
```
3. 查看阵列信息
```
[root@VM-16-5-opencloudos ~]#cat /proc/mdstat
```
4. 将md10作为一个整体磁盘，格式化并挂载
```
[root@VM-16-5-opencloudos ~]#mkfs.ext4 /dev/md10

[root@VM-16-5-opencloudos ~]#mount -t ext4 /dev/md10 /mnt/
```
至此，磁盘阵列已可用

### 11.3 卸载RAID阵列

以上述11.2节创建的RAID为例，执行以下命令卸载磁盘阵列
```
[root@VM-16-5-opencloudos ~]#umount /dev/md10

[root@VM-16-5-opencloudos ~]#mdadm -S /dev/md10

[root@VM-16-5-opencloudos ~]#mdadm --misc --zero-superblock /dev/vdb[1-4]
```


## 第十二章 分区管理

### 12.1 磁盘分区基础

- 基本分区（primary partion）

基本分区也称主分区，引导分区、每块磁盘分区主分区与扩展分区加起来不能大于四个。

基本分区创建后可以立即使用，但是有分区数量上限。

- 扩充分区(extension partion)

每块磁盘内只能划分一块扩展分区，扩展分区内可划分任意块逻辑分区

扩展分区创建后不能直接使用，需要在扩展分区内创建逻辑分区

- 逻辑分区（logical partion）

逻辑分区是在扩展分区内创建的分区

逻辑分区相当于一块存储介质，和其他逻辑分区主分区完全独立

磁盘分区主要借助fdisk -l工具

### 12.2 创建MBR分区

选择一块需要分区的磁盘，以/dev/vdb/为例，使用fdisk工具创建新分区
```
[root@VM-6-130-opencloudos /]# fdisk /dev/vdb

Command (m for help): n //创建一个新的分区

Partition type

p primary (0 primary, 0 extended, 4 free)

e extended (container for logical partitions)

Select (default p): p //选择分区类型，以主分区为例

Partition number (1-4, default 1):1 //选择分区号

First sector (2048-104857599, default 2048): 2048 //扇区起始地址

Last sector, +sectors or +size{K,M,G,T,P} (2048-104857599, default 104857599):+5G //指定扇区容量

Created a new partition 1 of type 'Linux' and of size 5 GiB.

Command (m for help): wq //保存修改并退出
```
至此，成功创建新分区！

其他操作参数如下：

| fdisk     -l | 查看，列出磁盘信息 |
| --- | --- |
| fdisk    /dev/vdb | 进行操作 |
| p | 列出磁盘现有分区情况，分区类型为主分区 |
| e | 分区类型为扩展分区 |
| n | 添加新的磁盘分区 |
| d | 删除现有磁盘分区 |
| w | 对分区操作进行保存 |
| q | 放弃更改退出 |
| t | 改变分区的id，即修改分区功能id |
| l | 列出已有分区类型 |

### 12.3 格式化分区

需要将新创建的分区格式化对应的文件系统,以ext4格式为例
```
[root@VM-16-5-opencloudos ~]#mkfs.ext4 /dev/vdb1
```
### 12.4 挂载/卸载分区

需要将格式化后的分区挂载到指定目录后方可使用，如将分区挂载到/mnt/分区下
```
[root@VM-16-5-opencloudos ~]#mount -t ext4 /dev/vdb1
```
分区不用后可以将该分区卸载
```
[root@VM-16-5-opencloudos ~]#umount /dev/vdb1
```
### 12.5 创建GPT分区

fdisk 命令用于创建和维护磁盘分区，而且fdisk只能对小于2TB的硬盘进行分区，对于大于2TB的硬盘，需要使用parted工具分区，使用fdisk创建分区，只能创建MBR分区方案。

1. 进入parted交互模式

```

[root@VM-6-130-opencloudos /]# parted

(parted) select /dev/vdb //选择磁盘/dev/vdb

(parted) print free //查看磁盘信息
```
2. 创建一个新分区
```
(parted) mkpart

Partition type? primary/extended? primary

File system type? [ext2]? ext4

Start? 1

End? 5G

(parted)quit
```

3. 将磁盘格式化为GPT磁盘
```
(parted) mklabel gpt
```
### 12.6 调整分区大小

parted可以调整分区的大小，注意，parted 调整已经挂载使用的分区时，不会影响分区中的数据的。但是一定要先卸载分区，再调整分区大小，否则数据会出现问题。另外，要调整大小的分区必须已经建立了文件系统（格式化），否则会报错。

以调整/dev/vdb1分区的大小为例

1. 卸载该分区
```
[root@VM-16-5-opencloudos ~]#umount /dev/vdb1
```
2. 进入parted交互模式

执行以下命令
```
[root@VM-6-130-opencloudos /]# parted /dev/vdb

(parted) resizepart

Partition number? 1

End? 10G

(parted) quit
```
### 12.7 删除GPT分区

1. 卸载该分区
```
[root@VM-16-5-opencloudos ~]#umount /dev/vdb1
```
2. 进入parted交互模式

执行以下命令
```
[root@VM-6-130-opencloudos /]# parted /dev/vdb

(parted) rm

Partition number? 1

(parted) quit
```
要注意的是，parted 中所有的操作都是立即生效的，没有保存生效的概念。这一点和 fdisk 交互命令明显不同，所以所有操作要加倍小心。å