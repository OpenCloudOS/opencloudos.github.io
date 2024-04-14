# Chapter 31 Linux Flow Control

Linux provides tools to manage and manipulate packet transmissions. The Linux Traffic Control (TC) subsystem helps in policing, classifying, controlling, and scheduling network traffic. TC can also take advantage of packet content segmentation during classification by using filters and actions. The TC subsystem does this through queuing disciplines (qdiscs), a fundamental element of the TC architecture.

Scheduling mechanisms determine or reschedule packets before entering or exiting different queues. The most common scheduler is the first-in-first-out (FIFO) scheduler. You can perform qdiscs operations temporarily using the tc tool, or permanently using NetworkManager.

This chapter explains queuing disciplines and describes how to update the default qdiscs in OpenCloudOS.

## 31.1. Overview of queuing disciplines

Queuing disciplines (qdiscs) help plan queuing and then schedule traffic delivery across network interfaces. qdisc has two operations:
- Queue requests to queue packets for later transmission
- Dequeue requests so that one of the queued packets can be selected for immediate transmission.

Each qdisc has a 16-digit hexadecimal identification number called handle , with an appended colon, like 1: or abcd: this number is called the qdisc main number. If the qdisc has a type, the identifier is a pair of two digits with the major number preceding the minor number, i.e. \<major>:\<minor>, eg abcd:1. The numbering scheme for minor numbers depends on the qdisc type. Sometimes the numbering is systematized, where the first class has an ID of < \<major>:1, the second class has an ID of \<major>:2, and so on. Some qdiscs allow the user to randomly set the minor number of a class when creating the class.


**Classful qdiscs**

There are different types of qdiscs that help to receive and transmit packets from network interfaces. You can configure qdiscs using root, parent, or subclasses. The places where subobjects can be attached are called classes. Classes in a qdisc are flexible and always contain multiple subclasses or a single qdisc. Classes containing the class qdisc itself are not prohibited, which facilitates complex flow control scenarios.

The class qdiscs itself does not store any packets. Instead, they pass enqueue and dequeue requests to one of their subclasses according to the criteria specified in the qdisc. Finally, this recursive packet is delivered to where the packet is finally saved (or extracted from if there is a queue).

**Classless qdiscs**

Some qdiscs do not contain subclasses, they are called classless qdiscs. Classless qdiscs require less customization than class-like qdiscs. Usually, it is sufficient to attach them to the interface.

## 31.2. qdiscs available in OpenCloudOS

Each qdisc solves uniquely corresponding and related network problems. Below is a list of qdiscs available in OpenCloudOS. You can use any of the following qdiscs to shape network traffic according to your network requirements.

| qdisc name | module in which | offload support |
|-----------|----------|----------|
| Asynchronous Transfer Mode (ATM) |kernel-modules-extra| |
| Class-based queues |kernel-modules-extra| |
| credit-based builder |kernel-modules-extra| yes |
|CHOose and Keep for responsive traffic, CHOose and Kill for unresponsive traffic (CHOKE) |kernel-modules-extra| |
|Controlled Latency (CoDel) |kernel-core| |
|Round Robin (DRR)|kernel-modules-extra| |
|Differentiated Services marker (DSMARK)|kernel-modules-extra|   |
|Enhanced Transmission Selection (ETS)|kernel-modules-extra| yes  |
|Fair Queue (FQ)|kernel-core|   |
|Fair Queuing Controlled Delay (FQ_CODel)|kernel-core|   |
|Generalized Random Early Detection (GRED)|kernel-modules-extra|   |
|Hierarchical Fair Service Curve (HSFC)|kernel-core|   |
|Heavy-Hitter Filter (HHF)|kernel-core|   |
|Hierarchy Token Bucket (HTB)|kernel-core|   |
|INGRESS|kernel-core| yes  |
|Multi Queue Priority (MQPRIO)|kernel-modules-extra| yes  |
|Multiqueue (MULTIQ)|kernel-modules-extra| yes  |
|Network Emulator (NETEM)|kernel-modules-extra|   |
|Proportional Integral-controller Enhanced (PIE)|kernel-core|   |
|PLUG|kernel-core|   |
|Quick Fair Queueing (QFQ)|kernel-modules-extra|   |
|Random Early Detection (RED)|kernel-modules-extra| yes  |
|Stochastic Fair Blue (SFB)|kernel-modules-extra|   |
|Stochastic Fairness Queueing (SFQ)|kernel-core|   |
|Token Bucket Filter (TBF)|kernel-core| yes  |
|Trivial Link Equalizer (TEQL)|kernel-modules-extra|   |
||||

