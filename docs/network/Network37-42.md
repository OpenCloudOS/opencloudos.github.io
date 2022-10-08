# 第37章 配置 802.3 链路设置

## 37.1.了解自动协商

自动协商是 IEEE 802.3u 快速以太网协议的一个特性。它以设备端口为目标，为链路上的信息交换提供速度、双工模式和流控制的最佳性能。使用自动协商协议时，您具有在以太网上进行数据传输的最佳性能。

注意，要最大限度地利用自动协商的性能，请在链接两端使用同样的配置。

## 37.2.使用 nmcli 工具配置802.3 链路设置

要配置以太网连接的 802.3 链接设置，请修改以下配置参数：

- 802-3-ethernet.auto-negotiate
- 802-3-ethernet.speed
- 802-3-ethernet.duplex

**流程**

1. 显示连接的当前设置：
    ```   
    # nmcli connection show Example-connection
    ...
    802-3-ethernet.speed:  0
    802-3-ethernet.duplex: --
    802-3-ethernet.auto-negotiate: no
    ...
    ```
    如果需要在有问题的时候重置参数，您可以使用这些值。

2. 设置速度和双工链路设置：

    ```
    # nmcli connection modify Example-connection 802-3-ethernet.auto-negotiate no 802-3-ethernet.speed 10000 802-3-ethernet.duplex full
    ```
    这个命令会禁用自动协商，并将连接的速度设为 10000 Mbit 全双工。

3. 重新激活连接：

    ```
    # nmcli connection up Example-connection
    ```
**验证**

- 使用 ethtool 工具验证以太网接口 enp1s0 的值：
    ```
    # ethtool enp1s0

    Settings for enp1s0:
        ...
        Advertised auto-negotiation: No
        ...
        Speed: 10000Mb/s
        Duplex: Full
        Auto-negotiation: off
        ...
        Link detected: yes
    ```

# 第38章 配置 ethtool offload 功能

网络接口卡可使用 TCP 卸载引擎（TOE）将某些操作推卸给网络控制器以提高网络吞吐量。

## 38.1.NetworkManager 支持的卸载功能

- ethtool.feature-esp-hw-offload
- ethtool.feature-esp-tx-csum-hw-offload
- ethtool.feature-fcoe-mtu
- ethtool.feature-gro
- ethtool.feature-gso
- ethtool.feature-highdma
- ethtool.feature-hw-tc-offload
- ethtool.feature-l2-fwd-offload
- ethtool.feature-loopback
- ethtool.feature-lro
- ethtool.feature-macsec-hw-offload
- ethtool.feature-ntuple
- ethtool.feature-rx
- ethtool.feature-rx-all
- ethtool.feature-rx-fcs
- ethtool.feature-rx-gro-hw
- ethtool.feature-rx-gro-list
- ethtool.feature-rx-udp_tunnel-port-offload
- ethtool.feature-rx-udp-gro-forwarding
- ethtool.feature-rx-vlan-filter
- ethtool.feature-rx-vlan-stag-filter
- ethtool.feature-rx-vlan-stag-hw-parse
- ethtool.feature-rxhash
- ethtool.feature-rxvlan
- ethtool.feature-sg
- ethtool.feature-tls-hw-record
- ethtool.feature-tls-hw-rx-offload
- ethtool.feature-tls-hw-tx-offload
- ethtool.feature-tso
- ethtool.feature-tx
- ethtool.feature-tx-checksum-fcoe-crc
- ethtool.feature-tx-checksum-ip-generic
- ethtool.feature-tx-checksum-ipv4
- ethtool.feature-tx-checksum-ipv6
- ethtool.feature-tx-checksum-sctp
- ethtool.feature-tx-esp-segmentation
- ethtool.feature-tx-fcoe-segmentation
- ethtool.feature-tx-gre-csum-segmentation
- ethtool.feature-tx-gre-segmentation
- ethtool.feature-tx-gso-list
- ethtool.feature-tx-gso-partial
- ethtool.feature-tx-gso-robust
- ethtool.feature-tx-ipxip4-segmentation
- ethtool.feature-tx-ipxip6-segmentation
- ethtool.feature-tx-nocache-copy
- ethtool.feature-tx-scatter-gather
- ethtool.feature-tx-scatter-gather-fraglist
- ethtool.feature-tx-sctp-segmentation
- ethtool.feature-tx-tcp-ecn-segmentation
- ethtool.feature-tx-tcp-mangleid-segmentation
- ethtool.feature-tx-tcp-segmentation
- ethtool.feature-tx-tcp6-segmentation
- ethtool.feature-tx-tunnel-remcsum-segmentation
- ethtool.feature-tx-udp-segmentation
- ethtool.feature-tx-udp_tnl-csum-segmentation
- ethtool.feature-tx-udp_tnl-segmentation
- ethtool.feature-tx-vlan-stag-hw-insert
- ethtool.feature-txvlan

