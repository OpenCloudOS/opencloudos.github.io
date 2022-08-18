# OpenCloudOS V8 操作系统图形安装指南

## 概述
作为企业级Linux服务器操作系统，OpenCloudOS基于Linux内核自主研发设计，OpenCloudOS  8.5 是稳定的企业级服务器Linux发行版，其稳定性、安全性、兼容性和性能等核心能力均已得到充分验证。

OpenCloudOS的ISO支持两种安装方式：

- **图形安装模式**，用户根据需要选择语言，软件源，安装磁盘，网络配置等操作。本文着重介绍图形安装的过程。
- 自动探测模式，引导之后安装就会执行安装操作。

### 1、介绍
OpenCloudOS 是企业级社区研发的定制化服务器操作系统。该系统集成了众多服务器系列的优点，加入自主研发的软件，便于用户操作使用，提供全方位（内核及用户态）的操作系统支持。系统特点：安全、易用、稳定、快速、长久支持。安装镜像提供了服务器常用的各种软件支持，同时可以使用线上软件源安装及更新软件。**此说明适用于OpenCloudOS 发行版的安装与使用。**

### 2、安装前准备
安装OpenCloudOS 服务器操作系统前，您的服务器需要满足以下要求：
- 服务器接入稳定电源
- 确保服务器至少拥有50GB硬盘空间，4GB内存空间
- 获取安装DVD光盘（需要服务器拥有DVD光驱）或USB安装（需要服务器拥有USB接口）
- **安装前请备份您的硬盘数据，以防数据丢失**
- 镜像获取地址：
  - http://mirrors.opencloudos.org/opencloudos/8/
  - https://mirrors.tencent.com/opencloudos/8/

### 3、光盘安装说明
- 插入安装光盘，启动时进入BIOS选择从CDROM驱动器启动
- 进入系统安装引导选择，选择Install进行安装               

### 4、USB安装说明
- 使用Rufus 或者 dd命令行烧录iso镜像到USB介质中
- 插入USB安装介质，启动时从BIOS选择USB启动
- 进入系统安装引导选择，选择Install进行安装

![OpenCloudOS V8 Installation example picture](../assets/OC_V8_Installation_example.png)

### 5、语言选择
OpenCloudOS 支持多语言的选择， 选择之后Continue 继续下一步操作。

![OpenCloudOS V8 software selection example picture](../assets/OC_V8_software_selection_example.png)

### 6、选择软件源
OpenCloudOS 支持用户根据需求选择软件源,点**Done**完成选择。

![OpenCloudOS V8 software selection example picture](../assets/OC_V8_software_selection_example.png)

### 7、选择安装目标
OpenCloudOS 支持选择安装路径，在服务器存在多个硬盘时，可以选择安装到指定硬盘中。
- 选择指定硬盘后，系统将自动分区，安装系统镜像
- 支持用户自定义分区，点**Custom**手动分区，完成Done

![OpenCloudOS V8 seclect destination disk example picture](../assets/OC_V8_seclect_destination_disk_example.png)

### 8、用户设置
系统安装支持用户设置root用户的口令，创建管理员或者普通用户及其口令。

![OpenCloudOS V8 user settings example picture](../assets/OC_V8_user_settings_1.png)

![OpenCloudOS V8 user settings example picture](../assets/OC_V8_user_settings_2.png)

### 9、完成安装
系统完成安装后会出现图中所示字样，Reboot可完成安装。

![OpenCloudOS V8 omplete installation example picture](../assets/OC_V8_complete_installation_example.png)

重启后，正常启动，可以看到系统目录中出现OpenCloudOS 引导界面。

![OpenCloudOS V8 boot interface example picture](../assets/OC_V8_boot_interface_example.png)

### 10、如何登陆
完成安装系统后，进入到登陆界面。输入用户名、口令 Enjoy OpenCloudOS

![OpenCloudOS V8 reboot and login example picture](../assets/OC_V8_reboot_and_login_example.png)
