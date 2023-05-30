# <center> OpenCloudOS-Basic Configuration

## 1 Initial Environment Setup

The basic environment configuration is part of the installation process. The following describes how to modify the basic configuration at the OS level after the system is installed. The basic environment configuration includes:

- time and date
- system locale

### 1.1 Configure time and date

Precise timing is critical in production systems. Under OpenCloudOS, the time accuracy is guaranteed through the NTP protocol. The NTP protocol is implemented through a daemon process in user mode, which updates the system clock in the kernel, and the system clock can use different clock sources to maintain time.
```
[root@VM-6-130-opencloudos /]# date

Tue Aug 2 19:44:17 CST 2022
```

To see more information, use the timedatectl command:
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

- **More help:** man date(1) / man timedatectl(1)

### 1.2 Configure system locale

System-wide locales are configured in the /etc/local/conf file, which is read during systemd's initial startup. Each service or user will inherit the configuration here by default. Of course, some programs and users can also customize and modify the configuration.

View available locale settings: localectl list-locales
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

View the current system locale settings
```
[root@VM-6-130-opencloudos /]# localectl status

System Locale: LANG=en\_US.UTF-8

VC Keymap: us

X11 Layout: us
```

- More help: man localectl(1) man locale(7) man locale.conf(5)

## 2 Configuration and Management Network Environment

This chapter introduces how to configure network connections on OpenCloudOS. The default management method of OpenCloudOS is NetworkManager.service, which supports all configuration methods of NetworkManager. However, the server-side management is more convenient through the traditional ifcfg configuration method. Here is an introduction to the ifcfg configuration method.

### 2.1 Static network configuration

1. Determine the configuration network card name

    `/proc/net/dev` file to view network devices

2. Configure ip for eth1

Here we directly adopt the traditional ifcfg configuration method, we only configure ip by default, and do not configure dns and gateway

Edit the file `/etc/sysconfig/network-scripts/ifcfg-eth1` directly
```
[root@VM-6-140-opencloudos /]# vim /etc/sysconfig/network-scripts/ifcfg-eth1

BOOTPROTO=static 
DEVICE=eth1

ONBOOT=yes

HWADDR=20:90:6F:EC:F4:52 # Network card mac address

TYPE=Ethernet

USERCTL=no # Whether to allow non-root users to control the device

IPADDR=172.27.16.39 # network card ip address

NETMASK=255.255.255.0 # subnet mask
```

3. Restart the network
```
[root@OpencloudOS~]#systemctl restart NetworkManager.service
```

### 2.2 DHCP network configuration

1. Determine the configuration network card name

    `/proc/net/dev` file to view network devices

2. Configure ip for eth1

Here directly adopt the traditional ifcfg configuration method, the method is dhcp
```
BOOTPROTO=dhcp

DEVICE=eth1

HWADDR=20:90:6F:EC:F4:52

ONBOOT=yes

TYPE=Ethernet

USERCTL=no
```

3. Restart the network

```
[root@OpencloudOS~]#systemctl restart NetworkManager.service
```

- For more help: man ifcfg

### 2.3 Custom DNS

edit /etc/resolv.conf
```
[root@VM-6-140-opencloudos /]# vim /etc/resolv.conf

nameserver 183.60.83.19

nameserver 183.60.82.98
```

- More help: man(5) resolve.conf

## 3 Systemd management

Through systemd services, users can enable services to automatically start in an elegant way.

### 3.1 Service startup settings

1. The service starts automatically
```
[root@OpencloudOS~]#systemctl enable service\name
```
2. Service disabled autostart
```
[root@OpencloudOS~]#systemctl disable service\name
```
Note: The masked service cannot be enabled or disabled, it needs to be unmask first, refer to section 3.2.

### 3.2 Service shielding
```
[root@OpencloudOS~]#systemctl mask service\_name
[root@OpencloudOS~]#systemctl unmask service\_name
```
### 3.3 Service Daily Operation

1. Start the service
```
[root@OpencloudOS~]#systemctl start service_name
```
2. restart service
```
[root@OpencloudOS~]#systemctl restart service_name
```
3. View service status
```
[root@OpencloudOS~]#systemctl status service_name
```
- More help: man systemctl

## 4 Account Management

OpenCloudOS supports multi-account management, allowing multiple accounts to log in to one system at the same time for operation. Accounts are generally divided into ordinary accounts and system accounts.

- Ordinary account

Ordinary accounts are generally created for specific system users, and can be added, deleted, and modified during system management. Mainly used for user login operations.

- system account

System accounts are generally used for specific applications, created when the application is installed, and are not modified afterwards