有关各个卸载特性的详情，请查看 ethtool 工具文档和内核文档。

## 38.2.使用 NetworkManager 配置 ethtool offload 功能

本节介绍了如何使用 NetworkManager 启用和禁用 ethtool 卸载特性，以及如何从 NetworkManager 连接配置文件中删除某一特性的设置。

**流程**

1. 例如：要启用 RX 卸载特性，并在 enp1s0 连接配置文件中禁用 TX 卸载，请输入：

    ```
    # nmcli con modify enp1s0 ethtool.feature-rx on ethtool.feature-tx off
    ```
    这个命令明确启用 RX 卸载并禁用 TX 卸载功能。

2. 要删除之前启用或禁用的卸载特性的设置，请将特性的参数设为 ignore。例如，要删除 TX 卸载的配置，请输入：

    ```
    # nmcli con modify enp1s0 ethtool.feature-tx ignore
    ```
3. 重新激活网络配置集：

    ```
    # nmcli connection up enp1s0
    ```

**验证步骤**

- 使用 ethtool -k 命令显示网络设备的当前卸载特性：

    ```
    # ethtool -k network_device
    ```

## 38.3.使用 rhel-system-roles 设置 ethtool 特性

您可以使用 rhel-system-roles 中的网络角色来配置 NetworkManager 连接的 ethtool 特性。

当您运行使用 Networking RHEL system roles 的脚本时，如果设置的值与脚本中指定的名称不匹配，则系统角色会覆盖具有相同名称的现有的连接配置文件。因此，始终在脚本中指定网络连接配置文件的整个配置，即使 IP 配置已经存在。否则，role 会将这些值重置为默认值。

**前提条件**

- ansible 和 rhel-system-roles 软件包已安装在控制节点上。
- 如果您使用非 root 用户运行 playbook ，请确保用户在受管节点上具有 sudo 权限。

**流程**

1. 如果您要在其上执行 playbook 中指令的主机还没有被列入清单，请将此主机的 IP 或名称添加到 /etc/ansible/hosts Ansible 清单文件中：

    ```
    node.example.com
    ```
2. 使用以下内容创建 ~/configure-ethernet-device-with-ethtool-features.yml playbook：

    ```
    ---
    - name: Configure an Ethernet connection with ethtool features
    hosts: node.example.com
    become: true
    tasks:
    - include_role:
        name: rhel-system-roles.network

        vars:
        network_connections:
            - name: enp1s0
            type: ethernet
            autoconnect: yes
            ip:
                address:
                - 198.51.100.20/24
                - 2001:db8:1::1/64
                gateway4: 198.51.100.254
                gateway6: 2001:db8:1::fffe
                dns:
                - 198.51.100.200
                - 2001:db8:1::ffbb
                dns_search:
                - example.com
            ethtool:
                features:
                gro: "no"
                gso: "yes"
                tx_sctp_segmentation: "no"
            state: up
    ```
3. 运行 playbook：

    - 要以 root 用户身份连接到受管主机，请输入：

        ```
        # ansible-playbook -u root ~/configure-ethernet-device-with-ethtool-features.yml
        ```
    - 以用户身份连接到受管主机，请输入：

        ```
        # ansible-playbook -u user_name --ask-become-pass ~/configure-ethernet-device-with-ethtool-features.yml
        ```
        --ask-become-pass 选项确保 ansible-playbook 命令提示输入 -u user_name 选项中定义的用户的 sudo 密码。

    如果没有指定 -u user_name 选项，ansible-playbook 以当前登录到控制节点的用户身份连接到受管主机。

# 第39章 配置 ethtool coalesce 设置

