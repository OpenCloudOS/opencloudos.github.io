# Chapter 13 Configure network team

## 13.1 Introduction of network team

The function of network team is merging or aggregating network interfaces. Network team provides an interface that has high throughput or redundance.

Network team use kernel drive to realize data packet flow, user-space library and quick process of other tasks. Therefore, network team is an resolution that is easy to expand. It can be used to meet the needs of balancing loads and redundance.

## 13.2 Configuration of the controller, ports and interfaces

While using NetworkManager service to manage or resolve the problems of team or binded ports/interfaces, please consider the following default configuration:
- Starting controller interfaces will not start ports and interfaces.
- Starting ports or interfaces will always start controller interfaces.
- Stopping controller interfaces will stop ports and interfaces.
- Controllers with no ports can start static IP connection.
- Controllers with no ports will wait for ports when starting DHCP connection.
- When adding ports that is a wave carrier, controllers with DHCP connection will wait for the ports until they finish.
- When adding ports that is not a wave carrier, controllers with DHCP connection will continue to wait for ports.

## 13.3 Comparasion of network team and bonding

Comparasion of network team and bonding:

|Function                                      |Network bonding|Network team|
|----------------------------------------------|---------------|------------|
|Broadcasting Tx strategy                      |yes            |yes         |
|Roundrobin  Tx strategy                          |yes            |yes         |
|Active-backup Tx strategy                     |yes            |yes         |
|LACP(802.3ad)support                          |yes(only activity)|yes         |
|The Tx strategy that based on hash            |yes            |yes         |
|Users can configure hash                      |no             |yes         |
|Support TX load balancing(TLB)                |yes            |yes         |
|LACP hash ports selection                     |yes            |yes         |
|The load balancing that LACP supports         |no             |yes         |
|The monitoring of ethtool link                |yes            |yes         |
|ARP chain monitoring                          |yes            |yes         |
|NS/NA(IPv6)chain monitoring                   |no             |yes         |
|Ports starting/closing latency                |yes            |yes         |
|Ports priority and stickiness("main" option strengthened) |no   |yes         |
|Separate chain monitor setting of each port   |no             |yes         |
|Multiple chain monitoring setting             |limited        |yes         |
|Lockless Tx/Rx path                           |no(rwlock)     |yes(RCU)    |
|VLAN support                                  |yes            |yes         |
|User space runtime control                    |limited        |yes         |
|The logic of user space                       |no             |yes         |
|Extensibility                                    |difficult      |easy        |
|Modularization design                         |no             |yes         |
|Performance cost                              |low            |very low    |
|D-Bus interface                               |no             |yes         |
|multiple devices stack                        |yes            |yes         |
|Zero configuration when using LLDP            |no             |(planing)   |
|Support NetworkManager                        |yes            |yes         |

## 13.4 Take a look at teamd service, running programs and link-watcher

One of the instances of team service teamd team drive program controls. This driver instance add hardware devices driver program instances to constitute a network interface group. Team driver program provide a network interface for the kernel,e.g. team0.

teamd service provide common logic for the realization of all team methods. Those functions are unique for different load shares and back-up methods(e.g. loop), and they are realized by a sole code unit which is called runners. Programmers use JSON format which is JavaScript object definition method to define runners, when creating instances, JSON codes are compiled into teamd instances. Moreover, When using NetworkManager, you can set runner in the parameters of team runners, NetworkManager will create the corresponding JSON codes automatically.

The available runners are as below:

- broadcast: transform the data on all ports.
- roundrobin: transform the data on all ports in turn.
- activebackup: transform the data on one port, the data on other ports are reserved.
- loadbalance: transform all the data with active Tx load balance and the data on the ports of Tx ports selector that based on Berkeley data packet filter(BPF).
- random: transform the data of the ports being selected randomly.
- lacp: realize 802.3ad link Aggregation Control Protocol).

teamd service use link watcher to monitor the state of slave devices. The available link-watchers are as below:

- ethtool: libteam library use ethtool tool to monitor the alteration of the link station. This is the default link-watcher.
- arp_ping: libteam library use arp_ping tool to monitor the existence of the remote hardware address that use address Resolution Protocol(ARP).
- nsna_ping: on the connection of IPv6, libteam library use the neighbor advertisement that comes from IPv6 neighbor detector protocol and the asking for help function of neighbors to detect the existence of neighbor interfaces.

Each runner can use any link watcher, but lacp is an exception, this runner can only use ethtool link watcher.

## 13.5. Install teamd service

To configure network team in NetworkManager, the teamd service and team plug-in units are needed. Those two elements are installed on OpenCloudOS by default.

```
# yum install teamd NetworkManager-team
```

## 13.6. Configure network team using nmcli command

**prerequisite**

- Install two or more than two physical or virtual network devices on the server.
- To use the Ethernet devices as the team ports, it's required to install physical Ethernet devices or virtual Ethernet devices and connect them to the switcher.
- To use bond, bridge or VLAN devices as team ports, you can create these devices when creating teams or create them beforehand.

**process**

1. Create team interface:
   ```
   # nmcli connection add type team con-name team0 ifname team0 team.runner activebackup
   ```
   This command create a network team that use activebackup runner with the name of team0.
2. Configure link watcher. for example, to configure ethtool link watcher in the connection configuration file of team0:
   ```
   # nmcli connection modify team0 team.link-wathers "name=ethtool"
   ```
   The link watcher supports different parameters. To configure parameters for the link watcher, please set them in the attribute of name and separate the parameters with space. Note, the name attribute is needed to be crowded by quotation mark. For example, to use ethtool link watcher and set its delay-up to 2500 ms(2.5s):
   ```
   # nmcli connection modify team0 team.link-watchers "name=ethtool delay-up=2500"
   ```
   To set multiple link watchers and each of them use specific parameters, different link watchers are separated by comma. The following example use delay-up parameter to configure ethtool link watcher, use souce-host and target-host to configure arp_ping link watcher:
   ```
   #nmcli connection modify team0 team.link-watchers "name=ethtool delay-up=2, name=arp_ping source-host=192.0.2.1 target-host=192.0.2.2"
   ```
3. Show network interface and record the interface name that you will add to the team:
   ```
   # nmcli device status
   DEVICE  TYPE      STATE               CONNECTION
   enp7s0  ethernet  disconnected        -- 
   enp8s0  ethernet  disconnected        --
   bond0   bond      connected           bond0
   bond1   bond      connected           bond1
   ...
   ```
   In the above example:
   - enp7s0 and enp8s0 are not configured. To use these devices as ports, please add connection configuration set. Note, You can only use Ethernet interface in the teams that are not allocated to any connection.
   - bond0 and bond1 already have connection configuration files. To use these devices as ports, please modify their configuration set in the next step.
4. Allocate ports and interfaces for teams:
   1. If the interface you will allocate to your team is not configured, create new connection set for it:
        ```
        # nmcli connection add type ethernet slave-type team con-name team0-port1 ifname enp7s0 master team0
        # nmcli connection add type ethernet slave-type team con-name team0-port2 ifname enp8s0 master team0
        ```
        The above commands create configuration files for enp7s0 and enp8s0, and add those configuration files to team0
   2. To allocate the existing connection configuration files to teams, set master parameter of those connections to team0:
        ```
        # nmcli connection modify bond0 master team0
        # nmcli connection modify bond1 master team0
        ```
        The above commands allocate the existing connection configuration file to team0 connection.

