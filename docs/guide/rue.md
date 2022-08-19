# 如意QoS用户指南

## 如意(RUE)接口说明
如意(RUE)定位为全场景混部系统底座，为混部底层提供资源隔离解决方案。
如意有如下特点：

- 为云而生：更适用云上环境，以容器为对象进行资源的调度。
- 多优先级：目前支持三档优先级，允许扩展更多优先级。
- 多种资源：对CPU、IO、网络、内存等多种服务器资源进行调度。
- 资源隔离：低优先级容器可以使用空闲资源，不会对高优先级容器造成影响。
- 配置方便：提供统一、易用的配置管理工具

以下将根据资源分类介绍RUE的API。


## CPU（调度、隔离）
### 全局开关
开启CPU QoS功能：
```
sysctl -w kernel.cpu_qos=1
```
关闭CPU QoS功能：
```
sysctl -w kernel.cpu_qos=0
```

### Per-cgroup开关
#### 绝对抢占
- 描述：高优先级容器能够绝对抢占低优先级容器
- 接口说明：
```
# 设置容器A所在cgroup为高优先级 (优先级0)
echo 0 > /sys/fs/cgroup/cpu/<Pod_A>/cgroup.priority

# 设置容器B所在cgroup为低优先级 (优先级7)
echo 7 > /sys/fs/cgroup/cpu/<Pod_B>/cgroup.priority
```

#### 低优先级容器share
- 描述：多个低优先级容器之间的CPU比例设置 (与CFS share机制相同)
- 接口说明：
```
# 设置容器A和容器B为低优先级容器
echo 7 > /sys/fs/cgroup/cpu/<Pod_A>/cgroup.priority
echo 7 > /sys/fs/cgroup/cpu/<Pod_B>/cgroup.priority

# 设置低优先级容器A所在cgroup为shares为2048
echo 2048 > /sys/fs/cgroup/cpu/<Pod_A>/cpu.bt_shares

# 设置低优先级容器B所在cgroup为shares为1024 
echo 1024 > /sys/fs/cgroup/cpu/<Pod_B>/cpu.bt_shares
```
当CPU繁忙时, 低优先级容器A和低优先级容器B获得的CPU比例约为2:1. 当CPU空闲时, 该设置不生效 (与CFS shares机制相同)

#### 低优先级容器CPU限额
- 描述：限制低优先级容器在一定周期内所能使用CPU的时间上限
- 接口说明：
```
# 设置容器A为低优先级容器
echo 7 > /sys/fs/cgroup/cpu/test/<Pod_A>/cgroup.priority
```
低优先级容器CPU限额是基于高优先级容器限额的比例值, 以达到如当高优和低优整体CPU利用率达到50%以上的时候, 低优先级容器被限流无法运行, 而高优能继续运行的效果。
    
```
# 设置容器A的父cgroup的限额为100% CPU
echo 100000 > /sys/fs/cgroup/cpu/test/cpu.cfs_period_us
echo 100000 > /sys/fs/cgroup/cpu/test/cpu.cfs_quota_us

# 设置低优容器的限额值为高优的50%
echo 50 > /sys/fs/cgroup/cpu/test/cpu.bt_suppress_percent
```
此时, 运行在/sys/fs/cgroup/cpu/test下的所有容器能跑满100%的CPU(每个100000ms周期内可以跑100000ms), 而低优容器所能跑的限额为50%, 即每个100000ms周期内可以跑50000ms, 因此低优容器的限额被设置为了50%CPU.

