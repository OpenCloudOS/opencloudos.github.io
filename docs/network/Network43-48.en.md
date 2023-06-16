# Chapter 43: Reusing the Same IP Address on Different Interfaces

By using Virtual Routing and Forwarding (VRF), administrators can utilize multiple routing tables simultaneously on the same host. VRF partitions the network at layer 3, enabling administrators to isolate traffic using independent routing tables for each VRF domain. This technique is similar to Virtual LANs (VLANs), which partition the network at layer 2, where different VLAN tags are used by the operating system to isolate traffic sharing the same physical media.

Compared to partitioning at layer 2, one advantage of VRF is that routing can scale better, taking into account the number of peer routes involved.

OpenCloudOS utilizes virtual vrt devices for each VRF domain and adds routes to the VRF domain by adding existing network devices to the VRF device. The addresses and routes previously attached to the original device will be moved to the VRF domain.

Note that each VRF domain is isolated from each other

## 43.1. Reusing the Same IP Address Permanently on Different Interfaces

This section describes how to use VRF functionality to permanently reuse the same IP address on different interfaces of the same server.

In order to allow remote peers to communicate with both VRF interfaces using the same IP address, the network interfaces must belong to different broadcast domains. A broadcast domain in a network is a group of nodes that receive any broadcast traffic sent by any node within the group. In most configurations, all nodes connected to the same switch belong to the same domain.

**Prerequisites.**

- Log in as the root user.
- No network interfaces are configured.

**The process is as follows:**

1. Create and configure the first VRF device:

   - Create a connection for the VRF device and assign it to a routing table. For example, to create a VRF device named "vrf0" and assign it to routing table 1001:


   ```
   # nmcli connection add type vrf ifname vrf0 con-name vrf0 table 1001 ipv4.method disabled ipv6.method disabled
   ```

   - Activate the vrf0 device:

   ```
   # nmcli connection up vrf0
   ```

   - Assign network devices to the newly created VRF. For example, to add the Ethernet device enp1s0 to the vrf0 VRF device and assign an IP address and subnet mask to enp1s0, enter:

   ```
   # nmcli connection add type ethernet con-name vrf.enp1s0 ifname enp1s0 master vrf0 ipv4.method manual ipv4.address 192.0.2.1/24
   ```

   - Activate the vrf.enp1s0 connection:

   ```
   # nmcli connection up vrf.enp1s0
   ```

2. Create and configure the next VRF device:

   - Create a VRF device and assign it to a routing table. For example, to create a VRF device named "vrf1" and assign it to routing table 1002:

   
   ```
   # nmcli connection add type vrf ifname vrf1 con-name vrf1 table 1002 ipv4.method disabled ipv6.method disabled
   ```
   
   - Activate the vrf1 device:
   
   
   ```
   # nmcli connection up vrf1
   ```

   - Assign network devices to the newly created VRF. For example, to add the Ethernet device enp7s0 to the vrf1 VRF device and assign an IP address and subnet mask to enp7s0, enter:
   
   
   ```
   # nmcli connection add type ethernet con-name vrf.enp7s0 ifname enp7s0 master vrf1 ipv4.method manual ipv4.address 192.0.2.1/24
   ```
   
   - Activate the vrf.enp7s0 device:
   
   ```
   # nmcli connection up vrf.enp7s0
   ```

## 43.2. Reusing the same IP address temporarily across different interfaces

This section describes how to use Virtual Routing and Forwarding (VRF) to temporarily reuse the same IP address across different interfaces of a server. This process is intended for testing purposes only, as the configuration is temporary and will be lost upon system reboot.

To enable remote peers to reach both VRF interfaces when reusing the same IP address, the network interfaces must belong to different broadcast domains. A broadcast domain is a group of nodes that receive broadcast traffic sent by any member of the group. In most configurations, all nodes connected to the same switch belong to the same domain.

**Prerequisites:**

- Log in as the root user.
- No network interfaces are configured.

**Process**

1. Create and configure the first VRF device:

   - Create a VRF device and assign it to a routing table. For example, to create a VRF device named "blue" and assign it to routing table 1001:

   ```
   # ip link add dev blue type vrf table 1001
   ```

   - Activate the "blue" device:

   ```
   # ip link set dev blue up
   ```

   - Assign network devices to the VRF device. For example, to add the Ethernet device "enp1s0" to the "blue" VRF device:

   ```
   # ip link set dev enp1s0 master blue
   ```

   - Activate the "enp1s0" device:

   ```
   # ip link set dev enp1s0 up
   ```

   - Assign an IP address and subnet mask to the "enp1s0" device. For example, set it to 192.0.2.1/24:

   ```
   # ip addr add dev enp1s0 192.0.2.1/24
   ```

