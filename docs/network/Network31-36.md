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



## 32.2.设置 NetworkManager DNS 默认服务器优先级值

NetworkManager 为连接使用以下 DNS 优先级默认值：

- 50 用于 VPN 连接
- 100 用于其他连接

这部分论述了如何使用 IPv4 和 IPv6 连接的自定义默认值覆盖这些系统范围的默认值。

**流程**

1. 编辑 /etc/NetworkManager/NetworkManager.conf 文件：
   1. 添加 [connection] 部分（如果不存在）：
    ```
    [connection]
    ```
   2. 将自定义默认值添加到 [connection] 部分。例如，要将 IPv4 和 IPv6 的新默认值设为 200，请添加：
    ```
    ipv4.dns-priority=200
    ipv6.dns-priority=200
    ```
    您可以将参数设为 -2147483647 和 2147483647 之间的值。请注意，将参数设置为 0 将启用内置的默认值（对于 VPN 连接为 50 ，对于其他连接为 100 ）。

2. 重新载入 NetworkManager 服务：
    ```
    # systemctl reload NetworkManager
    ```

## 32.3.设置 NetworkManager 连接的 DNS 优先级

本节介绍了如何在 NetworkManager 创建或更新 /etc/resolv.conf 文件时定义 DNS 服务器的顺序。

请注意，只有在您配置了多个不同 DNS 服务器的多个连接时，设置 DNS 优先级才有意义。如果您只有一个配置了多个 DNS 服务器的连接，请在连接配置集中按顺序手动设置 DNS 服务器。

**前提条件**

- 系统配置了多个 NetworkManager 连接。
- 系统在 /etc/NetworkManager/NetworkManager.conf 文件中未设置 dns 参数，或者该参数被设为了 default。

**流程**

1. 可选，显示可用的连接：
    ```
    # nmcli connection show
    NAME           UUID                                  TYPE      DEVICE
    Example_con_1  d17ee488-4665-4de2-b28a-48befab0cd43  ethernet  enp1s0
    Example_con_2  916e4f67-7145-3ffa-9f7b-e7cada8f6bf7  ethernet  enp7s0
    ...
    ```
2. 设置 ipv4.dns-priority 和 ipv6.dns-priority 参数。例如，对于 
Example_con_1 连接，将两个参数都设为 10 ：
    ```
    # nmcli connection modify Example_con_1 ipv4.dns-priority 10 ipv6.dns-priority 10
    ```
3. 重新激活您更新的连接：
    ```
    # nmcli connection up Example_con_1
    ```

**验证**

- 显示 /etc/resolv.conf 文件的内容以验证 DNS 服务器的顺序是否正确：
    ```
    # cat /etc/resolv.conf
    ```

# 第33章 使用 ifcfg 文件配置 ip 网络

本章介绍了如何通过编辑 ifcfg 文件来手动配置网络接口。

NetworkManager 支持以 keyfile 格式存储的配置集。但是，在使用 NetworkManager API 创建或更新配置文件时，NetworkManager 默认使用 ifcfg 格式。

接口配置(ifcfg)文件可控制各个网络设备的软件接口。当系统引导时，它使用这些文件来决定启动哪些界面以及如何进行配置。这些文件通常被命名为 ifcfg-name，其中后缀 name 指的是配置文件控制的设备的名称。按照惯例，ifcfg 文件的后缀与配置文件中 DEVICE 指令提供的字符串相同。

## 33.1.使用 ifcfg 文件配置带有静态网络设置的接口

本节介绍了如何使用 ifcfg 文件配置网络接口。

**流程**

- 如果需要为名为 enp1s0 的接口配置具有静态网络设置的接口，请在 /etc/sysconfig/network-scripts/ 目录中创建一个名为 ifcfg-enp1s0 的文件，其包含以下内容：
  - 对于 IPv4 配置：
    ```
    DEVICE=enp1s0
    BOOTPROTO=none
    ONBOOT=yes
    PREFIX=24
    IPADDR=10.0.1.27
    GATEWAY=10.0.1.1
    ```
  - 对于 IPv6 配置：
    ```
    DEVICE=enp1s0
    BOOTPROTO=none
    ONBOOT=yes
    IPV6INIT=yes
    IPV6ADDR=2001:db8:1::2/64
    ```

## 33.2.使用 ifcfg 文件配置带有动态网络设置的接口

