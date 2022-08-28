# <center />OpenCloudosOS-监控和管理系统状态和性能

## 第一章 TUNED入门

TuneD 是一项服务，可监控您的系统并优化特定工作负载下的性能。

TuneD 的核心是配置文件，可针对不同的用例调整您的系统。TuneD提供的配置文件分为以下几类:

- 节能配置文件
- 性能提升配置文件

### 1.1合并优化配置文件

TuneD可以一次选择更多配置文件,TuneD 将在加载期间尝试合并它们。如果存在冲突，则最后指定配置文件中的设置优先。

前提：安装tuned

例：虚拟访客中的低功耗：

1.启动tuned
```
[root@opencloudos ~]# service tuned start
Redirecting to /bin/systemctl start tuned.service
```

2.优化系统以在虚拟机中运行以获得最佳性能，并调优以实现低功耗
```
[root@opencloudos ~]# tuned-adm profile virtual-guest powersave
```

### 1.2优化策略

-   balanced：默认的省电配置文件。它旨在成为性能和功耗之间的折衷方案。它尽可能使用自动缩放和自动调整。唯一的缺点是延迟增加。
-   powersave：最大节能性能的配置文件。它可以限制性能以最小化实际功耗运行。
-   throughput-performance：针对高吞吐量优化的服务器配置文件。它禁用节能机制并启用sysctl设置，以提高磁盘和网络 IO 的吞吐量性能。
-   accelerator-performance：加速器性能配置文件包含与吞吐量性能配置文件相同的调整。可以将CPU锁定到低 C 状态，因此延迟小于 100us。提高了某些加速器的性能，例如 GPU。
-   virtual-guest：基于吞吐量性能配置文件，除其他任务外，该配置文件可减少虚拟内存交换并增加磁盘预读值。
-   virtual-host：该配置文件可减少虚拟内存交换，增加磁盘预读值，并启用更积极的脏页写回值。
-   optimize-serial-console：通过减少 printk 值来调整串行控制台的 I/O 活动的配置文件。

```
[root@opencloudos ~]# tuned-adm profile throughput-performance optimize-serial-console
```

-   intel-sst：针对具有用户定义的 Intel Speed Select Technology 配置的系统优化的配置文件。

```
[root@opencloudos ~]# tuned-adm profile cpu-partitioning intel-sst
```

### 1.3优化的 CPU 分区配置文件

在/etc/tuned/cpu-partitioning-variables.conf文件中配置 cpu‑partitioning 配置文件：具有负载平衡的隔离 CPU、没有负载平衡的隔离 CPU、内务处理 CPU。

### 1.4使用调整后的 CPU 分区配置文件以实现低延迟调整

前提：用root身份使用 yum install tuned-profiles-cpu-partitioning 命令安装cpu-partitioning TuneD配置文件。

1.编辑/etc/tuned/cpu-partitioning-variables.conf 文件并添加以下信息

```
Isolated CPUs with the kernel’s scheduler load balancing:
isolated_cores=2-23
Isolated CPUs without the kernel’s scheduler load balancing:
no_balance_cores=2,3
```

2.设置cpu-partitioning TuneD 配置文件

```
[root@opencloudos ~]# tuned-adm profile cpu-partitioning
```

3.重启
```
[root@opencloudos ~]# reboot
```

### 1.5安装和启动Tuned

1.安装tuned包

```
[root@opencloudos ~]# yum install tuned
```

2.启动并启用tuned服务

```
[root@opencloudos ~]# systemctl enable --now tuned
```

3.（可选）为实时系统安装 TuneD 配置文件：

```
[root@opencloudos ~]# yum install tuned-profiles-realtime tuned-profiles-nfv
```

4.验证 TuneD 配置文件是否处于活动状态并已应用：

```
[root@opencloudos ~]# tuned-adm active
Current active profile: virtual-guest
```

查看当前系统设置与预设配置文件匹配
```
[root@opencloudos ~]# tuned-adm verify

Verfication succeeded, current system settings match the preset profile.

See TuneD log file ('/var/log/tuned/tuned.log') for details.
```