User IDs below 1000 are reserved for system accounts. IDs higher than 1000 can be used as ordinary accounts, but it is recommended to allocate ordinary accounts starting from 5000.

- Group

A group can be bound to multiple accounts for unified authorization operations.

### 4.1 Manage Accounts and Groups

- View user and group IDs
```
[root@VM-16-5-opencloudos ~]# id

uid=0(root) gid=0(root) groups=0(root)
```
- create user
```
[root@OpencloudOS~]#adduser user_name
```
- Set a password for the user
```
[root@OpencloudOS~]#passwd user_name
```
- Add the user to the user group (such as the root user)
```
[root@OpencloudOS~]#usermod -g root user_name
```
## 5 Dump crash kernel

### 5.1 Kdump installation

kdump is a tool and service used to dump memory operating parameters when the system crashes, deadlocks or crashes. It is a new crash dump capture mechanism used to capture crash dumps generated during kernel crashes.

1. Install kexec-tools
```
[root@OpencloudOS~]#yum install kexec-tools
```
2. Update kexec-tools
```
[root@OpencloudOS~]#yum update kexec-tools
```
### 5.2 Configure capture kernel size

1. Edit the configuration file "/etc/default/grub", find the field "crashkernel=", and configure it according to the following parameters
```
crashkernel=1800M-64G:256M, 64G-128G:512M,128G-:768M
```
The user can set the amount of reserved memory as a variable based on the total amount of memory installed. The syntax to reserve memory to a variable is crashkernel=\<range1\>:\<size1\>, \<range2\>:\<size2\>. As shown in FIG. If the total system memory is between 1800MB and 64GB, the above example reserves 256MB of memory.

*It should be noted that for the first configuration, the reserved memory needs to be restarted to use kdump normally*

2. Update the grub2 configuration file
```
[root@OpencloudOS~]#grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 5.3 Configure the storage location of the capture kernel

Open the configuration file "/etc/kdump.conf"
```
[root@OpencloudOS~]#vim /etc/kdump.conf
```
The default storage address is "/var/crash/"

- Capture kernel dumps to a remote computer using the NFS protocol

Delete the "#" before "nfs test.example.com:/export/tmp", and then configure the address information correctly

- Use the SSH protocol to capture kernel dumps to remote computers

Similar to the above NFS protocol, delete the "#" before "ssh test.example.com", and then configure the address information correctly

### 5.4 Trigger a kernel panic

Force a kernel panic to test if kdump is working
```
[root@OpencloudOS~]#echo c > /proc/sysrq-trigger
```
### 5.5 Analyze vmcore

Before capturing the kernel analysis, you need to prepare:

- vmlinux debug image

The file is generally located in the "/boot/" directory corresponding to the kernel version, and the file naming format is as follows:

- vmlinux-5.4.109-x-xx-xxx

If there is no corresponding debug image in this directory, you can download the kernel-debuginfo toolkit corresponding to the kernel version from the link below

http://mirrors.tencent.com/opencloudos/8.5/BaseOS/x86_64/debug/tree/Packages/
And use the following command to install, after successful installation, the corresponding vmlinux image will be generated in /boot/

And use the following command to install, after successful installation, the corresponding vmlinux image will be generated in /boot/
```
#rpm -ivh kernel-xxx-debuginfo-common-xxxx.rpm#rpm -ivh kernel-xxx-debuginfo-common-xxxx.rpm
```
- vmcore file

Find the vmcore file in the corresponding directory (default is /var/crash/)

Start running the crash program to analyze vmcore

```
[root@VM-6-140-opencloudos /]# ls /var/crash/127.0.0.1-2022-08-19-02\:34\:00/
kexec-dmesg.log vmcore vmcore-dmesg.txt

crash\> q // exit crash

crash\> log //display message buffer

crash\> bt //Display the function call stack before the kernel crash

crash\> ps //Display the process status before the kernel crash

crash\> vm //Display the virtual memory information before the kernel crash

