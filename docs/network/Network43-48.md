# 第43章 在不同的接口上重复使用相同的 IP 地址

使用虚拟路由和转发(VRF)，管理员可以在同一主机上同时使用多个路由表。为此，VRF 将网络在第 3 层进行分区。这可让管理员使用每个 VRF 域的独立路由表隔离流量。这个技术与虚拟 LAN（虚拟 LAN）类似，后者在第二层对网络进行分区，其中操作系统使用不同的 VLAN 标签来隔离共享相同物理介质的流量。

与第二层上的分区相比，VRF 的一个优点是，考虑到所涉及的对等路由的数量，路由可以更好地扩展。

OpenCloudOS 为每个 VRF 域使用虚拟 vrt 设备，并通过向 VRF 设备添加现有网络设备来向 VRF 域添加路由。之前附加到原始设备的地址和路由将在 VRF 域中移动。

请注意，每个 VRF 域间都是相互隔离的

## 43.1.在不同接口上永久重复使用相同的 IP 地址

本节介绍了如何使用 VRF 功能在同一服务器的不同接口中永久使用相同的 IP 地址。

要在重新使用相同的 IP 地址时让远程对等两个 VRF 接口都能联系，网络接口必须属于不同的广播域。网络中的广播域是一组节点，它们接收其中任何一个节点发送的广播流量。在大多数配置中，所有连接到同一交换机的节点都属于相同的域。

**前提条件**

- 以 root 用户身份登录。
- 没有配置网络接口。

**流程**

1. 创建并配置第一个 VRF 设备：

    1. 为 VRF 设备创建连接并将其分配到路由表中。例如，要创建一个分配给 1001 路由表、名为 vrf0 的 VRF 设备：

        ```
        # nmcli connection add type vrf ifname vrf0 con-name vrf0 table 1001 ipv4.method disabled ipv6.method disabled
        ```
    2. 启用 vrf0 设备：

        ```
        # nmcli connection up vrf0
        ```
    3. 为刚刚创建的 VRF 分配网络设备。例如，要向 vrf0 VRF 设备添加 enp1s0 以太网设备，并向 enp1s0 分配 IP 地址和子网掩码，请输入：

        ```
        # nmcli connection add type ethernet con-name vrf.enp1s0 ifname enp1s0 master vrf0 ipv4.method manual ipv4.address 192.0.2.1/24
        ```
    4. 激活 vrf.enp1s0 连接：

        ```
        # nmcli connection up vrf.enp1s0
        ```
2. 创建并配置下一个 VRF 设备：

    1. 创建 VRF 设备并将其分配到路由表中。例如，要创建一个分配给 1002 路由表、名为 vrf1 的 VRF 设备，请输入：
        ```

        # nmcli connection add type vrf ifname vrf1 con-name vrf1 table 1002 ipv4.method disabled ipv6.method disabled
        ```
    2. 激活 vrf1 设备：
        ```

        # nmcli connection up vrf1
        ```
    3. 为刚刚创建的 VRF 分配网络设备。例如，要向 vrf1 VRF 设备添加 enp7s0 以太网设备，并给 enp7s0 分配 IP 地址和子网掩码，请输入：

        ```
        # nmcli connection add type ethernet con-name vrf.enp7s0 ifname enp7s0 master vrf1 ipv4.method manual ipv4.address 192.0.2.1/24
        ```
    4. 激活 vrf.enp7s0 设备：

        ```
        # nmcli connection up vrf.enp7s0
        ```

## 43.2.在不同接口中临时重复使用相同的 IP 地址

本节介绍了如何使用虚拟路由和转发（VRF）功能在某个服务器的不同接口中临时使用相同的 IP 地址。这个过程仅用于测试目的，因为配置是临时的并在重启系统后会丢失。

要在重新使用相同的 IP 地址时让远程对等两个 VRF 接口都能联系，网络接口必须属于不同的广播域。广播域是一组节点，它们接收被其中任何一个发送的广播流量。在大多数配置中，所有连接到同一交换机的节点都属于相同的域。