利用中断合并，系统收集网络数据包，并为多个数据包生成一个中断。这会增加一个硬件中断能够发送到内核的数据量，从而减少中断负载，并最大化吞吐量。

本节提供了用来设置 ethtool 合并设置的不同选项。

## 39.1. NetworkManager 支持的合并设置

您可以使用 NetworkManager 设置以下 ethtool 合并设置：
- coalesce-adaptive-rx
- coalesce-adaptive-tx
- coalesce-pkt-rate-high
- coalesce-pkt-rate-low
- coalesce-rx-frames
- coalesce-rx-frames-high
- coalesce-rx-frames-irq
- coalesce-rx-frames-low
- coalesce-rx-usecs
- coalesce-rx-usecs-high
- coalesce-rx-usecs-irq
- coalesce-rx-usecs-low
- coalesce-sample-interval
- coalesce-stats-block-usecs
- coalesce-tx-frames
- coalesce-tx-frames-high
- coalesce-tx-frames-irq
- coalesce-tx-frames-low
- coalesce-tx-usecs
- coalesce-tx-usecs-high
- coalesce-tx-usecs-irq
- coalesce-tx-usecs-low

## 39.2.使用 NetworkManager 配置 ethtool 合并设置

这部分描述了如何使用 NetworkManager 设置 ethtool 合并设置，以及如何从 NetworkManager 连接配置文件中删除设置。

**流程**

1. 例如，要在 enp1s0 连接配置文件中将接收的数据包的最大数量设置为延迟到 128，请输入：

    ```
    # nmcli connection modify enp1s0 ethtool.coalesce-rx-frames 128
    ```

2. 要删除合并设置，可将设置设为 ignore。例如，要删除 ethtool.coalesce-rx-frames 设置，请输入：

    ```
    # nmcli connection modify enp1s0 ethtool.coalesce-rx-frames ignore
    ```

3. 重新激活网络配置集：

    ```
    # nmcli connection up enp1s0
    ```

**验证步骤**

- 使用 ethtool -c 命令显示网络设备的当前卸载特性：

    ```
    # ethtool -c network_device
    ```

## 39.3.使用 rhel-system-roles 设置 ethtool 合并设置

您可以使用 rhel-system-roles 中的网络角色来配置 NetworkManager 连接的 ethtool 合并设置。

当您运行使用 Networking RHEL system roles 的脚本时，如果设置的值与脚本中指定的名称不匹配，则系统角色会覆盖具有相同名称的现有的连接配置文件。因此，始终在脚本中指定网络连接配置文件的整个配置，即使 IP 配置已经存在。否则，role 会将这些值重置为默认值。

**前提条件**

- ansible 和 rhel-system-roles 软件包已安装在控制节点上。
- 如果您使用非 root 用户运行 playbook ，请确保用户在受管节点上具有 sudo 权限。

**流程**

1. 如果您要在其上执行 playbook 中指令的主机还没有被列入清单，请将此主机的 IP 或名称添加到 /etc/ansible/hosts Ansible 清单文件中：

    ```
    node.example.com
    ```
2. 使用以下内容创建 ~/configure-ethernet-device-with-ethtool-features.yml playbook：

    ```
    ---
    - name: Configure an Ethernet connection with ethtool coalesce settings
     hosts: node.example.com
     become: true
     tasks:
     - include_role:
         name: rhel-system-roles.network

       vars:
         network_connections:
           - name: enp1s0
             type: ethernet
             autoconnect: yes
             ip:
               address:
                 - 198.51.100.20/24
                 - 2001:db8:1::1/64
               gateway4: 198.51.100.254
               gateway6: 2001:db8:1::fffe
               dns:
                 - 198.51.100.200
                 - 2001:db8:1::ffbb
               dns_search:
                 - example.com
             ethtool:
               coalesce:
                 rx_frames: 128
                 tx_frames: 128
             state: up
    ```
3. 运行 playbook：

    - 要以 root 用户身份连接到受管主机，请输入：

        ```
        # ansible-playbook -u root ~/configure-ethernet-device-with-ethtoolcoalesce-settings.yml
        ```
    - 以用户身份连接到受管主机，请输入：

        ```
        # ansible-playbook -u user_name --ask-become-pass ~/configure-ethernet-device-with-ethtoolcoalesce-settings.yml
        ```
        --ask-become-pass 选项确保 ansible-playbook 命令提示输入 -u user_name 选项中定义的用户的 sudo 密码。

    如果没有指定 -u user_name 选项，ansible-playbook 以当前登录到控制节点的用户身份连接到受管主机。