crash\> files //Display the file handle information before the kernel crash
```

## 6 Log Analysis

The kernel and system logs are uniformly managed by the system service rsyslog, which determines where to record kernel messages and various system program messages according to the settings in its main configuration file /etc/rsyslog.conf.

Commonly used system log files are shown in the following table:

| Path | Description |
| --- | --- |
| /var/log/messages | Records common log messages for the kernel and various applications |
| /var/log/cron | Record event information generated by crond scheduled tasks |
| /var/log/dmesg | Record various event information during the boot process of the operating system |
| /var/log/mailog | Logs email activity into or out of the system |
| /var/log/boot.log | Logs at system startup, including self-starting services |
| /var/log/yum.log | Contains information about yum installed packages |
| /var/log/audit/ | Contains the audit log of the audit daemon |
| /var/log/sa/ | Contains daily sar files collected by the sysstat package |

- User logs

User logs are used to record relevant information about operating system users logging in and out of the system, including user names, logged-in terminals, log-in time, source hosts, and process operations in use, etc.

Commonly used user log directories:

| Path | Description |
| --- | --- |
| /var/log/lastlog | Logs the last login events for each user |
| /var/log/secure | Record security event information related to user authentication |
| /var/log/wtmp | Logs every user login, logout, and system startup and downtime |
| /var/log/btmp | Logs failed, bad login attempts and authentication events |

View log method:
```
[root@VM-6-140-opencloudos /]#cat /var/log/xxx
```
Where xxx represents the log type, such as cat /var/log/lastlog

- program log

Usually, the application maintains a log file by itself to record various event information during the running of the program. Since the log management and rules of each program are independent, the log record formats used by different programs are quite different.

- Service log

Information generated by the kernel, initrd, and services can be processed through the Journal. Through the journalctl tool, access and manipulate the data inside the journal. For example, check the latest service information
```
[root@VM-6-140-opencloudos /]#journalctl -xe -u "service"
```


## 7 Systemd mechanism

systemd is the main system daemon process management tool on the OpenCloudOS system. On the one hand, init manages the process serially, which is prone to blockage. On the other hand, init only executes the startup script and cannot update the service itself. More management, so systemd replaced init as the default system process management tool.

All system resources managed by systemd are called Units, and these Units can be easily managed through the systemd command set. Such as systemctl, hostnamectl, timedatectl, localctl and other commands.

The syntax for systemd is as follows:

| systemctl [command] | [unit] (configured application name, take nginx as an example) |
| --- | --- |
| command options: |
| start: start the specified unit | systemctl start nginx|
| stop: close the specified unit | systemctl stop nginx |
| restart: restart the specified unit | systemctl restart nginx |
| reload: reload the specified unit | systemctl reload nginx |
| enable: Automatically start the specified unit when the system is turned on, provided that there are related configurations in the configuration file | systemctl enable nginx |
| disable: do not automatically run the specified unit when booting | systemctl disable nginx |
| status: view the current running status of the specified unit | systemctl status nginx |

**Systemd**  **Configuration file description**

Each Unit needs to have a configuration file to tell systemd how to manage the service

1. The configuration file is stored in /usr/lib/systemd/system/, and a soft link file will be created in the /etc/systemd/system directory after setting the boot
2. The configuration file configuration default suffix of each Unit is .service
3. In the /usr/lib/systemd/system/ directory, it is divided into two directories: system and user. Generally, the programs that can be run without logging in are stored in the system service, that is, /usr/lib/systemd/system
4. The configuration file is divided into multiple parts using square brackets and is case-sensitive
<br />

## 8 Chrony mechanism

chrony is a generic implementation of the Network Time Protocol (NTP). It can synchronize the system clock with NTP servers, reference clocks such as GPS receivers. It can also operate as an NTPv4 (RFC 5905) server and peer, providing time services to other computers on the network. chronyd is a daemon that can be started at boot time.

### 8.1 Installation configuration

1. Install chrony
```
[root@VM-6-140-opencloudos /]#yum install chrony
```
2. Start chrony
```
[root@VM-6-140-opencloudos /]#systemctl start chronyd

[root@VM-6-140-opencloudos /]#systemctl enable chronyd
```

3. View chrony status

```
[root@VM-6-140-opencloudos /]#systemctl status chronyd
```

### 8.2 Add time source

1. Open the configuration file
```
[root@VM-6-140-opencloudos /]#vim /etc/chrony.conf
```
2. Add time source parameters

Add the following parameters to the /etc/chrony.conf configuration file:
```
[root@VM-6-140-opencloudos /]#server time.tencentyun.com iburst
```
3. View the status of the time synchronization source
```
[root@VM-6-130-opencloudos ~]# chronyc sources -v

^+ 169.254.0.79 2 10 377 688 +149us[+281us] +/- 24ms

^\* 169.254.0.80 2 10 377 29 -380us[-245us] +/- 39ms

^+ 169.254.0.81 2 10 377 691 +31us[+162us] +/- 27ms

^+ 169.254.0.82 2 10 377 613 +158us[+290us] +/- 28ms

