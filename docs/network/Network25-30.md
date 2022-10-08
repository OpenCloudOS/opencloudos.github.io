# 第25章 创建 dummy 接口

作为 OpenCloudOS 用户，您可以创建并使用 dummy 网络接口进行调试和测试。dummy 接口提供了一个设备来路由数据包而无需实际传送数据包。它可让您额外创建使用 NetworkManager 管理的回环设备，使不活跃的 SLIP（Serial Line Internet Protocol）地址像具体地址一样与进程通信。

## 25.1.使用 nmcli 使用 IPv4 和 IPv6 地址创建 dummy 接口

您可以创建带有各种设置的 dummy 接口。本节描述了如何使用 IPv4 和 IPv6 地址创建 dummy 接口。创建虚拟接口后，NetworkManager 会自动将其分配给默认的 public 防火墙域。

注意，要配置没有 IPv4 或 IPv6 地址的虚拟接口，请将 ipv4.method 和 ipv6.method 参数设为 disabled。否则，IP 自动配置会失败，NetworkManager 会取消激活连接并删除 dummy 设备。

**流程**

1. 创建一个名为 dummy0 的、带有静态 IPv4 和 IPv6 地址的 dummy 接口：
    ```
    # nmcli connection add type dummy ifname dummy0 ipv4.method manual ipv4.addresses 192.0.2.1/24 ipv6.method manual ipv6.addresses 2001:db8:2::1/64
    ```
2. 可选： 要查看 dummy 接口，请输入：
    ```
    # nmcli connection show
    NAME            UUID                                  TYPE      DEVICE
    enp1s0          db1060e9-c164-476f-b2b5-caec62dc1b05  ethernet    ens3
    dummy-dummy0    
    ```

# 第26章 使用 nmstate-autoconf 自动配置使用 LLDP 的网络状态

网络设备可以使用链路层发现协议(LLDP)在 LAN 中表明自己的身份、功能和邻居。nmstate-autoconf 工具可使用此信息来自动配置本地网络接口。

## 26.1.使用 nmstate-autoconf 来自动配置网络接口

nmstate-autoconf 工具使用 LLDP 来识别连接到交换机接口的 VLAN 设置来配置本地设备。

此流程假设以下场景，以及交换机使用 LLDP 广播 VLAN 设置：

- 服务器的 enp1s0 和 enp2s0 接口连接到使用 VLAN ID 100 和 VLAN 名称 prod-net 配置的交换机端口。
- 服务器的 enp3s0 接口连接到使用 VLAN ID 200 和 VLAN 名称 mgmt-net 配置的交换机端口。

然后，nmstate-autoconf 工具使用此信息来在服务器上创建以下接口：

- bond100 - enp1s0 和 enp2s0 作为端口的绑定接口。
- prod-net - 在 VLAN ID 为 100 的 bond100 上面的 VLAN 接口。
- mgmt-net - 在 VLAN ID 为200 的 enp3s0 上面的 VLAN 接口

如果您将多个网络接口连接到 LLDP 用来广播同一 VLAN ID 的不同交换机的端口，则 nmstate-autoconf 会用这些接口来创建一个绑定，并在其上配置通用 VLAN ID。

**前提条件**

- nmstate 软件包已安装。
- 网络交换机上启用了 LLDP。
- 以太网接口已启用。

**流程**

1. 在以太网接口上启用 LLDP：
   1. 创建 ~/enable-lldp.yml YAML 文件，包含以下内容：
        ```
        interfaces:
        - name: enp1s0
            type: ethernet
            lldp:
            enabled: true
        - name: enp2s0
            type: ethernet
            lldp:
            enabled: true
        - name: enp3s0
            type: ethernet
            lldp:
            enabled: true
        ```
   2. 应用设置：
        ```
        # nmstatectl apply ~/enable-lldp.yml
        ```

2. 使用 LLDP 配置网络接口：
   1. 可选，启动一个空运行来显示并验证 nmstate-autoconf 生成的 YAML 配置：
        ```
        # nmstate-autoconf -d enp1s0,enp2s0,enp3s0
        ---
        interfaces:
        - name: prod-net
        type: vlan
        state: up
        vlan:
            base-iface: bond100
            id: 100
        - name: mgmt-net
        type: vlan
        state: up
        vlan:
            base-iface: enp3s0
            id: 200
        - name: bond100
        type: bond
        state: up
        link-aggregation:
            mode: balance-rr
            port:
            - enp1s0
            - enp2s0
        ```
   2. 使用 nmstate-autoconf 根据从 LLDP 接收的信息来生成配置，并将设置应用到系统：
        ```
        # nmstate-autoconf enp1s0,enp2s0,enp3s0
        ```