**前提条件**

- 以 root 用户身份登录。
- 没有配置网络接口。

**流程**

1. 创建并配置第一个 VRF 设备：

    1. 创建 VRF 设备并将其分配到路由表中。例如，要创建一个分配给 1001 路由表、名为 blue 的 VRF 设备：

    ```
    # ip link add dev blue type vrf table 1001
    ```

    2. 启用 blue 设备：

    ```
    # ip link set dev blue up
    ```

    3. 为 VRF 设备分配网络设备。例如，要向 blue VRF 设备添加 enp1s0 以太网设备：

    ```
    # ip link set dev enp1s0 master blue
    ```

    4. 启用 enp1s0 设备：

    ```
    # ip link set dev enp1s0 up
    ```

    5. 向 enp1s0 设备分配 IP 地址和子网掩码。例如，将其设为 192.0.2.1/24 ：

    ```
    # ip addr add dev enp1s0 192.0.2.1/24
    ```

2. 创建并配置下一个 VRF 设备：

    1. 创建 VRF 设备并将其分配到路由表中。例如，要创建一个分配给 1002 路由表、名为 red 的 VRF 设备：

    ```
    # ip link add dev red type vrf table 1002
    ```

    2. 启用 red 设备：

    ```
    # ip link set dev red up
    ```

    3. 为 VRF 设备分配网络设备。例如，要向 red VRF 设备添加 enp7s0 以太网设备：

    ```
    # ip link set dev enp7s0 master red
    ```

    4. 启用 enp7s0 设备：

    ```
    # ip link set dev enp7s0 up
    ```

    5. 为 enp7s0 设备分配与 blue VRF 域中 enp1s0 设备所使用的相同的 IP 地址和子网掩码：

    ```
    # ip addr add dev enp7s0 192.0.2.1/24
    ```

3. 可选，还可按照上述步骤创建更多 VRF 设备。

# 第44章 在隔离的 VRF 网络内启动服务

使用虚拟路由和转发(VRF)，您可以使用与操作系统主路由表不同的路由表创建隔离网络。然后，您可以启动服务和应用程序，以便它们只能访问该路由表中定义的网络。

## 44.1.配置 VRF 设备

要使用虚拟路由和转发(VRF)，您可以创建一个 VRF 设备，并将物理或虚拟网络接口和路由信息附加给它。

要防止您将自己远程锁定，请在本地控制台中或通过您不想分配给 VRF 设备的网络接口远程执行此流程。

**前提条件**

- 您已在本地登录或使用与您要分配给 VRF 设备不同的网络接口。
  
**流程**

1. 使用同命的虚拟设备创建 vrf0 连接，并将其附加到路由表 1000 ：

    ```
    # nmcli connection add type vrf ifname vrf0 con-name vrf0 table 1000 ipv4.method disabled ipv6.method disabled
    ```

2. 向 vrf0 连接添加 enp1s0 设备，并配置 IP 设置：

    ```
    # nmcli connection add type ethernet con-name enp1s0 ifname enp1s0 master vrf0 ipv4.method manual ipv4.address 192.0.2.1/24 ipv4.gateway 192.0.2.254
    ```
    此命令会创建 enp1s0 连接，来作为 vrf0 连接的端口。由于此配置，路由信息会自动分配给与 vrf0 设备关联的路由表 1000。

3. 如果您在隔离网络中需要静态路由：

    1. 添加静态路由：

        ```
        # nmcli connection modify enp1s0 +ipv4.routes "198.51.100.0/24 192.0.2.2"
        ```
    这向 198.51.100.0/24 网络添加了一个路由，该网络使用 192.0.2.2 作为路由器。

    2. 激活连接：

    ```
    # nmcli connection up enp1s0
    ```

**验证**

1. 显示与 vrf0 关联的设备的 IP 设置：

    ```
    # ip -br addr show vrf vrf0
    enp1s0    UP    192.0.2.15/24
    ```

2. 显示 VRF 设备及其关联的路由表：

    ```
    # ip vrf show
    Name              Table
    -----------------------
    vrf0              1000
    ```