^+ 169.254.0.83 2 10 377 767 +168us[+299us] +/- 37ms
```


### 8.3 The configuration takes effect

Restart chrony to make the configuration take effect
```
[root@VM-6-140-opencloudos /]#systemctl restart chronyd
```

View the status of the time synchronization source
```
[root@VM-6-130-opencloudos ~]# chronyc sourcestats -v

169.254.0.79 6 3 86m +0.081 0.289 +191us 178us

169.254.0.80 24 14 396m -0.015 0.017 -235us 156us

169.254.0.81 6 3 86m +0.044 0.257 +62us 140us

169.254.0.82 6 3 86m -0.062 0.420 -118us 142us

169.254.0.83 9 8 137m +0.031 0.035 +165us 47us
```

## 9 Bond Configuration

The bond technology is to bind multiple physical network cards to the same IP address to provide external services, and configure them in different modes to achieve high availability, load balancing, and link redundancy.

### 9.1 Bond mode

Bond has 7 modes, as shown in the table below

| Mode | Description |
| --- | --- |
| mode 0 | load balancing (round-robin) mode, multiple network cards work at the same time |
| mode 1 | fault-tolerance (active-backup) provides redundancy function, the working mode is the main-backup working mode |
| mode 2 | balance-x, providing load balancing and redundancy |
| mode 3 | means broadcast, this mode provides fault tolerance |
| mode 4 | Indicates 802.3ad, indicates support for 802.3ad protocol, and cooperates with the aggregation LACP mode of the switch (requires xmit\_hash\_policy) |
| mode 5 | means balance-tlb, which automatically adapts to load balancing and automatically switches over failures. On this basis, Ethtool supports drivers |
| mode 6 | Indicates that the broadcast information of arp is optimized on the basis of mode 5 |

Among them, mode4 is the common mode

### 9.2 Configure bond

1. Select 2 network cards that need to be bound for configuration. Example: eth0/eth1, the parameters of the configuration file ifcfg-eth0 of eth0 in the /etc/sysconfig/network-scripts/ directory are expressed as follows:
```
[root@VM-6-130-opencloudos ~]# /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0 #Network port name: eth0

NAME=eth0

TYPE=Ethernet #Network port type: Ethernet interface

ONBOOT=yes #The network port status is activated when the system starts

BOOTPROTO=none #Network port activation protocol: nono does not apply to any protocol static: set static ip dhcp: set dynamic acquisition ip

MASTER=bond\_test #Specify the name of the virtual network port

SLAVE=yes #Standby (slave device)
```

2. The parameters of the configuration file ifcfg-eth1 in the /etc/sysconfig/network-scripts/ directory of eth1 are expressed as follows:
```
[root@VM-6-130-opencloudos ~]# /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1 #Network port name: eth1

NAME=eth1

TYPE=Ethernet #Network port type: Ethernet interface

ONBOOT=yes #The network port status is activated when the system starts

BOOTPROTO=none #Network port activation protocol: nono does not apply to any protocol static: set static ip dhcp: set dynamic acquisition ip

MASTER=bond\_test #Specify the name of the virtual network port

SLAVE=yes #Standby (slave device)
```

3. Configure the bond\_test network card in the /etc/sysconfig/network-scripts/ifcfg-bond\_test directory
```
[root@VM-6-130-opencloudos ~]# /etc/sysconfig/network-scripts/ifcfg-bond_test

DEVICE=bond\_test #Network port name: bond\_test

NAME=bond\_test

TYPE=Ethernet #Network port type: Ethernet interface

ONBOOT=yes #The network port status is activated when the system starts

DELAY=0

NM\_CONTROLLED=yes

BOOTPROTO=static #Network port activation protocol: nono does not apply to any protocol static: set static ip dhcp: set dynamic acquisition ip

IPADDR='9.27.148.124' #ip address

NETMASK='255.255.255.192' #subnet mask

GATEWAY='9.27.148.65' #Gateway