本节介绍了如何使用 ifcfg 文件配置具有动态网络设置的网络接口。

**流程**

1. 如果需要为名为 em1 的接口配置具有动态网络设置的接口，请在 /etc/sysconfig/network-scripts/ 目录中创建一个名为 ifcfg-em1 的文件，其包含以下内容：
    ```
    DEVICE=em1
    BOOTPROTO=dhcp
    ONBOOT=yes
    ```

2. 要配置发送的接口：
 - 如果 DHCP 服务器使用不同的主机名，请在 ifcfg 文件中添加以下行：
    ```
    DHCP_HOSTNAME=hostname
    ```
 - DHCP 服务器使用不同的完全限定域名(FQDN)，请在 ifcfg 文件中添加以下行：
    ```
    DHCP_FQDN=fully.qualified.domain.name
    ```

注意，您只能使用这些设置中的一个。如果您同时指定了 DHCP_HOSTNAME 和 DHCP_FQDN，则只使用 DHCP_FQDN。

3. 要将接口配置为使用特定的 DNS 服务器，请在 ifcfg 文件中添加以下行：
    ```
    PEERDNS=no
    DNS1=ip-address
    DNS2=ip-address
    ```
    其中 ip-address 是 DNS 服务器的地址。这会导致网络服务使用指定的 DNS 服务器更新 /etc/resolv.conf。只需要一个 DNS 服务器地址，另一个是可选的。

## 33.3.使用 ifcfg 文件管理系统范围和专用的连接配置集

本节描述了如何配置 ifcfg 文件来管理系统范围和私有的连接配置文件。

**流程**
访问权限对应 ifcfg 文件中的 USERS 指令。如果没有 USERS 指令，则网络配置文件对所有用户可用。

- 例如，使用以下行修改 ifcfg 文件，这将使连接仅对列出的用户可用：
    ```
    USERS="joe bob alice"
    ```

# 第34章 使用 NetworkManager 为特定连接禁用 IPv6

这部分描述了如何在使用 NetworkManager 管理网络接口的系统上禁用 IPv6 协议。如果您禁用了 IPv6，则 NetworkManager 会在内核中自动设置相应的 sysctl 值。如果使用内核可调参数或内核引导参数禁用 IPv6，则必须额外考虑系统配置。

**前提条件**
- 系统使用 NetworkManager 来管理网络接口

## 34.1.使用 nmcli 在连接中禁用 IPv6

**流程**

1. 可选，显示网络连接列表：
    ```
    # nmcli connection show
    NAME    UUID                                  TYPE      DEVICE
    Example 7a7e0151-9c18-4e6f-89ee-65bb2d64d365  ethernet  enp1s0
    ...
    ```
2. 显示网络连接列表：
    ```
    # nmcli connection modify Example ipv6.method "disabled"
    ```
3. 重启网络连接：
    ```
    # nmcli connection up Example
    ```

**验证**

1. 输入 ip address show 命令来显示设备的 IP 设置：
    ```
    # ip address show enp1s0
    2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 52:54:00:6b:74:be brd ff:ff:ff:ff:ff:ff
        inet 192.0.2.1/24 brd 192.10.2.255 scope global noprefixroute enp1s0
        valid_lft forever preferred_lft forever
    ```
    如果没有显示 inet6 条目，则 IPv6 在该设备上被禁用。

2. 验证 /proc/sys/net/ipv6/conf/enp1s0/disable_ipv6 文件现在是否包含值 1 :
    ```
    # cat /proc/sys/net/ipv6/conf/enp1s0/disable_ipv6
    1
    ```
    值 1 表示针对该设备禁用 IPv6。

# 第35章 手动配置 /etc/resolv.conf 文件

默认情况下，OpenCloudOS 上的 NetworkManager 使用连接配置文件中的 DNS 设置动态更新 /etc/resolv.conf 文件。本章介绍了如何禁用此特性，来在 /etc/resolv.conf 中手动配置 DNS 设置。

## 35.1.在 NetworkManager 配置中禁用 DNS 处理

本节介绍了如何在 NetworkManager 配置中禁用 DNS 处理来手动配置 /etc/resolv.conf 文件。

**流程**

1. 以 root 用户身份，使用文本编辑器创建包含以下内容的 /etc/NetworkManager/conf.d/90-dns-none.conf 文件：
    ```
    [main]
    dns=none
    ```