5. IP configuration of the configuration team. If you want to use the team as the ports of other devices, please skip this step.
   1. Configure IPv4. For example: To set the static IPv4 address, network mask, default gateway, DNS server and DNS search domain, please input:
      ```
      # nmcli connection modify team0 ipv4.addresses '192.0.2.1/24'
      # nmcli connection modify team0 ipv4.gateway '192.0.2.254
      # nmcli connection modify team0 ipv4.dns '192.0.2.253'
      # nmcli connection modify team0 ipv4.dns-search 'example.com'
      # nmcli connection modify team0 ipv4.method manual
      ```
   2. Configure IPv6. For example: To set the  static IPv6 address, network mask, defualt gateway, DNS server and DNS search domain, please input:
      ```
      # nmcli connection modify team0 ipv6.addresses '2001:db8:1::1/64'
      # nmcli connection modify team0 ipv6.gateway '2001:db8:1::fffe'
      # nmcli connection modify team0 ipv6.dns '2001:db8:1::fffd'
      # nmcli connection modify team0 ipv6.dns-search 'example.com'
      # nmcli connection modify team0 ipv6.method manual
      ```
 6. Activate the connection:
      ```
      # nmcli connection up team0
      ```

**Verification**

- Show team state：
    ```
    # teamdctl team0 state
    setup:
    runner: activebackup
    ports:
    enp7s0
        link watches:
        link summary: up
        instance[link_watch_0]:
            name: ethtool
            link: up
            down count: 0
    enp8s0
        link watches:
        link summary: up
        instance[link_watch_0]:
            name: ethtool
            link: up
            down count: 0
    runner:
    active port: enp7s0
    ```
    In the above example, the two ports are all on line.

## 13.7. Use nm-connection-editor to configure network team

This part will introduce how to use nm-connection-editor application to configure network team.

Please note: nm-connection-editor can only add new ports to the team. To use the existing connection configuration file as the port, please use nmcli tool to create team, as described in Configure network team using nmcli command.

**prerequisite**

- Install two or more than two physical or virtual network devices on the server.
- To use the Ethernet devices as team ports, it's required  to install physical or virtual Ethernet device on the server. 
- To use team, bond or VLAN devices as team ports, please make sure that those devices are not configured yet.

**process**

1. Type nm-connection-editor in the terminal
2. Click + button to add a new connection.
3. Choose Team connection type and then click Create.
4. In the Team option:
   1. Optional: Set team interface name in the field of Interface name
   2. Click Add button to add new connection configuration set for the network interface, and add the configuration set to the team as ports.
      1. Choose the connection type of the interface, e.g., choose Ethernet for the wired connection.
      2. Optional: set connection name for the port
      3. If you will create a connection configuration file for the Ethernet device, please open the Ethernet option, and choose the network interface that you want to add to the team as ports in the Device field. If you have chosen different device types, please configure correspondly. Please note, you can only use Ethernet interface in the team that has not been allocated to any connection.
      4. Click confirmed
   3. Repeat the former steps with each interface that you want to add to the team.
   ![nm-connection-editorconfigurationteam_1](././assets/nm-connection-editor配置team_1.png)
   4. Click Advanced button and set the advanced option to team connection
      1. Choose runner in the Runner option
      2. Set link watcher and its available settings in Link Watcher option
      3. Click confirmed
5.Configure the IP configuration of the team. If you want to use this team as the port of other devices, please skip this step.
   1. In the option of IPv4 Settings, configure IPv4. For example, configure static IPv4 address, network mask, default gateway, DNS server and DNS search domain:
   ![nm-connection-editorconfigurationteam_2](././assets/nm-connection-editor配置team_2.png)
   2. In the option of IPv6 Settings, configure IPv6. For example, configure static IPv6 address, network mask, default gateway, DNS server and DNS search domain:
   ![nm-connection-editorconfigurationteam_3](././assets/nm-connection-editor配置team_3.png)
6. Save tean connection.
7. Close nm-connection-editor.

**Verification**

- Show team state:
    ```
    # teamdctl team0 state
    setup:
    runner: activebackup
    ports:
    enp7s0
        link watches:
        link summary: up
        instance[link_watch_0]:
            name: ethtool
            link: up
            down count: 0
    enp8s0
        link watches:
        link summary: up
        instance[link_watch_0]:
            name: ethtool
            link: up
            down count: 0
    runner:
    active port: enp7s0
    ```


# Chapter 14 Configure network bonding

## 14.1. Introduction of network bonding

Network bonding is a method of constituting or aggregating network interfaces,  a logic interface with high throughput or redundancy can be provided.

active-backup, balance-tlb and balance-alb mode do not need any specific configuration of the network switcher. However, it's needed for other bonding modes to configure the switcher to aggregate links. e.g., as for mode 0, 2 and 3, Cisco switcher needs EtherChannel, but for mode 4, it needs link aggregation control protocol(LACP) and EtherChannel.


## 14.2. Default configuration of controllers and port interfaces

When using NetworkManager service to manage or exclude team or bonded port interfaces errors, please consider the following default configuration:

- Starting controller interfaces will not start port interfaces automatically.
- Starting port interfaces will always start controller interfaces.
- Stop controller interfaces will also stop port interfaces.
- The controllers with no ports can start static IP connection.
- The controllers with no ports will wait for ports when it is starting DHCP connection.
- When you are adding the port that is a wave carrier, the controller that has DHCP connection will wait for the port to finish.
- When you are adding the port that is not a wave carrier, the controller that has DHCP connection will continuously wait for the port to finish.

## 14.3 Comparasion of network team and the bonding function

Comparasion of the network team and the bonding function:

 
|Function                                    |Network bonding|Network team|
|--------------------------------------------|---------------|------------|
|Broadcasting Tx strategy                    |yes            |yes         |
|Roundrobin Tx strategy                      |yes            |yes         |
|Active-backup Tx strategy                   |yes            |yes         |
|LACP（802.3ad）support                      |yes(only activity)|yes      |
|The Tx strategy that based on hash          |yes            |yes         |
|Users can configure hash                    |no             |yes         |
|Support TX load balancing（TLB）            |yes            |yes         |
|LACP hash port selection                    |yes            |yes         |
|The load balancing that supported by LACP   |no             |yes         |
|ethtool link monitoring                     |yes            |yes         |
|ARP link monitoring                         |yes            |yes         |
|NS/NA（IPv6）link monitoring                |no             |yes         |
|Port starting/closing latency               |yes            |yes         |
|Port priority and stickness("main" option strengthened)|no      |yes     |
|The link watcher configuration of each separate port   |no      |yes     |
|Multiple link watchers configuration                   |limited |yes     |
|Lockless Tx/Rx path                         |no(rwlock)     |yes(RCU)    |
|VLAN support                                |yes            |yes         |
|User space runtime control                  |limited        |yes         |
|The logic in user space                     |no             |yes         |
|Expansibility                               |difficult      |easy        |
|Modulization design                         |no             |yes         |
|Performance cost                            |low            |very low    |
|D-Bus interface                             |no             |yes         |
|Multiple device stack                       |yes            |yes         |
|Zero configuration when using LLDP          |no             |(Planning)  |
|NetworkManager support                      |yes            |yes         |

