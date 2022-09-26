# 第31章 Linux 流量控制

Linux 提供管理和操作数据包传输的工具。Linux 流量控制（TC）子系统帮助进行策略、分类、控制以及调度网络流量。TC 还可以通过使用过滤器和动作在分类过程中利用数据包内容分栏。TC 子系统通过排队规则(qdisc)，TC 架构的基本元素，来实现此目的。

调度机制在进入或退出不同的队列前确定或者重新安排数据包。最常见的调度程序是先入先出（FIFO）调度程序。您可以使用 tc 工具临时执行 qdiscs 操作，也可以使用 NetworkManager 永久执行操作。

本章解释了排队规则，并描述了如何在 OpenCloudOS 中更新默认的 qdiscs。

## 31.1.排队规则概述

排队规则(qdiscs)会帮助规划排队等候，之后通过网络接口调度流量传输。qdisc 有两个操作：
- 排队请求，以便在以后传输时对数据包进行排队
- 出队请求，以便可以选择其中一个排队的数据包进行即时传输。

每个 qdisc 都有一个 16 位十六进制标识数字，称为 handle ，带有一个附加的冒号，如 1: 或 abcd：这个数字被称为 qdisc 主号码。如果 qdisc 有类型，则标识符是由两个数字组成的对，主号码在次要号码之前，即 <major>:<minor>，例如 abcd:1。次要号码的编号方案取决于 qdisc 类型。有时，编号是系统化的，其中第一类的 ID 为 < <major>:1，第二类的 ID 为 <major>:2 ，等等。一些 qdiscs 允许用户在创建类时随机设置类的次要号码。

**Classful qdiscs**

存在不同类型的 qdiscs，帮助从网络接口接收和传输数据包。您可以使用根类、父类或子类配置 qdiscs。子对象可以被附加的位置称为类。qdisc 中的类是灵活的，始终包含多个子类或一个子类 qdisc。不禁止包含类 qdisc 本身的类，这有助于实现复杂的流量控制场景。

类 qdiscs 本身不存储任何数据包。相反，它们根据指定于 qdisc 的标准，将排队和出队请求传到它们其中的一个子类。最后，这个递归数据包传递到最终保存数据包的位置（或在出现排队时从中提取）。

**Classless qdiscs**

有些 qdiscs 不包含子类，它们称为无类别 qdiscs。与类 qdiscs 相比，无类别 qdiscs 需要较少的自定义。通常情况下，将它们附加到接口就足够了。

## 31.2. OpenCloudOS 中可用的 qdiscs

每个 qdisc 解决唯一对应的与相关网络问题。以下是 OpenCloudOS 中可用的 qdiscs 列表。您可以使用以下任何一个 qdisc 来根据您的网络要求来塑造网络流量。

| qdisc 名称| 所处模块 | 卸载支持 |
|-----------|----------|----------|
| 异步传输模式(ATM) |kernel-modules-extra|   |
| 基于类的队列 |kernel-modules-extra|   |
| 基于信用的塑造程序 |kernel-modules-extra| 是  |
|CHOose 和 Keep 用于有响应的流量，CHOose 和 Kill 用于没有响应的流量（CHOKE）|kernel-modules-extra|   |
|受控的延迟（CoDel）|kernel-core|   |
|轮循(DRR)|kernel-modules-extra|   |
|Differentiated Services marker (DSMARK)|kernel-modules-extra|   |
|Enhanced Transmission Selection (ETS)|kernel-modules-extra| 是  |
|Fair Queue (FQ)|kernel-core|   |
|Fair Queuing Controlled Delay (FQ_CODel)|kernel-core|   |
|Generalized Random Early Detection (GRED)|kernel-modules-extra|   |
|Hierarchical Fair Service Curve (HSFC)|kernel-core|   |
|Heavy-Hitter Filter (HHF)|kernel-core|   |
|Hierarchy Token Bucket (HTB)|kernel-core|   |
|INGRESS|kernel-core| 是  |
|Multi Queue Priority (MQPRIO)|kernel-modules-extra| 是  |
|Multiqueue (MULTIQ)|kernel-modules-extra| 是  |
|Network Emulator (NETEM)|kernel-modules-extra|   |
|Proportional Integral-controller Enhanced (PIE)|kernel-core|   |
|PLUG|kernel-core|   |
|Quick Fair Queueing (QFQ)|kernel-modules-extra|   |
|Random Early Detection (RED)|kernel-modules-extra| 是  |
|Stochastic Fair Blue (SFB)|kernel-modules-extra|   |
|Stochastic Fairness Queueing (SFQ)|kernel-core|   |
|Token Bucket Filter (TBF)|kernel-core| 是  |
|Trivial Link Equalizer (TEQL)|kernel-modules-extra|   |
||||