2. Create and configure the next VRF device:

   - Create a VRF device and assign it to a routing table. For example, to create a VRF device named "red" and assign it to routing table 1002:

   ```
   # ip link add dev red type vrf table 1002
   ```

   - Activate the "red" device:

   ```
   # ip link set dev red up
   ```

   - Assign network devices to the VRF device. For example, to add the Ethernet device "enp7s0" to the "red" VRF device:

   ```
   # ip link set dev enp7s0 master red
   ```

   - Activate the "enp7s0" device:

   ```
   # ip link set dev enp7s0 up
   ```

   - Assign the same IP address and subnet mask used by the "enp1s0" device in the "blue" VRF domain to the "enp7s0" device:

   ```
   # ip addr add dev enp7s0 192.0.2.1/24
   ```

3. Optionally, you can create more VRF devices by following the above steps.

# Chapter 44: Starting Services in an Isolated VRF Network

Using Virtual Routing and Forwarding (VRF), you can create an isolated network with a routing table that is different from the operating system's main routing table. Then, you can start services and applications so that they can only access the network defined in that routing table.

## 44.1. Configuring VRF Devices

To use Virtual Routing and Forwarding (VRF), you can create a VRF device and attach physical or virtual network interfaces and routing information to it.

To prevent yourself from being remotely locked out, execute this process on the local console or through a network interface that you do not want to assign to the VRF device.

**Prerequisites**

- You have logged in locally or are using a network interface different from the one you want to assign to the VRF device.

**Process**

1. Create a VRF device named "vrf0" using the same-named virtual device and attach it to routing table 1000:

   ```
   # nmcli connection add type vrf ifname vrf0 con-name vrf0 table 1000 ipv4.method disabled ipv6.method disabled
   ```

2. Add the "enp1s0" device to the "vrf0" connection and configure IP settings:

   ```
   # nmcli connection add type ethernet con-name enp1s0 ifname enp1s0 master vrf0 ipv4.method manual ipv4.address 192.0.2.1/24 ipv4.gateway 192.0.2.254
   ```

   This command creates an "enp1s0" connection as the port for the "vrf0" connection. With this configuration, routing information is automatically assigned to routing table 1000 associated with the "vrf0" device.

3. If you need static routes in the isolated network:

   - Add static routes:

      ```
      # nmcli connection modify enp1s0 +ipv4.routes "198.51.100.0/24 192.0.2.2"
      ```

      This adds a route to the network 198.51.100.0/24 using 192.0.2.2 as the router.

   - Activate the connection:

     ```
     # nmcli connection up enp1s0
     ```
   



**Verification**

1. Display the IP settings of the devices associated with "vrf0":

   ```
   # ip -br addr show vrf vrf0
   enp1s0    UP    192.0.2.15/24
   ```

2. Display the VRF device and its associated routing table:

   ```
   # ip vrf show
   Name              Table
   -----------------------
   vrf0              1000
   ```

3. Display the main routing table:

   ```
   # ip route show
   default via 192.168.0.1 dev enp1s0 proto static metric 100
   ```

4. Display routing table 1000:

   ```
   # ip route show table 1000
   default via 192.0.2.254 dev enp1s0 proto static metric 101
   broadcast 192.0.2.0 dev enp1s0 proto kernel scope link src 192.0.2.1
   192.0.2.0/24 dev enp1s0 proto kernel scope link src 192.0.2.1 metric 101
   local 192.0.2.1 dev enp1s0 proto kernel scope host src 192.0.2.1
   broadcast 192.0.2.255 dev enp1s0 proto kernel scope link src 192.0.2.1
   198.51.100.0/24 via 192.0.2.2 dev enp1s0 proto static metric 101
   ```

   The "default" entry indicates that services using this routing table will use 192.0.2.254 as their default gateway instead of the default gateway in the main routing table.

5. Execute the traceroute tool in the network associated with "vrf0" to verify that the tool is using the routing of table 1000:

   ```
   # ip vrf exec vrf0 traceroute 203.0.113.1
   traceroute to 203.0.113.1 (203.0.113.1), 30 hops max, 60 byte packets
   1  192.0.2.254 (192.0.2.254)  0.516 ms  0.459 ms  0.430 ms
   ...
   ```

   The first hop is the default gateway assigned to routing table 1000 instead of the default gateway in the system's main routing table.

   