### 1.6列出可用的调整配置文件

1.列出系统上所有可用的 TuneD 配置文件

```
[root@opencloudos ~]# tuned-adm list

Available profiles:

\- accelerator-performance - Throughput performance based tuning with disabled higher latency STOP states

\- balanced - General non-specialized tuned profile

\- cpu-partitioning - Optimize for CPU partitioning

\- desktop - Optimize for the desktop use-case

\- hpc-compute - Optimize for HPC compute workloads

\- intel-sst - Configure for Intel Speed Select Base Frequency

\- latency-performance - Optimize for deterministic performance at the cost of increased power consumption

\- my-profile

...

Current active profile: virtual-guest
```

2.仅显示当前活动的配置文件

```
[root@opencloudos ~]# tuned-adm active

Current active profile: virtual-guest
```

### 1.7设置一个经过调整的配置文件

此过程会在你的系统上激活选定的 TuneD 配置文件

1.你可以让TuneD为你的系统推荐最合适的配置文件：

```
[root@opencloudos ~]# tuned-adm recommend

virtual-guest
```

2.激活配置文件：

```
[root@opencloudos ~]# tuned-adm profile selected-profile
```

激活多个配置文件

```
[root@opencloudos ~]# tuned-adm profile profile1 profile2
```
3.重启系统：

```
[root@opencloudos ~]# reboot
```

4.验证TuneD 配置文件是否处于活动状态并已应用：
```
[root@opencloudos ~]# tuned-adm verify
Verfication succeeded, current system settings match the preset profile.

See TuneD log file ('/var/log/tuned/tuned.log') for details.
```
### 1.8禁用TUNED

1.暂时禁用所有tuned：

```
[root@opencloudos ~]# tuned-adm off
```

2.永久停止和禁用tuned服务：
```
[root@opencloudos ~]# systemctl disable --now tuned
```
## 第二章 磁盘管理

### 2.1查看磁盘

1.查看磁盘信息，但不会显示未挂载的磁盘
```
[root@opencloudos ~]# df -h

Filesystem Size Used Avail Use% Mounted on

devtmpfs 830M 0 830M 0% /dev

tmpfs 845M 24K 845M 1% /dev/shm

tmpfs 845M 250M 596M 30% /run

tmpfs 845M 0 845M 0% /sys/fs/cgroup

/dev/vda1 50G 4.5G 43G 10% /

tmpfs 169M 0 169M 0% /run/user/0
...
```

2.查看主机所有的磁盘列表
```
[root@opencloudos ~]# fdisk -lu

Disk /dev/vda: 50 GiB, 53687091200 bytes, 104857600 sectors

Units: sectors of 1 \* 512 = 512 bytes

Sector size (logical/physical): 512 bytes / 512 bytes

I/O size (minimum/optimal): 262144 bytes / 262144 bytes

Disklabel type: dos

Disk identifier: 0xe609e297

Device Boot Start End Sectors Size Id Type

/dev/vda1 2048 104857566 104855519 50G 83 Linux
...
```

3.查看指定目录磁盘使用情况

    在命令后直接放目录名：
```
[root@opencloudos ~]# df -h /dev

Filesystem Size Used Avail Use% Mounted on

devtmpfs 830M 0 830M 0% /dev
```
4.df命令介绍

- \-a：列出所有的文件系统，包括系统特有的/proc等文件系统
- \-k：以KB的容量显示各文件系统
- \-m：以MB的容量显示各文件系统
- \-h：以人们较易阅读的GB,MB,KB等格式自行显示
- \-H：以M=1000K替代M=1024K的进位方式
- \-T：显示文件系统类型
- \-i：不用硬盘容量，而以inode的数量来显示
- \-l：只显示本机的文件系统

5.du命令介绍

    作用：使用du命令查看指定目录的使用情况。

    选项：-h:输出文件系统分区使用的情况

    \-s:显示文件或整个目录的大小

### 2.2添加新磁盘

新添加硬盘后，我们如果想使用这块硬盘，必须通过分区-\>格式化-\>挂载后才可以使用。