3. 显示主路由表：

    ```
    # ip route show
    default via 192.168.0.1 dev enp1s0 proto static metric 100
    ```

4. 显示路由表 1000 ：

    ```
    # ip route show table 1000
    default via 192.0.2.254 dev enp1s0 proto static metric 101
    broadcast 192.0.2.0 dev enp1s0 proto kernel scope link src 192.0.2.1
    192.0.2.0/24 dev enp1s0 proto kernel scope link src 192.0.2.1 metric 101
    local 192.0.2.1 dev enp1s0 proto kernel scope host src 192.0.2.1
    broadcast 192.0.2.255 dev enp1s0 proto kernel scope link src 192.0.2.1
    198.51.100.0/24 via 192.0.2.2 dev enp1s0 proto static metric 101
    ```
    default 条目表示使用此路由表的服务，将 192.0.2.254 用作其默认网关，而不是主路由表中的默认网关。

5. 在与 vrf0 关联的网络中执行 traceroute 工具，以验证工具是否使用表 1000 的路由：

    ```
    # ip vrf exec vrf0 traceroute 203.0.113.1
    traceroute to 203.0.113.1 (203.0.113.1), 30 hops max, 60 byte packets
    1  192.0.2.254 (192.0.2.254)  0.516 ms  0.459 ms  0.430 ms
    ...
    ```
    第一跳是分配给路由表 1000 的默认网关，而不是系统的主路由表中的默认网关。

## 44.2.在隔离的 VRF 网络内启动服务

您可以将服务（如 Apache HTTP 服务器）配置为在隔离的虚拟路由和转发(VRF)网络中启动。

注意，服务只能绑定到同一 VRF 网络中的本地 IP 地址。

**前提条件**

- 您已配置了 vrf0 设备。
- 您已将 Apache HTTP 服务器配置为仅侦听分配给与 vrf0 设备关联的接口的 IP 地址。

**流程**

1. 显示 httpd systemd 服务的内容：

    ```
    # systemctl cat httpd
    ...
    [Service]
    ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
    ...
    ```
    在后续步骤中需要 ExecStart 参数的内容，以在隔离的 VRF 网络中运行相同的命令。

2. 创建 /etc/systemd/system/httpd.service.d/ 目录：

    ```
    # mkdir /etc/systemd/system/httpd.service.d/
    ```

3. 使用以下内容创建 /etc/systemd/system/httpd.service.d/override.conf 文件：

    ```
    [Service]
    ExecStart=
    ExecStart=/usr/sbin/ip vrf exec vrf0 /usr/sbin/httpd $OPTIONS -DFOREGROUND
    ```
    要覆盖 ExecStart 参数，您首先需要对其取消设置，然后将其设为所示的新值。

4. 重新加载 systemd 。

    ```
    # systemctl daemon-reload
    ```

5. 重新启动 httpd 服务。

    ```
    # systemctl restart httpd
    ```

**验证**

1. 显示 httpd 进程的进程 ID(PID)：

    ```
    # pidof -c httpd
    1904 ...
    ```

2. 显示 PID 的 VRF 关联，例如：

    ```
    # ip vrf identify 1904
    vrf0
    ```

3. 显示与 vrf0 设备关联的所有 PID：

    ```
    # ip vrf pids vrf0
    1904  httpd
    ...
    ```

# 第45章 为您的系统设置路由协议

这部分描述了如何使用 自由范围路由 （FRRouting 或 FRR）特性来为您的系统启用和设置所需的路由协议。

## 45.1.FRRouting 介绍

自由范围路由 （FRRouting 或 FRR）是一个路由协议堆栈，它由 AppStream 存储库中的 frr 软件包提供。

FRR 提供了基于 TCP/IP 的路由服务，并支持多个 IPv4 和 IPv6 路由协议。

支持的协议包括：

