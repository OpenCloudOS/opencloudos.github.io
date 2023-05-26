# Chapter 37: Configuring 802.3 Link Settings

## 37.1. Understanding Auto-Negotiation

Auto-Negotiation is a feature of the IEEE 802.3u Fast Ethernet protocol that targets device ports to provide optimal performance for speed, duplex mode, and flow control in information exchange on the link. By using the Auto-Negotiation protocol, you can achieve the best performance for data transmission on Ethernet.

Note that to maximize the performance of Auto-Negotiation, it is important to use the same configuration at both ends of the link.

## 37.2. Configuring 802.3 Link Settings using the nmcli Tool

To configure 802.3 link settings for an Ethernet connection, modify the following configuration parameters:

- 802-3-ethernet.auto-negotiate
- 802-3-ethernet.speed
- 802-3-ethernet.duplex

**process**

1. Display the current settings of the connection:

   ```
   # nmcli connection show Example-connection
   ...
   802-3-ethernet.speed:  0
   802-3-ethernet.duplex: --
   802-3-ethernet.auto-negotiate: no
   ...
   ```

   If you need to reset the parameters in case of issues, you can use these values.

2. Configure the speed and duplex settings of the link:

   ```
# nmcli connection modify Example-connection 802-3-ethernet.auto-negotiate no 802-3-ethernet.speed 10000 802-3-ethernet.duplex full
   ```
   
   This command will disable auto-negotiation and set the speed of the connection to 10000 Mbit full duplex.

3. Reactivate the connection:

   ```
   # nmcli connection up Example-connection
   ```


**verification**

4. Use the ethtool tool to verify the values of Ethernet interface enp1s0:

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



# Chapter 38: Configure Ethtool Offload Features

Network interface cards can use TCP Offload Engine (TOE) to offload certain operations to the network controller in order to improve network throughput.

## 38.1. Offload features supported by NetworkManager

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

For details on each offload feature, please refer to the ethtool tool documentation and the kernel documentation.

## 38.2. Configure Ethtool Offload Features with NetworkManager

This section describes how to enable and disable Ethtool offload features with NetworkManager, as well as how to remove a setting for a particular feature from NetworkManager connection configuration files.

**process**

1. For example, to enable the RX offload feature and disable the TX offload in the enp1s0 connection configuration file, enter:

   ```
   # nmcli con modify enp1s0 ethtool.feature-rx on ethtool.feature-tx off
   ```

   This command explicitly enables the RX offload and disables the TX offload feature.

2. To remove the setting for an offload feature previously enabled or disabled, set the parameter for the feature to "ignore". For example, to remove the configuration for TX offload, enter:

   ```
   # nmcli con modify enp1s0 ethtool.feature-tx ignore
   ```

3. Reactivate the network configuration set:

   ```
   # nmcli connection up enp1s0
   ```

**Validation Steps**

- Use the command "ethtool -k" to display the current offload features of a network device:

  ```
  # ethtool -k network_device
  ```

## 38.3. Set Ethtool Features with rhel-system-roles

You can use the network role in rhel-system-roles to configure Ethtool features for NetworkManager connections.

When you run a script using Networking RHEL system roles, if the value you set does not match the name specified in the script, the system role will overwrite existing connection configuration files with the same name. Therefore, always specify the entire configuration of the network connection configuration file in the script, even if the IP configuration already exists. Otherwise, the role will reset these values to the default.

**Prerequisites**

- Ansible and rhel-system-roles software packages are installed on the control node.
- If you are running the playbook using a non-root user, ensure that the user has sudo privileges on the managed nodes.

**process**

1. If the host on which you want to execute the instructions in the playbook is not yet listed in the inventory, add the IP or name of this host to the Ansible inventory file, /etc/ansible/hosts:

   ```
   node.example.com
   ```

2. Create the ~/configure-ethernet-device-with-ethtool-features.yml playbook with the following content:

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