分区：一共有两种分区方式
- 采用MBR的方式进行分区，即fdisk命令
- 采用GPT的方式进行分区，即gdisk命令
  
1.格式化磁盘：

```
[root@opencloudos ~]# mkfs
```

将指定分区格式化为ext4格式：

```
[root@opencloudos ~]# mkfs -t ext4 /dev/sdb1
```

格式化指定磁盘2G空间：

```
[root@opencloudos ~]# mkfs /dev/sdb2 2G
```

2.挂载：
    建立设备分区和系统目录的映射关系。

```
[root@opencloudos ~]# mount 设备名称 挂载目录
```

### 2.3删除磁盘分区

1.查看硬盘信息，找到自己想要删除哪块硬盘上的分区。
```
[root@opencloudos ~]# fdisk -l
```

2.进入该磁盘分区
```
[root@opencloudos ~]# fdisk 设备名称
```

3.输入d表示删除分区，若有多个分区，则会需要选择分区号
```
    command(m for help): d
```

4.输入w保存退出

### 2.4可用的磁盘调度程序

- none
  先入先出(FIFO)调度算法。它通过简单的最后一个缓存合并通用块层的请求。

- mq-deadline
尝试从请求到达调度程序时起为请求提供保证的延迟。此调度程序适用于大多数用例，特别是写操作大部分异步的情况。

- bfq
以桌面系统和互动任务为目标。此调度程序适合用于复制大型文件，在这种情况下，系统也不会变得无响应。

- kyber
调度程序通过计算提交至块 I/O 层的每个 I/O 请求的延迟来调整自身以达到延迟目标。这个调度程序适合快速设备，如 NVMe、SSD 或其他低延迟设备。

默认磁盘调度程序：

对于非易失性 Memory Express(NVMe)块设备，默认调度程序是 none

### 2.5确定活跃磁盘调度程序

查看哪个磁盘调度程序目前在给定块设备中活跃。

```
 [root@opencloudos ~]# cat /sys/block/设备/queue/scheduler** 将设备替换为块设备名称

[none] mq-deadline kyber bfq
```

### 2.6使用TuneD设置磁盘调度程序

1.查看哪个配置集当前处于活跃状态
```
[root@opencloudos ~]# tuned-adm active
```

2.创建保存 TuneD 配置集的新目录
```
[root@opencloudos ~]# mkdir /etc/tuned/my-profile
```
3.查找所选块设备系统唯一标识符：

```
[root@opencloudos ~]# udevadm info --query=property --name=/dev/device \| grep -E '(WWN\|SERIAL)'

[root@opencloudos ~]# udevadm info --query=property --name=/dev/sr0 \| grep -E '(WWN\|SERIAL)'

ID_SERIAL=QEMU_DVD-ROM_QM00002

ID_SERIAL_SHORT=QM00002
```

4.创建 /etc/tuned/my-profile/tuned.conf 配置文件。在该文件中设置：

    为与 WWN 标识符匹配的设备设置所选磁盘调度程序：
```
[disk]

devices_udev_regex=IDNAME=device system unique id

elevator=selected-scheduler
```

5.启用您的配置集：
```
[root@opencloudos ~]# tuned-adm profile my-profile
```

6.验证 TuneD 配置集是否活跃并应用：
```
[root@opencloudos ~]# tuned-adm active

Current active profile: my-profile
```
```
[root@opencloudos ~]# tuned-adm verify

Verfication succeeded, current system settings match the preset profile.

See TuneD log file ('/var/log/tuned/tuned.log') for details.
```

7.读取 /sys/block/设备/queue/scheduler文件的内容：
```
[root@opencloudos ~]# cat /sys/block/device/queue/scheduler
mq-deadline kyber [bfq] none
```

### 2.7使用 udev 规则设置磁盘调度程序

1.查找块设备系统唯一标识符
```
[root@opencloudos ~]# udevadm info --name=/dev/sr0 \| grep -E '(WWN\|SERIAL)'

[root@opencloudos ~]# udevadm info --name=/dev/sr0 \| grep -E '(WWN\|SERIAL)'

E: ID_SERIAL=QEMU_DVD-ROM_QM00002

E: ID_SERIAL_SHORT=QM00002
```