**验证**

- 显示单个接口的设置
    ```
    # nmstatectl show <interface_name>
    ```

# 第27章 使用 LLDP 来调试网络配置问题

您可以使用链路层发现协议(LLDP)来调试网络拓扑中的配置问题。这意味着 LLDP 可以报告与其他主机或路由器以及交换机的配置不一致问题。

## 27.1.使用 LLDP 信息调试不正确的 VLAN 配置

如果您将交换机端口配置为使用指定的 VLAN ，而主机没有收到这些 VLAN 数据包，那么您可以使用链路层发现协议(LLDP)来调试问题。请在没有收到数据包的主机上执行这个流程。

**前提条件**

- nmstate 软件包已安装。
- 交换机支持 LLDP。
- LLDP 在邻居设备上已启用。

**流程**

1. 使用以下内容创建 ~/enable-LLDP-enp1s0.yml 文件：
    ```
    interfaces:
    - name: enp1s0
        type: ethernet
        lldp:
        enabled: true
    ```
2. 使用以下内容创建 ~/enable-LLDP-enp1s0.yml 文件：
    ```
    # nmstatectl apply ~/enable-LLDP-enp1s0.yml
    ```
3. 显示 LLDP 信息：
    ```
    # nmstatectl show enp1s0
    - name: enp1s0
    type: ethernet
    state: up
    ipv4:
        enabled: false
        dhcp: false
    ipv6:
        enabled: false
        autoconf: false
        dhcp: false
    lldp:
        enabled: true
        neighbors:
        - - type: 5
            system-name: Summit300-48
        - type: 6
            system-description: Summit300-48 - Version 7.4e.1 (Build 5)
            05/27/05 04:53:11
        - type: 7
            system-capabilities:
            - MAC Bridge component
            - Router
        - type: 1
            _description: MAC address
            chassis-id: 00:01:30:F9:AD:A0
            chassis-id-type: 4
        - type: 2
            _description: Interface name
            port-id: 1/1
            port-id-type: 5
        - type: 127
            ieee-802-1-vlans:
            - name: v2-0488-03-0505
            vid: 488
            oui: 00:80:c2
            subtype: 3
        - type: 127
            ieee-802-3-mac-phy-conf:
            autoneg: true
            operational-mau-type: 16
            pmd-autoneg-cap: 27648
            oui: 00:12:0f
            subtype: 1
        - type: 127
            ieee-802-1-ppvids:
            - 0
            oui: 00:80:c2
            subtype: 2
        - type: 8
            management-addresses:
            - address: 00:01:30:F9:AD:A0
            address-subtype: MAC
            interface-number: 1001
            interface-number-subtype: 2
        - type: 127
            ieee-802-3-max-frame-size: 1522
            oui: 00:12:0f
            subtype: 4
    mac-address: 82:75:BE:6F:8C:7A
    mtu: 1500
    ```
4. 验证输出，以确保设置与您预期的配置匹配。例如，连接到交换机的接口的 LLDP 信息显示此主机连接的交换机端口使用 VLAN ID 448:
    ```
    - type: 127
            ieee-802-1-vlans:
            - name: v2-0488-03-0505
            vid: 488
    ```
    如果 enp1s0 接口的网络配置使用不同的 VLAN ID，请相应地进行修改。

# 第28章 以 keyfile 格式手动创建 NetworkManager 配置集

NetworkManager 支持以 keyfile 格式存储的配置集。但是，默认情况下，如果您使用 NetworkManager 工具（如 nmcli、networking rhel-system-roles或 nmstate API）来管理配置文件，NetworkManager 仍然会使用 ifcfg 格式的配置文件。

## 28.1. keyfile 格式的 NetworkManager 配置集

当在磁盘上存储连接配置集时， NetworkManager 将使用 INI 样式的 keyfile 格式。

**keyfile 格式的以太网连接配置集示例**

```
[connection]
id=example_connection
uuid=82c6272d-1ff7-4d56-9c7c-0eb27c300029
type=ethernet
autoconnect=true

[ipv4]
method=auto

[ipv6]
method=auto

[ethernet]
mac-address=00:53:00:8f:fa:66
```

每个部分都对应一个 NetworkManager 设置名称，NetworkManager keyfile 文件中的大多数变量都有一个一对一的映射。这意味着 NetworkManager 的属性作为相同名称的变量和相同格式存储在 keyfile 中。然而，有一些例外情况，主要是为了使 keyfile 语法更易于阅读。

