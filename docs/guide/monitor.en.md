# <center />OpenCloudosOS-Monitor and manage system status and performance

## Chapter 1 Getting Started with TUNED

TuneD is a service that monitors your system and optimizes performance under specific workloads.

At the heart of TuneD are configuration files that tune your system for different use cases. The configuration files provided by TuneD are divided into the following categories:

- Energy Saving Profiles
- Performance boost profile

### 1.1 Merge Optimization Profiles

TuneD can select more profiles at once, TuneD will try to merge them during loading. If there is a conflict, the setting in the last specified configuration file takes precedence.

**Prerequisite**: tuned installed

Example: Low Power Consumption in Virtual Guest:

1. start tuned

```
[root@opencloudos ~]# service tuned start
Redirecting to /bin/systemctl start tuned.service
```

2. Optimize the system to run in a virtual machine for maximum performance and tune for low power consumption

```
[root@opencloudos ~]# tuned-adm profile virtual-guest powersave
```

### 1.2 Optimization Strategy

-   balanced: The default power saving profile. It is intended to be a compromise between performance and power consumption. It uses autoscaling and autoresizing where possible. The only downside is increased latency.
-   powersave: Profile for maximum energy saving performance. It can limit performance to run with minimum actual power consumption.
-   throughput-performance: A server profile optimized for high throughput. It disables power saving mechanisms and enables sysctl settings to improve throughput performance of disk and network IO.
-   accelerator-performance: The Accelerator performance profile contains the same tuning as the Throughput performance profile. The CPU can be locked into a low C state, so the latency is less than 100 us. Improved performance of some accelerators, such as GPUs.
-   virtual-guest: Throughput-based performance profile that, among other tasks, reduces virtual memory swapping and increases disk read-ahead values.
-   virtual-host: This profile reduces virtual memory swapping, increases disk read-ahead values, and enables more aggressive dirty page writeback values.
-   optimize-serial-console: Tweak the profile of the serial console's I/O activity by reducing the printk value.

```
[root@opencloudos ~]# tuned-adm profile throughput-performance optimize-serial-console
```

-   intel-sst: Profile optimized for systems with user-defined Intel Speed Select Technology configurations.

```
[root@opencloudos ~]# tuned-adm profile cpu-partitioning intel-sst
```

### 1.3 Optimized CPU partition profile

Configure the cpu‑partitioning profile in the /etc/tuned/cpu-partitioning-variables.conf file: isolated CPU with load balancing, isolated CPU without load balancing, housekeeping CPU.

### 1.4 Use tuned CPU partition profile for low latency tuning

Prerequisite: Use the yum install tuned-profiles-cpu-partitioning command as root to install the cpu-partitioning TuneD configuration file.

1. Edit the /etc/tuned/cpu-partitioning-variables.conf file and add the following information

```
Isolated CPUs with the kernel’s scheduler load balancing:
isolated_cores=2-23
Isolated CPUs without the kernel’s scheduler load balancing:
no_balance_cores=2,3
```

2. Set the cpu-partitioning TuneD configuration file

```
[root@opencloudos ~]# tuned-adm profile cpu-partitioning
```

3. Restart

```
[root@opencloudos ~]# reboot
```

### 1.5 Install and start Tuned

1. Install the tuned package

```
[root@opencloudos ~]# yum install tuned
```

2. Start and enable the tuned service

```
[root@opencloudos ~]# systemctl enable --now tuned
```

3. (Optional) Install the TuneD configuration file for the live system:

```
[root@opencloudos ~]# yum install tuned-profiles-realtime tuned-profiles-nfv
```

4. Verify that the TuneD profile is active and applied:

```
[root@opencloudos ~]# tuned-adm active
Current active profile: virtual-guest
```

5. Check that the current system settings match the preset configuration files

```
[root@opencloudos ~]# tuned-adm verify

Verfication succeeded, current system settings match the preset profile.

See TuneD log file ('/var/log/tuned/tuned.log') for details.
```

### 1.6 List available tuning profiles

1. List all available TuneD configuration files on the system

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

### 2. Show only currently active profiles

This process activates the selected TuneD profile on your system