2.配置udev 规则。使用以下内容创建 /etc/udev/rules.d/99-scheduler.rules文件：

    注意：将 IDNAME 替换为要使用的标识符的名称（如 ID_WWN）

    将 设备系统唯一 id 替换为所选标识符值（例0x5002538d00000000）
```
ACTION=="add\|change", SUBSYSTEM=="block", ENV{ID_SERIAL_SHORT}=="QM00002", ATTR{queue/scheduler}="bfq"
```

3.重新载入udev规则：
```
[root@opencloudos ~]# udevadm control --reload-rules
```
4.应用调度程序配置：
```
[root@opencloudos ~]# udevadm trigger --type=devices --action=change
```

5.验证活跃的调度程序：
```
[root@opencloudos ~]# cat /sys/block/device/queue/scheduler

[root@opencloudos ~]# cat /sys/block/sr0/queue/scheduler

mq-deadline kyber [bfq] none
```
### 2.8为特定磁盘临时设置调度程序

此流程为特定块设备设置给定磁盘调度程序。系统重启后该设置不会保留。

1.将所选调度程序的名称写入 /sys/block/设备/queue/scheduler文件：
```
[root@opencloudos ~]# echo selected-scheduler \> /sys/block/device/queue/scheduler
```
在文件名中，将 device 替换为块设备名称，如vda

2.验证调度程序是否在该设备中活跃：
```
[root@opencloudos ~]# cat /sys/block/device/queue/scheduler

[root@opencloudos ~]# cat /sys/block/vda/queue/scheduler

[none] mq-deadline kyber bfq
```
## 第三章 CPU使用率以及调优

### 3.1显示CPU的架构信息

1.收集CPU的架构信息，如 CPU、线程、内核、插槽和 NUMA 节点的数量：
```
[root@opencloudos ~]# lscpu

Architecture: x86_64

CPU op-mode(s): 32-bit, 64-bit

Byte Order: Little Endian

CPU(s): 2

On-line CPU(s) list: 0,1

Thread(s) per core: 2

Core(s) per socket: 1

Socket(s): 1

NUMA node(s): 1

Vendor ID: GenuineIntel

BIOS Vendor ID: Smdbmds

CPU family: 6

Model: 106
...
```

### 3.2查看CPU的使用率：

1.展示CPU的性能：
```
[root@opencloudos ~]# top

%Cpu(s): 0.2 us, 0.2 sy, 0.0 ni, 99.7 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
```

- user（通常缩写为 us），代表用户态 CPU 时间。注意，它不包括下面的 nice 时间，但包括了 guest 时间。
- nice（通常缩写为 ni），代表低优先级用户态 CPU 时间，也就是进程的 nice 值被调整为 1-19 之间时的 CPU 时间。这里注意，nice 可取值范围是 -20 到 19，数值越大，优先级反而越低。
- system（通常缩写为 sy），代表内核态 CPU 时间。
- idle（通常缩写为 id），代表空闲时间。注意，它不包括等待 I/O 的时间（iowait）。
- iowait（通常缩写为 wa），代表等待 I/O 的 CPU 时间。
- irq（通常缩写为 hi），代表处理硬中断的 CPU 时间。
- softirq（通常缩写为 si），代表处理软中断的 CPU 时间。
- steal（通常缩写为 st），代表当系统运行在虚拟机中的时候，被其他虚拟机占用的 CPU 时间。
- guest（通常缩写为 guest），代表通过虚拟化运行其他操作系统的时间，也就是运行虚拟机的 CPU 时间。
- guest_nice（通常缩写为 gnice），代表以低优先级运行虚拟机的时间。

CPU 使用率，就是除了空闲时间外的其他时间占总 CPU 时间的百分比

公式：CPU 使用率 = 1 - 空闲时间 / 总 CPU 时间

2.查看CPU使用率：

    安装sysstat工具：