# 第40章 使用 MACsec 加密同一物理网络中的第 2 层流量

介质访问控制安全(MACsec)是一种第 2 层的协议，用于保护以太网链路上的不同的流量类型，您可以使用 MACsec 来保护两个设备（点到点）之间的通信。MACsec 包括：
- 动态主机配置协议(DHCP)
- 地址解析协议(ARP)
- 互联网协议版本 4 / 6(IPv4 / IPv6)以及
- 任何使用 IP的流量（如 TCP 或 UDP）

MACsec 默认使用 GCM-AES-128 算法加密并验证 LAN 中的所有流量，使用预共享密钥在参与的主机之间建立连接。如果要更改预共享密钥，您需要更新网络中使用 MACsec 的所有主机上的 NM 配置。

MACsec 连接将以太网设备（如以太网网卡、VLAN 或隧道设备）用作父设备。您可以只在 MACsec 设备上设置 IP 配置，以便只使用加密连接与其他主机进行通信，也可以在父设备上设置 IP 配置。在后者的情况下，您可以使用父设备使用未加密连接和 MACsec 设备加密连接与其他主机通信。

Macsec 不需要任何特殊硬件。例如，您可以使用任何交换机，如果您只想在主机和交换机之间加密流量。在这种情况下，交换机还必须支持 MACsec。

配置 MACsec 有两种常用的用法：
- 主机到主机
- 主机到交换机，然后交换机到其他主机

注意，您只能在相同（物理或虚拟）LAN 的主机间使用 MACsec。

## 40.1.使用 nmcli 配置 MACsec 连接

您可以使用 nmcli 工具将以太网接口配置为使用 MACsec。以下流程描述了如何在通过以太网连接的两个主机之间创建 MACsec 连接。

**流程**

1. 在配置 MACsec 的第一个主机上：

    - 为预共享密钥创建连接关联密钥(CAK)和连接关联密钥名称(CKN)：

        1. 创建一个 16 字节的十六进制 CAK：

            ```
            # dd if=/dev/urandom count=16 bs=1 2> /dev/null | hexdump -e '1/2 "%04x"'
            50b71a8ef0bd5751ea76de6d6c98c03a
            ```
        2. 创建一个 32 字节的十六进制 CKN：

            ```
            # dd if=/dev/urandom count=32 bs=1 2> /dev/null | hexdump -e '1/2 "%04x"'
            f2b4297d39da7330910a74abc0449feb45b5c0b9fc23df1430e1898fcf1c4550
            ```
2. 在您要通过 MACsec 连接连接的两个主机上，创建 MACsec 连接：

    ```
    # nmcli connection add type macsec con-name macsec0 ifname macsec0 connection.autoconnect yes macsec.parent enp1s0 macsec.mode psk macsec.mka-cak 50b71a8ef0bd5751ea76de6d6c98c03a macsec.mka-ckn f2b4297d39da7330910a7abc0449feb45b5c0b9fc23df1430e1898fcf1c4550
    ```
    在macsec.mka-cak 和 macsec.mka-ckn 参数中使用上一步生成的 CAK 和 CKN。在 MACsec-protected 网络的每个主机上，这些值必须相同。

3. 配置 MACsec 连接中的 IP 设置。

    1. 配置 IPv4 设置。例如，要为 macsec0 连接设置静态 IPv4 地址、网络掩码、默认网关和 DNS 服务器，请输入：

        ```
        # nmcli connection modify macsec0 ipv4.method manual ipv4.addresses '192.0.2.1/24' ipv4.gateway '192.0.2.254' ipv4.dns '192.0.2.253'
        ```
    2. 配置 IPv6 设置。例如，要为 macsec0 连接设置静态 IPv6 地址、网络掩码、默认网关和 DNS 服务器，请输入：

        ```
        # nmcli connection modify macsec0 ipv6.method manual ipv6.addresses '2001:db8:1::1/32' ipv6.gateway '2001:db8:1::fffe' ipv6.dns '2001:db8:1::fffd'
        ```
4. 激活连接：

    ```
    # nmcli connection up macsec0
    ```