## 44.2. Starting Services in an Isolated VRF Network

You can configure services, such as the Apache HTTP server, to start within an isolated Virtual Routing and Forwarding (VRF) network.

Note that services can only bind to local IP addresses within the same VRF network.

**Prerequisites**

- You have configured the "vrf0" device.
- You have configured the Apache HTTP server to only listen on the IP address assigned to the interface associated with the "vrf0" device.

**Process**

1. Display the contents of the httpd systemd service:

   ```
# systemctl cat httpd
   ...
   [Service]
   ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
   ...
   ```
   
   The contents of the ExecStart parameter will be required in the subsequent steps to run the same command within the isolated VRF network.

2. Create the /etc/systemd/system/httpd.service.d/ directory:

   ```
   # mkdir /etc/systemd/system/httpd.service.d/
   ```

3. Create the /etc/systemd/system/httpd.service.d/override.conf file with the following content:

   ```
   [Service]
   ExecStart=
   ExecStart=/usr/sbin/ip vrf exec vrf0 /usr/sbin/httpd $OPTIONS -DFOREGROUND
   ```

   To override the ExecStart parameter, you need to unset it first and then set it to the new value shown.

4. Reload systemd:

   ```
   # systemctl daemon-reload
   ```

5. Restart the httpd service:

   ```
   # systemctl restart httpd
   ```

**Verification**

1. Display the process ID (PID) of the httpd process:

   ```
   # pidof -c httpd
   1904 ...
   ```

2. Display the VRF association of the PID, for example:

   ```
   # ip vrf identify 1904
   vrf0
   ```

3. Display all PIDs associated with the "vrf0" device:

   ```
   # ip vrf pids vrf0
   1904  httpd
   ...
   ```

# Chapter 45: Setting up Routing Protocols for Your System

This section describes how to use the Free Range Routing (FRRouting or FRR) feature to enable and configure the desired routing protocols for your system.

## 45.1. Introduction to FRRouting

Free Range Routing (FRRouting or FRR) is a routing protocol stack provided by the "frr" package in the AppStream repository.

FRR provides TCP/IP-based routing services and supports multiple IPv4 and IPv6 routing protocols.

Supported protocols include:

- Border Gateway Protocol (BGP)
- Intermediate System to Intermediate System (IS-IS)
- Open Shortest Path First (OSPF)
- Protocol Independent Multicast (PIM)
- Routing Information Protocol (RIP)
- Routing Information Protocol next generation (RIPng)
- Enhanced Interior Gateway Routing Protocol (EIGRP)
- Next Hop Resolution Protocol (NHRP)
- Bidirectional Forwarding Detection (BFD)
- Policy Based Routing (PBR)

FRR is a collection of the following services:

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

If FRR is installed, the system can act as a dedicated router that exchanges routing information with other routers in internal or external networks using routing protocols.

## 45.2. Setting up FRRouting

This section explains how to set up Free Range Routing (FRRouting or FRR).

**Prerequisites**

- Make sure that the "frr" package is installed on your system:

  ```
  # yum install frr
  ```

**Process**

1. Edit the /etc/frr/daemons configuration file and enable the required daemons for your system.

   For example, to enable the "ripd" daemon, include the following line:

   ```
   ripd=yes
   ```

   The "zebra" daemon must always be enabled, so you must set "zebra=yes" to use FRR. By default, the /etc/frr/daemons file contains [daemon_name]=no entries for all daemons, which disables all daemons. Therefore, starting FRR is ineffective after a fresh installation of the system.

2. Start the FRR service:

   ```
   # systemctl start frr
   ```

3. Alternatively, you can set FRR to start automatically at boot:

   ```
   # systemctl enable frr
   ```

## 45.3. Modifying the Configuration of FRR

This section covers:

- How to enable additional daemons after setting up FRR
- How to disable daemons after setting up FRR

Prerequisites

- FRR has been set up as described in "Setting up FRRouting".

**Process**

1. Edit the /etc/frr/daemons configuration file and change the lines for the desired daemons from "no" to "yes".

   For example, enable the "ripd" daemon:

   ```
   ripd=yes
   ```

2. Reload the FRR service:

   ```
   # systemctl reload frr
   ```

## 45.4. Modifying the Configuration of Specific Daemons

With the default configuration, each routing daemon in FRR can only act as a regular router.