Note that qdisc offload requires hardware and driver support for the NIC.

## 31.3. Check qdiscs of a network interface using the tc tool

By default, the OpenCloudOS system uses fq_codel , qdisc. This procedure describes how to check qdisc counters.

**process**

1. Optional: View your current qdisc:
    ```
    # tc qdisc show dev enp0s1
    ```
2. Check the current qdisc counters:
    ```
    # tc -s qdisc show dev enp0s1
    qdisc fq_codel 0: root refcnt 2 limit 10240p flows 1024 quantum 1514 target 5.0ms interval 100.0ms memory_limit 32Mb ecn
    Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
    backlog 0b 0p requeues 0
    maxpacket 0 drop_overlimit 0 new_flow_count 0 ecn_mark 0
    new_flows_len 0 old_flows_len 0
    ```
3. Update an existing qdisc:
     ```
     # sysctl -w net.core.default_qdisc=pfifo_fast
     ```
4. Reload the network driver to apply the changes:
     ```
     # rmmod NETWORKDRIVERNAME
     # modprobe NETWORKDRIVERNAME
     ```
5. Start the network interface:
     ```
     # IP link set enp0s1 up
     ```

**verify**

- View qdiscs for ethernet connections:
     ```
     # tc -s qdisc show dev enp0s1
     qdisc pfifo_fast 0: root refcnt 2 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
     Sent 373186 bytes 5333 pkt (dropped 0, overlimits 0 requeues 0)
     backlog 0b 0p requeues 0
     ....
     ```

## Temporarily set the qdisk of the network interface using the tc tool

You can update the current temporary qdisc without changing the default qdisc.

**process**

**process**

1. Optional: View the current qdisc:
     ```
     # tc -s qdisc show dev enp0s1
     ```
2. Update the current qdisc:
     ```
     # tc qdisc replace dev enp0s1 root htb
     ```

**verify**

- View updated current qdisc:
     ```
     # tc -s qdisc show dev enp0s1
     qdisc htb 8001: root refcnt 2 r2q 10 default 0 direct_packets_stat 0 direct_qlen 1000
     Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
     backlog 0b 0p requeues 0
     ```

## 31.6. Use NetworkManager to permanently set the current qdisk of a network interface

**process**

1. Optional: View the current qdisc:
     ```
     # tc qdisc show dev enp0s1
      qdisc fq_codel 0: root refcnt 2
     ```
2. Update the current qdisc:
     ```
     # nmcli connection modify enp0s1 tc.qdiscs 'root pfifo_fast'
     ```
3. Optional: To add another qdisc on top of an existing qdisc, use the +tc.qdisc option:
     ```
     # nmcli connection modify enp0s1 +tc.qdisc 'ingress handle ffff:'
     ```
4. Activate the changes:
     ```
     # nmcli connection up enp0s1
     ```

**verify**

- View updated current qdisc:
     ```
     # tc qdisc show dev enp0s1
     qdisc pfifo_fast 8001: root refcnt 2 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
     qdisc ingress ffff: parent ffff:fff1 ----------------
     ```

# Chapter 32 Configure DNS server order

Most applications use the getaddrinfo() function of the glibc library to resolve DNS requests. By default, glibc sends all DNS requests to the first DNS server specified in the /etc/resolv.conf file. If this server does not reply, OpenCloudOS will use the next server in this file.

This chapter describes how to customize the DNS server order.

## 32.1. Order DNS servers in /etc/resolv.conf using NetworkManager