## 14.4. The corresponding configuration between bonding mode and upstream switchers

It is described which configurations are needed for upstream switchers according to the bonding modes.

|bonding mode        |Configuraions on the switcher              |
|--------|--------------|
|0 - balance-rr        |It's needed to start static Etherchannel(LACP negotiation is not started)              |
|1 - active-backup        |Free ports are needed              |
|2 - balance-xor        |It's needed to start static Etherchannel(LACP negotiation is not started)              |
|3 - broadcast        ||It's needed to start static Etherchannel(LACP negotiation is not started)                |
|4 - 802.3ad        |It's needed to start the Etherchannel with negotiated LACP             |
|5 - balance-tlb        |Free ports are needed               |
|6 - balance-alb        |Free ports are needed              |

As for configuring these configurations in the switcher, please refer to the switcher docomentations.

## 14.5. Using nmcli commands to configure network bonding

**prerequisite**

- Install two or more than two physical or virtual network devices on the server.
- To use the Ethernet devices as team ports, it's needed to intall physical or virtual Ethernet devices on the server and connect those devices to the switcher. 
- To use bond, bridge or VLAN devices as team ports, you can create these devices when creating teams or you can create them beforehand.

**process**

1. Create bonding interfaces
     ``` 
     # nmcli connection add type bond con-name bond0 ifname bond0 bond.options "mode=active-backup"
     ```
     This command will create a bond named bond0, which use active-backup mode.
     
     It's needed to set additional medium independent interface(IIM) to monitor interval, please add miimon=interval option to bond.options attribute. For example, if you want to use the same command and set MII to 1000ms(1s), please input:
     ```
    # nmcli connection add type bond con-name bond0 ifname bond0 bond.options "mode=active-backup,miimon=1000"

2. Show the interfaces that been chosen by the network interface to add to the bond
    ```
    # nmcli device status
    DEVICE   TYPE      STATE         CONNECTION
    enp7s0   ethernet  disconnected  --
    enp8s0   ethernet  disconnected  --
    bridge0  bridge    connected     bridge0
    bridge1  bridge    connected     bridge1
    ...
    ```
   
    In the above example:
    - enp7s0 and enp8s0 are not configured. To use these devices as ports, please add connection configuration set in the next step.
    - Both bridge0 and bridge1 have existing connection configuration files. To use these devices as ports, please modify their configuration sets.

3. Allocate interfaces  for bonds
   1. If the interface you want to allocate to the bond has not been configured, create new connection configuration set for the interface:
 
        ```
        # nmcli connection add type ethernet slave-type bond con-name bond0-port1 ifname enp7s0 master bond0
        # nmcli connection add type ethernet slave-type bond con-name bond0-port2 ifname enp8s0 master bond0
        ```
        The above two commands create configuration files for enp7s0 and enp8s0, and add them to the bond0 connection.
   2. If you want to allocate the configuration file of the existiong connection to the bond, please set the master parameter of these connections to bond0:

        ```
        # nmcli connection modify bridge0 master bond0
        # nmcli connection modify bridge1 master bond0
        ```
        The above two commands allocate the existing configuration files of bridge0 and bridge1 to connection bond0.

4. Configure the IP configuration of the bond. If the bond is to be used as the interface of other devices, please skip this step.
   1. Configure IPv4 configuration. For example, to set static IPv4 address, network mask, default gateway, DNS server and DNS search domain of bond0, please input:
        ```
        # nmcli connection modify bond0 ipv4.addresses '192.0.2.1/24'
        # nmcli connection modify bond0 ipv4.gateway '192.0.2.254'
        # nmcli connection modify bond0 ipv4.dns '192.0.2.253'
        # nmcli connection modify bond0 ipv4.dns-search 'example.com'
        # nmcli connection modify bond0 ipv4.method manual
        ```
   2. Configure IPv6 configuration. For example, To set static IPv6 address, network mask, default gateway, DNS server and DNS search domain of bond0, please input:
        ```
        # nmcli connection modify bond0 ipv6.addresses '2001:db8:1::1/64'
        # nmcli connection modify bond0 ipv6.gateway '2001:db8:1::fffe'
        # nmcli connection modify bond0 ipv6.dns '2001:db8:1::fffd'
        # nmcli connection modify bond0 ipv6.dns-search 'example.com'
        # nmcli connection modify bond0 ipv6.method manual
        ```

5. Activate the connection:
    ```
    # nmcli connection up bond0
    ```

6. Verify whether the port is connected and the CONNECTION column shows the connection name:
    ```
    # nmcli device
    DEVICE   TYPE      STATE      CONNECTION
    ...
    enp7s0   ethernet  connected  bond0-port1
    enp8s0   ethernet  connected  bond0-port2
    ```
    Once you activate any port of the connection, NetworkManager will activate the bond, however, other ports will not be activated. You can configure OpenCloudOS to start all ports when starting the bond.:
    1. Enable connection.autoconnect-slaves parameter of connection bonding:
       ```
       # nmcli connection modify bond0 connection.autoconnect-slaves 1
       ```
    2. re-activate the bridge connection:
       ```
       # nmcli connection up bond0
       ```

**Verification**

1. Unplug the network cable temporarily from the host.

Note that it's impossible to use software tools to test link failure event. The tool used to stop connection(nmcli) only shows bonded driver program which can process ports configuration alteration, it cannot show the real link failure event.

2. Show bonding state
   ```
   # cat /proc/net/bonding/bond0
   ```

## 14.6. Use nm-connection-editor to configure network bonding

It is introduced how to use nm-connection-editor application to configure network bonding.

Please note: nm-connection-editor can only add new ports to the bonding. To use the existing connection configuration files to be the ports, please use nmcli tool to create bonding.

**Prerequisite**

- Install two or more than two physical or virtual network devices on the server.
- To use the Ethernet devices as the bonding interfaces, it's required to install physical or virtual Ethernet devices on the server.
- To use team, bond or VLAN devices as the bonding interfaces, please make sure that these devices have not been configured.

**Process**

1. Type in nm-connection-editor in the terminal.
2. Click + button to add a new connection.
3. Choose Bond connection type and click Create.
4. In the option Bond:
   1. Optional: Set bonding interface name in Interface name field.
   2. Click Add button, add the network interface to the bonding as a port.
      1. Choose the interface connection type. e.g., choose Ethernet for wired connection.
      2. Optional: Set connection name for the port.
      3. If you will create connection configuration files for Ethernet devices, please open the Ethernet option, choose the network interface that you will add to the bond as a port. If you choose different device types, please make corresponding configurations. Note, you can only use Ethernet interface in the bonds that has not been configured.
      4. Click Save.
   3. Repeat the former steps on the interfaces that you will add to the bond:
   ![nm-connection-editorconfigurationbond_1](././assets/nm-connection-editor配置bond_1.png)
   4. Optional: configure the other options, e.g.,medium independent interface（MII) monitor intervals.
5. Configure the bonded IP address. If this bond is to be used as the port of other devices, please skip this step.
   1. Configure IPv4 in IPv4 settings option. For example, set static IPv4 address, network mask, default gateway, DNS server and DNS search domain:
   ![nm-connection-editorconfigurationbond_2](././assets/nm-connection-editor配置bond_2.png)
   2. Configure IPv6 in IPv6 settings option. For example, set static IPv6 address, network mask, default gateway, DNS server and DNS search domain:
   ![nm-connection-editorconfigurationbond_3](././assets/nm-connection-editor配置bond_3.png)
6. Click Save to save bonding connection
7. Close nm-connection-editor.

**Verification**
1. Unplug the network cable from the host.

Note, it's impossible to use software tools to test link failure events. stop using connection tool(e.g. nmcli), it only shows bonded driver program can process ports configuration alteration, but it does not show the actual link failure event.

2. Show the bonding state
   ```
   # cat /proc/net/bonding/bond0
   ```

## 14.7. Use nmstatectl to configure network bonding

**Prerequisite**

- Install two or more than two physical or virtual network devices on the server.
- To use the Ethernet devices as the ports in bondings, it's required to install physical or virtual Ethernet devices on the server.
- To use team, network bridge or WLAN devices as ports in the bondings, please set the port name in the port list and define the corresponding interfaces.
- nmstate software is already installed.

**Process**

1. Create YAML file ~/create-bond.yml:

    ```
    ---
    interfaces:
    - name: bond0
    type: bond
    state: up
    ipv4:
        enabled: true
        address:
        - ip: 192.0.2.1
        prefix-length: 24
        dhcp: false
    ipv6:
        enabled: true
        address:
        - ip: 2001:db8:1::1
        prefix-length: 64
        autoconf: false
        dhcp: false
    link-aggregation:
        mode: active-backup
        port:
        - enp1s0
        - enp7s0
    - name: enp1s0
    type: ethernet
    state: up
    - name: enp7s0
    type: ethernet
    state: up

    routes:
    config:
    - destination: 0.0.0.0/0
        next-hop-address: 192.0.2.254
        next-hop-interface: bond0
    - destination: ::/0
        next-hop-address: 2001:db8:1::fffe
        next-hop-interface: bond0

    dns-resolver:
    config:
        search:
        - example.com
        server:
        - 192.0.2.200
        - 2001:db8:1::ffbb
    ```
2. Application configurations
    ```
    # nmstatectl apply /create-bond.yml
    ```

**Verification**
1. Show the state of devices and connections:
    ```
    # nmcli device status
    DEVICE      TYPE      STATE      CONNECTION
    bond0       bond      connected  bond0
    ```
2. Show the whole configurations of the connection configuration set:
    ```
    # nmcli connection show bond0
    connection.id:              bond0
    connection.uuid:            79cbc3bd-302e-4b1f-ad89-f12533b818ee
    connection.stable-id:       --
    connection.type:            bond
    connection.interface-name:  bond0
    ...
    ```
3. Show the connection configurations using YAML format:
    ```
    # nmstatectl show bond0
    ```

## 14.8. Using rhel-system-roles to configure network bonding

**Prerequisite**

- ansible and rhel-system-roles software packages are already installed on the control node.
- If you use users other than root when running playbook, it's required for the non-root user to have corresponding sudo permission on the node being managed.
- Install two or more than two physical or virtual network devices on the server.

**Process**

1. If the host you use to execute playbook has not been listed in the detailed list, you need to add the host IP or nameto Ansible detailed list file /etc/ansible/hosts.
    ```
    node.example.com
    ```
2. Create ~/bond-ethernet.yml playbook:

    ```
    ---
    - name: Configure a network bond that uses two Ethernet ports
    hosts: node.example.com
    become: true
    tasks:
    - include_role:
        name: rhel-system-roles.network

        vars:
        network_connections:
            # Define the bond profile
            - name: bond0
            type: bond
            interface_name: bond0
            ip:
                address:
                - "192.0.2.1/24"
                - "2001:db8:1::1/64"
                gateway4: 192.0.2.254
                gateway6: 2001:db8:1::fffe
                dns:
                - 192.0.2.200
                - 2001:db8:1::ffbb
                dns_search:
                - example.com
            bond:
                mode: active-backup
            state: up

            # Add an Ethernet profile to the bond
            - name: bond0-port1
            interface_name: enp7s0
            type: ethernet
            controller: bond0
            state: up

            # Add a second Ethernet profile to the bond
            - name: bond0-port2
            interface_name: enp8s0
            type: ethernet
            controller: bond0
            state: up
    ```
3. run playbook:
     - Connect to the host that been managed using root user identity, please input:
        ```
        # ansible-playbook -u root ~/bond-ethernet.yml
        ```
     - Connect to the host that been managed using users identity, please input:
        ```
        # ansible-playbook -u user_name --ask-become-pass ~/bond-ethernet.yml
        ```
        --ask-become-pass option ensures the sudo password of the user defined in -u user_name which is prompted by command ansible-playbook.
     If -u user_name is not defined, ansible-playbook will use the existing user identity which logins to the controlled node to connect to the node being managed.

## 14.9. Create network bonding so that we can switch between the Ethernet and the wireless connection without breaking the VPN

The users who need to use the internal company network to work always use VPN to access remote resources. However, if the host switch between the Ethernet and the Wi-Fi, e.g.,the VPN will interrupt if you release your notebook computer from the dock station that has the Ethernet. To avoid this situation, you can create network bonding that uses Ethernet and Wi-Fi connection in active-backup mode.

**Prerequisite**

- The host has Ethernet devices and Wi-Fi devices.
- The network manager connection configuration set of the Ethernet and the Wi-Fi is already created and those two connections are able to work independently.
- The process use the following connection configuration files to create the network bonding with the name of bond0:
  - The Docking_station that connectes with enp11s0u1 Ethernet device
  - Wi-Fi connects with wlp1s0 Wi-Fi devices

**Prerequisite**

1. Create a bonding interface in active-backup mode:
    ```
    # nmcli connection add type bond con-name bond0 ifname bond0 bond.options "mode=active-backup"
    ```
    The above command names the interface and the connection configuration files bond0
2. Configure IPv4 settings of the bonding:
    - If the DHCP server allocates IPv4 address for the host in your network, you do not need to to perform any operation.
    - If your local network needs static IPv4 address, please set the address, network mask, defualt gateway, DNS server and DNS search domain as bond0 connection:
        ```
        # nmcli connection modify bond0 ipv4.addresses '192.0.2.1/24'
        # nmcli connection modify bond0 ipv4.gateway '192.0.2.254'
        # nmcli connection modify bond0 ipv4.dns '192.0.2.253'
        # nmcli connection modify bond0 ipv4.dns-search 'example.com'
        # nmcli connection modify bond0 ipv4.method manual
        ```
3. Configure IPv6 settings of the bonding:
    - If the router or the DHCP server allocates the IPv6 address for the host in your network, you do not need to perform any operation.
    - If your local network needs static IPv6 address, please set the address, network mask, default gateway, DNS server and DNS search domain as bond0 connection:
        ```
        # nmcli connection modify bond0 ipv6.addresses '2001:db8:1::1/64'
        # nmcli connection modify bond0 ipv6.gateway '2001:db8:1::fffe'
        # nmcli connection modify bond0 ipv6.dns '2001:db8:1::fffd'
        # nmcli connection modify bond0 ipv6.dns-search 'example.com'
        # nmcli connection modify bond0 ipv6.method manual
        ```
4. Show the connection configuration set:
    ```
    # nmcli connection show
    NAME             UUID                                  TYPE      DEVICE
    Docking_station  256dd073-fecc-339d-91ae-9834a00407f9  ethernet  enp11s0u1
    Wi-Fi            1f1531c7-8737-4c60-91af-2d21164417e8  wifi      wlp1s0
    ...
    ```
    It's needed to connect the connection configuration set name and the Ethernet device name.
5. Allocate the Ethernet connection configuration for the bonding:
    ```
    # nmcli connection modify Docking_station master bond0
    ```
6. Allocate connection configuration set of the Wi-Fi connection for the bonding:
    ```
    # nmcli connection modify Wi-Fi master bond0
    ```
7. If your Wi-Fi network use MAC filtering to give permission only to those MAC addresses that are in the permission list, please configure the NetworkManager to allocate MAC addresses to the active ports for the bonding.
    ```
    # nmcli connection modify bond0 +bond.options fail_over_mac=1
    ```
8. Set the device that connected with the Ethernet connection as the primary device:
    ```
    # nmcli con modify bond0 +bond.options "primary=enp11s0u1"
    ```
9. Perform the configuration that the NetworkManager will activate the port automatically once the bond0 device is activated:
    ```
    # nmcli connection modify bond0 connection.autoconnect-slaves 1
    ```
10. Activate bond0 connection:
    ```
    # nmcli connection up bond0
    ```

**Verification**

- Show the current active devices, bonding and the state of its port:
    ```
    # cat /proc/net/bonding/bond0
    Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

    Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
    Primary Slave: enp11s0u1 (primary_reselect always)
    Currently Active Slave: enp11s0u1
    MII Status: up
    MII Polling Interval (ms): 1
    Up Delay (ms): 0
    Down Delay (ms): 0
    Peer Notification Delay (ms): 0

    Slave Interface: enp11s0u1
    MII Status: up
    Speed: 1000 Mbps
    Duplex: full
    Link Failure Count: 0
    Permanent HW addr: 00:53:00:59:da:b7
    Slave queue ID: 0

    Slave Interface: wlp1s0
    MII Status: up
    Speed: Unknown
    Duplex: Unknown
    Link Failure Count: 2
    Permanent HW addr: 00:53:00:b3:22:ba
    Slave queue ID: 0
    ```

# Chapter15 Configure the VPN connection

This chapter introduces how to configure Virtual Private Network(VPN) connection.

VPN is an approach of connecting to the local network through the internet. Libreswan is the method being used to create VPN in this chapter. Libreswan is realized through the VPN userspace IPsec. VPN realizes the communication between one LAN and the other remote LAN through setting tunnel in the intermediate network(e.g. internet). For the sake of safety, VPN tunnel always use authentication and encryption. As for the encryption operation, Libreswan uses NSS library.

## 15.1 Using control-center to configure the VPN connection

**Prerequisite**

- The software NetworkManager-libreswan-gnome is already installed.

**Process**

1. enter Settings.
2. Choose the Network on the left side.
3. Click the + icon on the right side of VPN.
4. Choose VPN.
5. Choose Identity menu to check the basic configuration options:
    - General : Gateway - remote VPN gateway name or IP address.
    - Authentication: type
      - IKEv2(Certificate)- The identity authentication work is finished through certifications on the server. It's more safer(default).
      - IKEv1(XAUTH) - The identity authentication work is finished through username and password or the preshared secret key(PSK).
    ![control-centerconfigurationVPN_1](./assets/control-center配置VPN_1.png)
    The following configurations are provided in the Advanced part:
    - Identification
      - domain input domain name if needed.
    - Security
      - phase1 Algorithms corresponds to ike Libreswan parameter, type in the algorithm that used to authenticate identities and set encrypted channels.
      - phase2 Algorithms corresponds to esp Libreswan parameter, type in the algorithm used for IPsec negotiation.
      Check the Disable PFS field to close Perfect Forward Secrecy(PFS) and make sure that it's compatible with the old server which does not support PFS.
      - Phase1 Lifetime corresponds to ikelifetime libreswan parameter, it's used to encrypt the lifespan of the secret key of the traffic.
      - phase2 Liftime corresponds to salifetime Libreswan parameter, the duration of the specific connection instance before it expires.
        Note: For the sake of safety, the encryption secret key should be updated now and then.
    - Connectivity
      - Remote Network corresponds to rightsubnet Libreswan parameter, the remote private network that should be accessed through VPN.
      - Enable fragmentation corresponds to fragmentation Libreswan parameter, whether it allows IKE fragments. Valid values include yes(default) and no.
      - Enable Mobike corresponds to mobike Libreswan parameter, whether it allows that the mobility and multi-functional protocol(MOBIKE, RFC 4555) start connection to move its end points and there is no need to restart the connection from the scratch. It can be used in the mobile devices which switches among wired network, wireless network and mobile data connections. Its values include no(default) and yes.                       
    ![control-centerconfigurationVPN_2](./assets/control-center配置VPN_2.png)
6. Choose the items of IPv4 menu:
    - IPv4 Method
     - Automatic(DHCP) - If the network you want to connect to uses the DHCP to allocate dynamic IP address, Please choose this option.
     - Only link - If the netowrk you want to connect to does not have the DHCP server and you do not want to allocate the IP address manually, choose this option. Random address will allocate prefix 169.254/16 according to the RFC 3927.
     - Manual - If you allocate IP addresses manually, please choose this option.
     - disable - The connection bans using IPv4.
    - DNS:

       When the Automatic is ON, change it to OFF to input the IP address of the DNS server, you will separate the IP address by comma.
    - Routes

       Please note, in Routes part, when the Automatic is ON, the routes come from the DHCP will be used, you can also add the other static routes. When the Automatic if OFF, only static routes are used.
     - Address Type in the IP address of the remote network or the host
     - Netmask The length of the subnet mask or prefix of the IP address being typed in in the preceding step.
     - Gateway The length of the subnet mask or prefix of the IP address being typed in in the preceding step.
     - Metric network cost, the primary value that been granted to the route. The lower the value, the higher the priority.
    - The connection is only used for the resources on its network.
       
       Select the checkbox in case that the connection becomes the default route. Selecting this option means that only the traffic that being used for routing specicially can be gotten automatically through the connection, or you can input manually to be on the connection.



7. To configure IPv6 settings in the VPN connection, please choose the items of IPv6 menu:  
    - IPv6 Method
      - Automatic - Choose this option to use the IPv6 Stateless address auto configuration(SLAAC), create automatic stateless configuration based on the hardware address and the router advertisement(RA).
      - Automatic, only DHCP - Choose this option to avoid using RA, and apply for information directly from DNCPv6 to create configurations that have states.
      - Only link - If the network you want to connect to do not has the DHCP server and you do not want to allocate the IP address manually, choose this option. The random addresses will be allocated through the RFC 4862, the prefix is FE80::0.
      - Manual - If it's needed to allocate IP addresses manually, choose this option.
      - disable - This connection bans using IPv6.

    Please note,  it's very common that DNS, Route s only uses the connection on the resources of its network.

8. After finishing editing the VPN connection, click Add button to customize configuration, or use Apply buttion to save it for the current configuration.

9. Configure the configuration set to ON to activate the VPN.

## 15.2. Use nm-connection-editor to configure the VPN connection

**Prerequisite**

- The NetworkManager-libreswan-gnome software package is already installed.
- If you have configured the Internet Key Exchange（IKEv2) verification:
  - Import the certificate to the database of the IPsec network safety service(NSS).
  - The name of the certificate in the NSS database is known.

**Process**

1. Type in in the terminal: nm-connection-editor
2. Click + button to add a new connection.
3. Choose the IPsec based VPN connection type and click Create.
4. In the VPN options:
   1. Input the hostname or IP address of the VPN gateway in the Gateway field and then choose the verification type, you are required to input different additional information according to the verification type.
     - IKEv2(Certificate)- It's safter that the client verifies the identity through certificates. This setting needs to specify the certificate name in the IPsec NSS database.
     - IKEv1(CAUTH) - The client performs identity verification through the username or the pre-shared key. You need to input the following information in this setting:
       - username
       - password
       - group name
       - secret key
   2. If the remote server already specifies the local identifier for the IKE exchange, input accurate characters in the Remote ID field. As for the remote server that runs Libreswan, this value is set in the leftid parameter of the remote server.                         
   ![nm-connection-editorconfigurationVPN_1](././assets/nm-connection-editor配置VPN_1.png)
   3. (optional) click Advanced buttion to configure additional settings. You can configure the following configurations:
      - Identification
        - domain Input the domain name if needed.
      - Security
        - phase1 Algorithms corresponds to ike Libreswan parameter, input the algorithm of Identity Verification and setting encryption channel.
        - phase2 Algorithms corresponds to esp Libreswan parameter, input the algorithm of IPsec negotiating.
        Check Disable PFS field to close Perfect Forward Secrecy(PFS) and make sure that it's compatible with the old server which does not support PFS.
        - Phase1 Lifetime corresponds to ikelifetime Libreswan parameter, It's used to encrypt the lifespan of the traffic key.
        - Phase2 Lifetime corresponds to salifetime Libreswan parameter, the duration of the specific instance before it expires.
          Note: For the sake of safety, the encryption key should be modified now and then.
      - Connectivity
        - Remote Network corresponds to rightsubnet Libreswan parameter, The object private remote network that should be accessed through the VPN.
        - Enable fragmentation corresponds to fragmentation Libreswan parameter, Whether the IKE fragments are allowed, valid values include yes(default) or no.
        - Enable Mobike corresponds to mobik Libreswan parameter, Whether it allows the mobility and multi-functional protocol(MOBIKE, RFC 4555) to start the connection so that the end points can be moved and there is no need to start the connection from the scratch. It can be used on the mobile devices that switch among wired network, wireless network and mobile data connection. The value includes no(default) and yes.

5. In the options of IPv4 Settings, select the IP allocation method, and you can also choose to set the static address, DNS server, search domain and route. 
![nm-connection-editorconfigurationVPN_2](./assets/nm-connection-editor配置VPN_2.png)
6. Save the configurations.
7. Close nm-connection-editor.

## 15.3. Configure automatic detection and use ESP hardware unload to accelerate the connection

Unload the hardware Encapsulation Safety Payload(ESP) to accelerate the Ethernet IPsec connection. By default, Libreswan will detect whether the harware supports the given function and start ESP hardware unload by default. This part describes how to start auto-detection under the condition of banning using this function or using this function explicitly.

**Prerequisite**

- The network card supports ESP hardware unload.
- The network dirver program supports ESP hardware unload.
- IPsec connection is configured and works normally.

**Process**

1. Edit the Libreswan configuration file of VPN connection in directory of /etc/ipsec.d/, this file should use the auto-dection that the ESP hardware supports.
2. Make sure that the nic-offload parameter is not set in the connection configuration.
3. If you have deleted nic-offload, please restart ipsec service:
    ```
    # systemctl restart ipsec
    ```

**Verification**

If the network card supports the ESP hardware unload support, please verify the result following the steps below:

1. Show the Ethernet devices counter tx_ipsec and rx_ipsec that IPsec connection uses:
    ```
    # ethtool -S enp1s0 | egrep "_ipsec"
        tx_ipsec: 10
        rx_ipsec: 10
    ```
2. Send the traffic through the IPsec tunnel. e.g., ping remote IP address:
    ```
    # ping -c 5 remote_ip_address
    ```
3. Show the Ethernet device counters tx_ipsec and rx_ipsec again:
    ```
    # ethtool -S enp1s0 | egrep "_ipsec"
        tx_ipsec: 15
        rx_ipsec: 15
    ```
    The increase of the counter value indicates that the ESP hardware unload works normally.

## 15.4. Configure the ESP hardware unload in the bonding to accelerate the IPsec connection

Unloading the Encapsulation Safety Payload(ESP) from the hardware can accelerate the IPsec connection. If you use network bonding because of errors transfer, The requirements and process of configuring ESP hardware unloading are different from that of using normal Ethernet devices. e.g., In this case, you can start unloading support on the bonding, the kernel will apply the settings to the bonded ports.

**Prerequisite**

- All the network card in the bonding supports ESP hardware unloading.
- The network driver program supports ESP hardware unloading of the bonded devices. In RHEL, only ixgbe driver program supports this function.
- The bonding is already configured and can work normally.
- This bonding uses active-backup mode. The bonding driver program does not support any other mode of this function.
- IPsec connection is configured and can work normally.

**Process**

1. Start ESP hardware unload support on the network bonding:
    ```
    # nmcli connection modify bond0 ethtool.feature-esp-hw-offload on
    ```
2. re-activate bond0 connection:
    ```
    # nmcli connection up bond0
    ```
3. Edit the Libreswan configuration file of the connection that should use ESP hardware unloading, the configuration file is in /etc/ipsec.d/ directory. Additionally, add nic-offload=yes to the connection items:
    ```
    conn example
    ...
    nic-offload=yes
    ```
4. Restart the ipsec service:
    ```
    # systemctl restart ipsec
    ```

**Verification**

1. Show the bonded active port:
    ```
    # grep "Currently Active Slave" /proc/net/bonding/bond0
    Currently Active Slave: enp1s0
    ```
2. Show the Ethernet device counter tx_ipsec and rx_ipsec that the IPsec connection uses:
    ```
    # ethtool -S enp1s0 | egrep "_ipsec"
        tx_ipsec: 10
        rx_ipsec: 10
    ```
3. Send the traffic through the IPsec tunnel. e.g., ping the remote IP address:
    ```
    # ping -c 5 remote_ip_address
    ```
4. Show the Ethernet devices counter tx_ipsec and rx_ipsec again:
    ```
    # ethtool -S enp1s0 | egrep "_ipsec"
        tx_ipsec: 15
        rx_ipsec: 15
    ```
    The increase of the counter value indicates that the ESP hardware unloading works normally.

# Chapter16 Configuring the IP tunnel

It's similar with the VPN, IP tunnel connects the two subnets directly through the thrird-party network(e.g. internet). However, not all the tunnel protocols support encryption.

It requires at least two interfaces to create the route of the tunnel network:

- One interface to connect to the local network.
- One interface to connect to the network that creates the tunnel.

To create the tunnel, you can use the IP addresses that comes from the remote subnet to create a virtual interface in the two routers.

NetworkManager supports the following IP tunnels:

- General Route Encapsulation(GRE)
- IPv6 General Route Encapsulation(IP6GRE)
- General Route Encapsulation Terminal Access Point(GRETAP)
- General Route Encapsulation Terminal Access Point on IPv6(IP6GRETAP) 
- IPv4 over IPv4（IPIP）
- IPv4 over IPv6（IPIP6）
- IPv6 over IPv6（IP6IP6）
- Simple Internet Transformation(SIT)

According to the types, these tunnels work on the second and third level of the Open Systems Interconnection(OSI) model.

## 16.1. Using nmcli to configure the IPIP tunnel
 
IP over IP(IPIP) tunnel works on the third level of the OSI model, The data sent through the IPIP tunnel is not encrypted. For the sake of safety, you should use tunnels in the encrypted data, e.g., HTTPS.

Please note, IPIP tunnel only supports unicast packet. If you need the IPv4 tunnel that supports multicast, please refer to  Encapsulate the third level traffic of the IPv4 packet through using nmcli to configure the GRE tunnel.

The following picture shows how to create the IPIP tunnel between two routers so that the two internal subnets can be connected through the internet:
![IPIP-tunnel](././assets/IPIP-tunnel.png)

**Prerequisite**

- Each router has a network interface, which connects to its local subnet.
- Each router has a network interface, which connects to the internet.
- The traffic that you need to send through the tunnel is IPv4 unicast.

**Process**

1. On the router of network A:
   1. Create the IPIP tunnel interface named tun0:
        ```
        # nmcli connection add type ip-tunnel ip-tunnel.mode ipip con-name tun0 ifname tun0 remote 198.51.100.5 local 203.0.113.10
        ```
        remote and local set the public IP address of the remote router and that of the local router respectively.
   2. Set the IPv4 address as tun0 device:
        ```
        # nmcli connection modify tun0 ipv4.addresses '10.0.1.1/30'
        ```
        Please note that the /30 subnet that has two useful IP addresses meets the needs of the tunnel is enough to meet the needs of the tunnel.
   3. Configure tun0 connection to use manual IPv4 configuration: 
        ```
        # nmcli connection modify tun0 ipv4.method manual
        ```
   4. Add a static route, it routes the traffic which arrives at 172.16.0.0/24 network to the tunnel IP of router B:
        ```
        # nmcli connection modify tun0 +ipv4.routes "172.16.0.0/24 10.0.1.2"
        ```
   5. Start tun0 connection:
        ```
        # nmcli connection up tun0
        ```
   6. Start packet transfer:
        ```
        # echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/95-IPv4-forwarding.conf
        # sysctl -p /etc/sysctl.d/95-IPv4-forwarding.conf
        ```
2. On the router of network B:
   1. Create the IPIP tunnel interface named tun0:
        ```
        # nmcli connection add type ip-tunnel ip-tunnel.mode ipip con-name tun0 ifname tun0 remote 203.0.113.10 local 198.51.100.5
        ```
        remote and local sets the IP address of the remote router and the local router respectively.
   2. Set the IPv4 address as tun0 device:
        ```
        # nmcli connection modify tun0 ipv4.addresses '10.0.1.2/30'
        ```
   3. Configure tun0 connection to use manual IPv4 configuration:
        ```
        # nmcli connection modify tun0 ipv4.method manual
        ```
   4. Add a static route, it routes the traffic that should arrive at 192.0.2.0/24 network to the tunnel IP of router A:
        ```
        # nmcli connection modify tun0 +ipv4.routes "192.0.2.0/24 10.0.1.1"
        ```
   5. Start tun0 connection:
        ```
        # nmcli connection up tun0
        ```
   6. Start packet transfer:
        ```
        # echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/95-IPv4-forwarding.conf
        # sysctl -p /etc/sysctl.d/95-IPv4-forwarding.conf
        ```

**Verification**

- On each router, ping the router internal interface IP address:
  - On router A, ping 172.16.0.1:
        ```
        # ping 172.16.0.1
        ```
  - On router B, ping 192.0.2.1:
        ```
        # ping 192.0.2.1
        ```

## 16.2. Encapsulate the third layer traffic of the IPv4 packet through using nmcli to configure the GRE tunnel 

As RFC 2784 describes, Generic Routing Encapsulation(GRE) tunnel encapsulates the third layer traffic of the IPv4 packet. GRE tunnel can encapsulate any third layer protocol using effective Ethernet type. The data sent by GRE tunnel is not encrypted. For the sake of safety, tunnels should only be used in the encrypted data, e.g., HTTPS.

The following picture shows how to create GRE tunnel between two routers and connects the two internal subnets through the internet:
![GRE-tunnel](./assets/GRE-tunnel.png)
Note, gre0 is a reserved word, please use gre1 as the tunnel name.

**Prerequisite**

- Each route has a network interface, it connects to its local subnet.
- Each route has a network interface, it connects to the internet.

**Process**

1. On the route of network A:
   1. Create the GRE tunnel interface named gre1:
        ```
        # nmcli connection add type ip-tunnel ip-tunnel.mode gre con-name gre1 ifname gre1 remote 198.51.100.5 local 203.0.113.10
        ```
        remote and local set the public IP address of the remote router and that of the local router.
   2. Set the IPv4 address as gre1 device:
        ```
        # nmcli connection modify gre1 ipv4.addresses '10.0.1.1/30'
        ```
        Please note, The /30 subnet which has two useful IP addresses is enough to meet the needs of the tunnel.
   3. Set the gre1 connection configuration as using manual IPv4 configuration:
        ```
        # nmcli connection modify gre1 ipv4.method manual
        ```
   4. Add a static route, it routes the traffic which arrives at 172.16.0.0/24 network to the tunnel IP of router B.
        ```
        # nmcli connection modify gre1 +ipv4.routes "172.16.0.0/24 10.0.1.2"
        ```
   5. Start gre1 connection.
        ```
        # nmcli connection up gre1
        ```
   6. Start packet transfer:
        ```
        # echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/95-IPv4-forwarding.conf
        # sysctl -p /etc/sysctl.d/95-IPv4-forwarding.conf
        ```
2. On the router of network B:
   1. Create the IPIP tunnel interface named gre1:
        ```
        # nmcli connection add type ip-tunnel ip-tunnel.mode ipip con-name gre1 ifname gre1 remote 203.0.113.10 local 198.51.100.5
        ```
   2. Set the IPv4 address as the gre1 device:
        ```
        # nmcli connection modify gre1 ipv4.addresses '10.0.1.2/30'
        ```
   3. Configure the gre1 connection as using manual IPv4 configuration:
        ```
        # nmcli connection modify gre1 ipv4.method manual
        ```
   4. Add a static route, it routes the traffic which should arrive at 192.0.2.0/24 network to the tunnel IP of router A:
        ```
        # nmcli connection modify gre1 +ipv4.routes "192.0.2.0/24 10.0.1.1"
        ```
   5. Start gre1 connection.
        ```
        # nmcli connection up gre1
        ```
   6. Start the packet transfer:
        ```
        # echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/95-IPv4-forwarding.conf
        # sysctl -p /etc/sysctl.d/95-IPv4-forwarding.conf
        ```

**Verification**

- On each router, ping the router internal interface IP address:
  - On router A, ping 172.16.0.1:
        ```
        # ping 172.16.0.1
        ```
  - On router B, ping 192.0.2.1:
        ```
        # ping 192.0.2.1
        ```

## 16.3. Configure GRETAP tunnel to transfer Ethernet frames through IPv4

General Route Encapsulation Terminal Access Point(GRETAP) tunnel runs at the second layer of the OSI model and encapsulates the Ethernet traffic of IPv4 packet, as described in RFC 2784. The data sent by GRETAP tunnel is not encrypted. For the sake of safety, please create the tunnel through VPN or different encryption connections.

The following picture shows how to create GRETAP tunnel between tow routers to use bridging to connect two networks:
![GRETAP-tunnel](./assets/GRETAP-tunnel.png)
Please note, gretap0 is reserved, greptap1 is used as the connection name here.

**Prerequisite**

- Each router has a network interface, it connects to its local network, no IP address is allocated to the interface.
- Each router has a network interface, it connects to the internet.

**Process**

1. On the router of network A:
   1. Create the brige interface named bridge0:
        ```
        # nmcli connection add type bridge con-name bridge0 ifname bridge0
        ```
   2. Configure the IP settings of the bridge:
        ```
        # nmcli connection modify bridge0 ipv4.addresses '192.0.2.1/24'
        # nmcli connection modify bridge0 ipv4.method manual
        ```
   3. Add new connection configuration set which directs to the bridge for the interface that connects to the local network:
        ```
        # nmcli connection add type ethernet slave-type bridge con-name bridge0-port1 ifname enp1s0 master bridge0
        ```
   4. Add new connection configuration set of GRETAP tunnel for the bridge:
        ```
        # nmcli connection add type ip-tunnel ip-tunnel.mode gretap slave-type bridge con-name bridge0-port2 ifname gretap1 remote 198.51.100.5 local 203.0.113.10 master bridge0
        ```
        remote and local set the public IP address of the remote router and that of the local router.
   5. Close the STP(Spanning Tree Protocol) if you do not need it:
        ```
        # nmcli connection modify bridge0 bridge.stp no

        ```
   6. Configure that once the bridge0 is activated the bridge interface will be activated automatically:
        ```
        # nmcli connection modify bridge0 connection.autoconnect-slaves 1
        ```
   7. Activate bridge0 connection:
        ```
        # nmcli connection up bridge0
        ```
2. On the router of network B:
   1. Create the bridge interface named bridge0:
        ```
        # nmcli connection add type bridge con-name bridge0 ifname bridge0
        ```
   2. Configure the IP settings of the bridge:
        ```
        # nmcli connection modify bridge0 ipv4.addresses '192.0.2.2/24'
        # nmcli connection modify bridge0 ipv4.method manual
        ```
   3. Add new connection configuration sets which directs to the bridge for the interface that connects to the local network:
        ```
        # nmcli connection add type ethernet slave-type bridge con-name bridge0-port1 ifname enp1s0 master bridge0
        ```
   4. Add new connection configuration sets of the GRETAP tunnel interface for the bridge:
        ```
        # nmcli connection add type ip-tunnel ip-tunnel.mode gretap slave-type bridge con-name bridge0-port2 ifname gretap1 remote 203.0.113.10 local 198.51.100.5 master bridge0
        ```
        remote and local set the public IP address of the remote router and that of the local router.
   5. Close STP(Spanning Tree Protocol) if you do not need it:
        ```
        # nmcli connection modify bridge0 bridge.stp no

        ```
   6. Configure that once the bridge0 connection is activated, the bridge interface will be activated automatically:
        ```
        # nmcli connection modify bridge0 connection.autoconnect-slaves 1
        ```
   7. Activate bridge0 connection:
        ```
        # nmcli connection up bridge0
        ```

**Verification**

1. On the two routers, verify that whether the connection between enp1s0 and gretap1 already exists and whether the CONNECTION column shows the connection name:
    ```
    # nmcli device
    nmcli device
    DEVICE   TYPE      STATE      CONNECTION
    ...
    bridge0  bridge    connected  bridge0
    enp1s0   ethernet  connected  bridge0-port1
    gretap1  iptunnel  connected  bridge0-port2
    ```
2. On each router, ping the router internal interface IP address:
   1. On router A, ping 192.0.2.2:
        ```
        # ping 192.0.2.2
        ```
   2. On router B, ping 192.0.2.1:
        ```
        # ping 192.0.2.1
        ```


# Chapter 17  Using network-scripts

By default, OpenCloudOS use NetworkManager to configure and manage network connections, the two scripts /usr/sbin/ifup and /usr/sbin/ifdown use NetworkManager to process the ifcfg file in directory /etc/sysconfig/network-scripts/.


If you need to use network-scripts to configure and manage network, you can install it. Then,the scripts /usr/sbin/ifup and /usr/sbin/ifdown will link to the shell scripts that manage network configurations.

- Install network-scripts:
    ```
    # yum install network-scripts
    ```

# Chapter 18 Ports mirroring

You can use the NetworkManager to configure the ports mirroring, the following process shows copying the network traffic from enp1s0 to enp7s0 through adding the traffic control(tc) rules and filters to enp1s0.

**Prerequisite**

- There is a network interface that functions as network traffic mirror

**Process**

1. Add the network connection configuration set which you will use to copy network traffic:
    ```
    # nmcli connection add type ethernet ifname enp1s0 con-name enp1s0 autoconnect no
    ```
2. Append prio qdisc to the traffic exit enp1s0 using 10: handele mode:
    ```
    # nmcli connection modify enp1s0 +tc.qdisc "root prio handle 10:"
    ```
3. Add qdisc for the entrance traffic, add qdisc using ffff: handle:
    ```
    # nmcli connection modify enp1s0 +tc.qdisc "ingress handle ffff:"
    ```
4. Add the following filter to match the data package of qdiscs on the entrance and exit and copy the data package to enp7s0:
    ```
    # nmcli connection modify enp1s0 +tc.tfilter "parent ffff: matchall action mirred egress mirror dev enp7s0"

    # nmcli connection modify enp1s0 +tc.tfilter "parent 10: matchall action mirred egress mirror dev enp7s0"
    ```
    matchall filter matches with all the data package, the mirred operation will redirect the data package to the destination.
5. Activate the connection:
    ```
    # nmcli connection up enp1s0
    ```

**Verification**

1. Install tcpdump tool:
    ```
    # yum install tcpdump
    ```
2. Check the mirror traffic on the object device(enp7s0):
    ```
    # tcpdump -i enp7s0
    ```