```
[root@opencloudos ~]# yum install sysstat
```

查看CPU占用
```
[root@opencloudos ~]# pidstat 1

Linux 5.4.119-19-0009.1 (VM-6-117-centos) 08/19/2022 \_x86_64\_ (2 CPU)

03:11:04 PM UID PID %usr %system %guest %wait %CPU CPU Command

03:11:05 PM 0 561098 1.00 0.00 0.00 0.00 1.00 0 barad_agent

03:11:05 PM UID PID %usr %system %guest %wait %CPU CPU Command

03:11:06 PM 0 561098 0.00 1.00 0.00 0.00 1.00 0 barad_agent
...
```

### 3.3perf工具

1.安装perf工具。perf用户空间工具是基于内核的 Linux 子系统性能计数器 (PCL)接口。Perf是一个强大的工具，它使用性能监控单元(PMU)来测量、记录和监控各种硬件和软件事件。
```
[root@opencloudos ~]# yum install perf
```

2.使用 perf top 分析 CPU 使用率

perf top命令用于实时系统分析，其功能与 top 实用程序类似。但是，top实用程序通常显示给定进程或线程使用的 CPU 时间，perf top会显示每个特定功能使用的 CPU 时间。
```
[root@opencloudos ~]# perf top

6.48% perf [.] \__symbols__insert

5.21% [kernel] [k] kallsyms_expand_symbol.constprop.1

4.80% perf [.] rb_next

2.48% perf [.] rust_demangle_callback

2.42% libc-2.28.so [.] \__GI_____strtoull_l_internal

2.32% [kernel] [k] vsnprintf

2.10% [kernel] [k] number
...
```

### 3.4perf调查忙碌CPU

1.计数禁用 CPU 数量聚合的事件：
```
[root@opencloudos ~]# perf stat -a -A sleep seconds
```

指定周期等事件：
```
[root@opencloudos ~]# perf stat -a -A -e cycles sleep seconds
```

2.显示使用 perf 报告进行的 CPU 样本

    前提：当前目录中创建了 perf.data文件，该文件含有perf 记录。

```
[root@opencloudos ~]# perf report --sort cpu
按 CPU 和命令排序：
[root@opencloudos ~]# perf report --sort cpu,comm
```

### 3.5使用 perf top 分析时显示特定的 CPU
```
[root@opencloudos ~]# perf top --sort cpu
按 CPU 和命令排序：
[root@opencloudos ~]# perf top --sort cpu,comm
```

### 3.6记录报告和监控特定CPU

1.在特定 CPU 中记录性能数据，生成 perf.data文件：

    使用以逗号分隔的 CPU 列表
```
[root@opencloudos ~]# perf record -C 0,1 sleep seconds

使用一组 CPU:

[root@opencloudos ~]# perf record -C 0-2 sleep seconds
```

2.显示 perf.data文件的内容，以进一步分析：
```
[root@opencloudos ~]# perf report
```

## 第四章 内存使用率

### 4.1查看内存使用情况

1.使用free命令：
```
[root@opencloudos ~]# free
total used free shared buff/cache available

Mem: 1730432 375364 130400 255480 1224668 932144

Swap: 0 0 0
```

-   total表示总共有1.6G的内存
-   used表示物理内存的使用量，大概215M
-   free表示空闲内存
-   shared表示共享内存
-   buff/cache表示缓存和缓冲内存量; Linux 系统会将很多东西缓存起来以提高性能，这部分内存可以在必要时进行释放，给其他程序使用。
-   available表示可用内存

Swap表示交换内存

2.使用vmstat
```
[root@opencloudos ~]# vmstat -m

Cache Num Total Size Pages

rpc_inode_cache 46 46 704 23

rpc_buffers 16 16 2048 16

rpc_tasks 16 16 256 16

ext4_groupinfo_4k 420 420 144 28

fscrypt_info 0 0 104 39

fscrypt_ctx 0 0 64 64

bridge_fdb_cache 0 0 128 32

ip6-frags 0 0 208 19

fib6_nodes 128 128 64 64

ip6_dst_cache 32 32 256 16
...
```