1. You can let TuneD recommend the most suitable configuration file for your system:

```
[root@opencloudos ~]# tuned-adm recommend

virtual-guest
```

2. Activate the configuration file:

```
[root@opencloudos ~]# tuned-adm profile selected-profile
```

Activate multiple profiles

```
[root@opencloudos ~]# tuned-adm profile profile1 profile2
```
3. Restart the system:

```
[root@opencloudos ~]# reboot
```

4. Verify that the TuneD profile is active and applied:

```
[root@opencloudos ~]# tuned-adm verify
Verfication succeeded, current system settings match the preset profile.

See TuneD log file ('/var/log/tuned/tuned.log') for details.
```
### 1.8 Disable TUNED

1. Temporarily disable all tuned:

```
[root@opencloudos ~]# tuned-adm off
```

2. Permanently stop and disable the tuned service:

```
[root@opencloudos ~]# systemctl disable --now tuned
```
## Chapter 2 Disk Management

### 2.1 View disk

1. View disk information, but unmounted disks will not be displayed

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

2. View the list of disks owned by the host

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

3. View the disk usage of the specified directory

Put the directory name directly after the command:

```
[root@opencloudos ~]# df -h /dev

Filesystem Size Used Avail Use% Mounted on

devtmpfs 830M 0 830M 0% /dev
```
4. df command introduction

- \-a: List all file systems, including system-specific /proc and other file systems
- \-k: Display each file system in KB capacity
- \-m: display each file system in MB capacity
- \-h: Display in GB, MB, KB and other formats that are easier for people to read
- \-H: Replace the carry method of M=1024K with M=1000K
- \-T: display file system type
- \-i: Use the number of inodes instead of the hard disk capacity to display
- \-l: only display the file system of the local machine

5. du command introduction

    Function: Use the du command to view the usage of the specified directory.
    
    Options: -h: output file system partition usage
    
    \-s: display the size of the file or the entire directory

### 2.2 Add a new disk

After adding a new hard disk, if we want to use this hard disk, we must go through partition-\>format-\>mount before we can use it.

Partition: There are two partition methods

- Use the MBR method to partition, that is, the fdisk command
- Use GPT to partition, that is, the gdisk command

1. Format the disk:

```
[root@opencloudos ~]# mkfs
```

Format the specified partition as ext4:

```
[root@opencloudos ~]# mkfs -t ext4 /dev/sdb1
```

Format the specified disk with 2G space:

```
[root@opencloudos ~]# mkfs /dev/sdb2 2G
```

2. Mount:
    Establish a mapping relationship between device partitions and system directories.

```
[root@opencloudos ~]# mount [Device name] [Mount directory]
```

### 2.3 Delete disk partition

1. Check the hard disk information and find the partition on which hard disk you want to delete.

```
[root@opencloudos ~]# fdisk -l
```

2. Enter the disk partition

```
[root@opencloudos ~]# fdisk [Device name]
```

3. Enter d to delete the partition. If there are multiple partitions, you need to select the partition number

```
    command(m for help): d
```

4. Enter w to save and exit

### 2.4 Available disk schedulers

- none
  First-in-first-out (FIFO) scheduling algorithm. It coalesces requests for common block layers with a simple last cache.

- mq-deadline
Attempts to provide a guaranteed latency for requests from the time the request reaches the scheduler. This scheduler is suitable for most use cases, especially where write operations are mostly asynchronous.

- bfq
Target desktop systems and interactive tasks. This scheduler is suitable for copying large files without the system becoming unresponsive.

- kyber
The scheduler tunes itself to meet latency goals by calculating the latency of each I/O request submitted to the block I/O layer. This scheduler is suitable for fast devices like NVMe, SSD or other low latency devices.

Default disk scheduler:

For Non-Volatile Memory Express (NVMe) block devices, the default scheduler is none

Default disk scheduler:

For Non-Volatile Memory Express (NVMe) block devices, the default scheduler is none

### 2.5 Determine Active Disk Scheduler

See which disk scheduler is currently active on a given block device.

```
[root@opencloudos ~]# cat /sys/block/[device]/queue/scheduler** replace device with the block device name

[none] mq-deadline kyber bfq
```