To perform additional configuration for a daemon, use the following steps.

**Process**

1. In the /etc/frr/ directory, create a configuration file for the desired daemon and name the file:

   ```
[daemon_name].conf
   ```
   
   For example, to further configure the "eigrpd" daemon, create an "eigrpd.conf" file in the directory mentioned above.

2. Populate the new file with the desired content.

   For examples of configuration for specific FRR daemons, see the /usr/share/doc/frr/ directory.

3. Reload the FRR service:

   ```
   # systemctl reload frr
   ```

# Chapter 46: Testing Basic Network Settings

## 46.1. Verifying Connectivity to Other Hosts with the Ping Program

The ping tool sends ICMP packets to a remote host, which can be used to test if the IP connectivity to different hosts is working correctly.

**Process**

- Put the IP address of the host in the same subnet as your default gateway:

  ```
# ping 192.0.2.3
  ```
  
  If the command fails, verify the default gateway settings.

- Specify the IP address of the host in a remote subnet:

  ```
# ping 198.162.3.1
  ```
  
  If the command fails, verify the default gateway settings and ensure that the gateway is forwarding packets between connected networks.


## 46.2. Verifying Domain Name Resolution with the Host Program

**Process**

- Use the host utility to verify that name resolution is working correctly. For example, to resolve the hostname "client.example.com" to an IP address, enter:

  ```
# host client.example.com
  ```
  
  If the command returns an error, such as "connection timed out" or "no servers could be reached," verify your DNS settings.


# Chapter 47: Using the NetworkManager Program to Schedule dhclient Exit Hooks Scripts

## 47.1. Concept of Scheduling Scripts with the NetworkManager Program

When a network event occurs, the NetworkManager-dispatcher service executes user-provided scripts in alphabetical order. These scripts are usually shell scripts, but can also be any executable script or application. For example, you can use scheduled scripts to adjust network-related settings that you cannot manage with NetworkManager.

You can store scheduled scripts in the following directories:

- /etc/NetworkManager/dispatcher.d/: A common location for scheduled scripts that root users can edit.
- /usr/lib/NetworkManager/dispatcher.d/: For pre-deployed immutable scheduled scripts.

For security reasons, the NetworkManager-dispatcher service only executes scripts if the following conditions are met:

- The script is owned by root user.
- The script is only readable and writable by root.
- The setuid bit is not set on the script.

The NetworkManager-dispatcher service runs each script with two arguments:

1. The interface name of the device where the action occurred.
2. The interface action, such as "up" when the interface is activated.

The NetworkManager-dispatcher service runs one script at a time, but asynchronously with respect to the main NetworkManager process. Note that if a script is queued, the service will run it, even if subsequent events make it obsolete. However, when the NetworkManager-dispatcher service runs scripts that are symbolic links to files in /etc/NetworkManager/dispatcher.d/no-wait.d/, it does so without waiting for the termination of previous scripts and runs them in parallel.

## 47.2.Creating a NetworkManager Scheduled Script to Run dhclient Exit Hooks

This section explains how to write a NetworkManager scheduled script that runs the dhclient exit hooks saved in the /etc/dhcp/dhclient-exit-hooks.d/ directory when an IPv4 address is assigned or updated from the DHCP server.

**Prerequisites**

- The dhclient exit hooks are stored in the /etc/dhcp/dhclient-exit-hooks.d/ directory.

**Process**