NetworkManager sorts the DNS servers in the /etc/resolv.conf file according to the following rules:
- If there is only one connection profile, NetworkManager will use the order of IPv4 and IPv6 DNS servers specified in that connection.
- If multiple connection profiles are activated, NetworkManager sorts the DNS servers according to the DNS priority value. If you set DNS priority, NetworkManager's behavior depends on the value set in the dns parameter. You can set this parameter in the [main] section of the /etc/NetworkManager/NetworkManager.conf file:

   - dns=default or if the dns parameter is not set:

     NetworkManager sorts the DNS servers for different connections according to the ipv4.dns-priority and ipv6.dns-priority parameters in each connection.

     If no value is set, or if you set ipv4.dns-priority and ipv6.dns-priority to 0 , NetworkManager will use the global defaults. See the default value for the DNS Priority parameter.

   - dns=dnsmasq or dns=systemd-resolved:

     When you use one of these settings, NetworkManager sets 127.0.0.1 or 127.0.0.53 for dnsmasq as the nameserver entry in the /etc/resolv.conf file.

     The dnsmasq and systemd-resolved services forward queries for search domains set in a NetworkManager connection to the DNS servers specified in that connection, and forward queries for other domains to connections with default routes. When multiple connections have the same set of search domains, dnsmasq and systemd-resolved forward queries for this domain to the DNS server set in the connection with the lowest priority value.

Default value for the DNS priority parameter
NetworkManager uses the following defaults for connections:

- 50 for VPN connections
- 100 for other connections


Valid DNS priority values:
You can set the global default and connection-specific ipv4.dns-priority and ipv6.dns-priority parameters to values between -2147483647 and 2147483647.

- Lower values have higher priority.
- Negative values have a special effect that exclude other configurations with larger values. For example, if at least one connection has a negative priority value, NetworkManager only uses the DNS server with the lowest priority specified in the connection configuration set.
- If multiple connections have the same DNS priority, NetworkManager prioritizes DNS in the following order:

     1. VPN connection
     2. A connection with an active default route. The active default route is the default route with the lowest metric.



## 32.2. Set NetworkManager DNS default server priority value

NetworkManager uses the following DNS priority defaults for connections:

- 50 for VPN connections
- 100 for other connections

This section discusses how to override these system-wide defaults with custom defaults for IPv4 and IPv6 connections.

**process**

1. Edit the /etc/NetworkManager/NetworkManager.conf file:
    1. Add the [connection] section (if not present):
     ```
     [connection]
     ```
    2. Add custom defaults to the [connection] section. For example, to set a new default of 200 for IPv4 and IPv6, add:
     ```
     ipv4.dns-priority=200
     ipv6.dns-priority=200
     ```
     You can set the parameter to a value between -2147483647 and 2147483647. Note that setting the parameter to 0 will enable the built-in default ( 50 for VPN connections and 100 for other connections).

2. Reload the NetworkManager service:
     ```
     # systemctl reload NetworkManager
     ```

## 32.3. Set DNS priority for NetworkManager connections

This section describes how to define the order of DNS servers when NetworkManager creates or updates the /etc/resolv.conf file.

Note that setting DNS priority only makes sense if you have configured multiple connections to multiple different DNS servers. If you have only one connection configured with multiple DNS servers, manually set the DNS servers sequentially in the connection configuration set.

**Prerequisites**

- The system is configured with multiple NetworkManager connections.
- The system does not set the dns parameter in the /etc/NetworkManager/NetworkManager.conf file, or it is set to default.

**process**

1. Optionally, display available connections:
     ```
     # nmcli connection show
     NAME UUID TYPE DEVICE
     Example_con_1 d17ee488-4665-4de2-b28a-48befab0cd43 ethernet enp1s0
     Example_con_2 916e4f67-7145-3ffa-9f7b-e7cada8f6bf7 ethernet enp7s0
     ...
     ```
2. Set the ipv4.dns-priority and ipv6.dns-priority parameters. For example, for
Example_con_1 connection, with both parameters set to 10 :
     ```
     # nmcli connection modify Example_con_1 ipv4.dns-priority 10 ipv6.dns-priority 10
     ```
3. Reactivate your updated connection:
     ```
     # nmcli connection up Example_con_1
     ```

**verify**

- Display the contents of the /etc/resolv.conf file to verify that the DNS servers are in the correct order:
     ```
     # cat /etc/resolv.conf
     ```

# Chapter 33 Using ifcfg file to configure ip network

This chapter describes how to manually configure network interfaces by editing the ifcfg file.

NetworkManager supports configuration sets stored in keyfile format. However, NetworkManager uses the ifcfg format by default when creating or updating configuration files using the NetworkManager API.

Interface configuration (ifcfg) files control the software interfaces of individual network devices. When the system boots, it uses these files to decide which interfaces to start and how to configure them. These files are usually named ifcfg-name, where the suffix name refers to the name of the device controlled by the configuration file. By convention, the ifcfg file has the same suffix as the string provided by the DEVICE directive in the configuration file.