出于安全考虑，由于连接配置文件可以包含敏感信息，如私钥和密语，NetworkManager 仅使用由 root 拥有的配置文件，并且仅可由 root 读和写。

根据连接配置文件的目的，将其保存在以下目录中：

- /etc/NetworkManager/system-connections/ ：用户创建的持久配置文件的通用位置，也可以对其进行编辑。NetworkManager 将它们自动复制到 /etc/NetworkManager/system-connections/。
- /run/NetworkManager/system-connections/ ：用于在重启系统时自动删除的临时配置文件。
- /usr/lib/NetworkManager/system-connections/ ：用于预先部署的不可变的配置文件。当您使用 NetworkManager API 编辑此类配置文件时，NetworkManager 会将此配置文件复制到持久性存储或临时存储中。

NetworkManager 不会自动从磁盘重新加载配置文件。当您以 keyfile 格式创建或更新连接配置集时，请使用 nmcli connection reload 命令告知 NetworkManager 更改。

## 28.2.以 keyfile 格式创建 NetworkManager 配置集

本节介绍如何以 keyfile 格式手动创建 NetworkManager 连接配置集。

注意，手动创建或更新配置文件可能会导致意外或无法正常工作的网络配置。OpenCloudOS 建议您使用 NetworkManager 工具，如 nmcli、网络 RHEL 系统角色或 nmstate API 来管理 NetworkManager 连接。

**流程**

1. 如果您为硬件接口（如以太网卡）创建了一个配置文件，请显示此接口的 MAC 地址：
    ```
    # ip address show enp1s0
    2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 00:53:00:8f:fa:66 brd ff:ff:ff:ff:ff:ff
    ```

2. 创建连接配置文件。例如，对于使用 DHCP 的以太网设备的连接配置文件，请使用以下内容创建 /etc/NetworkManager/system-connections/example.nmconnection 文件：
    ```
    [connection]
    id=example_connection
    type=ethernet
    autoconnect=true

    [ipv4]
    method=auto

    [ipv6]
    method=auto

    [ethernet]
    mac-address=00:53:00:8f:fa:66
    ```

您可以使用任何以 .nmconnection 为后缀的文件名。但是，当您稍后使用 nmcli 命令来管理连接时，您必须在引用此连接 id 变量中设置的连接名称。当省略 id 变量时，请使用不带 .nmconnection 的文件名来引用此连接。

3. 对配置文件设置权限，以便只有 root 用户可以读和更新它：
    ```
    # chown root:root /etc/NetworkManager/system-connections/example.nmconnection
    # chmod 600 /etc/NetworkManager/system-connections/example.nmconnection
    ```

4. 重新加载连接配置文件：
    ```
    # nmcli connection reload
    ```

5. 验证 NetworkManager 是否从配置文件读取配置文件：
    ```
    # nmcli -f NAME,UUID,FILENAME connection
    NAME                UUID                                  FILENAME
    example-connection  86da2486-068d-4d05-9ac7-957ec118afba  /etc/NetworkManager/system-connections/example.nmconnection
    ...
    ```
    如果命令未显示新添加的连接，请验证文件权限和您在文件中使用的语法是否正确。

6. 可选：如果您将配置文件中的 autoconnect 变量设为 false，请激活连接：
    ```
    # nmcli connection up example_connection
    ```

**验证**

1. 显示连接配置文件：
    ```
    # nmcli connection show example_connection
    ```

2. 显示接口的 IP 设置：
    ```
    # ip address show enp1s0
    ```

## 28.3.将 NetworkManager 配置集从 ifcfg 迁移到 keyfile 格式

您可以使用 nmcli connection migrate 命令将现有 ifcfg 连接配置集迁移到 keyfile 格式。这样，所有连接配置集都将位于一个位置和首选格式。

**前提条件**

- 在 /etc/sysconfig/network-scripts/ 目录中有 ifcfg 格式的连接配置集。

**流程**

- 迁移连接配置集
    ```
    # nmcli connection migrate
    Connection 'enp1s0' (43ed18ab-f0c4-4934-af3d-2b3333948e45) successfully migrated.
    Connection 'enp2s0' (883333e8-1b87-4947-8ceb-1f8812a80a9b) successfully migrated.
    ...
    ```

**验证**

- 验证是否成功迁移了连接配置集
    ```
    # nmcli -f TYPE,FILENAME,NAME connection
    TYPE      FILENAME                                                           NAME
    ethernet  /etc/NetworkManager/system-connections/enp1s0.nmconnection         enp1s0
    ethernet  /etc/NetworkManager/system-connections/enp2s0.nmconnection         enp2s0
    ...
    ```

## 28.4.使用 nmcli 以离线模式创建密钥文件连接配置集