**验证步骤**

1. 验证流量是否加密：

    ```
    # tcpdump -nn -i enp1s0
    ```
2. 可选：显示未加密的流量：

    ```
    # tcpdump -nn -i macsec0
    ```
3. 显示 MACsec 统计信息：

    ```
    # ip macsec show
    ```
4. 显示每种保护类型的单独的计数器：仅完整性（关闭加密）和加密（打开加密）

    ```
    # ip -s macsec show
    ```

# 第41章 在不同域中使用不同的 DNS 服务器

默认情况下，OpenCloudOS 将所有 DNS 请求发送到 /etc/resolv.conf 文件中指定的第一个 DNS 服务器。如果这个服务器没有回复，OpenCloudOS 会使用这个文件中的下一个服务器。

在一个 DNS 服务器无法解析所有域的环境中，管理员可将 OpenCloudOS 配置为将特定域的 DNS 请求发送到所选 DNS 服务器。例如，您可以配置一个 DNS 服务器来解析对 example.com 的查询，配置另一个 DNS 服务器来解析对 example.net 的查询。对于所有其他 DNS 请求，OpenCloudOS 使用与默认网关连接中配置的 DNS 服务器。

## 41.1.将特定域的 DNS 请求发送到所选 DNS 服务器

本节配置了 systemd-resolved 服务和 NetworkManager ，来将对特定域的 DNS 查询发送到所选的 DNS 服务器。

如果您完成了本节中的流程，OpenCloudOS 将使用 /etc/resolv.conf 文件中 systemd-resolved 提供的 DNS 服务。systemd-resolved 服务启动一个 DNS 服务，该服务侦听端口 53 ，IP 地址 127.0.0.53。该服务会动态将 DNS 请求路由到 NetworkManager 中指定的对应 DNS 服务器。请将 IP 替换为实际生产中需要的 IP 地址。

**前提条件**

- 系统配置了多个 NetworkManager 连接。
- 在负责解析特定域的 NetworkManager 连接中配置 DNS 服务器和搜索域

例如：如果 VPN 连接中指定的 DNS 服务器应该可以解析对 example.com 域的查询，则 VPN 连接配置文件必须有：

1. 配置一个可以解析 example.com 的 DNS 服务器
2. 在 ipv4.dns-search 和 ipv6.dns-search 参数中将搜索域配置为 example.com

**流程**

1. 启动并启用 systemd-resolved 服务：

    ```
    # systemctl --now enable systemd-resolved
    ```

2. 编辑 /etc/NetworkManager/NetworkManager.conf 文件，并在 [main] 部分中设置以下条目：

    ```
    dns=systemd-resolved
    ```

3. 重新载入 NetworkManager 服务：

    ```
    # systemctl reload NetworkManager

    ```

**验证**

1. 验证 /etc/resolv.conf 文件中的 nameserver 条目是否指向 127.0.0.53 ：

    ```
    # cat /etc/resolv.conf
    nameserver 127.0.0.53
    ```

2. 验证 systemd-resolved 服务是否监听本地 IP 地址 127.0.0.53 上的端口 53 ：

    ```
    # ss -tulpn | grep "127.0.0.53"
    udp  UNCONN 0  0      127.0.0.53%lo:53   0.0.0.0:*    users:(("systemd-resolve",pid=1050,fd=12))
    tcp  LISTEN 0  4096   127.0.0.53%lo:53   0.0.0.0:*    users:(("systemd-resolve",pid=1050,fd=13))
    ```

# 第42章 使用 IPVLAN

本章介绍了 IPVLAN 驱动程序。

## 42.1.IPVLAN 概述

IPVLAN 是虚拟网络设备的驱动程序，可在容器环境中用于访问主机网络。IPVLAN 会将一个 MAC 地址公开给外部网络，而不管主机网络中所创建的 IPVLAN 设备的数量。这意味着，用户可以在多个容器中有多个 IPVLAN 设备，相应的交换机会读取单个 MAC 地址。当本地交换机对它可管理的 MAC 地址的总数施加约束时，IPVLAN 驱动程序很有用。

## 42.2.IPVLAN 模式

IPVLAN 有以下模式可用：