- 边界网关协议(BGP)
- 中间系统到中间系统(IS-IS)
- 首先打开最短路径(OSPF)
- 协议独立多播(PIM)
- 路由信息协议(RIP)
- 下一代路由信息协议(RIPng)
- 增强的内部网关路由协议(EIGRP)
- 下一跳解析协议(NHRP)
- 双向转发检测(BFD)
- 基于策略的路由(PBR)

FRR 是下列服务的集合：

- zebra
- bgpd
- isisd
- ospfd
- ospf6d
- pimd
- ripd
- ripngd
- eigrpd
- nhrpd
- bfdd
- pbrd
- staticd
- fabricd

如果安装了 frr，系统可以充当专用的路由器，其使用路由协议与内部或外部网络中的其他路由器交换路由信息。

## 45.2.设置 FRRouting

本节解释了如何设置自由范围路由（FRRouting 或 FRR）。

**前提条件**

- 确保在您的系统中安装了 frr 软件包：
    ```
    # yum install frr
    ```

**流程**

1. 编辑 /etc/frr/daemons 配置文件，并为您的系统启用所需的守护进程。

    例如，要启用 ripd 守护进程，请包含以下行：

    ```
    ripd=yes
    ```
    必须始终启用 zebra 守护进程，所以您必须设置 zebra=yes 才能使用 FRR。
    默认情况下，/etc/frr/daemons 包含所有守护进程的 [daemon_name]=no 条目。所有守护进程都被禁用，因此，在新安装系统后启动 FRR 无效。

2. 启动 frr 服务：

    ```
    # systemctl start frr
    ```

3. 另外，您也可以将 FRR 设为在引导时自动启动：

    ```
    # systemctl enable frr
    ```

## 45.3. 修改 FRR 的配置

本节介绍了：

- 设置 FRR后如何启用额外的守护进程
- 设置 FRR后如何禁用守护进程

**先决条件**

- FRR 已设置，如 设置 FRRouting 中所述。

**流程**

1. 编辑 /etc/frr/daemons 配置文件，并将所需守护进程的行更改为 yes，而不是 no。

    例如，启用 ripd 守护进程：

    ```
    ripd=yes
    ```

2. 重新加载 frr 服务：

    ```
    # systemctl reload frr
    ```

## 45.4.修改特定守护进程的配置

使用默认配置，FRR 中的每个路由守护进程都只能充当普通路由器。

要进行守护进程的额外配置，请使用以下步骤。

**流程**

1. 在 /etc/frr/ 目录中，为所需守护进程创建一个配置文件，并将该文件命名为：

    ```
    [daemon_name].conf
    ```
    例如，要进一步配置 eigrpd 守护进程，请在上述目录中创建 eigrpd.conf 文件。

2. 使用所需内容填充新文件。

    有关特定的 FRR 守护进程的配置示例，请查看 /usr/share/doc/frr/ 目录。

3. 重新加载 frr 服务：

    ```
    # systemctl reload frr
    ```

# 第46章 测试基本网络设置

## 46.1.使用 ping 程序验证 IP 到其他主机的连接

ping 工具向远程主机发送 ICMP 数据包。您可以使用此功能来测试 IP 与不同主机的连接是否正常工作。

**流程**

- 将主机的 IP 地址放在同一子网中，如您的默认网关：

    ```
    # ping 192.0.2.3
    ```
    如果命令失败，请验证默认网关设置。

- 在远程子网中指定主机的 IP 地址：

    ```
    # ping 198.162.3.1
    ```
    如果命令失败，请验证默认网关设置，并确保网关在连接的网络间转发数据包。

## 46.2.使用 host 程序验证域名解析

**流程**

- 使用 host 实用程序来验证名称解析是否正常工作。例如：要将 client.example.com 主机名解析为 IP 地址，请输入：

    ```
    # host client.example.com
    ```
    如果命令返回错误，如 connection timed out 或 no servers could be reached，请验证您的 DNS 设置。

# 第47章 使用 NetworkManager 程序调度脚本运行 dhclient exit hooks

## 47.1.NetworkManager 程序调度脚本的概念