#### 超线程隔离
- 描述：避免运行在一对超线程上的低优先级容器进程对高优先级容器进程产生干扰
- 接口说明：
```
# 设置容器A为高优先级容器
echo 0 > /sys/fs/cgroup/cpu/<Pod_A>/cgroup.priority
    
# 设置容器B为低优先级容器
echo 7 > /sys/fs/cgroup/cpu/<Pod_B>/cgroup.priority
    
# 设置高优先级容器A为超线程敏感
echo 1 > /sys/fs/cgroup/cpu/<Pod_A>/cpu.ht_sensi_type
```
假设CPU 0和CPU 40为物理核0上的一对超线程, 当高优先级容器A的进程运行在CPU 0上时, 如果CPU 40上运行的是低优先级容器B的进程, 那么CPU 40上的低优先级容器进程会在高优先级容器进程运行期间不会被调度执行, 达到超线程隔离的目的



## 网络
### 全局开关
开启网络QoS功能：
```
sysctl -w net.core.net_qos=1
```
关闭网络QoS功能：
```
sysctl -w net.core.net_qos=0
```

### Per-cgroup开关
#### 容器入方向限速
- 描述：容器网络入方向限速，限制容器的入方向速率不超过设置值
- 接口说明：
```
# 设置容器A所在cgroup的入方向最大带宽为50M
echo "rx_bps=50" >  /sys/fs/cgroup/net_cls/<POD A>/net_cls.limit
```
容器A 的入方向最大流量被限制在50Mbps

#### 容器出方向限速
- 描述：容器网络出方向限速，限制容器的出方向速率不超过设置值
- 接口说明：
```
# 设置容器A所在cgroup的出方向最大带宽为50M
echo "tx_bps=50" >  /sys/fs/cgroup/net_cls/<POD A>/net_cls.limit
```
容器A 的出方向最大流量被限制在50Mbps

#### 入方向绝对抢占
- 描述：入方向高优先级容器能够绝对抢占低优先级容器
- 接口说明：
```
# 配置网卡ethX的带宽限速，设置离线容器入方向的最低保证带宽为100Mbps，最大使用带宽为1000Mbps
echo  "<ethX> rx_bps_min=100 rx_bps_max=1000 tx_bps_min=100 tx_bps_max=1000"  > /sys/fs/cgroup/net_cls/net_cls.dev_bps_config

# 设置离线容器A 绝对优先级大于离线容器B
echo 1 > /sys/fs/cgroup/net_cls/<POD A>/cgroup.priority
echo 7 > /sys/fs/cgroup/net_cls/<POD B>/cgroup.priority
```
所有离线容器在入方向的最大使用带宽为1000Mbps，最低保障带宽为100Mbps
入方向的高优先级容器A会抢占低优先级容器B的带宽


#### 出方向绝对抢占
- 描述：出方向高优先级容器能够绝对抢占低优先级容器
- 接口说明：
```
# 配置网卡ethX的带宽限速，设置离线容器出方向的最低保证带宽为100Mbps，最大使用带宽为1000Mbps
echo  "<ethX> rx_bps_min=100 rx_bps_max=1000 tx_bps_min=100 tx_bps_max=1000" > /sys/fs/cgroup/net_cls/net_cls.dev_bps_config

# 设置离线容器A 绝对优先级大于离线容器B
echo 1 > /sys/fs/cgroup/net_cls/<POD A>/cgroup.priority
echo 7 > /sys/fs/cgroup/net_cls/<POD B>/cgroup.priority
```
所有离线容器在出方向的最大使用带宽为1000Mbps，最低保障带宽为100Mbps
出方向的高优先级容器A会抢占低优先级容器B的带宽

#### 容器端口白名单
- 描述：设置容器内特定端口为白名单端口，不做限流
- 接口说明：
```
# 设置容器A入方向最大带宽为50M
echo "rx_bps=50" >  /sys/fs/cgroup/net_cls/<POD A>/net_cls.limit

# 设置容器A内本地端口2000和远端端口2001为白名单，不做限流
echo "lports=2000,rports=2001" > /sys/fs/cgroup/net_cls/<POD A>/net_cls.whitelist_ports
```
容器A 的入方向带宽限速为50Mbps，本地端口2000和远端端口2001被设置为白名单端口，不会受到50Mbps的流量限制