2. 重新载入 NetworkManager 服务：
    ```
    # systemctl reload NetworkManager
    ```
    注意，重新加载服务后，NetworkManager 不再更新 /etc/resolv.conf 文件。但是该文件的最后内容将被保留。

3. （可选）从 /etc/resolv.conf 中删除 NetworkManager 生成的 注释，方便阅读。

**验证**

1. 编辑 /etc/resolv.conf 文件并手动更新配置。
2. 重新载入 NetworkManager 服务：
    ```
    # systemctl reload NetworkManager
    ```
3. 显示 /etc/resolv.conf 文件：
    ```
    # cat /etc/resolv.conf
    ```
    如果您成功禁用了 DNS 处理，NetworkManager 不会覆盖手动配置的设置。

## 35.2.使用符号链接替换 /etc/resolv.conf 来手动配置 DNS 设置

如果 /etc/resolv.conf 是符号链接，则 NetworkManager 不会自动更新 DNS 配置。本节描述了如何将带有符号链接的 /etc/resolv.conf 替换成带有 DNS 配置的文件。

**前提条件**

- rc-manager 选项没有设为 file。要验证，请使用 NetworkManager --print-config 命令。

**流程**

1. 创建一个文件，如 /etc/resolv.conf.manually-configured，并将您环境的 DNS 配置添加到其中。使用与原始 /etc/resolv.conf 中一样的参数和语法。
2. 删除 /etc/resolv.conf 文件：
    ```
    # rm /etc/resolv.conf
    ```
3. 创建名为 /etc/resolv.conf 的符号链接，该链接指向 /etc/resolv.conf.manually-configured ：
    ```
    # ln -s /etc/resolv.conf.manually-configured /etc/resolv.conf
    ```

# 第36章 监控并调整网卡环缓冲

接收环形缓冲区是设备驱动程序和网络接口控制器 (NIC) 之间共享的共享缓冲区。该卡会分配一个发送 (TX) 和接收 (RX) 环形缓冲区。顾名思义，环形缓冲区是一个循环缓冲区，溢出数据会覆盖现有数据。将数据从 NIC 移动到内核有两种方法，硬件中断和软件中断，也称为 SoftIRQ。

内核使用 RX 环形缓冲区来存储传入的数据包，直到它们可以被设备驱动程序处理。设备驱动程序通常使用 SoftIRQ排空 RX 环，它将传入的数据包放入称为sk_buff或skb的内核数据结构中，数据包开始其通过内核的过程，直到到达拥有相关套接字的应用程序。

内核使用 TX 环形缓冲区来保存发往线路的传出数据包。

这些环形缓冲区位于堆栈的底部，是可能发生丢包的关键点，同时，这也会对网络性能产生不利影响。

## 36.1.显示丢弃的数据包数量

ethtool 工具可让管理员查询、配置或控制网络驱动程序设置。

RX 环缓冲的耗尽会导致计数器的递增，例如 ethtool -S interface_name 的输出中的 "discard" 或 "drop"。丢弃的数据包表示可用缓冲区的填满速度要快于内核可以处理数据包的速度。

本节描述了如何使用 ethtool 显示丢弃计数器。

**流程**

- 要查看 enp1s0 接口的丢弃计数器，请输入：
    ```
    $ ethtool -S enp1s0
    ```

## 36.2.增加 RX 环缓冲以降低数据包丢弃的比率

ethtool 工具可以帮助提高 RX 缓冲大小，以减少数据包的高丢弃率。

**流程**

1. 查看 RX 环缓冲的最大值：
    ```
    # ethtool -g enp1s0
    Ring parameters for enp1s0:
    Pre-set maximums:
    RX:             4080
    RX Mini:        0
    RX Jumbo:       16320
    TX:             255
    Current hardware settings:
    RX:             255
    RX Mini:        0
    RX Jumbo:       0
    TX:             255
    ```

2. 如果 Pre-set maximums 部分中的值大于 Current hardware settings 部分，请增加 RX 环缓冲：
    - 要临时将 enp1s0 设备的 RX 环缓冲改为 4080，请输入：
        ```
        # ethtool -G enp1s0 rx 4080
        ```
    - 要永久更改 RX 环缓冲，请创建一个 NetworkManager 分配程序脚本。

注意，根据您的网卡使用的驱动，环缓冲的改变会很快中断网络连接。