注意，qdisc 卸载需要对 NIC 的硬件和驱动程序的支持。

## 31.3.使用 tc 工具检查网络接口的 qdiscs

默认情况下，OpenCloudOS 系统使用 fq_codel 、 qdisc。本节流程描述了如何检查 qdisc 计数器。

**流程**

1. 可选：查看您当前的 qdisc ：
    ```
    # tc qdisc show dev enp0s1
    ```
2. 检查当前的 qdisc 计数器：
    ```
    # tc -s qdisc show dev enp0s1
    qdisc fq_codel 0: root refcnt 2 limit 10240p flows 1024 quantum 1514 target 5.0ms interval 100.0ms memory_limit 32Mb ecn
    Sent 1008193 bytes 5559 pkt (dropped 233, overlimits 55 requeues 77)
    backlog 0b 0p requeues 0
    ....
    ```
    - dropped - 由于所有队列已满而丢弃数据包的次数
    - overlimits - 配置的链路容量已满的次数
    - sent - 出队的数量

## 31.4.更新默认的 qdisc

**流程**

1. 查看当前的默认 qdisc ：
    ```
    # sysctl -a | grep qdisc
    net.core.default_qdisc = fq_codel
    ```
2. 查看当前以太网连接的 qdisc ：
    ```
    # tc -s qdisc show dev enp0s1
    qdisc fq_codel 0: root refcnt 2 limit 10240p flows 1024 quantum 1514 target 5.0ms interval 100.0ms memory_limit 32Mb ecn
    Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
    backlog 0b 0p requeues 0
    maxpacket 0 drop_overlimit 0 new_flow_count 0 ecn_mark 0
    new_flows_len 0 old_flows_len 0
    ```
3. 更新现有的 qdisc ：
    ```
    # sysctl -w net.core.default_qdisc=pfifo_fast
    ```
4. 重新加载网络驱动程序应用更改：
    ```
    # rmmod NETWORKDRIVERNAME
    # modprobe NETWORKDRIVERNAME
    ```
5. 启动网络接口：
    ```
    # IP link set enp0s1 up
    ```

**验证**

- 查看以太网连接的 qdisc ：
    ```
    # tc -s qdisc show dev enp0s1
    qdisc pfifo_fast 0: root refcnt 2 bands 3 priomap  1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
    Sent 373186 bytes 5333 pkt (dropped 0, overlimits 0 requeues 0)
    backlog 0b 0p requeues 0
    ....
    ```

## 使用 tc 工具临时设置网络接口的 qdisk

您可以更新当前临时的 qdisc 而不更改默认的 qdisc 。

**流程**

1. 可选：查看当前的 qdisc ：
    ```
    # tc -s qdisc show dev enp0s1
    ```
2. 更新当前的 qdisc ：
    ```
    # tc qdisc replace dev enp0s1 root htb
    ```

**验证**

- 查看更新后的当前 qdisc ：
    ```
    # tc -s qdisc show dev enp0s1
    qdisc htb 8001: root refcnt 2 r2q 10 default 0 direct_packets_stat 0 direct_qlen 1000
    Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
    backlog 0b 0p requeues 0
    ```

## 31.6.使用 NetworkManager 永久设置网络接口的当前 qdisk

**流程**