## IO
### 全局开关
IO测试之前，需要对被测试磁盘开启bfq调度器并配置bfq
```
echo bfq > /sys/block/${disk}/queue/scheduler
```

针对多容器场景下，提升调度均衡性
```
echo 0 > /sys/block/${disk}/queue/iosched/low_latency
echo 1 > /sys/block/${disk}/queue/iosched/better_fairness
echo 50 > /sys/block/${disk}/queue/iosched/timeout_sync
```
增大磁盘队列深度
```
echo 1000 >  /sys/block/${disk}/queue/nr_requests
```

cgroup V1中，为实现buffer IO限速，需要**显式绑定**memcg与blkcg的关系，如下命令:
```
echo "path of blkcg" > memcg.bind_blkio
```

具体配置方法：

一个 pod 处于如下 memory 和 blkio 两个 cgroup: /sys/fs/cgroup/memory/A 和
/sys/fs/cgroup/blkio/A

则配置命令为：
```
echo /sys/fs/cgroup/blkio/A > /sys/fs/cgroup/memory/A/memcg.bind_blkio
```

### Per-cgroup开关
#### BPS限速
- 描述：设置该blk cgroup的读写BPS上限
- 接口说明：
```
配置接口：
1. blkio.throttle.read_bps_device 读bps上限
2. blkio.throttle.write_bps_device 写bps 上限
3. blkio.throttle.readwrite_bps_device  读写bps上限
    
配置方式：
echo MAJ:MIN BPS > blkio.throttle.xxx_bps_device

如：配置cgroup 对sda 盘的写BPS 最大为10MB/s
echo 253:32 10485760 > blkio.throttle.write_bps_device

注意：三个接口可以同时进行配置，且同时生效
```

#### IOPS 限速
- 描述：设置该blk cgroup的读写IOPS上限
- 接口说明：
```
配置接口：
1. blkio.throttle.read_iops_device 读iops上限
2. blkio.throttle.write_iops_device 写iops上限
3. blkio.throttle.readwrite_iops_device  读写iops上限

配置方式：
echo MAJ:MIN IOPS > blkio.throttle.xxx_iops_device

如：配置cgroup 对sda 盘的写IOPS 最大为1000
echo 253:32 1000 > blkio.throttle.write_iops_device

注意：三个接口可以同时进行配置，且同时生效
```

#### 按权重分配IO带宽
- 描述：blk-cgroup按照权重派发IO
- 接口说明：
```
blkio.bfq.weight
用来配置同级同class 的cgroup 之间的带宽权重。

如下图所示，blk cgroup A 与 blk cgroup B之间按照1：2 的比例派发IO 进入 blk cgroup C，blk cgroup C与 blk cgroup D之间按照1：4 的比例派发进入驱动队列。
```

![](../assets/按权重分配IO带宽.png)

#### IO 绝对抢占
- 描述：通过配置，实现高优先级blk cgroup抢占低优先级blk cgroup 下发的IO
- 接口说明：
```
blkio.bfq.ioprio_class
按照优先级，可配置如下：
1. rt
2. be 
3. idle
配置之后，同级blk cgroup之间，高优先级blk cgroup 优先向父blk cgroup派发IO。

如下图所示，bfq调度器支持层级，每个IO需要从当前进程所处的blk cgroup开始，逐级向上调度，直到离开root cgroup，最终才被派发给驱动。
每个cgroup根据子cgroup 的class 等级进行派发，如cgroup A中产生IO 相对于B优先被调度进入cgroup C，之后C 和 D 之间继续按照class 等级将IO派发给root cgroup。
```

![](../assets/IO%20%E7%BB%9D%E5%AF%B9%E6%8A%A2%E5%8D%A0.png)


## MEMORY
### 全局开关
#### memory qos全局开关

开启Mem Qos:
```
sysctl -w vm.memory_qos=1
```

关闭Mem Qos:
```
sysctl -w vm.memory_qos=0
```
注：下面的memory qos功能均需要打开全局开关才能生效。