BONDING\_OPTS='mode=4 miimon=100 lacp\_rate=fast xmit\_hash\_policy=1'
```

Among them, miimon means link monitoring: miimon=100 means that the system monitors the link status every 100ms, and if one line fails, it will transfer to another line

4. Configure bonding
```
[root@VM-6-140-opencloudos /]#vim /etc/modprobe.d/dist.conf
alias bond\_test bonding
```
5. View the currently used network port
```
[root@VM-6-140-opencloudos /]#cat /proc/net/bonding/bond_test
```
<br />


## 10 LVM Management

LVM (Logical Volume Manager) logical volume management is a logical layer added between the hard disk partition and the file system. It shields the underlying hard disk partition layout for the file system and provides an abstract volume on which the file system is established. System administrators can use LVM to dynamically adjust the size of the file system without repartitioning the hard disk, and the file system managed by LVM can span physical hard disks. When a new hard disk is added to the server, the administrator does not need to move the original files to the new hard disk, but directly expands the file system through LVM to span the physical hard disk.

### 10.1 Basic concepts

Physical Volume (PV): Initialize conventional block devices (hard disks, partitions and other devices that can read and write data) through the pvcreate command to form a physical volume

Volume group (VG): The capacity of multiple physical volumes is formed into a logical whole, and the capacity can be flexibly allocated from it

Logical Volume (LV): Divide part of the space from the volume group into a logical unit that can read and write data, which needs to be formatted and then mounted for use

### 10.2 Deploy lvm

1. Add a physical disk and create a physical volume
```
[root@VM-16-5-opencloudos ~]# lsblk | grep "vd[bcd]"

vdb 253:0 0 50G 0 disk

vdc 253:16 0 50G 0 disk

vdd 253:32 0 50G 0 disk
```
2. Add disk to pv

```
[root@VM-16-5-opencloudos ~]# pvcreate /dev/vdb

Physical volume "/dev/sdb" successfully created.//Check pv creation

[root@VM-16-5-opencloudos ~]# pvs

PV VG Fmt Attr PSize PFree

/dev/vdb lvm2 --- 50.00g 50.00g
```

3. Create a volume group named datavg

```
[root@VM-16-5-opencloudos ~]# vgcreate datavg /dev/vdb

Volume group "datavg" successfully created

[root@VM-16-5-opencloudos ~]# vgs

VG #PV #LV #SN Attr VSize VFree

datavg 1 0 0 wz--n- \<50.00g \<50.00g
```

4. Create logical volumes, assign names, and sizes, and specify volume groups

```
[root@VM-16-5-opencloudos ~]# lvcreate -L 100M -n lv1 datavg

Wiping ext4 signature on /dev/datavg/lv1.

Logical volume "lv1" created.

[root@VM-16-5-opencloudos ~]# lvscan

ACTIVE '/dev/datavg/lv1' [100.00 MiB] inherit
```

5. Format file system

```
[root@VM-16-5-opencloudos ~]# mkfs.ext4 /dev/datavg/lv1

Allocating group tables: done

Writing inode tables: done

Creating journal (4096 blocks): done

Writing superblocks and filesystem accounting information: done
```

6. Mount and use

```
[root@VM-16-5-opencloudos ~]#mkdir /lv1

[root@VM-16-5-opencloudos ~]#mount /dev/datavg/lv1 /lv1/

[root@VM-16-5-opencloudos ~]#df -hl

Filesystem Size Used Avail Use% Mounted on

/dev/mapper/datavg-lv1 93M 1.6M 85M 2% /lv1
```

### 10.3 Volume group management

Extend the volume group to add new disks to the volume group

1. Add new disk to pv
```
[root@VM-6-140-opencloudos /]#pvcreate /dev/vdc
```
2. Use vgextend extension
```
[root@VM-6-140-opencloudos /]#vgextend datavg /dev/vdc
```

3. Shrink volume group
```
[root@VM-6-140-opencloudos /]#vgreduce datavg /dev/vdb
```
Data migration volume group.

Only disks in the same volume group can be migrated online.

4. Check the PV usage in the current logical volume VG
```
[root@VM-16-5-opencloudos ~]#pvs

PV VG Fmt Attr PSize PFree

/dev/vdb vg1 lvm2 a -- 2.00g 1.76g

/dev/vdc vg1 lvm2 a -- 2.00g 2.00g
```

5. Pvmove migrates /dev/vdb data to /dev/vdc online

```
[root@VM-16-5-opencloudos ~]#pvmove /dev/vdb /dev/vdc
```

6. Check whether the migration is successful
```
[root@VM-16-5-opencloudos ~]#pvs

/dev/vdb vg1 lvm2 a -- 2.00g 2.00g

/dev/vdc vg1 lvm2 a -- 2.00g 1.76g
```
7. Delete volume group
```
[root@VM-16-5-opencloudos ~]#vgremove vgname
```
Before deleting the volume group, please ensure that the logical volumes in the volume group are not mounted.

### 10.4 Logical volume management

Logical volume extension

The expansion of the logical volume depends on the capacity in the volume group, and the capacity of the logical volume expansion cannot exceed the capacity of the volume group

1. Add 1G capacity to the logical volume
```
[root@VM-16-5-opencloudos ~]#lvextend -L +1G /dev/datavg/lv1 (Note: 1G and +1G have different meanings)
```
2. Expand the file system
```
xfs\_growfs /dev/datavg/lv1 //xfs file system expansion