3. Run the playbook:

   - To connect to a managed host as the root user, enter:

     ```
     # ansible-playbook -u root ~/configure-ethernet-device-with-ethtool-features.yml
     ```
     

   - To connect to a managed host as a user, enter:

     ```
     # ansible-playbook -u user_name --ask-become-pass ~/configure-ethernet-device-with-ethtool-features.yml
     ```

     The --ask-become-pass option ensures that the ansible-playbook command prompts for the sudo password of the user defined in the -u user_name option.

     If the -u user_name option is not specified, ansible-playbook connects to the managed host as the user currently logged in to the control node.

# Chapter 39: Configure Ethtool Coalesce Settings

With interrupt coalescing, the system collects network packets and generates a single interrupt for multiple packets. This increases the amount of data that a hardware interrupt can send to the kernel, reducing interrupt overhead and maximizing throughput.

This section provides different options for setting Ethtool coalesce settings.

## 39.1. Coalesce settings supported by NetworkManager

You can use NetworkManager to set the following Ethtool coalesce settings: - coalesce-adaptive-rx - coalesce-adaptive-tx - coalesce-pkt-rate-high - coalesce-pkt-rate-low - coalesce-rx-frames - coalesce-rx-frames-high - coalesce-rx-frames-irq - coalesce-rx-frames-low - coalesce-rx-usecs - coalesce-rx-usecs-high - coalesce-rx-usecs-irq - coalesce-rx-usecs-low - coalesce-sample-interval - coalesce-stats-block-usecs - coalesce-tx-frames - coalesce-tx-frames-high - coalesce-tx-frames-irq - coalesce-tx-frames-low - coalesce-tx-usecs - coalesce-tx-usecs-high - coalesce-tx-usecs-irq - coalesce-tx-usecs-low

## 39.2.Configure Ethtool coalesce settings with NetworkManager

This section describes how to configure Ethtool coalesce settings with NetworkManager, and how to remove the settings from NetworkManager connection configuration files.

**process**

1. For example, to set the maximum number of received packets to delay to 128 in the enp1s0 connection configuration file, enter:

   ```
   # nmcli connection modify enp1s0 ethtool.coalesce-rx-frames 128
   ```

2. To remove a coalesce setting, you can set the setting to "ignore". For example, to remove the ethtool.coalesce-rx-frames setting, enter:

   ```
   # nmcli connection modify enp1s0 ethtool.coalesce-rx-frames ignore
   ```

3. Reactivate the network configuration set:

   ```
   # nmcli connection up enp1s0
   ```

**Validation Steps**

- Use the command "ethtool -c" to display the current coalesce settings of a network device:

  ```
  # ethtool -c network_device
  ```

## 39.3. Set Ethtool Coalesce Settings with rhel-system-roles

You can use the network role in rhel-system-roles to configure Ethtool coalesce settings for NetworkManager connections.

When you run a script using Networking RHEL system roles, if the value you set does not match the name specified in the script, the system role will overwrite existing connection configuration files with the same name. Therefore, always specify the entire configuration of the network connection configuration file in the script, even if the IP configuration already exists. Otherwise, the role will reset these values to the default.

**Prerequisites**

- Ansible and rhel-system-roles software packages are installed on the control node.
- If you are running the playbook using a non-root user, ensure that the user has sudo privileges on the managed nodes.

**process**

1. If the host on which you want to execute the instructions in the playbook is not yet listed in the inventory, add the IP or name of this host to the Ansible inventory file, /etc/ansible/hosts:

   ```
   node.example.com
   ```

2. Create the ~/configure-ethernet-device-with-ethtool-features.yml playbook with the following content:

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

3. Run the playbook:

   - To connect to a managed host as the root user, enter:

     ```
     # ansible-playbook -u root ~/configure-ethernet-device-with-ethtoolcoalesce-settings.yml
     ```

   - To connect to a managed host as a user, enter:

     ```
     # ansible-playbook -u user_name --ask-become-pass ~/configure-ethernet-device-with-ethtoolcoalesce-settings.yml
     ```

     The --ask-become-pass option ensures that the ansible-playbook command prompts for the sudo password of the user defined in the -u user_name option.

     If the -u user_name option is not specified, ansible-playbook connects to the managed host as the user currently logged in to the control node.