#### memory cgroup优先级
```
/sys/fs/cgroup/memory/<POD A>/cgroup.priority
```
表示pod的内存QoS优先级，取值范围0-7，值越小，优先级越高。默认为0(最高优先级)

### Per-cgroup开关
#### 全局分级水位
- 描述：根据优先级动态调整全局MIN水位，减少进入直接内存回收流程（全局）
- 接口说明：
```
1. 接口说明
水位分级接口
/sys/fs/cgroup/memory/<Pod>/memory.priority_wmark_ratio

2. 取值
/sys/fs/cgroup/memory/<Pod>/memory.priority_wmark_ratio = {-75~75}
-75~0：表示在线服务，会根据watermark[MIN] 按照百分比降低水位
0~75 ：表示离线服务，根据watermark[low]-watermark[MIN] 差值，按照百分比抬升水位，在free不满足的条件下，会根据比例进行 限速；

3. 举例
# 设置pod A 为在线pod , 水位为-16；设置pod B 为离线pod，水位为60；
echo -60 > /sys/fs/cgroup/memory/<Pod_A>/memory.priority_wmark_ratio
echo 60  > /sys/fs/cgroup/memory/<Pod_B>/memory.priority_wmark_ratio
```


#### 优先级OOM/group OOM
- 描述：memcg内部根据优先级Kill进程
- 接口说明：
```
1.  接口说明
# 优先级OOM
/sys/fs/cgroup/memory/<Pod>/memory.use_priority_oom
# group OOM
/sys/fs/cgroup/memory/<Pod>/memory.oom.group

2. 取值
memory.use_priority_oom = 1表示开启优先级OOM
memory.oom.group = 1 表示开启group OOM

3. 举例
# 对pod A开启优先级OOM
echo  1 > /sys/fs/cgroup/memory/<Pod_A>/memory.use_priority_oom
# 对pod A开启group OOM
echo  1 > /sys/fs/cgroup/memory/<Pod_A>/memory.group.oom
```

#### 内存异步回收
- 描述：memcg内部异步回收内存，减少直接回收的概率
- 接口说明：
```
1. 接口说明
# 异步回收水位设置
/sys/fs/cgroup/memory/<pod_A>/memory.async_ratio
# 异步回收步长设置（代表memcg异步回收开始水线async_high和结束水线async_low之间的distance，值越大回收力度越大）
/sys/fs/cgroup/memory/<pod_A>/memory.async_distance_factor

2. 取值
/sys/fs/cgroup/memory/<pod_A>/memory.async_ratio ={ 0-100 }
/sys/fs/cgroup/memory/<pod_A>/memory.async_distance_factor = { 0-150000}

3. 举例
# 设置pod A 异步回收水位为90 （memcg内存使用超过90%触发异步回收）
echo 90 > /sys/fs/cgroup/memory/<pod_A>/memory.async_ratio
# 设置pod A 回收步长为20
echo 20 > /sys/fs/cgroup/memory/<pod_A>/memory.async_distance_factor
```

#### 全局内存预留
- 描述：全局预留部分内存，可以保证高优内存分配latency，同时限制低优内存分配速度
- 接口说明：
```
1. 接口说明
全局内存预留功能开关：
sysctl参数：vm.qos_prio_reclaim
回收优先级设置：
sysctl参数：vm.qos_highest_reclaim_prio
内存预留百分比：
sysctl参数：vm.qos_prio_reclaim_ratio

2. 取值：
vm.qos_prio_reclaim=[0,1]
vm.qos_highest_reclaim_prio=[1-7]
vm.qos_prio_reclaim_ratio=[1-20]

3. 举例
# 空闲内存不够时，从优先级5开始回收内存，优先级为5、6、7的容器的内存可能被回收
sysctl -w vm.qos_highest_reclaim_prio=5
# 预留全部内存的10%
sysctl -w vm.qos_prio_reclaim_ratio=10
```


欢迎贡献OpenCloudOS！