您可以使用 nmcli --offline connection add 命令，以离线模式 keyfile 格式创建各种连接配置集。

离线模式可确保 nmcli 在没有 NetworkManager 服务的情况下运行，以通过标准输出生成密钥文件连接配置集。此功能在以下情况下很有用：

- 您想要创建需要预部署位置的连接配置集。例如，在容器镜像中，或 RPM 软件包。
- 您需要在 NetworkManager 服务不可用的环境中创建连接配置集。例如，当您想要使用 chroot 工具时。或者，当您要通过 Kickstart %post 脚本创建或修改 OpenCloudOS 系统的网络配置时。

您可以创建以下连接配置集类型：

- 静态以太网连接
- 动态以太网连接
- 网络绑定
- 网桥
- VLAN 或任何支持的连接类型

注意，手动创建或更新配置文件可能会导致意外或无法正常工作的网络配置。

**前提条件**

- NetworkManager 服务已停止。

**流程**

1. 以 keyfile 格式创建新连接配置集。例如，对于不使用 DHCP 的以太网设备的连接配置文件，请运行类似的 nmcli 命令：
    ```
    # nmcli --offline connection add type ethernet con-name Example-Connection ipv4.addresses 192.0.2.1/24 ipv4.dns 192.0.2.200 ipv4.method manual > /etc/NetworkManager/system-connections/output.nmconnection
    ```
    
    使用 con-name 键指定的连接名称保存在生成的配置集的 id 变量中。当您稍后使用 nmcli 命令管理这个连接时，请按如下所示指定连接：
     - 如果没有省略 id 变量，请使用连接名称，如 Example-Connection。
     - 当省略 id 变量时，请使用不带 .nmconnection 后缀的文件名，如 输出。

2. 对配置文件设置权限，以便只有 root 用户可以读和更新它：
    ```
    # chmod 600 /etc/NetworkManager/system-connections/output.nmconnection
    # chown root:root /etc/NetworkManager/system-connections/output.nmconnection
    ```

3. 启动 NetworkManager 服务：
    ```
    # systemctl start NetworkManager.service
    ```

4. 可选：如果您将配置文件中的 autoconnect 变量设为 false，请激活连接：
    ```
    # nmcli connection up Example-Connection
    ```

**验证**

1. 验证 NetworkManager 服务是否正在运行：
    ```
    # systemctl status NetworkManager.service
    ● NetworkManager.service - Network Manager
    Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; enabled; vendor preset: enabled)
    Active: active (running) since Wed 2022-08-03 13:08:32 CEST; 1min 40s ago
        Docs: man:NetworkManager(8)
    Main PID: 7138 (NetworkManager)
        Tasks: 3 (limit: 22901)
    Memory: 4.4M
    CGroup: /system.slice/NetworkManager.service
            └─7138 /usr/sbin/NetworkManager --no-daemon

    Aug 03 13:08:33 example.com NetworkManager[7138]: <info>  [1659524913.3600] device (vlan20): state change: secondaries -> activated (reason 'none', sys-iface-state: 'assume')
    Aug 03 13:08:33 example.com NetworkManager[7138]: <info>  [1659524913.3607] device (vlan20): Activation: successful, device activated.
    ...
    ```
2. 验证 NetworkManager 是否可以从配置文件中读取配置集：
    ```
    # nmcli -f TYPE,FILENAME,NAME connection
    TYPE      FILENAME                                                    NAME
    ethernet /etc/NetworkManager/system-connections/output.nmconnection Example-Connection
    ethernet  /etc/sysconfig/network-scripts/ifcfg-enp1s0                 enp1s0
    ...
    ```
    如果输出没有显示新创建的连接，请验证 keyfile 权限和您所用的语法是否正确。
3. 显示连接配置文件：
    ```
    # nmcli connection show Example-Connection
    connection.id:                          Example-Connection
    connection.uuid:                        232290ce-5225-422a-9228-cb83b22056b4
    connection.stable-id:                   --
    connection.type:                        802-3-ethernet
    connection.interface-name:              --
    connection.autoconnect:                 yes
    ...
    ```

# 第29章 使用 netconsole 通过网络记录内核信息

使用 netconsole 内核模块和同名的服务，当磁盘失败或不可能使用串行控制台时，您可以在日志通过网络记录内核消息，方便调试。

## 29.1.配置 netconsole 服务为将内核信息记录到远程主机

使用 netconsole 内核模块，您可以将内核信息记录到远程系统日志服务。

**前提条件**

- 远程主机上已安装系统日志服务，如 rsyslog 。
- 远程系统日志服务被配置为接收来自此主机的日志条目。