# Chapter 40: Encrypting Layer 2 Traffic with MACsec on the Same Physical Network

Media Access Control Security (MACsec) is a Layer 2 protocol used to protect different types of traffic on Ethernet links. MACsec can be used to secure communication between two devices (point-to-point). MACsec includes:

- Dynamic Host Configuration Protocol (DHCP)
- Address Resolution Protocol (ARP)
- Internet Protocol version 4/6 (IPv4/IPv6)
- Any traffic that uses IP (such as TCP or UDP).

MACsec uses the GCM-AES-128 algorithm by default to encrypt and authenticate all traffic in the LAN, using a pre-shared key to establish a connection between the participating hosts. If you want to change the pre-shared key, you need to update the NM configuration on all hosts using MACsec in the network.

MACsec connections use Ethernet devices (such as Ethernet NICs, VLANs, or tunnel devices) as parent devices. You can set IP configurations only on the MACsec device to communicate with other hosts using encrypted connections, or you can set IP configurations on the parent device. In the latter case, you can use the parent device to communicate with other hosts using unencrypted connections and the MACsec device to communicate using encrypted connections.

MACsec does not require any special hardware. For example, you can use any switch if you only want to encrypt traffic between hosts and the switch. In this case, the switch must also support MACsec.

There are two common use cases for configuring MACsec:

- Host-to-Host
- Host-to-Switch and then switch-to-other hosts

Note that MACsec can only be used between hosts in the same (physical or virtual) LAN.

## 40.1. Configure MACsec Connection with nmcli

You can use the nmcli tool to configure an Ethernet interface to use MACsec. The following procedure describes how to create a MACsec connection between two hosts connected via Ethernet.

**process**

1. On the first host where you want to configure MACsec:

   - Create a Connection Association Key (CAK) and Connection Association Key Name (CKN) for the pre-shared key:

     - Create a 16-byte hexadecimal CAK:

        ```
        # dd if=/dev/urandom count=16 bs=1 2> /dev/null | hexdump -e '1/2 "%04x"'
        50b71a8ef0bd5751ea76de6d6c98c03a
        ```

     - Create a 32-byte hexadecimal CKN:

       ```
       # dd if=/dev/urandom count=32 bs=1 2> /dev/null | hexdump -e '1/2 "%04x"'
       f2b4297d39da7330910a74abc0449feb45b5c0b9fc23df1430e1898fcf1c4550
       ```

2. On both hosts that you want to connect via MACsec, create the MACsec connection:

   ```
   # nmcli connection add type macsec con-name macsec0 ifname macsec0 connection.autoconnect yes macsec.parent enp1s0 macsec.mode psk macsec.mka-cak 50b71a8ef0bd5751ea76de6d6c98c03a macsec.mka-ckn f2b4297d39da7330910a7abc0449feb45b5c0b9fc23df1430e1898fcf1c4550
   ```

   Use the CAK and CKN generated in the previous step in the macsec.mka-cak and macsec.mka-ckn parameters. These values must be the same on each host in the MACsec-protected network.

3. Configure IP settings in the MACsec connection.

   - To configure IPv4 settings, for example, to set a static IPv4 address, netmask, default gateway, and DNS server for the macsec0 connection, enter:

      ```
      # nmcli connection modify macsec0 ipv4.method manual ipv4.addresses '192.0.2.1/24' ipv4.gateway '192.0.2.254' ipv4.dns '192.0.2.253'
      ```

   - To configure IPv6 settings, for example, to set a static IPv6 address, netmask, default gateway, and DNS server for the macsec0 connection, enter:

     ```
     # nmcli connection modify macsec0 ipv6.method manual ipv6.addresses '2001:db8:1::1/32' ipv6.gateway '2001:db8:1::fffe' ipv6.dns '2001:db8:1::fffd'
     ```