1. 可选：查看当前的 qdisc ：
    ```
    # tc qdisc show dev enp0s1
     qdisc fq_codel 0: root refcnt 2
    ```
2. 更新当前的 qdisc ：
    ```
    # nmcli connection modify enp0s1 tc.qdiscs ‘root pfifo_fast’
    ```
3. 可选：要在现有的 qdisc 上添加另一个 qdisc，请使用 +tc.qdisc 选项：
    ```
    # nmcli connection modify enp0s1 +tc.qdisc ‘ingress handle ffff:’
    ```
4. 激活更改：
    ```
    # nmcli connection up enp0s1
    ```

**验证**

- 查看更新后的当前 qdisc ：
    ```
    # tc qdisc show dev enp0s1
    qdisc pfifo_fast 8001: root refcnt 2 bands 3 priomap  1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
    qdisc ingress ffff: parent ffff:fff1 ----------------
    ```

# 第32章 配置DNS服务器顺序

大多数应用程序使用 glibc 库的 getaddrinfo（） 函数来解析 DNS 请求。默认情况下，glibc 将所有 DNS 请求发送到 /etc/resolv.conf 文件中指定的第一个 DNS 服务器。如果这个服务器没有回复，OpenCloudOS 会使用这个文件中的下一个服务器。

本章介绍了如何自定义 DNS 服务器顺序。

## 32.1.使用 NetworkManager 在 /etc/resolv.conf 中对 DNS 服务器进行排序

NetworkManager 根据以下规则对 /etc/resolv.conf 文件中的 DNS 服务器进行排序：
- 如果只有一个连接配置集，NetworkManager 将使用那个连接中指定的 IPv4 和 IPv6 DNS 服务器顺序。
- 如果激活多个连接配置集，NetworkManager 会根据 DNS 优先级值对 DNS 服务器进行排序。如果您设置了 DNS 优先级，NetworkManager 的行为取决于 dns 参数中设置的值。您可以在 /etc/NetworkManager/NetworkManager.conf 文件的 [main] 部分中设置此参数：

  - dns=default 或者如果 dns 参数没有设置：

    NetworkManager 根据每个连接中的 ipv4.dns-priority 和 ipv6.dns-priority 参数对不同的连接的 DNS 服务器进行排序。

    如果没有设置值，或者您将 ipv4.dns-priority 和 ipv6.dns-priority 设为 0 ，则 NetworkManager 将使用全局默认值。请参阅 DNS 优先级参数的默认值。

  - dns=dnsmasq 或 dns=systemd-resolved ：

    当您使用这些设置中的一个时，NetworkManager 将 dnsmasq 的 127.0.0.1 或 127.0.0.53 设为 /etc/resolv.conf 文件中的 nameserver 条目。

    dnsmasq 和 systemd-resolved 服务将对 NetworkManager 连接中设置的搜索域的查询转发到该连接中指定的 DNS 服务器，并将对其他域的查询转发到带有默认路由的连接。当多个连接有相同的搜索域集时，dnsmasq 和 systemd-resolved 将这个域的查询转发到在具有最低优先级值的连接中设置的 DNS 服务器。

DNS 优先级参数的默认值
NetworkManager 对连接使用以下默认值：

- 50 用于 VPN 连接
- 100 用于其他连接


有效的 DNS 优先级值：
您可以将全局默认值和特定于连接的 ipv4.dns-priority 和 ipv6.dns-priority 参数设为 -2147483647 和 2147483647 之间的值。

- 低的值具有更高的优先级。
- 负值具有一个特殊的效果，它会排除其他带有更大值的配置。例如，如果至少有一个连接具有负优先级值，NetworkManager 只使用在连接配置集中指定的具有最低优先级的 DNS 服务器。
- 如果多个连接具有相同的 DNS 优先级，NetworkManager 会按照以下顺序排列 DNS 的优先顺序：

    1. VPN 连接
    2. 带有活跃的默认路由的连接。活跃的默认路由是具有最低指标的默认路由。


# 