resize2fs /dev/datavg/lv1//ext file system expansion
```
Cut the capacity of logical volumes in ext format

Taking cutting the capacity of the logical volume to 512M as an example, the operation steps are as follows:

3. First create a 1G logical volume as the object to be trimmed
```
[root@VM-16-5-opencloudos ~]#lvcreate -n rm_test -L 1G datavg //datavg为卷组

[root@VM-16-5-opencloudos ~]#mkfs.ext4 /dev/datavg/rm_test

[root@VM-16-5-opencloudos ~]#mkdir -p /rm_test

[root@VM-16-5-opencloudos ~]#mount /dev/datavg/rm_test /rm_test/
```
4. If it is already mounted, it must be uninstalled first
```
[root@VM-16-5-opencloudos ~]#umount /dev/datavg/rm_test
```
5. To cut the capacity, the file system must be detected first
```
[root@VM-16-5-opencloudos ~]#e2fsck -f /dev/datavg/rm_test

[root@VM-16-5-opencloudos ~]#resize2fs /dev/datavg/rm_test 512M
```
6. After the adjustment, cut the logical volume capacity
```
[root@VM-16-5-opencloudos ~]#lvreduce -L 512M /dev/datavg/rm_test
```
7. Mounting test, if it can be mounted successfully, the file system is not damaged
```
[root@VM-16-5-opencloudos ~]#mount /dev/datavg/rm_test
```
Delete logical volume

8. Make sure the logical volume being deleted is not in use

```
[root@VM-16-5-opencloudos ~]#umount /dev/datavg/rm_test # umount /dev/volume group name/logical volume name
```
9. Delete the logical volume

    `lvremove <volume_group>/<logical_volume>`
```
[root@VM-16-5-opencloudos ~]#lvremove /dev/datavg/rm_test
```

## 11 Soft RAID Management

RAID (redundant array of independent disks) is a redundant array of disks. By combining multiple hard disk devices into a disk array with larger capacity and better security, the data is cut into multiple segments and stored in different physical disks. On the hard disk device, and then use distributed read and write technology to improve the overall performance of the disk array, and at the same time synchronize multiple copies of important data to different physical hard disk devices, thus achieving a very good data redundancy backup effect and fault tolerance.

RAID is divided into soft RAID and hard RAID according to the use of hardware

Soft RAID: no independent RAID controller card, all RAID functions are implemented by the operating system and CPU

### 11.1 RAID logical classification

- RAID0

Data striping, no parity, no data protection. Data is written to multiple hard disks concurrently.

Advantages: 

1. The read and write performance is the highest among all RAIDs. 

2. 2.100% disk space utilization

Disadvantage: 

It does not provide data redundancy protection, and once the data is damaged, it cannot be recovered.

Application scenarios: RAID 0 is suitable for scenarios that require fast reading and writing, but do not require high data security and reliability, such as video and printing.

- RAID1

Data mirroring, no checksum. Half of the space stores redundant data, and the data security is the highest among all RAIDs.

Advantages: 

1. The security is the highest among all RAIDs, even if half of the disks fail, they can still operate normally. 

2. If all the mirrored disks do not fail, the data will not be lost.

Disadvantages: 1. The disk space utilization rate is 50%, and half of the space is used to store redundant data. 2. High cost.

- RAID5

Data striping, parity data (1 group) is evenly distributed on each physical disk. When a physical disk fails, the damaged data can be reconstructed according to other data blocks and corresponding parity data in the same stripe.

Advantages: 

1. Allow 1 physical disk to fail without data loss. 

2. The read performance is relatively high, and the disk space utilization rate is greater than that of RAID 10.

Disadvantages: 

1. The write performance is relatively low. 

2. When rebuilding data, the performance will be greatly affected.

- RAID10

The combination of RAID 1 and RAID 0, from the perspective of disks, first group four disks in pairs to form two mirror pairs, use RAID1 mode for data backup in the group first, and use RAID0 for data splitting between groups to improve read performance. write rate. Compared with RAID0, RAID1 uses more disks.

Advantages: 

1. The read performance is second only to RAID 0. 

2. If all the disks in the mirror pair do not fail, the data will not be lost.

3. When half of the physical disks fail, they can still operate normally.

Disadvantages: 

1. High cost. 

2. The disk space utilization rate is 50%, and half of the space is used to store redundant data.

- RAID6

Data striping, parity data (2 groups) are evenly distributed on each physical disk. Even if two disks fail at the same time, the damaged data on the two disks can be reconstructed through two sets of parity data.

Advantages: 

1. Allows 2 physical disks to fail without data loss. 

2. The read performance is high, and the disk space utilization rate is greater than that of RAID 10.

Cons: Higher cost than RAID 5, lower write performance (lower than RAID 5).

- RAID50

The combination of RAID 5 and RAID 0, create RAID 5 first, and then create RAID 0. Effectively improves the performance of RAID 5. Divide the constituent disks into several identical RAID 5s. Configuring RAID 50 requires at least 6 disks, divided into 2 RAID 5s with 3 disks in each group.

Advantages: 

1. The read and write performance is higher than RAID 5. 

2. The fault tolerance is higher than RAID 0 or RAID 5. 

3. The failed disks are in different RAID 5, and at most n physical disks are allowed to fail (n is RAID 5 number) without loss of data.

Disadvantages: 

1. When rebuilding a failed disk, if another disk fails in the same RAID 5, all data will be lost. 

2. More space is needed in the disk to store the verification data.

- RAID60

The combination of RAID 6 and RAID 0 creates RAID 6 first and then RAID 0. Effectively improves the performance of RAID6. Divide the constituent disks into several identical RAID 6s. Configuring RAID 60 requires at least 8 disks, divided into two RAID 6 groups of 4 disks each.

Advantages: 

1. The read and write performance is higher than that of RAID 6. 

2. The fault tolerance is higher than that of RAID 0 or RAID 6. 

3. There are no more than two failed disks in the same RAID 6, and a maximum of 2n physical disks can be allowed to fail (n for RAID 6) without data loss.

Disadvantages: 

1. When rebuilding a failed disk, if a third disk fails in the same RAID 6, all data will be lost. 

2. More space is needed in the disk to store the verification data.

The selection of RAID mode is shown in the table below:

| RAID level | Fault tolerance | Number of hard disks | Available capacity | Allowable number of faulty hard disks | Usage scenarios |
| --- | --- | --- | --- | --- | --- |
| RAID 0 | None | N\>=2 | All | 0 | High read/write, no protection |
| RAID 1 | Yes | N\>=2 and N%2=0 | Half |
| RAID 5 | Yes | N\>=3 | (N-1)\*single disk capacity | 1 | Random data transmission, high security requirements |
| RAID 6 | Yes | N\>=4 | (N-2)\*single disk capacity | 2 | Use less |
| RAID 10 | Yes | N\>=4 and N%2=0 | Half | Half | High read/write, high protection |
| RAID 50 | Yes | N\>=6 | Each RAID5 has 1 disk to store parity data | Each RAID5 allows 1 disk to fail | Less usage |
| RAID 60 | Yes | N\>=8 | Each RAID6 has 2 disks for parity data storage | Each RAID6 allows 2 disk failures | Less usage |

### 11.2 Soft RAID10 Deployment

There is an md (multiple devices) module in the Linux kernel to manage RAID devices at the bottom layer. It will provide users with an application tool mdadm at the application layer. mdadm is a command for creating and managing software RAID under Linux.

This document divides the disk /dev/vdb/ into four partitions as four disks to build soft RAID10;

1. Create raid10
```
[root@VM-16-5-opencloudos ~]#mdadm -C -v /dev/md10 -l 10 -n 4 /dev/vdb[1-4]