对内存使用情况进行统计

3.查看/proc/meminfo

    ```
[root@opencloudos ~]# cat /proc/meminfo

MemTotal: 1730432 kB

MemFree: 131156 kB

MemAvailable: 933048 kB

Buffers: 80508 kB

Cached: 1070344 kB

SwapCached: 0 kB
...
```

/proc目录下都是虚拟文件，包含内核以及操作系统相关的动态信息,/proc/meminfo是了解Linux系统内存使用状况的主要接口，我们最常用的”free”、”vmstat”等命令就是通过它获取数据的 ，/proc/meminfo所包含的信息比”free”等命令要丰富得多。

### 4.2查看物理内存信息

1.dmidecode命令
```
[root@opencloudos ~]# dmidecode -t 17
```
展示的信息包括 内存大小, 类型，带宽等信息。

### 4.3虚拟内存

2.介绍：物理内存是有限的（即使支持了热插拔）、非连续的，不同的CPU架构对物理内存的组织都不同。这使得直接使用物理内存非常复杂，为了降低使用内存的复杂度，引入了虚拟内存机制。
3.使用方法：通过虚拟内存机制，每个进程都以为自己占用了全部内存，进程访问内存时，操作系统都会把进程提供的虚拟内存地址转换为物理地址，再去对应的物理地址上获取数据。
4.虚拟内存不仅通过内存地址转换解决了多个进程访问内存冲突的问题，还带来更多的益处：
-   进程内存管理：内存完整性：由于虚拟内存对进程的”欺骗”，每个进程都认为自己获取的内存是一块连续的地址。我们在编写应用程序时，就不用考虑大块地址的分配，总是认为系统有足够的大块内存即可。
-   数据共享：通过虚拟内存更容易实现内存和数据的共享。

### 4.4段页机制

现代操作系统的内存管理机制有两种：段式管理和页式管理。

- 段式内存管理，就是将内存分成段，每个段的起始地址就是段基地址。地址映射的时候，由逻辑地址加上段基地址而得到物理地址。
- 页式内存管理，内存分成固定长度的一个个页片。地址映射的时候，需要先建立页表，页表中的每一项都记录了这个页的基地址。通过页表，由逻辑地址的高位部分先找到逻辑地址对应的页基地址，再由页基地址偏移一定长度就得到最后的物理地址，偏移的长度由逻辑地址的低位部分决定。
-  段页式内存管理需要通过段表和页表的一同使用，段表中包含：段号、段内页表的起始地址，页表包含：页号、页内偏移量。通过虚拟地址中的段号在段表中找到对应的段号项，通过段表中的段号项可以得到段内页表的起始地址（也就是这个分段对应的页表），在得到该段内存对应的页表后可以通过虚拟地址中的段内页号找到对应的页表项得到物理起始地址，加上虚拟地址中的页内偏移量就对应的物理地址。

### 4.5内存回收

1.内存回收的目标：对于内核并不是所有的物理内存都可以参与回收，比如内核的代码段，如果被内核回收了，系统就无法正常运行了，所以一般内核代码段、数据段、内核申请的内存、内核线程占用的内存等都是不可以回收的，除此之外的内存都可以是我们要回收的目标。

2. 内存回收时机：

- 内核需要为任何时刻突发到来的内存申请提供足够的内存，以便cache的使用和其他相关内存的使用不至于让系统的剩余内存长期处于很少的状态。

- 当真的有大于空闲内存的申请到来的时候，会触发强制内存回收。
3.回收机制：

- 针对第1种，Linux系统设计了kswapd后台程序，当内核分配物理页面时，由于系统内存短缺，没法在低水位情况下分配内存，因此会唤醒kswapd内核线程来异步回收内存。也就是周期性内存回收机制。

- 针对第2种，Linux系统会触发直接内存回收(direct reclaim)，在内核调用页分配函数分配物理页面时，由于系统内存短缺，不能满足分配请求，内核就会直接触发页面回收机制，尝试回收内存来解决问题。也就是直接内存回收。