### 2.6 Setting up the disk scheduler with TuneD

1. See which configuration set is currently active

```
[root@opencloudos ~]# tuned-adm active
```

2. Create a new directory to hold TuneD configuration sets

```
[root@opencloudos ~]# mkdir /etc/tuned/my-profile
```
3. Find the selected block device system unique identifier:

```
[root@opencloudos ~]# udevadm info --query=property --name=/dev/device \| grep -E '(WWN\|SERIAL)'

[root@opencloudos ~]# udevadm info --query=property --name=/dev/sr0 \| grep -E '(WWN\|SERIAL)'

ID_SERIAL=QEMU_DVD-ROM_QM00002

ID_SERIAL_SHORT=QM00002
```

4. Create the /etc/tuned/my-profile/tuned.conf configuration file. In this file set:

    Set the selected disk scheduler for devices matching WWN identifiers:
```
[disk]

devices_udev_regex=IDNAME=device system unique id

elevator=selected-scheduler
```

5. Enable your configuration set:

```
[root@opencloudos ~]# tuned-adm profile my-profile
```

6. Verify that the TuneD configuration set is active and apply:

```
[root@opencloudos ~]# tuned-adm active

Current active profile: my-profile
```
```
[root@opencloudos ~]# tuned-adm verify

Verfication succeeded, current system settings match the preset profile.

See TuneD log file ('/var/log/tuned/tuned.log') for details.
```

7. Read the contents of the /sys/block/device/queue/scheduler file:

```
[root@opencloudos ~]# cat /sys/block/device/queue/scheduler
mq-deadline kyber [bfq] none
```

### 2.7 Setting up the disk scheduler with udev rules

1. Find the block device system unique identifier

```
[root@opencloudos ~]# udevadm info --name=/dev/sr0 \| grep -E '(WWN\|SERIAL)'

[root@opencloudos ~]# udevadm info --name=/dev/sr0 \| grep -E '(WWN\|SERIAL)'

E: ID_SERIAL=QEMU_DVD-ROM_QM00002

E: ID_SERIAL_SHORT=QM00002
```

2. Configure udev rules. Create the /etc/udev/rules.d/99-scheduler.rules file with the following content:

    NOTE: Replace IDNAME with the name of the identifier you want to use (such as ID_WWN)
    
    Replace device system unique id with the selected identifier value (eg 0x5002538d00000000)
```
ACTION=="add\|change", SUBSYSTEM=="block", ENV{ID_SERIAL_SHORT}=="QM00002", ATTR{queue/scheduler}="bfq"
```

3. Reload the udev rules:

```
[root@opencloudos ~]# udevadm control --reload-rules
```
4. Apply scheduler configuration:

```
[root@opencloudos ~]# udevadm trigger --type=devices --action=change
```

5. Verify the active scheduler:

```
[root@opencloudos ~]# cat /sys/block/device/queue/scheduler

[root@opencloudos ~]# cat /sys/block/sr0/queue/scheduler

mq-deadline kyber [bfq] none
```
### 2.8 Temporarily set the scheduler for a specific disk

This process sets a given disk scheduler for a specific block device. This setting will not be retained after system restart.

1. Write the name of the selected scheduler to the /sys/block/devices/queue/scheduler file:

```
[root@opencloudos ~]# echo selected-scheduler \> /sys/block/device/queue/scheduler
```
In the filename, replace device with the block device name, such as vda

2. Verify that the scheduler is active on the device:

```
[root@opencloudos ~]# cat /sys/block/device/queue/scheduler

[root@opencloudos ~]# cat /sys/block/vda/queue/scheduler

[none] mq-deadline kyber bfq
```
## Chapter 3 CPU usage and tuning

### 3.1 Display CPU architecture information

1. Collect CPU architectural information, such as the number of CPUs, threads, cores, sockets, and NUMA nodes:

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

### 3.2 View CPU usage:

1. Show the performance of the CPU:

```
[root@opencloudos ~]# top

%Cpu(s): 0.2 us, 0.2 sy, 0.0 ni, 99.7 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
```