mdadm: layout defaults to n2

mdadm: layout defaults to n2

mdadm: chunk size defaults to 512K

mdadm: size set to 5237760K

mdadm: Defaulting to version 1.2 metadata

mdadm: array /dev/md10 started.
```

Parameter Description:

- `-C` create

- `-v` visualize

- `-l` specifies the RAID level

- `-n` specifies the number of devices


2. View saved configuration
```
[root@VM-16-5-opencloudos ~]#mdadm -D /dev/md10

[root@VM-16-5-opencloudos ~]#mdadm -Dsv \> /etc/mdadm.conf
```
3. View array information
```
[root@VM-16-5-opencloudos ~]#cat /proc/mdstat
```
4. Format and mount md10 as a whole disk
```
[root@VM-16-5-opencloudos ~]#mkfs.ext4 /dev/md10

[root@VM-16-5-opencloudos ~]#mount -t ext4 /dev/md10 /mnt/
```
At this point, the disk array is available

### 11.3 Unmounting the RAID array

Taking the RAID created in Section 11.2 above as an example, execute the following command to uninstall the disk array
```
[root@VM-16-5-opencloudos ~]#umount /dev/md10

[root@VM-16-5-opencloudos ~]#mdadm -S /dev/md10

[root@VM-16-5-opencloudos ~]#mdadm --misc --zero-superblock /dev/vdb[1-4]
```


## 12 Partition Management

### 12.1 Disk Partition Basics

- Basic partition (primary partition)

Basic partitions are also called primary partitions. The sum of boot partitions, primary partitions and extended partitions for each disk partition cannot exceed four.

Basic partitions can be used immediately after creation, but there is an upper limit on the number of partitions.

- Extension partion

Each disk can only be divided into one extended partition, and any logical partition can be divided into the extended partition

After the extended partition is created, it cannot be used directly, and a logical partition needs to be created in the extended partition

- Logical partition

A logical partition is a partition created within an extended partition

A logical partition is equivalent to a storage medium, completely independent from other logical partitions and primary partitions

The disk partition mainly uses the fdisk -l tool

### 12.2 Create MBR partition

Select a disk that needs to be partitioned, take /dev/vdb/ as an example, use the fdisk tool to create a new partition
```
[root@VM-6-130-opencloudos /]# fdisk /dev/vdb