## 33.1. Using the ifcfg file to configure an interface with static network settings

This section describes how to configure network interfaces using the ifcfg file.

**process**

- If you need to configure an interface with static network settings for an interface named enp1s0, create a file named ifcfg-enp1s0 in the /etc/sysconfig/network-scripts/ directory with the following content:
   - For IPv4 configuration:
     ```
     DEVICE=enp1s0
     BOOTPROTO=none
     ONBOOT=yes
     PREFIX=24
     IPADDR=10.0.1.27
     GATEWAY=10.0.1.1
     ```
   - For IPv6 configuration:
     ```
     DEVICE=enp1s0
     BOOTPROTO=none
     ONBOOT=yes
     IPV6INIT=yes
     IPV6ADDR=2001:db8:1::2/64
     ```

## 33.2. Using the ifcfg file to configure an interface with dynamic network settings

This section describes how to use the ifcfg file to configure a network interface with dynamic network settings.

**process**

1. If you need to configure an interface with dynamic network settings for an interface named em1, create a file named ifcfg-em1 in the /etc/sysconfig/network-scripts/ directory with the following content:
     ```
     DEVICE=em1
     BOOTPROTO=dhcp
     ONBOOT=yes
     ```

2. To configure the sending interface:
  - If the DHCP server uses a different hostname, add the following line to the ifcfg file:
     ```
     DHCP_HOSTNAME=hostname
     ```
  - The DHCP server uses a different Fully Qualified Domain Name (FQDN), add the following line to the ifcfg file:
     ```
     DHCP_FQDN=fully.qualified.domain.name
     ```

Note that you can only use one of these settings. If you specify both DHCP_HOSTNAME and DHCP_FQDN, only DHCP_FQDN is used.

3. To configure the interface to use specific DNS servers, add the following line to the ifcfg file:
     ```
     PEERDNS=no
     DNS1=ip-address
     DNS2=ip-address
     ```
     where ip-address is the address of the DNS server. This causes the web service to update /etc/resolv.conf with the specified DNS servers. Only one DNS server address is required, the other is optional.

## 33.3. Using the ifcfg file to manage system-wide and private connection configuration sets

This section describes how to configure the ifcfg file to manage system-wide and private connection configuration files.

**process**
Access rights correspond to the USERS directive in the ifcfg file. Without the USERS directive, the network configuration file is available to all users.

- For example, modify the ifcfg file with the following line, which will make the connection only available to the listed users:
     ```
     USERS="joe bob alice"
     ```

# Chapter 34 Disabling IPv6 for Specific Connections Using NetworkManager

This section describes how to disable the IPv6 protocol on systems using NetworkManager to manage network interfaces. If you disable IPv6, NetworkManager automatically sets the corresponding sysctl value in the kernel. If IPv6 is disabled using a kernel tunable or kernel boot parameter, additional system configuration considerations must be taken.

**Prerequisites**
- The system uses NetworkManager to manage network interfaces

## 34.1. Disable IPv6 in connection using nmcli

**process**

1. Optionally, display a list of network connections:
     ```
     # nmcli connection show
     NAME UUID TYPE DEVICE
     Example 7a7e0151-9c18-4e6f-89ee-65bb2d64d365 ethernet enp1s0
     ...
     ```
2. Display a list of network connections:
     ```
     # nmcli connection modify Example ipv6.method "disabled"
     ```
3. Restart the network connection:
     ```
     # nmcli connection up Example
     ```

**verify**

1. Enter the ip address show command to display the IP settings of the device:
     ```
     # ip address show enp1s0
     2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
         link/ether 52:54:00:6b:74:be brd ff:ff:ff:ff:ff:ff
         inet 192.0.2.1/24 brd 192.10.2.255 scope global noprefixroute enp1s0
         valid_lft forever preferred_lft forever
     ```
     If no inet6 entry is displayed, IPv6 is disabled on the device.

2. Verify that the /proc/sys/net/ipv6/conf/enp1s0/disable_ipv6 file now contains the value 1 :
     ```
     # cat /proc/sys/net/ipv6/conf/enp1s0/disable_ipv6
     1
     ```
     A value of 1 disables IPv6 for this device.

# Chapter 35 Manually configure the /etc/resolv.conf file

By default, NetworkManager on OpenCloudOS dynamically updates the /etc/resolv.conf file with the DNS settings in the connection configuration file. This chapter describes how to disable this feature to manually configure DNS settings in /etc/resolv.conf.