1. Create the /etc/NetworkManager/dispatcher.d/12-dhclient-down file with the following content:

   ```
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

2. Set root as the owner of the file:

   ```
   # chown root:root /etc/NetworkManager/dispatcher.d/12-dhclient-down
   ```

3. Set the permissions so that only the root user can execute it:

   ```
   # chmod 0700 /etc/NetworkManager/dispatcher.d/12-dhclient-down
   ```

4. Restore the SELinux context:

   ```
   # restorecon /etc/NetworkManager/dispatcher.d/12-dhclient-down
   ```

# Chapter 48: Introduction to NetworkManager Debugging

Raising the log level for all or certain domains can help to capture more details about the operations performed by NetworkManager. Administrators can use this information to troubleshoot issues. NetworkManager provides different levels and domains to generate log information. The /etc/NetworkManager/NetworkManager.conf file is the main configuration file for NetworkManager. Logs are stored in the journal.

This chapter covers information on enabling debug logging for NetworkManager and configuring the amount of logging using different log levels and domains.

## 48.1. Debug Levels and Domains

You can use the "levels" and "domains" parameters to manage the debugging of NetworkManager. "Levels" define the level of detail, while "domains" define the category of messages to be logged at a given severity level ("level").

| Log levels. | Descriptions                                                 |
| :---------- | :----------------------------------------------------------- |
| OFF         | Not logging any information related to NetworkManager.       |
| ERR         | Only logging critical errors.                                |
| WARN        | Logging warning messages that can reflect actions.           |
| INFO        | Logging various information that can help trace the status and operations. |
| DEBUG       | Enabling verbose logging for debugging.                      |
| TRACE       | Enabling logging that is more detailed than the DEBUG level. |
|             |                                                              |

Please note that subsequent levels of logging include all the information from the previous levels. For example, setting the log level to INFO will also log messages that are included in the ERR and WARN log levels.

## 48.2. Setting the NetworkManager log level.

By default, all logging domains are set to record logs at the INFO level. Disable rate limiting before collecting debug logs. With rate limiting, if there is too much information within a short period of time, systemd-journald may discard it. This can happen when the log level is set to TRACE.

The following procedure disables rate limiting and enables logging debug messages for all domains:

**Process**

1. To disable rate limiting, edit the /etc/systemd/journald.conf file, uncomment the **RateLimitBurst** parameter in the [Journal] section, and set its value to 0:

   ```
   RateLimitBurst=0
   ```

2. Restart the systemd-journald service.

   ```
   # systemctl restart systemd-journald
   ```

3. Create the /etc/NetworkManager/conf.d/95-nm-debug.conf file with the following content:

   ```
[logging]
   domains=ALL:TRACE
   ```
   
   The "domains" parameter can contain multiple domain:level pairs separated by commas.

4. Restart the NetworkManager service.

   ```
   # systemctl restart NetworkManager
   ```

**Verification**

- Query the systemd journal to display the log entries for the NetworkManager unit:

  ```
  # journalctl -u NetworkManager
  ...
  Jun 30 15:24:32 server NetworkManager[164187]: <debug> [1656595472.4939] active-connection[0x5565143c80a0]: update activation type from assume to managed
  Jun 30 15:24:32 server NetworkManager[164187]: <trace> [1656595472.4939] device[55b33c3bdb72840c] (enp1s0): sys-iface-state: assume -> managed
  Jun 30 15:24:32 server NetworkManager[164187]: <trace> [1656595472.4939] l3cfg[4281fdf43e356454,ifindex=3]: commit type register (type "update", source "device", existing a369f23014b9ede3) -> a369f23014b9ede3
  Jun 30 15:24:32 server NetworkManager[164187]: <info>  [1656595472.4940] manager: NetworkManager state is now CONNECTED_SITE
  ...
  ```

## 48.3. Using nmcli to temporarily set log level during runtime.

You can use nmcli to change the log level during runtime. However, OpenCloudOS recommends enabling debugging and restarting NetworkManager using a configuration file. Updating the "levels" and "domains" in a .conf file helps with debugging startup issues and captures all logs from the initial state.

**Process**

1. Optional: To display the current logging settings:

   ```
   # nmcli general logging
   LEVEL  DOMAINS
   INFO   PLATFORM,RFKILL,ETHER,WIFI,BT,MB,DHCP4,DHCP6,PPP,WIFI_SCAN,IP4,IP6,A
   UTOIP4,DNS,VPN,SHARING,SUPPLICANT,AGENTS,SETTINGS,SUSPEND,CORE,DEVICE,OLPC,
   WIMAX,INFINIBAND,FIREWALL,ADSL,BOND,VLAN,BRIDGE,DBUS_PROPS,TEAM,CONCHECK,DC
   B,DISPATCH
   ```

2. To modify the log level and domains, use the following options:

   - To set the log level to the same level for all domains, use the following command:

     ```
     # nmcli general logging level LEVEL domains ALL
     ```

   - To change the level for a specific domain, use the following command:

     ```
     # nmcli general logging level LEVEL domains DOMAINS
     ```

     Please note that using this command to update the log level will disable logging for all other domains.

   - To change the level for specific domains while keeping the level for other domains unchanged, use the following command:

     ```
     # nmcli general logging level KEEP domains DOMAIN:LEVEL,DOMAIN:LEVEL
     ```

## 48.4. Viewing NetworkManager logs.

**Process**

- To view logs, use the following command:

  ```
  # journalctl -u NetworkManager -b
  ```