4. Activate the connection:

   ```
   # nmcli connection up macsec0
   ```

**Validation Steps**

1. Verify that the traffic is encrypted:

   ```
   # tcpdump -nn -i enp1s0
   ```

2. Optional: Display unencrypted traffic:

   ```
   # tcpdump -nn -i macsec0
   ```

3. Display MACsec statistics:

   ```
   # ip macsec show
   ```


4. Display separate counters for each type of protection: integrity only (encryption off) and encryption (encryption on)

   ```
   # ip -s macsec show
   ```



# Chapter 41: Using Different DNS Servers in Different Domains

By default, OpenCloudOS sends all DNS requests to the first DNS server specified in the /etc/resolv.conf file. If this server does not respond, OpenCloudOS will use the next server specified in the file.

In an environment where a DNS server fails to resolve all domains, administrators can configure OpenCloudOS to direct DNS requests for specific domains to a chosen DNS server.For instance, you can configure one DNS server to resolve queries for example.com while setting up another DNS server to handle queries for example.net. For all other DNS requests, OpenCloudOS will utilize the DNS server configured in the connection with the default gateway.

## 41.1. Directing DNS requests for specific domains to a selected DNS server.

This section configures the systemd-resolved service and NetworkManager to route DNS queries for specific domains to the selected DNS server.

If you have completed the steps in this section, OpenCloudOS will utilize the DNS service provided by systemd-resolved in the /etc/resolv.conf file. systemd-resolved starts a DNS service that listens on port 53 with the IP address 127.0.0.53. This service dynamically routes DNS requests to the corresponding DNS server specified in NetworkManager. Please replace the IP address with the actual one required in your production environment.

**Prerequisites**

- The system is configured with multiple NetworkManager connections.
- Configure the DNS servers and search domains in the NetworkManager connection responsible for resolving the specific domain.

For example, if the DNS server specified in the VPN connection should be able to resolve queries for the example.com domain, the VPN connection configuration file must include the following:

1. Configure a DNS server that can resolve example.com:
2. Configure the search domain as example.com in the ipv4.dns-search and ipv6.dns-search parameters:

**process**

1. Start and enable the systemd-resolved service:

   ```
   # systemctl --now enable systemd-resolved
   ```

2. Edit the /etc/NetworkManager/NetworkManager.conf file and set the following entries in the [main] section:

   ```
   dns=systemd-resolved
   ```

3. Reload the NetworkManager service:

   ```
   # systemctl reload NetworkManager
   ```

**Validation**

1. To verify if the nameserver entry in the  /etc/resolv.conf  file points to 127.0.0.53, you can use the following command:

   ```
   # cat /etc/resolv.conf
   nameserver 127.0.0.53
   ```

2. To check if the systemd-resolved service is listening on port 53 of the local IP address 127.0.0.53, you can use the following command:

   ```
   # ss -tulpn | grep "127.0.0.53"
   udp  UNCONN 0  0      127.0.0.53%lo:53   0.0.0.0:*    users:(("systemd-resolve",pid=1050,fd=12))
   tcp  LISTEN 0  4096   127.0.0.53%lo:53   0.0.0.0:*    users:(("systemd-resolve",pid=1050,fd=13))
   ```

# Chapter 42: Using IPVLAN

This chapter introduces the IPVLAN driver.

## 42.1. IPVLAN Overview

IPVLAN is a driver for virtual network devices that can be used in container environments to access the host network. IPVLAN exposes a single MAC address to the external network, regardless of the number of IPVLAN devices created in the host network. This means that users can have multiple IPVLAN devices in different containers, while the corresponding switches read only a single MAC address. The IPVLAN driver is useful when there are constraints on the total number of MAC addresses that the local switch can manage.

## 42.2. IPVLAN Modes

IPVLAN has the following available modes:

- **L2 mode (Layer 2 mode)**

  In IPVLAN L2 mode, the virtual device receives and responds to Address Resolution Protocol (ARP) requests. The netfilter framework operates only within the container that owns the virtual device. Netfilter chains are not executed in the default namespace of containerized traffic. Using L2 mode provides good performance but offers limited control over network traffic.

- **L3 mode (Layer 3 mode)**

  In L3 mode, the virtual device only handles traffic at the L3 and higher layers. The virtual device does not respond to ARP requests, and users need to manually configure neighbor entries for the IPVLAN IP addresses on the relevant points. The egress traffic from the associated containers goes through the netfilter POSTROUTING and OUTPUT chains in the default namespace, while the ingress traffic is handled in a threaded manner similar to L2 mode. Using L3 mode provides good control but may potentially impact network traffic performance.

- **L3S mode (Layer 3 mode with MAC Source Check)**

  In L3S mode, the virtual device operates similarly to L3 mode in terms of traffic handling. However, the egress and ingress traffic of associated containers are both processed through netfilter chains in the default namespace. The behavior of L3S mode is similar to L3 mode but provides greater control over the network.

For L3 and L3S modes, IPVLAN virtual devices do not receive broadcast and multicast traffic.

## 42.3. MACVLAN Overview

The MACVLAN driver allows the creation of multiple virtual network devices on a single NIC, with each having its own unique MAC address. Packets on the physical NIC are multiplexed to the corresponding MACVLAN devices based on the destination MAC address. MACVLAN devices do not add any level of encapsulation.

## 42.4. Comparison between IPVLAN and MACVLAN

The following table presents the key distinctions between MACVLAN and IPVLAN:

| MACVLAN                                                      | IPVLAN                                                       |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Use a MAC address for each MACVLAN device. MAC address restrictions in the MAC table in the exchange may cause connection loss. | Use a single MAC address with unlimited IPVLAN devices.      |
| The netfilter rules for the global namespace do not affect network traffic to or from MACVLAN devices in sub-namespaces. | It is possible to control network traffic to and from IPVLAN devices in L3 mode and L3S mode. |

Please note that both IPVLAN and MACVLAN do not require any level of encapsulation.

## 42.5. Creating and Configuring IPVLAN Devices with iproute2

本节介绍了如何使用 iproute2 设置 IPVLAN 设备。

**process**

1. To create an IPVLAN device, please enter the following command:

   ```
   # ip link add link real_NIC_device name IPVLAN_device type ipvlan mode l2
   ```
   
   Please note that a Network Interface Controller (NIC) is a hardware component that connects a computer to a network.

   Example 43.1. Creating an IPVLAN Device

   ```
   # ip link add link enp0s31f6 name my_ipvlan type ipvlan mode l2
   # ip link
   47: my_ipvlan@enp0s31f6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000 link/ether e8:6a:6e:8a:a2:44 brd ff:ff:ff:ff:ff:ff
   ```
   
2. To assign an IPv4 or IPv6 address to an interface, please enter the following command:

   ```
   # ip addr add dev IPVLAN_device IP_address/subnet_mask_prefix
   ```

3. If configuring an IPVLAN device in L3 mode or L3S mode, please perform the following settings:

   - Configure neighbor settings for the remote peer on the remote host.

      ```
   # ip neigh add dev peer_device IPVLAN_device_IP_address lladdr MAC_address
      ```
   
      Where MAC_address is the MAC address of the underlying physical network card on which the IPVLAN device is based.

   - Use the following command to configure an IPVLAN device in L3 mode:

      ```
   # ip route add dev <real_NIC_device> <peer_IP_address/32>
      ```

      For L3S mode:

      ```
      # ip route add dev real_NIC_device peer_IP_address/32
      ```

      Where IP-address represents the address of the remote peer.

4. To set an active IPVLAN device, please enter the following command:

   ```
   # ip link set dev IPVLAN_device up
   ```

5. To check if the IPVLAN device is active, please execute the following command on the remote host:

   ```
   # ping IP_address
   ```
   
   Where IP_address is the IP address of the IPVLAN device being used.