- L2 模式

    在 IPVLAN L2 模式 中，虚拟设备接收并响应地址解析协议(ARP)请求。netfilter 框架仅在拥有虚拟设备的容器中运行。容器化流量的默认命名空间中不会执行 netfilter 链。使用L2 模式会提供良好的性能，但对网络流量的控制要小。

- L3 模式

    在 L3 模式 中，虚拟设备只处理 L3 以上的流量。虚拟设备不响应 ARP 请求，用户必须手动为相关点上的 IPVLAN IP 地址配置邻居条目。相关容器的出口流量位于默认命名空间中的 netfilter POSTROUTING 和 OUTPUT 链上，而入口流量以与 L2 模式 相同的方式被线程化。使用L3 模式会提供很好的控制，但可能会降低网络流量性能。

- L3S 模式

    在 L3S 模式 中，虚拟设备处理方式与 L3 模式 中的处理方式相同，但相关容器的出口和入口流量都位于默认命名空间中的 netfilter 链上。L3S 模式 的行为方式和 L3 模式 相似，但提供了对网络的更大控制。

对于 L3 和 L3S 模式，IPVLAN 虚拟设备不接收广播和多播流量。

## 42.3.MACVLAN 概述

MACVLAN 驱动程序允许在一个 NIC 上创建多个虚拟网络设备，每个网卡都由其自身唯一的 MAC 地址标识。物理 NIC 上的数据包通过目的地的 MAC 地址与相关的 MACVLAN 设备进行多路复用。MACVLAN 设备不添加任何级别的封装。

## 42.4.IPVLAN 和 MACVLAN 的比较

下表列出了 MACVLAN 和 IPVLAN 的主要区别。

|MACVLAN|IPVLAN|
|-----------------------------------------------------------------------------|----------------------------------------------------------------------------|
|为每个 MACVLAN 设备使用 MAC 地址。交换中 MAC 表的 MAC 地址限制可能会导致连接丢失。|使用不限制 IPVLAN 设备数的单个 MAC 地址。|
|全局命名空间的 netfilter 规则不会影响子命名空间中到达或从 MACVLAN 设备经过的网络流量。|有可能在 L3 模式和 L3S 模式中控制到 IPVLAN 设备或者来自 IPVLAN 设备的网络流量。|

请注意，IPVLAN 和 MACVLAN 不需要任何级别的封装。

## 42.5.使用 iproute2 创建和配置 IPVLAN 设备

本节介绍了如何使用 iproute2 设置 IPVLAN 设备。

**流程**

1. 要创建 IPVLAN 设备，请输入以下命令：

    ```
    # ip link add link real_NIC_device name IPVLAN_device type ipvlan mode l2

    ```
    请注意：网络接口控制器（NIC）是将计算机连接到网络的一个硬件组件。

    例 43.1. 创建 IPVLAN 设备

    ```
    # ip link add link enp0s31f6 name my_ipvlan type ipvlan mode l2
    # ip link
    47: my_ipvlan@enp0s31f6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000 link/ether e8:6a:6e:8a:a2:44 brd ff:ff:ff:ff:ff:ff
    ```

2. 要给接口分配 IPv4 或 IPv6 地址，请输入以下命令：

    ```
    # ip addr add dev IPVLAN_device IP_address/subnet_mask_prefix

    ```

3. 如果在 L3 模式或 L3S 模式中配置 IPVLAN 设备，请进行以下设置：

    1. 在远程主机上为远程 peer 配置邻居设置：

        ```
        # ip neigh add dev peer_device IPVLAN_device_IP_address lladdr MAC_address
        ```
        其中 MAC_address 是 IPVLAN 设备所基于的实际网卡的 MAC 地址。

    2. 使用以下命令为 L3 模式 配置 IPVLAN 设备：

        ```
        # ip route add dev <real_NIC_device> <peer_IP_address/32>
        ```
        对于 L3S 模式：

        ```
        # ip route add dev real_NIC_device peer_IP_address/32
        ```
        其中 IP-address 代表远程 peer 的地址。

4. 要设置活跃的 IPVLAN 设备，请输入以下命令：

    ```
    # ip link set dev IPVLAN_device up

    ```

5. 要检查 IPVLAN 设备是否活跃，请在远程主机中执行以下命令：

    ```
    # ping IP_address

    ```
    其中 IP_address 使用 IPVLAN 设备的 IP 地址。