## 35.1. Disable DNS handling in NetworkManager configuration

This section describes how to manually configure the /etc/resolv.conf file to disable DNS processing in the NetworkManager configuration.

**process**

1. As the root user, use a text editor to create the /etc/NetworkManager/conf.d/90-dns-none.conf file with the following content:
     ```
     [main]
     dns=none
     ```

2. Reload the NetworkManager service:
     ```
     # systemctl reload NetworkManager
     ```
     Note that NetworkManager no longer updates the /etc/resolv.conf file after a service reload. But the last content of the file will be preserved.

3. (Optional) Remove NetworkManager-generated comments from /etc/resolv.conf for easier reading.

**verify**

1. Edit the /etc/resolv.conf file and manually update the configuration.
2. Reload the NetworkManager service:
     ```
     # systemctl reload NetworkManager
     ```
3. Display the /etc/resolv.conf file:
     ```
     # cat /etc/resolv.conf
     ```
     If you successfully disable DNS processing, NetworkManager will not override manually configured settings.

## 35.2. Manually configure DNS settings by replacing /etc/resolv.conf with symlinks

NetworkManager does not automatically update the DNS configuration if /etc/resolv.conf is a symbolic link. This section describes how to replace /etc/resolv.conf with symlinks to files with DNS configuration.

**Prerequisites**

- The rc-manager option is not set to file. To verify, use the NetworkManager --print-config command.

**process**

1. Create a file, such as /etc/resolv.conf.manually-configured, and add your environment's DNS configuration to it. Use the same parameters and syntax as in the original /etc/resolv.conf.
2. Delete the /etc/resolv.conf file:
     ```
     # rm /etc/resolv.conf
     ```
3. Create a symbolic link named /etc/resolv.conf that points to /etc/resolv.conf.manually-configured :
     ```
     # ln -s /etc/resolv.conf.manually-configured /etc/resolv.conf
     ```

# Chapter 36 Monitoring and Adjusting NIC Ring Buffer

The receive ring buffer is a shared buffer shared between the device driver and the network interface controller (NIC). The card allocates a transmit (TX) and receive (RX) ring buffer. As the name suggests, a ring buffer is a circular buffer where overflowing data overwrites existing data. There are two methods of moving data from the NIC to the core, hardware interrupts and software interrupts, also known as SoftIRQs.

The RX ring buffer is used by the kernel to store incoming packets until they can be processed by a device driver. Device drivers typically use SoftIRQ to drain the RX ring, which places incoming packets into a kernel data structure called sk_buff or skb, and the packet begins its journey through the kernel until it reaches the application owning the associated socket .

The kernel uses a TX ring buffer to hold outgoing packets destined for the wire.

These ring buffers sit at the bottom of the stack and are key points where packet loss can occur, which can also adversely affect network performance.

## 36.1. Display the number of dropped packets

The ethtool utility allows administrators to query, configure, or control network driver settings.

Exhaustion of the RX ring buffer results in incremented counters such as "discard" or "drop" in the output of ethtool -S interface_name. A dropped packet means that the available buffer is filling up faster than the kernel can process the packet.

This section describes how to display drop counters using ethtool.

**process**

- To view the drop counters for the enp1s0 interface, enter:
     ```
     $ ethtool -S enp1s0
     ```

## 36.2. Increase the RX ring buffer to reduce the rate of packet discarding

The ethtool tool can help to increase the RX buffer size to reduce the high drop rate of packets.

**process**

1. View the maximum value of the RX ring buffer:
     ```
     # ethtool -g enp1s0
     Ring parameters for enp1s0:
     Pre-set maximums:
     RX: 4080
     RX Mini: 0
     RX Jumbo: 16320
     TX: 255
     Current hardware settings:
     RX: 255
     RX Mini: 0
     RX Jumbo: 0
     TX: 255
     ```

2. If the value in the Pre-set maximums section is greater than the Current hardware settings section, increase the RX ring buffer:
     - To temporarily change the RX ring buffer of the enp1s0 device to 4080, enter:
         ```
         # ethtool -G enp1s0 rx 4080
         ```
     - To permanently change the RX ring buffer, create a NetworkManager allocator script.

Note that, depending on the drivers used by your network card, changes to the ring buffer can break network connections very quickly.