**流程**

1. 安装 netconsole-service 软件包：
    ```
    # yum install netconsole-service
    ```
2. 编辑 /etc/sysconfig/netconsole 文件，并将 SYSLOGADDR 参数设为远程主机的 IP 地址：
    ```
    # SYSLOGADDR=192.0.2.1
    ```
3. 启用并启动 netconsole 服务：
    ```
    # SYSLOGADDR=192.0.2.1
    ```

**验证**

- 在远程系统日志服务器上显示 /var/log/messages 文件。

# 第30章 systemd 网络目标和服务

NetworkManager 在系统引导过程中配置网络。但是，当使用远程 root(/)引导时，例如，如果 root 目录存储在 iSCSI 设备上，网络设置会在 RHEL 启动之前在初始 RAM 磁盘(initrd)中应用。例如，如果网络配置是在内核命令行上使用 rd.neednet=1 指定的，或者配置被指定为挂载远程文件系统，则网络设置将在 initrd 上应用。

本章描述了应用网络设置时使用的不同目标，如 network 、network-online、和 NetworkManager-wait-online 服务，以及如何配置 systemd 服务以使其在 network-online 服务启动后启动。

## 30.1.network 和 network-online systemd target 的不同

systemd 维护 network 和 network-online 目标单元。特殊单元，如 NetworkManager-wait-online.service，具有 WantedBy=network-online.target 和 Before=network-online.target 参数。如果启用了，这些单元将启动 network-online.target ，它们会延迟 network-online 目标，直到网络连接了。

network-online 目标启动一个服务，这会对进一步执行增加更长的延迟。systemd 会自动使用这个目标单元的 Wants 和 After 参数来向所有 System V(SysV) init 脚本服务单元添加依赖项，这些服务单元具有一个指向 $network 工具的 Linux Standard Base(LSB)头。LSB 头是 init 脚本的元数据。您可以使用它指定依赖项。这与 systemd 目标类似。

network 目标不会显著延迟引导进程的执行。到达 network 目标意味着，负责设置网络的服务已启动。但并不意味着已经配置了一个网络设备。这个目标在关闭系统的过程中非常重要。例如，如果您在引导过程中有一个排在 network 目标之后的服务，则这个依赖关系在关闭过程中会反过来。在服务停止后，网络才会断开连接。远程网络文件系统的所有挂载单元都会自动启动 network-online 目标单元，并在其之后排序。

注意，network-online 目标单元只在系统启动过程中有用。系统完成引导后，这个目标不会跟踪网络的在线状态。因此，您无法使用 network-online 来监控网络连接。这个目标提供了一个一次性系统启动概念。

## 30.2. NetworkManager-wait-online 概述

同步传统网络脚本会遍历所有配置文件来设置设备。它们应用所有与网络相关的配置并确保网络在线。

NetworkManager-wait-online 服务会等待要配置的网络的超时时间。这个网络配置涉及插入以太网设备、扫描 Wi-Fi 设备等。NetworkManager 会自动激活配置为自动启动的适当配置集。因 DHCP 超时或类似事件导致自动激活失败，网络管理器（NetworkManager）可能会在一定时间内处于忙碌状态。根据配置，NetworkManager 会重新尝试激活同一配置集或不同的配置集。

当启动完成后，所有配置集都处于断开连接的状态，或被成功激活。您可以配置配置集来自动连接。以下是一些参数示例，这些参数设定超时或者在连接被视为活跃时定义：

- connection.wait-device-timeout - 设置用来检测设备的驱动程序的超时时间
- ipv4.may-fail 和 ipv6.may-fail - 使用一个 IP 地址系列设置激活，或者一个特定的地址系列是否必须已完成配置。
- ipv4.gateway-ping-timeout - 延迟激活。

## 30.3.将 systemd 服务配置为在网络已启动后再启动

OpenCloudOS 在 /usr/lib/systemd/system/ 目录中安装 systemd 服务文件。此流程为 /etc/systemd/system/service_name.service.d/ 中的服务文件创建一个置入段，该文件与 /usr/lib/systemd/system/ 中的服务文件一起使用，以便在网络在线后启动特定的 服务。如果置入段中的设置与 /usr/lib/systemd/system/ 中服务文件中的设置重叠，则它具有更高的优先级。

**流程**

1. 要在编辑器中打开服务文件，请输入：
    ```
    # systemctl edit service_name
    ```
2. 输入以下内容并保存更改：
    ```
    [Unit]
    After=network-online.target
    ```
3. 输入以下内容并保存更改：
    ```
    # systemctl daemon-reload
    ```