在发生网络事件时，NetworkManager-dispatcher 服务会按字母顺序执行用户提供的脚本。这些脚本通常是 shell 脚本，但也可以是任何可执行的脚本或应用程序。例如，您可以使用程序调度脚本来调整您无法使用 NetworkManager 进行管理的与网络相关的设置。

您可以在以下目录中存储程序调度脚本：
- /etc/NetworkManager/dispatcher.d/ ：root 用户可以编辑的程序调度脚本的通用位置。
- /usr/lib/NetworkManager/dispatcher.d/: 用于预先部署的不可变程序调度脚本。

为了安全起见，NetworkManager-dispatcher 服务只有在满足以下条件时才执行脚本：

- 脚本归 root 用户所有。
- 该脚本仅可由 root 读写。
- 脚本上没有设置 setuid 位。

NetworkManager-dispatcher 服务使用两个参数运行每个脚本：

  1. 操作所在的设备的接口名称。
  2. 接口行为，如当接口被激活时的 up 操作。

NetworkManager-dispatcher 服务一次运行一个脚本，但与主 NetworkManager 进程异步运行。请注意，如果脚本已排队，服务一定会运行它，即使后续事件使其过时。但是，NetworkManager-dispatcher 服务运行脚本，它们是指向 /etc/NetworkManager/dispatcher.d/no-wait.d/ 中的文件的符号链接，而无需等待之前脚本的终止，并行运行。

## 47.2.创建运行 dhclient exit hooks 的 NetworkManager 程序调度脚本

本节解释了当从 DHCP 服务器分配或更新 IPv4 地址时，如何编写 NetworkManager 程序调度脚本，该脚本会运行保存在 /etc/dhcp/dhclient-exit-hooks.d/ 目录下的 dhclient exit hooks 。

**前提条件**

- dhclient exit hooks 存储在 /etc/dhcp/dhclient-exit-hooks.d/ 目录下。

**流程**

1. 使用以下内容创建 /etc/NetworkManager/dispatcher.d/12-dhclient-down 文件：

    ```shell
    #!/bin/bash
    # Run dhclient.exit-hooks.d scripts

    if [ -n "$DHCP4_DHCP_LEASE_TIME" ] ; then
    if [ "$2" = "dhcp4-change" ] || [ "$2" = "up" ] ; then
        if [ -d /etc/dhcp/dhclient-exit-hooks.d ] ; then
        for f in /etc/dhcp/dhclient-exit-hooks.d/*.sh ; do
            if [ -x "${f}" ]; then
            . "${f}"
            fi
        done
        fi
    fi
    fi
    ```

2. 将 root 用户设为文件的所有者：

    ```
    # chown root:root /etc/NetworkManager/dispatcher.d/12-dhclient-down
    ```

3. 设置权限，以便只有 root 用户才能执行它：

    ```
    # chmod 0700 /etc/NetworkManager/dispatcher.d/12-dhclient-down
    ```

4. 恢复 SELinux 上下文：

    ```
    # restorecon /etc/NetworkManager/dispatcher.d/12-dhclient-down
    ```


# 第48章 NetworkManager 调试介绍

提高所有或某些域的日志级别有助于记录 NetworkManager 所执行的操作的更多详情。管理员可以使用这些信息排除问题。NetworkManager 提供不同的级别和域来生成日志信息。/etc/NetworkManager/NetworkManager.conf 文件是 NetworkManager 的主配置文件。日志存储在日志中。

本章介绍有关为 NetworkManager 启用调试日志以及使用不同日志级别和域配置日志量的信息。

## 48.1. 调试级别和域

您可以使用 levels 和 domains 参数来管理 NetworkManager 的调试。levels 定义了详细程度，而 domains 定义了消息的类别，以记录给定的严重程度（ level ）的日志。

|日志级别|描述|
|--------|---------|
|OFF|不记录任何有关 NetworkManager 的信息|
|ERR|仅记录严重错误|
|WARN|记录可以反映操作的警告信息|
|INFO|记录各种有助于跟踪状态和操作的信息|
|DEBUG|为调试启用详细日志记录|
|TRACE|启用比 DEBUG 级别更详细的日志|
|||