- user (usually abbreviated as us), stands for user mode CPU time. Note that it does not include nice time below, but includes guest time.
  -nice (usually abbreviated as ni), stands for low-priority user mode CPU time, that is, the CPU time when the nice value of the process is adjusted to be between 1-19. Note here that the value range of nice is -20 to 19, the larger the value, the lower the priority.
- system (often abbreviated as sy), stands for kernel-mode CPU time.
- idle (often abbreviated to id), which stands for idle time. Note that it does not include time waiting for I/O (iowait).
- iowait (often abbreviated to wa), stands for CPU time waiting for I/O.
- irq (often abbreviated to hi), stands for CPU time handling hard interrupts.
- softirq (often abbreviated to si), stands for CPU time handling softirqs.
- steal (usually abbreviated as st), which represents the CPU time occupied by other virtual machines when the system is running in a virtual machine.
- guest (usually abbreviated as guest), which represents the time to run other operating systems through virtualization, that is, the CPU time to run a virtual machine.
- guest_nice (often abbreviated as gnice), which represents when the virtual machine is running at low priority.

CPU usage, which is the percentage of total CPU time other than idle time

Formula: CPU Usage = 1 - Idle Time / Total CPU Time

2. View CPU usage:

    Install the sysstat tool:
```
[root@opencloudos ~]# yum install sysstat
```

View CPU usage
```
[root@opencloudos ~]# pidstat 1

Linux 5.4.119-19-0009.1 (VM-6-117-centos) 08/19/2022 \_x86_64\_ (2 CPU)

03:11:04 PM UID PID %usr %system %guest %wait %CPU CPU Command

03:11:05 PM 0 561098 1.00 0.00 0.00 0.00 1.00 0 barad_agent

03:11:05 PM UID PID %usr %system %guest %wait %CPU CPU Command

03:11:06 PM 0 561098 0.00 1.00 0.00 0.00 1.00 0 barad_agent
...
```

### 3.3 perf tool

1. Install the perf tool.  The perf userspace tool is a kernel-based interface to the Performance Counters (PCL) of the Linux subsystem. Perf is a powerful tool that uses a Performance Monitoring Unit (PMU) to measure, record, and monitor various hardware and software events.

```
[root@opencloudos ~]# yum install perf
```

2. Use perf top to analyze CPU usage. The perf top command is used for real-time system analysis, and its functionality is similar to the top utility. However, the top utility usually shows the CPU time used by a given process or thread, and perf top will show the CPU time used by each specific function.

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

### 3.4 perf investigates busy CPU

1. Count events that disable CPU count aggregation:

```
[root@opencloudos ~]# perf stat -a -A sleep seconds
```

Events such as the specified period:
```
[root@opencloudos ~]# perf stat -a -A -e cycles sleep seconds
```

2. Show CPU samples taken with perf report

    Prerequisite: A perf.data file is created in the current directory, which contains perf records.

```
[root@opencloudos ~]# perf report --sort cpu
Sort by CPU and command:
[root@opencloudos ~]# perf report --sort cpu,comm
```

### 3.5 Display specific CPU when profiling with perf top
```
[root@opencloudos ~]# perf top --sort cpu
Sort by CPU and command:
[root@opencloudos ~]# perf top --sort cpu,comm
```

### 3.6 Log reports and monitor specific CPUs

1. Record performance data in a specific CPU and generate a perf.data file

    Use a comma-separated list of CPUs
```
[root@opencloudos ~]# perf record -C 0,1 sleep seconds
Use a set of CPUs:
[root@opencloudos ~]# perf record -C 0-2 sleep seconds
```

2. Display the contents of the perf.data file for further analysis:

```
[root@opencloudos ~]# perf report
```

## Chapter 4 Memory Usage

### 4.1 View memory usage

1. Use the free command:

```
[root@opencloudos ~]# free
total used free shared buff/cache available

Mem: 1730432 375364 130400 255480 1224668 932144

Swap: 0 0 0
```