Command (m for help): n //Create a new partition

Partition type

p primary (0 primary, 0 extended, 4 free)

e extended (container for logical partitions)

Select (default p): p //Select the partition type, take the primary partition as an example

Partition number (1-4, default 1):1 //Select partition number

First sector (2048-104857599, default 2048): 2048 //sector start address

Last sector, +sectors or +size{K,M,G,T,P} (2048-104857599, default 104857599):+5G //Specify sector capacity

Created a new partition 1 of type 'Linux' and of size 5 GiB.

Command (m for help): wq //Save changes and exit
```
So far, the new partition has been successfully created!

Other operating parameters are as follows:

| fdisk -l | view, list disk information |
| --- | --- |
| fdisk /dev/vdb | operate |
| p | List the existing partitions of the disk, the partition type is primary partition |
| e | partition type is extended partition |
| n | add new disk partition |
| d | delete an existing disk partition |
| w | save partition operations |
| q | discard changes and exit |
| t | Change the id of the partition, that is, modify the partition function id |
| l | list existing partition types |

### 12.3 Format partition

The newly created partition needs to be formatted with the corresponding file system, taking the ext4 format as an example
```
[root@VM-16-5-opencloudos ~]#mkfs.ext4 /dev/vdb1
```
### 12.4 Mount/unmount partitions

You need to mount the formatted partition to a specified directory before it can be used, such as mounting the partition to the /mnt/ partition
```
[root@VM-16-5-opencloudos ~]#mount -t ext4 /dev/vdb1
```
After the partition is not used, the partition can be unmounted
```
[root@VM-16-5-opencloudos ~]#umount /dev/vdb1
```
### 12.5 Create GPT partition

The fdisk command is used to create and maintain disk partitions, and fdisk can only partition hard disks smaller than 2TB. For hard disks larger than 2TB, you need to use the parted tool to partition. Using fdisk to create partitions can only create MBR partition schemes.

1. Enter parted interactive mode

```
[root@VM-6-130-opencloudos /]# parted

(parted) select /dev/vdb //select disk /dev/vdb

(parted) print free //View disk information
```
2. Create a new partition
```
(parted) mkpart

Partition type? primary/extended? primary

File system type? [ext2]? ext4

Start? 1

End? 5G

(parted)quit
```

3. Format the disk as a GPT disk
```
(parted) mklabel gpt
```
### 12.6 Resize Partition

Parted can adjust the size of the partition. Note that when parted adjusts the mounted partition, it will not affect the data in the partition. But be sure to unmount the partition first, and then resize the partition, otherwise there will be problems with the data. In addition, the partition to be resized must have established a file system (formatted), otherwise an error will be reported.

Take resizing the /dev/vdb1 partition as an example

1. Unmount the partition
```
[root@VM-16-5-opencloudos ~]#umount /dev/vdb1
```
2. Enter parted interactive mode

Execute the following command
```
[root@VM-6-130-opencloudos /]# parted /dev/vdb

(parted) resizepart

Partition number? 1

End? 10G

(parted) quit
```
### 12.7 Delete GPT partition

1. Unmount the partition
```
[root@VM-16-5-opencloudos ~]#umount /dev/vdb1
```
2. Enter parted interactive mode

Execute the following command
```
[root@VM-6-130-opencloudos /]# parted /dev/vdb

(parted) rm

Partition number? 1

(parted) quit
```
It should be noted that all operations in parted take effect immediately, and there is no concept of saving and taking effect. This is obviously different from the fdisk interactive command, so all operations should be done with care.