请注意，后续的级别记录包含以前级别的所有信息。例如，将日志级别设为 INFO 也会记录包含在 ERR 和 WARN 日志级别中的消息。

## 48.2.设置 NetworkManager 日志级别

默认情况下，所有日志域都设为记录 INFO 日志级别。在收集调试日志前禁用速率限制。通过速率限制，如果短时间内有太多信息，systemd-journald 会丢弃它们。当日志级别为 TRACE 时，可能会发生这种情况。

以下流程禁用速率限制，并为所有域启用记录调试日志。

**流程**

1. 要禁用速率限制，请编辑 /etc/systemd/journald.conf 文件，取消 [Journal] 部分中的 RateLimitBurst 参数的注释，并将其值设为 0 ：

    ```
    RateLimitBurst=0
    ```

2. 重启 systemd-journald 服务。

    ```
    # systemctl restart systemd-journald
    ```

3. 使用以下内容创建 /etc/NetworkManager/conf.d/95-nm-debug.conf 文件：

    ```
    [logging]
    domains=ALL:TRACE
    ```
    domains 参数可以包含多个用逗号分隔的 domain:level 对。

4. 重启 NetworkManager 服务。

    ```
    # systemctl restart NetworkManager
    ```

**验证**

- 查询 systemd 日志以显示 NetworkManager 单元的日志条目：

    ```
    # journalctl -u NetworkManager
    ...
    Jun 30 15:24:32 server NetworkManager[164187]: <debug> [1656595472.4939] active-connection[0x5565143c80a0]: update activation type from assume to managed
    Jun 30 15:24:32 server NetworkManager[164187]: <trace> [1656595472.4939] device[55b33c3bdb72840c] (enp1s0): sys-iface-state: assume -> managed
    Jun 30 15:24:32 server NetworkManager[164187]: <trace> [1656595472.4939] l3cfg[4281fdf43e356454,ifindex=3]: commit type register (type "update", source "device", existing a369f23014b9ede3) -> a369f23014b9ede3
    Jun 30 15:24:32 server NetworkManager[164187]: <info>  [1656595472.4940] manager: NetworkManager state is now CONNECTED_SITE
    ...
    ```

## 48.3.在程序运行时使用 nmcli 临时设置日志级别

您可以使用 nmcli 在程序运行时更改日志级别。但是，OpenCloudOS建议使用配置文件启用调试并重启 NetworkManager。使用 .conf 文件更新 levels和 domains 有助于调试启动问题，并捕获初始状态的所有日志。

**流程**

1. 可选：显示当前的日志设置：

    ```
    # nmcli general logging
    LEVEL  DOMAINS
    INFO   PLATFORM,RFKILL,ETHER,WIFI,BT,MB,DHCP4,DHCP6,PPP,WIFI_SCAN,IP4,IP6,A
    UTOIP4,DNS,VPN,SHARING,SUPPLICANT,AGENTS,SETTINGS,SUSPEND,CORE,DEVICE,OLPC,
    WIMAX,INFINIBAND,FIREWALL,ADSL,BOND,VLAN,BRIDGE,DBUS_PROPS,TEAM,CONCHECK,DC
    B,DISPATCH
    ```

2. 要修改日志级别和域，请使用以下选项：

    - 要将所有域的日志级别都设为同样的 LEVEL，请输入：

        ```
        # nmcli general logging level LEVEL domains ALL
        ```

    - 要更改特定域的级别，请输入：

        ```
        # nmcli general logging level LEVEL domains DOMAINS
        ```
        请注意，使用这个命令更新日志级别会禁用所有其他域的日志功能。

    - 要更改特定域的级别并保持其它域的级别，请输入：

        ```
        # nmcli general logging level KEEP domains DOMAIN:LEVEL,DOMAIN:LEVEL
        ```

## 48.4.查看 NetworkManager 日志

**流程**

- 要查看日志，请输入：

    ```
    # journalctl -u NetworkManager -b
    ```