-   total indicates a total of 1.6G of memory
- used indicates the usage of physical memory, about 215M
- free means free memory
- shared means shared memory
- buff/cache indicates the amount of cache and buffer memory; the Linux system will cache many things to improve performance, and this part of memory can be released when necessary for other programs to use.
- available means available memory

Swap means swap memory

2. Use vmstat

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

Statistics on memory usage

3. View /proc/meminfo

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

The /proc directory is full of virtual files, including dynamic information related to the kernel and operating system. /proc/meminfo is the main interface to understand the memory usage of the Linux system. Our most commonly used commands such as "free" and "vmstat" are through it For data acquisition, /proc/meminfo contains much richer information than commands such as "free".

### 4.2 View physical memory information

1.dmidecode command

```
[root@opencloudos ~]# dmidecode -t 17
```
The displayed information includes memory size, type, bandwidth and other information.

### 4.3 Virtual memory

1. Introduction: Physical memory is limited (even if it supports hot swapping), non-sequential, and different CPU architectures have different organizations for physical memory. This makes it very complicated to use physical memory directly. In order to reduce the complexity of using memory, a virtual memory mechanism is introduced.
2. How to use: Through the virtual memory mechanism, each process thinks that it occupies all the memory. When the process accesses the memory, the operating system will convert the virtual memory address provided by the process into a physical address, and then go to the corresponding physical address to obtain data. .
3. Virtual memory not only solves the problem of multiple processes accessing memory conflicts through memory address translation, but also brings more benefits:

- Process memory management: memory integrity: Due to the "deception" of the process by virtual memory, each process thinks that the memory it acquires is a continuous address. When we write an application program, we don't need to consider the allocation of large blocks of addresses. We always think that the system has enough large blocks of memory.
- Data sharing: It is easier to share memory and data through virtual memory.

### 4.4 Segment page mechanism

There are two types of memory management mechanisms in modern operating systems: segment management and page management.

- Segmented memory management is to divide the memory into segments, and the starting address of each segment is the segment base address. During address mapping, the physical address is obtained by adding the segment base address to the logical address.
- Paging memory management, the memory is divided into pages of fixed length. When address mapping, a page table needs to be established first, and each item in the page table records the base address of this page. Through the page table, the page base address corresponding to the logical address is first found from the high-order part of the logical address, and then the final physical address is obtained by offsetting the page base address by a certain length. The length of the offset is determined by the low-order part of the logical address.
- Segment page memory management needs to be used together with the segment table and the page table. The segment table includes: the segment number, the starting address of the page table in the segment, and the page table includes: the page number and the offset in the page. Find the corresponding segment number item in the segment table through the segment number in the virtual address, and get the start address of the page table in the segment (that is, the page table corresponding to this segment) through the segment number item in the segment table. After getting After the page table corresponding to this segment of memory, the corresponding page table entry can be found through the page number in the segment in the virtual address to obtain the physical start address, and the corresponding physical address is obtained by adding the offset in the page in the virtual address.

### 4.5 Memory recovery

1. The goal of memory recycling: For the kernel, not all physical memory can participate in recycling. For example, if the code segment of the kernel is recycled by the kernel, the system will not be able to run normally. Therefore, the general kernel code segment, data segment, and kernel application The memory occupied by the kernel thread and the memory occupied by the kernel thread cannot be reclaimed, and the other memory can be the target we want to recycle.

2. Memory recovery timing:

- The kernel needs to provide enough memory for any sudden memory request at any time, so that the use of cache and other related memory will not make the remaining memory of the system stay in a low state for a long time.

- When there is an application larger than free memory, it will trigger forced memory recovery.


3. Recycling mechanism:

- For the first type, the Linux system has designed the kswapd background program. When the kernel allocates physical pages, due to the shortage of system memory, memory cannot be allocated under low water level conditions, so the kswapd kernel thread will be awakened to reclaim memory asynchronously. That is, the periodic memory recovery mechanism.

- For the second type, the Linux system will trigger direct reclaim (direct reclaim). When the kernel calls the page allocation function to allocate physical pages, due to the shortage of system memory, the allocation request cannot be satisfied, and the kernel will directly trigger the page reclaim mechanism to try to reclaim memory to solve the problem. That is, direct memory reclamation.
