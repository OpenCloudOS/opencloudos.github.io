# 第 1 章 逻辑卷管理概述

逻辑卷管理(LVM)在物理存储上创建抽象层，帮助您创建逻辑存储卷。这比直接使用物理存储的方式具有更大的灵活性。

此外，硬件存储配置在软件中隐藏，因此可以调整大小并移动，无需停止应用或卸载文件系统。这可降低操作成本。

## 1.1 LVM 构架

以下是 LVM 组件：

**物理卷**

物理卷(PV)是指定为 LVM 使用的分区或整个磁盘。如需更多信息，请参阅[管理 LVM 物理卷](#第-3-章-管理-lvm-物理卷)。

**卷组**

卷组(VG)是物理卷(PV)的集合，它会创建一个磁盘空间池，从中可以分配逻辑卷。如需更多信息，请参阅[管理 LVM 卷组](#第-4-章-管理-lvm-卷组)。

**逻辑卷**

逻辑卷代表可挂载的存储设备。如需更多信息，请参阅[管理 LVM 逻辑卷](#第-5-章-管理-lvm-逻辑卷)。

下图显示了 LVM 的组件：

**图 1.1 LVM 逻辑卷组件**

![LVM 逻辑卷组件](./images/LVM%20%E9%80%BB%E8%BE%91%E5%8D%B7%E7%BB%84%E4%BB%B6.png)

## 1.2 LVM 的优点

与直接使用物理存储相比，逻辑卷具有以下优势：

**灵活的容量**

使用逻辑卷时，您可以将设备和分区聚合到一个逻辑卷中。借助此功能，文件系统可以扩展到多个设备中，就像它们是一个单一的大型设备一样。

**存储卷大小**

您可以使用简单的软件命令扩展逻辑卷或减小逻辑卷大小，而无需重新格式化和重新分区基础设备。

**在线数据重新定位**

部署更新、更快或者更弹性的存储子系统，可以在系统活跃时移动数据。在磁盘处于使用状态时可以重新分配磁盘。例如，您可以在删除热插拔磁盘前将其清空。

**方便的设备命名**

逻辑卷可以使用用户定义的名称和自定义名称进行管理。

**条带化卷**

您可以创建一个在两个或者多个设备间条带化分布数据的逻辑卷。这可显著提高吞吐量。

**RAID 卷**

逻辑卷为您对数据配置 RAID 提供了一种便捷的方式。这可防止设备故障并提高性能。

**卷快照**

您可以对数据进行快照（逻辑卷在一个特点时间点上的副本）用于一致性备份或测试更改的影响，而不影响实际数据。

**精简卷**

逻辑卷可以使用精简模式置备。这可让您创建大于可用物理空间的逻辑卷。

**缓存卷**

缓存逻辑卷使用快速块设备，如 SSD 驱动器，以提高更大、较慢的块设备的性能。

# 第 2 章 使用系统角色管理本地存储

要使用 Ansible 管理 LVM 和本地文件系统(FS)，您可以使用 Storage 角色，这是 OpenCloud OS 中可用的系统角色之一。

使用存储角色可让您自动管理多台机器上的磁盘和逻辑卷上的文件系统，以及系统的版本。

## 2.1 Storage （存储）角色简介

Storage 角色可以管理：

- 磁盘上未被分区的文件系统
- 完整的 LVM 卷组，包括其逻辑卷和文件系统

使用 Storage 角色，您可以执行以下任务：

- 创建文件系统
- 删除文件系统
- 挂载文件系统
- 卸载文件系统
- 创建 LVM 卷组
- 删除 LVM 卷组
- 创建逻辑卷
- 删除逻辑卷
- 创建 RAID 卷
- 删除 RAID 卷
- 创建带有 RAID 的 LVM 池
- 删除带有 RAID 的 LVM 池

## 2.2 识别存储设备参数（存储系统角色）

您的存储角色配置仅影响您在以下变量中列出的文件系统、卷和池.

`storage_volumes`

在所有要管理的未分区磁盘中的文件系统列表。

当前不支持的分区。

`storage_pools`

要管理的池列表。

目前唯一支持的池类型是 LVM。使用 LVM 时，池代表卷组（VG）。每个池中都有一个要由角色管理的卷列表。对于 LVM，每个卷对应一个带文件系统的逻辑卷（LV）。

## 2.3 在块存储设备中创建XFS 文件系统的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色，来使用默认参数在块设备上创建 XFS 文件系统。

> **警告**
> 
> 存储角色只能在未分区、整个磁盘或逻辑卷（LV）上创建文件系统。它不能在分区中创建文件系统。

> **例 2.1 在/dev/vdb上创建 XFS 的playbook**
> 
> ```
> ---
> - hosts: all
>  vars:
>    storage_volumes:
>      - name: barefs
>        type: disk
>        disks:
>          - vdb
>        fs_type: xfs
>  roles:
>    - rhel-system-roles.storage
> ```
> 
> - 卷名称（示例中的 barefs ）目前是任意的。存储角色根据 disk: 属性下列出的磁盘设备来识别卷。
> - 您可以省略 `fs_type: xfs` 行，因为 XFS 是 OpenCloud OS 中的默认文件系统。
> - 要在 LV 上创建文件系统，请在 `disks: `属性下提供 LVM 设置，包括括起来的卷组。详情请参阅 管理逻辑卷 的 Ansible playbook 示例。
> 
> 不要提供到 LV 设备的路径。

## 2.4 永久挂载系统文件的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色用来立即和永久挂载 XFS 文件系统。

> **例 2.2 将 /dev/vdb 上的文件系统挂载到 /mnt/data 的 playbook**
> 
> ```
> ---
> - hosts: all
>  vars:
>   storage_volumes:
>      - name: barefs
>        type: disk
>        disks:
>          - vdb
>        fs_type: xfs
>        mount_point: /mnt/data
>  roles:
>    - rhel-system-roles.storage
> ```
> 
> - 此 playbook 将文件系统添加到 `/etc/fstab` 文件中，并立即挂载文件系统。
> - 如果 `/dev/vdb` 设备上的文件系统或挂载点目录不存在，则 playbook 会创建它们。

## 2.5 管理逻辑卷的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色在卷组中创建 LVM 逻辑卷。

> **例 2.3 在 myvg 卷组中创建 mylv 逻辑卷的 playbook**
> 
> ```
> - hosts: all
>  vars:
>    storage_pools:
>      - name: myvg
>        disks:
>          - vda
>          - vdb
>          - vdc
>        volumes:
>          - name: mylv
>            size: 2G
>            fs_type: ext4
>            mount_point: /mnt
>  roles:
>    - rhel-system-roles.storage
> ```
> 
> - `myvg` 卷组由以下磁盘组成：
>   
>   - `/dev/vda`
>   - `/dev/vdb`
>   - `/dev/vdc`
> 
> - 如果 `myvg` 卷组已存在，则 playbook 会将逻辑卷添加到卷组。
> 
> - 如果 `myvg` 卷组不存在，则 playbook 会创建它。
> 
> - playbook 在 `mylv` 逻辑卷上创建 Ext4 文件系统，并在 `/mnt` 上永久挂载文件系统。

## 2.5 启用在线快丢弃的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色用来挂载启用了在线块丢弃的 XFS 文件系统。

> **例 2.4 一个 playbook，它在 `/mnt/data/` 上启用在线块丢弃功能**
> 
> ```
> ---
> - hosts: all
>  vars:
>    storage_volumes:
>      - name: barefs
>        type: disk
>        disks:
>          - vdb
>        fs_type: xfs
>        mount_point: /mnt/data
>        mount_options: discard
>  roles:
>    - rhel-system-roles.storage
> ```

## 2.7 创建挂载 Ext4 文件系统的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色来创建和挂载 Ext4 文件系统。

> **例 2.5 在 `/dev/vdb` 上创建 Ext4 并挂载到 `/mnt/data` 的 playbook**
> 
> ```
> ---
> - hosts: all
>  vars:
>    storage_volumes:
>      - name: barefs
>        type: disk
>        disks:
>          - vdb
>        fs_type: ext4
>        fs_label: label-name
>        mount_point: /mnt/data
>  roles:
>    - rhel-system-roles.storage
> ```
> 
> - playbook 在 `/dev/vdb` 磁盘上创建文件系统。
> - playbook 将文件系统永久挂载在 `/mnt/data` 目录。
> - 文件系统的标签是 `label-name`。

## 2.8 创建和挂载 ext3 文件系统的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色来创建和挂载 Ext3 文件系统。

> **例 2.6 在 `/dev/vdb` 上创建 Ext3 ，并将其挂载在 `/mnt/data` 的 playbook**
> 
> ```
> ---
> - hosts: all
>  vars:
>    storage_volumes:
>      - name: barefs
>        type: disk
>        disks:
>          - vdb
>        fs_type: ext3
>        fs_label: label-name
>        mount_point: /mnt/data
>  roles:
>    - rhel-system-roles.storage
> ```
> 
> - playbook 在 `/dev/vdb` 磁盘上创建文件系统。
> - playbook 将文件系统永久挂载在 `/mnt/data` 目录。
> - 文件系统的标签是 `label-name`。

## 2.9 使用存储 OpenCloud OS系统角色对现有 Ext4 或 Ext3 文件系统调整大小的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色来调整块设备上现有的 Ext4 或 Ext3 文件系统的大小。

> **例 2.7 在磁盘上设置单个卷的 playbook**
> 
> ```
> ---
> - name: Create a disk device mounted on /opt/barefs
> - hosts: all
>  vars:
>    storage_volumes:
>      - name: barefs
>        type: disk
>        disks:
>          - /dev/vdb
>    size: 12 GiB
>        fs_type: ext4
>        mount_point: /opt/barefs
>  roles:
>    - rhel-system-roles.storage
> ```

- 如果上例中的卷已存在，要调整卷的大小，您需要运行相同的 playbook，使用不同的 `size` 参数值。例如：

> **例 2.8 在 `/dev/vdb`上调整 `ext4` 大小的 playbook**
> 
> ```
> ---
> - name: Create a disk device mounted on /opt/barefs
> - hosts: all
>  vars:
>    storage_volumes:
>      - name: barefs
>        type: disk
>        disks:
>          - /dev/vdb
>    size: 10 GiB
>        fs_type: ext4
>        mount_point: /opt/barefs
>  roles:
>    - rhel-system-roles.storage
> ```
> 
> - 卷名称（示例中的 barefs）当前是任意的。Storage 角色根据 disk: 属性中列出的磁盘设备标识卷。

> **注意**
> 
> 在其他文件系统中使用 调整大小 操作可能会破坏您正在使用的设备上的数据

## 2.10 使用存储 OpenCloud OS 系统角色在 LVM 上对现有文件系统的大小进行调整的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储 OpenCloud OS 系统角色来使用文件系统重新定义 LVM 逻辑卷大小。

> **警告**
> 
> 在其他文件系统中使用 调整大小 操作可能会破坏您正在使用的设备上的数据。

> **例 2.9 调整 myvg 卷组中现有 mylv1 和 myvl2 逻辑卷大小的 playbook**
> 
> ```
> ---
> 
> - hosts: all
>   vars:
>    storage_pools:
>      - name: myvg
>        disks:
>          - /dev/vda
>          - /dev/vdb
>          - /dev/vdc
>        volumes:
>            - name: mylv1
>              size: 10 GiB
>              fs_type: ext4
>              mount_point: /opt/mount1
>            - name: mylv2
>              size: 50 GiB
>              fs_type: ext4
>              mount_point: /opt/mount2
> 
> - name: Create LVM pool over three disks
>  incude_role:
>    name: rhel-system-roles.storage
> ```
> 
> - 此 playbook 调整以下现有文件系统的大小：
>   
>   - `mylv1` 卷上的 Ext4 文件系统挂载在 `/opt/mount1`，大小调整为 10 GiB。
>     - `mylv2` 卷上的 Ext4 文件系统挂载在 `/opt/mount2`，大小调整为 50 GiB。

## 2.11 使用存储 OpenCloud OS 系统角色创建交换分区的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色来创建交换分区（如果不存在的话），或者使用默认参数在块设备上修改交换分区（如果已存在的话）。

> **例 2.10 创建或修改 `/dev/vdb` 上现有 XFS 的 playbook**
> 
> ```
> name: Create a disk device with swap
> hosts: all
> vars:
>  storage_volumes:```
> - name: swap_fs
>  type: disk
>  disks:
>    - /dev/vdb
> size: 15 GiB
> fs_type: swap
> roles:
> - rhel-system-roles.storage 
> ```
> 
> - 卷名称（示例中的 `swap_fs` ）目前是任意的。存储角色根据 `disk:` 属性下列出的磁盘设备来识别卷。

## 2.12 使用存储系统角色配置 RAID 卷

使用存储系统角色，您可以使用 Ansible Automation Platform 在 OpenCloud OS 上配置 RAID 卷。在本小节中，您将了解如何使用可用参数设置 Ansible playbook 来配置 RAID 卷以满足您的要求。

**前提条件**

- Ansible Core 软件包安装在控制机器上。
- 您已在要运行 playbook 的系统上安装了 `rhel-system-roles` 软件包。
- 您有一个清单文件详细描述了您要使用存储系统角色部署 RAID 卷的系统。

**流程**

1. 使用以下内容创建一个新的 *`playbook.yml`* 文件：
   
   ```
   - hosts: all
    vars:
        storage_safe_mode: false
        storage_volumes:
         - name: data
           type: raid
           disks: [vdd, vde, vdf, vdg]
           raid_level: raid0
           raid_chunk_size: 32 KiB
           mount_point: /mnt/data
           state: present
    roles:
        - name: rhel-system-roles.storage
   ```
   
   > **警告**
   > 
   > 设备名称在某些情况下可能会改变，例如：当您在系统中添加新磁盘时。因此，为了避免数据丢失,我们不建议在 playbook 中使用特定的磁盘名称。

2. 可选。验证 playbook 语法。
   
   ```
    # ansible-playbook --syntax-check playbook.yml*
   ```

3. 在清单文件上运行 playbook:
   
   ```
   # ansible-playbook -i inventory.file /path/to/file/playbook.yml
   ```

## 2.13 使用存储系统角色使用 RAID 配置 LVM 池

使用 Storage 系统角色，您可以使用 Ansible Automation Platform 在 OpenCloud OS 上使用 RAID 配置 LVM 池。在本小节中，您将了解如何使用可用参数设置 Ansible playbook 来配置使用 RAID 的 LVM 池。

**前提条件**

- Ansible Core 软件包安装在控制机器上。
- 您已在要运行 playbook 的系统上安装了 `rhel-system-roles` 软件包。
- 您有一个清单文件详细描述了您要使用存储系统角色在其上配置带有 RAID 的 LVM 池的系统。

**流程**

1. 使用以下内容创建一个新的 *`playbook.yml`* 文件：
   
   ```
   - hosts: all
   vars:
       storage_safe_mode: false
       storage_pools:
       - name: my_pool
           type: lvm
           disks: [vdh, vdi]
           raid_level: raid1
           volumes:
           - name: my_pool
               size: "1 GiB"
               mount_point: "/mnt/app/shared"
               fs_type: xfs
               state: present
   roles:
       - name: rhel-system-roles.storage
   ```

> **注意**
> 
> 要创建带有 RAID 的 LVM 池，您必须使用 raid_level 参数指定 RAID 类型。

2. 可选。验证 playbook 语法。
   
   ```
   # ansible-playbook --syntax-check playbook.yml
   ```

3. 在清单文件上运行 playbook:
   
   ```
   # ansible-playbook -i inventory.file /path/to/file/playbook.yml
   ```

## 2.14 使用存储 OpenClous OS 系统角色在 LVM 上压缩和删除数据重复的 VDO 卷的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储 OpenCloud OS 系统角色来使用虚拟数据优化(VDO)，让卷对逻辑管理器卷(LVM)的进行压缩和删除重复数据操作。

**例 2.11 在 `myvg` 卷组中创建 `mylv1` LVM VDO 卷的 playbook**

> ```
> ---
> - name: Create LVM VDO volume under volume group 'myvg'
>  hosts: all
>  roles:
>    - rhel-system-roles.storage
>  vars:
>    storage_pools:
>     - name: myvg
>       disks:
>         - /dev/vdb
>       volumes:
>         - name: mylv1
>           compression: true
>           deduplication: true
>           vdo_pool_size: 10 GiB
>           size: 30 GiB
>           mount_point: /mnt/app/shared
> ```

在本例中，`compression` 和 `deduplication` 都被设置为 true，这里指定使用 VDO。下面描述了这些参数的用法：

- `deduplication` 用于删除存储在存储卷上重复的数据。
- `compression` 用于压缩存储卷上存储的数据，从而增加存储容量。
- `vdo_pool_size` 指定卷在设备上的实际大小。VDO 卷的虚拟大小由 `size` 参数设置。注：由于 Storage 角色使用 LVM VDO，每个池只有一个卷可以使用压缩和删除重复数据。

## 2.15 使用使用存储系统角色创建 LUKS 加密卷

您可以使用存储角色运行 Ansible playbook 来创建和配置那些使用 LUKS 加密的卷。

**前提条件**

- 对一个或多个 *受管节点* 的访问和权限，*受管节点* 是您要使用加密策略系统角色配置的系统。

- 对控制节点的访问和权限，这是 Ansible Core 配置其他系统的系统。

- 在控制节点上：
  
  - `ansible-core` 和 `rhel-system-roles` 软件包已安装 。

> **重要**
> 
> OpenCloud OS 提供对基于 Ansible 的自动化需要 Ansible Engine 的独立 Ansible 存储库的访问权限。Ansible Engine 包含命令行工具，如 ansible、ansible-playbook、连接器（如 docker 和 podman ）以及许多插件和模块。
> 
> OpenCloud OS 引入了 Ansible Core（以 ansible-core 软件包的形式提供），其中包含 Ansible 命令行工具、命令以及小型内置 Ansible 插件。

- 列出受管节点的清单文件。

**流程**

1. 使用以下内容创建一个新的 *`playbook.yml`* 文件：
   
   ```
   - hosts: all
   vars:
       storage_volumes:
       - name: barefs
           type: disk
           disks:
           - vdb
           fs_type: xfs
           fs_label: label-name
           mount_point: /mnt/data
           encryption: true
           encryption_password: your-password
   roles:
   - rhel-system-roles.storage
   ```

2. 可选：验证 playbook 语法：
   
   ```
   # ansible-playbook --syntax-check playbook.yml
   ```

3. 在清单文件上运行 playbook:
   
   ```
   # ansible-playbook -i inventory.file /path/to/file/playbook.yml
   ```

## 2.16 使用存储 OpenCloud OS 系统角色以百分比形式表示池卷大小的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储系统角色来通过存储池百分比的形式表示逻辑卷管理卷(LVM)的大小。

> **例 2.12 将卷大小表示为池容量百分比的 playbook**
> 
> ```
> ---
> - name: Express volume sizes as a percentage of the pool's total size
>  hosts: all
>  roles
>    - rhel-system-roles.storage
>  vars:
>    storage_pools:
>    - name: myvg
>      disks:
>        - /dev/vdb
>      volumes:
>        - name: data
>          size: 60%
>          mount_point: /opt/mount/data
>        - name: web
>          size: 30%
>          mount_point: /opt/mount/web
>        - name: cache
>          size: 10%
>          mount_point: /opt/cache/mount
> ```

这个示例将 LVM 卷的大小以存储池容量的百分比表示，例如："60%"。另外，您还可以将 LVM 卷的大小以文件系统中人类可读的类型（如 "10g" 或 "50 GiB"）使用存储池容量的百分比表示。

# 第 3 章 管理 LVM 物理卷

物理卷(PV)是 LVM 要使用的分区或整个磁盘。要将设备用于 LVM 逻辑卷，必须将设备初始化为物理卷。

如果您将整个磁盘作为您的物理卷使用，那么磁盘就不能有分区表。对于 DOS 磁盘分区，应该使用 `fdisk` 或 `cfdisk` 命令或对等命令将分区 id 设置为 0x8e。如果您将整个磁盘作为您的物理卷使用，那么磁盘就不能有分区表。必须擦除任何现有分区表，这样会有效地破坏该磁盘中的所有数据。您可以使用 `wipefs -a <PhysicalVolume>'` 命令作为 root 删除现有分区表。

## 3.1 物理卷概述

将块设备初始化为物理卷会在接近设备起始的位置放置一个标签。下面描述了 LVM 标签：

- LVM 标签为物理设备提供正确的标识和设备排序。未标记的非 LVM 设备可以在重新引导后更改名称，具体取决于系统在启动过程中发现它们的顺序。LVM 标签在重新引导时具有持久性并在整个集群中可用。

- LVM 标签可将该设备识别为 LVM 物理卷。它包含一个随机唯一标识符，即物理卷的 UUID。它还以字节为单位保存块设备的大小，并记录 LVM 元数据存储在该设备中的位置。

- 默认情况下，LVM 标签是放在第二个 512 字节扇区。您可以在创建物理卷时将标签放在前 4 个扇区的任意一个扇区，从而覆盖此默认设置。如果需要，LVM 卷可与其它使用这些扇区的用户共同存在。

下面描述了 LVM 元数据：

- LVM 元数据包含您系统中 LVM 卷组的配置详情。默认情况下，卷组中的每个物理卷的元数据区域都会保留一个一样的元数据副本。LVM 元数据很小，它以 ASCII 格式保存。

- 目前 LVM 允许您在每个物理卷中存储 0、1 或 2 个元数据副本。默认为 1 个副本。当您在物理卷中配置元数据副本数后，您将无法再更改该号码。第一个副本保存在设备的起始位置，紧随在标签后面。如果有第二个副本，会将其放在设备的末尾。如果您不小心写入了不同于您想要写入的磁盘覆盖了磁盘起始部分，那么您可以使用在设备末尾的元数据的第二个副本恢复元数据。

下图说明了 LVM 物理卷的布局。LVM 标签在第二个扇区，接下来是元数据区域，后面是设备的可用空间。

> **注意**
> 
> 在 Linux 内核和整个文档中，扇区的大小被视为 512 字节。

**图 3.1 物理卷布局**

![物理卷布局](./images/%E7%89%A9%E7%90%86%E5%8D%B7%E5%B8%83%E5%B1%80.png)

## 3.2. 一个磁盘上的多个分区

您可以使用 LVM 从磁盘分区中创建物理卷(PV)。

建议您创建一个覆盖整个磁盘的单一分区，将其标记为 LVM 物理卷，理由如下：

**方便管理**

如果每个真实磁盘只出现一次，那么在系统中追踪硬件就比较容易。特别是当磁盘失败时。

**条带化性能**

LVM 无法告知两个物理卷位于同一个物理磁盘中。如果您在两个物理卷位于同一物理磁盘时创建了条状逻辑卷，那么条带就可能在同一磁盘的不同分区中。这可能会降低性能，而不是提高性能。

**RAID 冗余**

LVM 无法确定两个物理卷是否位于同一设备中。如果您在位于同一设备上的两个物理卷上创建 RAID 逻辑卷，则性能和容错可能会丢失。

虽然不建议您这样做，但在某些情况下可能需要将磁盘分成独立的 LVM 物理卷。例如：在有多个磁盘的系统中，当您要将现有系统迁移到 LVM 卷时，可能需要将数据在分区间转移。另外，如果您有一个很大的磁盘，并且因为管理的原因想要有一个以上卷组，那么对磁盘进行分区是很必要的。如果您的磁盘有一个以上的分区，且这些分区在同一卷组中，在创建卷时指定逻辑卷中应包含哪些分区。

请注意，虽然 LVM 支持将非分区磁盘用作物理卷，但建议创建一个全磁盘分区，因为在混合操作系统环境中创建不含分区的 PV 可能会有问题。其他操作系统可能会将该设备解释为可用，并覆盖驱动器开头的 PV 标签。

## 3.3 创建 LVM 物理卷

这个步骤描述了如何创建和标记 LVM 物理卷（PV）。

在此过程中，将 */dev/vdb1*、*/dev/vdb2* 和 */dev/vdb3* 替换为您系统中的可用存储设备。

**前提条件**

- 已安装 `lvm2` 软件包。

**流程**

1. 使用以空格分隔的设备名称作为 `pvcreate` 命令的参数来创建多个物理卷：
   
   ```
   # pvcreate /dev/vdb1 /dev/vdb2 /dev/vdb3
   Physical volume "/dev/vdb1" successfully created.
   Physical volume "/dev/vdb2" successfully created.
   Physical volume "/dev/vdb3" successfully created.
   ```
   
   这会在 */dev/vdb1*、*/dev/vdb2* 和 */dev/vdb3* 上放置一个标签，将它们标记为属于 LVM 的物理卷。

2. 根据您的要求，使用以下命令之一查看创建的物理卷：
   
   1. `pvdisplay` 命令，它为每个物理卷提供详细的多行输出。它以固定格式显示物理属性，如大小、扩展、卷组和其他选项：
      
      ```
      # pvdisplay
      
      "/dev/vdb1" is a new physical volume of "5.00 GiB"
      --- NEW Physical volume ---
      PV Name               /dev/vdb1
      VG Name
      PV Size               5.00 GiB
      Allocatable           NO
      PE Size               0
      Total PE              0
      Free PE               0
      Allocated PE          0
      PV UUID               2P70iQ-wrmM-sdLS-p4or-ezbt-iN5H-TIei1C
      
      "/dev/vdb2" is a new physical volume of "5.00 GiB"
      --- NEW Physical volume ---
      PV Name               /dev/vdb2
      VG Name
      PV Size               5.00 GiB
      Allocatable           NO
      PE Size               0
      Total PE              0
      Free PE               0
      Allocated PE          0
      PV UUID               iweiWy-KiY7-iGOv-3Vcr-ne33-mZ9A-IGK5gz
      
      "/dev/vdb3" is a new physical volume of "5.00 GiB"
      --- NEW Physical volume ---
      PV Name               /dev/vdb3
      VG Name
      PV Size               5.00 GiB
      Allocatable           NO
      PE Size               0
      Total PE              0
      Free PE               0
      Allocated PE          0
      PV UUID               RVdXvl-rxYI-8gUo-J2Mx-6aQN-34JQ-QMX0OY
      ```
   
   2. `pvs` 命令提供了可以对其进行格式配置的物理卷信息，每行显示一个物理卷。
      
      ```
      # pvs
      PV         VG          Fmt  Attr PSize   PFree
        /dev/vda2  opencloudos lvm2 a--  <49.00g    0
        /dev/vdb1              lvm2 ---    5.00g 5.00g
        /dev/vdb2              lvm2 ---    5.00g 5.00g
        /dev/vdb3              lvm2 ---    5.00g 5.00g
      ```
   
   3. `pvscan` 命令扫描系统中所有支持的物理卷 LVM 块设备。您可以在 `lvm.conf` 文件中定义过滤器，以便这个命令避免扫描特定物理卷：
      
      ```
      # pvscan
        PV /dev/vda2   VG opencloudos     lvm2 [<49.00 GiB / 0    free]
        PV /dev/vdb1                      lvm2 [5.00 GiB]
        PV /dev/vdb2                      lvm2 [5.00 GiB]
        PV /dev/vdb3                      lvm2 [5.00 GiB]
        Total: 4 [<64.00 GiB] / in use: 1 [<49.00 GiB] / in no VG: 3 [15.00 GiB]
      ```

## 3.4 删除 LVM 物理卷

如果 LVM 不再需要某个设备，您可以使用 `pvremove` 命令删除 LVM 标签。执行 `pvremove` 命令会将空物理卷上的 LVM 元数据归零。

**流程**

1. 删除物理卷：
   
   ```
   # pvremove /dev/vdb3
   Labels on physical volume "/dev/vdb3" successfully wiped.
   ```

2. 查看现有物理卷并验证是否已删除所需卷：
   
   ```
   # pvs
   PV         VG          Fmt  Attr PSize   PFree
   /dev/vda2  opencloudos lvm2 a--  <49.00g    0
   /dev/vdb1              lvm2 ---    5.00g 5.00g
   /dev/vdb2              lvm2 ---    5.00g 5.00g
   ```

如果您要删除的物理卷当前是卷组的一部分，则必须使用 vgreduce 命令将其从卷组中删除。如需更多信息，请参阅[从卷组中删除物理卷](#43-从卷组中删除物理卷)

# 第 4 章 管理 LVM 卷组

卷组(VG)是物理卷(PV)的集合，它会创建一个磁盘空间池，从中可以分配逻辑卷。

在卷组中，可用于分配的磁盘空间被分成固定大小的单元，我们称之为扩展。一个扩展就是可被分配的最小空间单位。在物理卷中，扩展被称为物理扩展。

逻辑卷被分配成与物理卷扩展大小相同的逻辑扩展。因此卷组中的所有逻辑卷的扩展大小都是一样的。卷组将逻辑扩展与物理扩展匹配。

## 4.1 创建 LVM 卷组

此流程描述了如何使用 */dev/vdb1* 和 */dev/vdb2* 物理卷创建 LVM 卷组(VG) myvg。

**前提条件**

- 已安装 `lvm2` 软件包。
- 创建一个或多个物理卷。有关创建物理卷的更多信息，请参阅[创建 LVM 物理卷](#33-创建-lvm-物理卷)。

**流程**

1. 创建卷组：
   
   ```
   # vgcreate myvg /dev/vdb1 /dev/vdb2
   Volume group "myvg" successfully created.
   ```
   
   这将创建一个名为 *myvg* 的 VG。PV */dev/vdb1* 和 */dev/vdb2* 是 myvg VG 的基础存储级别。

2. 根据您的要求，使用以下命令之一查看创建的卷组：
   
   1. `vgs` 命令以可配置的形式提供卷组信息，每行显示一个卷组：
      
      ```
      # vgs
      VG          #PV #LV #SN Attr   VSize   VFree
      myvg          2   0   0 wz--n-   9.99g 9.99g
      ```
   
   2. `vgdisplay` 命令以固定格式显示卷组属性，如大小、区块、物理卷数量和其他选项。以下示例显示了卷组 *myvg* 的 `vgdisplay` 命令的输出。如果没有指定卷组，则会显示所有现有卷组：
      
      ```
      # vgdisplay myvg 
      --- Volume group ---
      VG Name               myvg
      System ID
      Format                lvm2
      Metadata Areas        2
      Metadata Sequence No  1
      VG Access             read/write
      VG Status             resizable
      MAX LV                0
      Cur LV                0
      Open LV               0
      Max PV                0
      Cur PV                2
      Act PV                2
      VG Size               9.99 GiB
      PE Size               4.00 MiB
      Total PE              2558
      Alloc PE / Size       0 / 0
      Free  PE / Size       2558 / 9.99 GiB
      VG UUID               HmLe8g-552z-YVnh-CIsi-VWdf-hoWz-7ycLj
      [..]
      ```
   
   3. `vgscan` 命令为卷组扫描系统中所有受支持的 LVM 块设备：
      
      ```
      # vgscan
      Found volume group "myvg" using metadata type lvm2
      ```

3. 可选：通过添加一个或多个空闲的物理卷来增加卷组的容量：
   
   ```
   # vgextend myvg /dev/vdb3
   Physical volume "/dev/vdb3" successfully created.
   Volume group "myvg" successfully extended
   ```

## 4.2 合并 LVM 卷组

要将两个卷组合并为一个卷组，请使用 `vgmerge` 命令。如果这两个卷的物理扩展大小相等，且两个卷组的物理卷和逻辑卷的描述符合目的卷组的限制，您可以将一个不活跃的"源"卷与一个活跃或者不活跃的"目标"卷合并。

**流程**

- 将不活跃卷组 数据库 合并到活跃或者不活跃的卷组 myvg 中，提供详细的运行时信息：
  
  ```
  # vgmerge -v myvg databases
  ```

## 4.3 从卷组中删除物理卷

要从卷组中删除未使用的物理卷，请使用 `vgreduce` 命令。`vgreduce` 命令通过删除一个或多个空物理卷来缩小卷组的容量。这样就可以使不同的卷组自由使用那些物理卷，或者将其从系统中删除。

**流程**

1. 如果物理卷仍在使用，则将数据从同一卷组中迁移到另一个物理卷中：
   
   ```
   # pvmove /dev/vdb3
   /dev/vdb3: Moved: 2.0%
   ...
   /dev/vdb3: Moved: 79.2%
   ...
   /dev/vdb3: Moved: 100.0%
   ```

2. 如果现有卷组中的其他物理卷中没有足够的可用扩展：
   
   1. 从 */dev/vdb4* 创建一个新物理卷：
      
      ```
      # pvcreate /dev/vdb4
      Physical volume "/dev/vdb4" successfully created
      ```
   
   2. 将新创建的物理卷添加到 *myvg* 卷组：
      
      ```
      # vgextend myvg /dev/vdb4
      Volume group "myvg" successfully extended
      ```
   
   3. 将数据从 */dev/vdb3* 移到 */dev/vdb4* 中 ：
      
      ```
      # pvmove /dev/vdb3 /dev/vdb4
      /dev/vdb3: Moved: 33.33%
      /dev/vdb3: Moved: 100.00%
      ```

3. 从卷组中删除物理卷 */dev/vdb3*:
   
   ```
   # vgreduce myvg /dev/vdb3
   Removed "/dev/vdb3" from volume group "myvg"
   ```

**验证**

- 验证 */dev/vdb3* 物理卷是否已从 *myvg* 卷组中删除：
  
  ```
  # pvs
  PV         VG          Fmt  Attr PSize   PFree
  /dev/vdb1  myvg        lvm2 a--   <5.00g <5.00g
  /dev/vdb2  myvg        lvm2 a--   <5.00g <5.00g
  /dev/vdb3              lvm2 ---    5.00g  5.00g
  ```

## 4.4 分割 LVM 卷组

这个步骤描述了如何分割现有卷组。如果在物理卷中有足够的空闲空间，就可在不添加新磁盘的情况下创建新的卷组。

在初始设置中，卷组 *myvg* 由 */dev/vdb1*、*/dev/vdb2* 和 */dev/vdb3* 组成。完成此步骤后，卷组 *myvg* 将包含 */dev/vdb1* 和 */dev/vdb2*，第二个卷组 *yourvg* 将包含 */dev/vdb3*。

**前提条件**

- 卷组中有足够的空间。使用 `vgscan` 命令确定卷组中当前有多少可用空间。

- 根据现有物理卷中的可用容量，使用 `pvmove` 命令将所有使用的物理区块移动到其他物理卷。如需更多信息，请参阅[从卷组中删除物理卷](#43-从卷组中删除物理卷)。

**流程**

1. 将现有卷组 *myvg* 拆分到新卷组 *yourvg* ：
   
   ```
   # vgsplit myvg yourvg /dev/vdb3
     New volume group "yourvg" successfully split from "myvg"
   ```
   
   > **注意**
   > 
   > 如果您使用现有卷组创建了逻辑卷，请使用以下命令取消激活逻辑卷：
   > 
   > ```
   > # lvchange -a n /dev/myvg/mylv
   > ```
   > 
   > 有关创建逻辑卷的更多信息，请参阅[管理 LVM 逻辑卷](#第-5-章-管理-lvm-逻辑卷)。

2. 查看两个卷组的属性：
   
   ```
   # vgs
     VG          #PV #LV #SN Attr   VSize   VFree
     myvg          3   0   0 wz--n- <11.99g <11.99g
     yourvg        1   0   0 wz--n-  <5.00g  <5.00g
   ```

**验证**

- 验证新创建的卷组 *yourvg* 是否由 */dev/vdb3* 物理卷组成：
  
  ```
  # pvs
  PV         VG          Fmt  Attr PSize   PFree
  /dev/vdb1  myvg        lvm2 a--   <5.00g <5.00g
  /dev/vdb2  myvg        lvm2 a--   <5.00g <5.00g
  /dev/vdb3  yourvg      lvm2 a--   <5.00g <5.00g
  /dev/vdb4  myvg        lvm2 a--   <2.00g <2.00g
  ```

## 4.5 重命名 LVM 卷组

此流程将现有卷组 *myvg* 重命名为 *myvg1*。

**流程**

1. 取消激活卷组。如果是一个集群卷组，在每个这样的节点上使用以下命令取消激活它的所有节点上的卷组：
   
   ```
   # vgchange --activate n myvg
   ```

2. 重命名现有卷组：
   
   ```
   # vgrename myvg myvg1
     Volume group "myvg" successfully renamed to "myvg1"
   ```
   
    您还可以通过指定设备的完整路径来重命名卷组：
   
   ```
   # vgrename /dev/myvg /dev/myvg1
   ```

## 4.6 将卷组移动到另一个系统中

您可以将整个 LVM 卷组移动到另一个系统中。建议您在执行此操作时使用 `vgexport` 和 `vgimport` 命令。

> **注意**
> 
> 您可以使用 `vgimport` 命令的 `--force` 参数。这可让您导入缺少物理卷的卷组，然后运行 `vgreduce --removemissing` 命令。

`vgexport` 命令使系统无法访问非活动卷组，这样您可以将其物理卷分离。在 `vgexport` 命令使卷组不活动后，`vgimport` 命令可使卷组再次可供计算机访问。

要从一个系统移动卷组到另一个系统,，执行以下步骤：

1. 确定没有用户正在访问卷组中激活卷中的文件，然后卸载逻辑卷。

2. 使用 `vgchange` 命令的 `-a n` 参数将卷组标记为不活动，这样可防止对卷组的任何其他活动。

3. 使用 `vgexport` 命令导出卷组。这样可防止您要将其从中删除的系统访问该卷组。
   
    导出卷组后，执行 `pvscan` 命令时物理卷会在导出的卷组中显示，如下例所示：
   
   ```
   # pvscan
   PV /dev/vdb3   VG yourvg          lvm2 [<5.00 GiB / <5.00 GiB free]
   PV /dev/vdb1   VG myvg1           lvm2 [<5.00 GiB / <5.00 GiB free]
   PV /dev/vdb2   VG myvg1           lvm2 [<5.00 GiB / <5.00 GiB free]
   PV /dev/vdb4   VG myvg1           lvm2 [<2.00 GiB / <2.00 GiB free]
   ...
   ```
   
    当关闭系统时，您可以拔出组成该卷组的磁盘并将其连接到新系统。

4. 磁盘插入新系统后，请使用 `vgimport` 命令导入卷组，使其能被新系统访问。

5. 使用 `vgchange` 命令的 `-a y` 参数激活卷组。

6. 挂载文件系统使其可使用。

## 4.7 删除 LVM 卷组

这个步骤描述了如何删除现有卷组。

**前提条件**

- 卷组没有包含逻辑卷。要从卷组中删除逻辑卷，请参阅[删除 LVM 逻辑卷](#57-删除-lvm-逻辑卷)。

**流程**

1. 如果卷组存在于集群的环境中，在所有节点上停止卷组的锁定空间。在除您要删除的节点外的所有节点上使用以下命令：
   
   ```
   # vgchange --lockstop vg-name
   ```
   
    等待锁定停止。

2. 删除卷组：
   
   ```
   # vgremove vg-name
   Volume group "vg-name" successfully removed
   ```

# 第 5 章 管理 LVM 逻辑卷

逻辑卷是文件系统、数据库或应用可以使用的虚拟块存储设备。要创建 LVM 逻辑卷，物理卷(PV)合并为一个卷组(VG)。这会创建一个磁盘空间池，用于分配 LVM 逻辑卷（LV）。

## 5.1 逻辑卷概述

管理员可以在不损坏数据的情况下增大或缩小逻辑卷，这与标准磁盘分区不同。如果卷组中的物理卷位于不同的驱动器或者 RAID 阵列中，那么管理员也可以跨存储设备分配逻辑卷。

如果您缩小逻辑卷到比卷中数据所需的容量小的容量时，则可能会丢失数据。此外，某些文件系统无法缩小。为确保灵活性最大化，请创建逻辑卷以满足您的当前需求，并将过量存储容量保留为未分配。您可以根据需要安全地扩展逻辑卷使用未分配空间。

> **重要**
> 
> 在 AMD、Intel、ARM 和 IBM Power Systems 服务器中,引导装载程序无法读取 LVM 卷。您必须为您的 `/boot` 分区创建一个标准的非 LVM 磁盘分区。在 IBM Z 中，`zipl` 引导装载程序通过线性映射在 LVM 逻辑卷中支持 `/boot`。默认情况下，安装过程总是在 LVM 卷中创建 `/` 和 swap 分区，物理卷中有一个单独的 `/boot` 分区。

以下是不同类型的逻辑卷：

**线性卷**

线性卷将来自一个或多个物理卷的空间集合到一个逻辑卷中。例如：如果您有两个 60GB 的磁盘，您可以创建一个 120GB 的逻辑卷。物理存储是连在一起的。

**条带化逻辑卷**

当您向 LVM 逻辑卷写入数据时，文件系统会在基本物理卷之间部署数据。您可以通过创建一个条状逻辑卷来控制将数据写入物理卷的方法。对于大量连续的读取和写入，这样可以提高数据输入/输出的效率。

条带化通过以 round-robin 模式向预定数目的物理卷写入数据来提高性能。使用条带，I/O 可以并行执行。在某些情况下，这可能会导致条带中的每个额外的物理卷提高近线的性能。

**RAID 逻辑卷**

LVM 支持 RAID 0、1、4、5、6 和 10。RAID 逻辑卷不是集群感知型逻辑卷。当您创建 RAID 逻辑卷时，LVM 会为阵列中的每个数据或奇偶校验子卷创建一个元数据子卷。

**精简配置的逻辑卷（精简卷）**

使用精简配置的逻辑卷，您可以创建大于可用物理存储的逻辑卷。通过创建精简配置的卷集合，系统可以分配您使用的内容，而不是分配请求的完整存储量

**快照卷**

LVM 快照功能提供在特定时间创建设备的虚拟镜像且不会造成服务中断的功能。在提取快照后，当对原始设备进行修改时，快照功能会生成有变化的数据区域的副本，以便重建该设备的状态。

**精简配置的快照卷**

使用精简配置的快照卷，可以有更多虚拟设备存储在同一个数据卷中。精简置备的快照很有用，因为您不会复制在给定时间要捕获的所有数据。

**缓存卷**

LVM 支持在较慢的块设备中使用快速块设备（比如 SSD 驱动器）作为写入或者写入缓存。用户可以创建缓存逻辑卷以提高其现有逻辑卷的性能，或者创建新的缓存逻辑卷，这些逻辑卷由小、快速设备组成，结合大型且缓慢的设备。

## 5.2 使用 CLI 命令

以下小节描述了 LVM CLI 命令的一些一般操作功能。

**在命令行参数中指定单元**

当在命令行参数中需要大小时，可以明确指定其单位。如果您没有指定单位，那么就使用默认单位，通常为 KB 或者 MB。LVM CLI 命令不接受分数。

当在命令行参数中指定单位时，LVM 是不区分大小写的；例如， M 和 m 是等效的，都使用 2 的幂（ 1024 的倍数）。但是，当在命令中指定 **--units** 参数时，小写表示该单位是 1024 的倍数，大写表示该单位是 1000 的倍数。

**指定卷组和逻辑卷**

在 LVM CLI 命令中指定卷组或者逻辑卷时请注意以下几点。

- 如果命令使用卷组或者逻辑卷名称作为参数，则完整路径名称是可选的。卷组 `vg 0` 中名为 `lvol 0` 的逻辑卷可以指定为 `vg0/lvol0`。
- 当需要卷组列表但为空时，则使用所有卷组的列表替代。
- 当需要列出逻辑卷但提供了一个卷组，则使用在那个卷组中的所有逻辑卷列表替代。例如，`lv` `display` `vg0` 命令将显示卷组 `vg0` 中的所有逻辑卷。

**增加输出详细程度**

所有 LVM 命令都接受 `-v` 参数，该参数可多次输入来提高输出详细程度。以下示例显示了 `lvcreate` 命令的默认输出。

    ```
    # lvcreate -L 50MB new_vg
      Rounding up size to full physical extent 52.00 MB
      Logical volume "lvol0" created
    ```

以下命令显示带有 `-v` 参数的 `lvcreate` 命令的输出。

    ```
    # lvcreate -v -L 50MB new_vg
    Rounding up size to full physical extent 52.00 MiB
    Creating logical volume lvol1
    Archiving volume group "new_vg" metadata (seqno 1).
    Activating logical volume new_vg/lvol0.
    activation/volume_list configuration setting not defined: Checking only host tags for new_vg/lvol0.
    Creating new_vg-lvol0
    Loading table for new_vg-lvol0 (251:2).
    Resuming new_vg-lvol0 (251:2).
    Wiping known signatures on logical volume new_vg/lvol0.
    Initializing 4.00 KiB of logical volume new_vg/lvol0 with value 0.
    Logical volume "lvol0" created.
    Creating volume group backup "/etc/lvm/backup/new_vg" (seqno 2).
    ```

`v` `vv`、`-vvv` 和 `-vvvv` 参数显示更多关于命令执行的详细信息。`vvvv` 参数提供目前的最大信息量。以下示例显示了 `lvcreate` 命令的输出行前几行，指定了 `-vvvvv` 参数。

    ```
    # lvcreate -vvvv -L 50MB new_vg
    #lvmcmdline.c:913         Processing: lvcreate -vvvv -L 50MB new_vg
    #lvmcmdline.c:916         O_DIRECT will be used
    #config/config.c:864       Setting global/locking_type to 1
    #locking/locking.c:138       File-based locking selected.
    #config/config.c:841       Setting global/locking_dir to /var/lock/lvm
    #activate/activate.c:358       Getting target version for linear
    #ioctl/libdm-iface.c:1569         dm version   OF   [16384]
    #ioctl/libdm-iface.c:1569         dm versions   OF   [16384]
    #activate/activate.c:358       Getting target version for striped
    #ioctl/libdm-iface.c:1569         dm versions   OF   [16384]
    #config/config.c:864       Setting activation/mirror_region_size to 512
    ...
    ```

显示 LVM CLI 命令的帮助信息
您可以使用 命令的 `--help` 参数显示任何 LVM CLI 命令的帮助信息。

```
# commandname --help
```

要显示命令的 man page，执行 man 命令：

```
# man commandname
```

`man lvm` 命令提供有关 LVM 的常规在线信息。

## 5.3 创建 LVM 逻辑卷

此流程描述了如何从 *myvg* 卷组中创建 *mylv* LVM 逻辑卷(LV)，该组使用 /*dev/vdb1*、*/dev/vdb2* 和 */dev/vdb3* 物理卷创建。

**前提条件**

- 已安装 lvm2 软件包。
- 已创建卷组。如需更多信息，请参阅[创建 LVM 卷组](#41-创建-lvm-卷组)。

**流程**

1. 创建逻辑卷：
   
   ```
   # lvcreate -n mylv -L 500M myvg
       Logical volume "mylv" created.
   ```
   
   使用 `-n` 选项将 LV 名称设置为 *mylv*，并使用 `-L` 选项以 Mb 为单位设置 LV 的大小，但可以使用任何其他单元。默认情况下 LV 类型是线性的，但用户可以使用 `--type` 选项指定所需的类型。
   
   > **重要**
   > 
   > 如果 VG 没有足够数量的可用物理扩展用于请求的大小和类型，该命令将失败。

2. 根据您的要求，使用以下命令之一查看创建的逻辑卷：
   
   1. `lvs` 命令提供了可以对其进行格式配置的逻辑卷信息，每行显示一个逻辑卷。
      
      ```
      # lvs
      LV    VG          Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      mylv  myvg        -wi-a----- 500.00m
      ```
   
   2. `lvdisplay` 命令以固定格式显示逻辑卷属性，如大小、布局和映射：
      
      ```
      --- Logical volume ---
      LV Path                /dev/myvg/mylv
      LV Name                mylv
      VG Name                myvg
      LV UUID                aHKLlz-ToBk-ZGYi-ExZH-C6Cl-MUun-sfYU8p
      LV Write Access        read/write
      LV Creation host, time OpencloudOS, 2022-10-25 08:57:16 +0800
      LV Status              available
      # open                 0
      LV Size                500.00 MiB
      Current LE             125
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     8192
      Block device           251:6
      ```
   
   3. `lvscan` 命令扫描系统中所有逻辑卷并列出它们：
      
      ```
      # lvscan
        ACTIVE            '/dev/myvg/mylv' [500.00 MiB] inherit
      ```

3. 在逻辑卷中创建文件系统。以下命令在逻辑卷中创建 `xfs` 文件系统：
   
   ```
   # mkfs.xfs /dev/myvg/mylv
   meta-data=/dev/myvg/mylv         isize=512    agcount=4, agsize=32000 blks
           =                       sectsz=512   attr=2, projid32bit=1
           =                       crc=1        finobt=1, sparse=1, rmapbt=0
           =                       reflink=1
   data     =                       bsize=4096   blocks=128000, imaxpct=25
           =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
   log      =internal log           bsize=4096   blocks=1368, version=2
           =                       sectsz=512   sunit=0 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   ```

4. 挂载逻辑卷并报告文件系统磁盘空间使用情况：
   
   ```
   # mount /dev/myvg/mylv /mnt
   
   # df -h
   Filesystem                    Size  Used Avail Use% Mounted on
   
   /dev/mapper/myvg-mylv         495M   29M  466M   6% /mnt
   ```

## 5.4 创建 RAID0（条状）逻辑卷

RAID0 逻辑卷以条的大小为单位，将逻辑卷数据分散到多个数据子卷中。

创建 RAID0 卷的命令格式如下。

    ```
    lvcreate --type raid0[_meta] --stripes Stripes --stripesize StripeSize VolumeGroup [PhysicalVolumePath ...]
    ```

**表 5.1 RAID0 命令创建参数**

| **参数**                     | **描述**                                                                                                                                                                                                                                                   |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--type raid0[_meta]`      | 指定 `raid0` 创建一个没有元数据卷的 RAID0 卷。指定 `raid0_meta` 会创建一个带有元数据卷的 RAID0 卷。因为 RAID0 是不弹性的，所以不需要保存任何已镜像的数据块（如 RAID1/10），或者计算并保存任何奇偶校验块（如 RAID4/5/6）。因此，它不需要元数据卷来保持有关镜像或奇偶校验块重新同步进程的状态。但是，在从 RAID0 转换到 RAID4/5/6/10 时元数据卷是强制的，指定 `raid0_meta` 会预先分配这些元数据卷以防止分配失败。 |
| `--stri pes stripes`       | 指定在其中分割逻辑卷的设备数。                                                                                                                                                                                                                                          |
| `--stripe size stripeSize` | 以 KB 为单位指定每个条的大小。这是在移动到下一个设备前写入一个设备的数据量。                                                                                                                                                                                                                 |
| `VolumeGroup`              | 指定要使用的卷组。                                                                                                                                                                                                                                                |
| `PhysicalVolumePath` …​    | 指定要使用的设备。如果没有指定，LVM 会选择 *Stripes* 选项指定的设备数，每个条带一个。                                                                                                                                                                                                       |

这个示例步骤创建一个名为 `mylv` 的 LVM RAID0 逻辑卷，该逻辑卷可在 `/dev/sda1`、`/dev/sdb1` 和 `/dev/sdc1` 磁盘间条状分布数据。

1. 使用 `pvcreate` 命令将卷组中使用的磁盘标记为 LVM 物理卷。
   
   > **警告**
   > 
   > 此命令将销毁 `/dev/sda1`、`/dev/sdb1` 和 `/dev/sdc1` 上的任何数据。
   
   ```
   # pvcreate /dev/sda1 /dev/sdb1 /dev/sdc1
   Physical volume "/dev/sda1" successfully created
   Physical volume "/dev/sdb1" successfully created
   Physical volume "/dev/sdc1" successfully created
   ```

2. 创建卷组 `myvg`。以下命令将创建卷组 `myvg` ：
   
   ```
   # vgcreate myvg /dev/sda1 /dev/sdb1 /dev/sdc1
   Volume group "myvg" successfully created
   ```
   
    您可以使用 `vgs` 命令显示新卷组的属性。
   
   ```
   # vgs
     VG          #PV #LV #SN Attr   VSize   VFree
     myvg          1   1   0 wz--n-  <5.00g <4.51g
   ```

3. 从您创建的卷组中创建 RAID0 逻辑卷。以下命令从卷组 my vg 创建 RAID0 卷 my lv。这个示例创建的逻辑卷大小为 2GB，有三个条带，条带的大小为 4KB。
   
   ```
   # lvcreate --type raid0 -L 2G --stripes 3 --stripesize 4 -n mylv myvg
   Rounding size 2.00 GiB (512 extents) up to stripe boundary size 2.00 GiB(513 extents).
   Logical volume "mylv" created.
   ```

4. 在 RAID0 逻辑卷中创建文件系统。下面的命令在逻辑卷中创建 `ext4` 文件系统。
   
   ```
   # mkfs.ext4 /dev/myvg/mylv
   Creating filesystem with 512000 1k blocks and 128016 inodes
   Filesystem UUID: bdaf6199-d5be-4bc2-aa80-b8d402570fbe
   Superblock backups stored on blocks:
           8193, 24577, 40961, 57345, 73729, 204801, 221185, 401409
   
   Allocating group tables: done
   Writing inode tables: done
   Creating journal (8192 blocks): done
   Writing superblocks and filesystem accounting information: done
   ```
   
    下面的命令挂载逻辑卷并报告文件系统磁盘空间用量。
   
   ```
   # mount /dev/myvg/mylv /mnt
   
   # df -h
   Filesystem                    Size  Used Avail Use% Mounted on
   
   /dev/mapper/myvg-mylv         477M  2.3M  445M   1% /mnt
   ```

## 5.5 重命名 LVM 逻辑卷

这个步骤描述了如何将现有逻辑卷 *mylv* 重命名为 *mylv1*。

**流程**

1. 如果逻辑卷当前已挂载，卸载该卷：
   
   ```
   # umount /mnt
   ```
   
   使用挂载点替换 /mnt。

2. 如果在集群环境中存在逻辑卷，则在所有其激活的节点上取消激活逻辑卷。对每个这样的节点运行以下命令：
   
   ```
   # lvchange --activate n myvg/mylv
   ```

3. 重命名现有逻辑卷：
   
   ```
   # lvrename myvg mylv mylv1
   Logical volume "mylv" successfully renamed to "mylv1"
   ```
   
    您还可以通过指定设备的完整路径来重命名逻辑卷：
   
   ```
   # lvrename /dev/myvg/mylv /dev/myvg/mylv1
   ```

## 5.6 从逻辑卷中删除磁盘

这个步骤描述了如何从现有逻辑卷中删除磁盘，替换磁盘或者将磁盘用作不同卷的一部分。

要删除磁盘，您必须首先将 LVM 物理卷中的扩展移动到不同的磁盘或者一组磁盘中。

**流程**

1. 在使用 LV 时查看物理卷的已用和可用空间：
   
   ```
   # pvs -o+pv_used
     PV         VG          Fmt  Attr PSize   PFree  Used
   /dev/vdb1  new_vg      lvm2 a--   <5.00g  4.79g 208.00m
   /dev/vdb2  myvg        lvm2 a--   <5.00g <4.51g 500.00m
   /dev/vdb3  yourvg      lvm2 a--   <5.00g <5.00g      0
   ```

2. 将数据移到其他物理卷中：
   
   1. 如果现有卷组中的其他物理卷中有足够的可用扩展，请使用以下命令移动数据：
      
      ```
      # pvmove /dev/vdb3
      /dev/vdb3: Moved: 2.0%
      ...
      /dev/vdb3: Moved: 79.2%
      ...
      /dev/vdb3: Moved: 100.0%
      ```
   
   2. 如果现有卷组中的其他物理卷上没有足够的可用扩展，请使用以下命令来添加新物理卷，使用新创建的物理卷扩展卷组，并将数据移动到此物理卷中：
      
      ```
      # pvcreate /dev/vdb4
      Physical volume "/dev/vdb4" successfully created
      
      # vgextend myvg /dev/vdb4
      Volume group "myvg" successfully extended
      
      # pvmove /dev/vdb3 /dev/vdb4
      /dev/vdb3: Moved: 33.33%
      /dev/vdb3: Moved: 100.00%
      ```

3. 删除物理卷：
   
   ```
   # vgreduce myvg /dev/vdb3
   Removed "/dev/vdb3" from volume group "myvg"
   ```
   
   如果逻辑卷包含失败的物理卷，您就无法使用该逻辑卷。要从卷组中删除缺少的物理卷，如果缺少的物理卷上没有分配逻辑卷，您可以使用 `vgreduce` 命令的 `--removemissing` 参数：
   
   ```
   # vgreduce --removemissing myvg
   ```

## 5.7 删除 LVM 逻辑卷

此流程描述了如何从卷组 *myvg* 中删除现有逻辑卷 */dev/myvg/mylv1*。

**流程**

1. 如果逻辑卷当前已挂载，卸载该卷：
   
   ```
   # umount /mnt
   ```

2. 如果在集群环境中存在逻辑卷，则在所有其激活的节点上取消激活逻辑卷。对每个这样的节点运行以下命令：
   
   ```
   # lvchange --activate n vg-name/lv-name
   ```

3. 使用 `lvremove` 工具删除逻辑卷：
   
   ```
   # lvremove /dev/myvg/mylv1
   
   Do you really want to remove active logical volume "mylv1"? [y/n]: y
   Logical volume "mylv1" successfully removed
   ```
   
   > **注意**
   > 
   > 在这种情况下，逻辑卷还没有被取消激活。如果您在删除逻辑卷前明确取消激活了逻辑卷，则无法看到验证您是否要删除活跃逻辑卷的提示信息。

## 5.8 配置持久的设备号码

在载入模块的时候会自动分配主设备号码和副设备号码。如果总是使用相同的设备（主和副）号码激活块设备，有些应用程序效果最好。您可以使用以下参数通过 `lvcreate` 和 `lvchange` 命令指定这些：

```
--persistent y --major major --minor minor
```

使用大的副号码以确定还没有动态分配给另一个设备。

如果您使用 NFS 导出文件系统，在 exports 文件中指定 `fsid` 参数可能会不需要在 LVM 中设置持久的设备号码。

## 5.9 指定 LVM 扩展大小

当使用物理卷创建卷组时，默认情况下它的磁盘空间被分成 4MB 扩展。这个扩展是增大或者减小逻辑卷容量的最小值。大量的扩展不会影响逻辑卷的 I/O 性能。

如果默认扩展大小不合适，您可以使用 `vgcreate` 命令的 `-s` 选项指定扩展大小。您可以使用 `vgcreate` 命令的 `-p` 和 `-l` 参数限制卷组的物理或逻辑卷数量。

## 5.10 使用系统角色管理 LVM 逻辑卷

本节论述了如何应用 `storage` 角色来执行以下任务：

- 在由多个磁盘组成的卷组中创建 LVM 逻辑卷。
- 在逻辑卷中创建一个带给定标签的 ext4 文件系统。
- 永久挂载 ext4 文件系统。

**前提条件**

- 包含 `storage` 角色的 Ansible playbook

### 5.10.1 管理逻辑卷的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色在卷组中创建 LVM 逻辑卷。

> **例 5.1 在 myvg 卷组中创建 mylv 逻辑卷的 playbook**
> 
> ```
> - hosts: all
>   vars:
>     storage_pools:
>       - name: myvg
>         disks:
>           - sda
>           - sdb
>           - sdc
>         volumes:
>           - name: mylv
>             size: 2G
>             fs_type: ext4
>             mount_point: /mnt/data
>   roles:
>     - rhel-system-roles.storage
> ```
> 
> - `myvg` 卷组由以下磁盘组成：
>   
>   - /dev/sda
>   - /dev/sdb
>   - /dev/sdc
> 
> - 如果 `myvg` 卷组已存在，则 playbook 会将逻辑卷添加到卷组。
> 
> - 如果 `myvg` 卷组不存在，则 playbook 会创建该组。
> 
> - playbook 在 `mylv` 逻辑卷上创建 Ext4 文件系统，并在 `/mnt` 上持久挂载文件系统。

## 5.11 删除 LVM 卷组

这个步骤描述了如何删除现有卷组。

**前提条件**

- 卷组没有包含逻辑卷。要从卷组中删除逻辑卷，请参阅[删除 LVM 逻辑卷](#57-删除-lvm-逻辑卷)。

**流程**

1. 如果卷组存在于集群的环境中，在所有节点上停止卷组的锁定空间。在除您要删除的节点外的所有节点上使用以下命令：
   
   ```
   # vgchange --lockstop vg-name
   ```
   
   等待锁定停止。

2. 删除卷组：
   
   ```
   # vgremove vg-name
   Volume group "vg-name" successfully removed
   ```

# 第 6 章 修改逻辑卷的大小

创建逻辑卷后，您可以修改卷的大小。

## 6.1 增大逻辑卷和文件系统

这个步骤描述了如何扩展逻辑卷并在同一逻辑卷中增大文件系统。

要增大逻辑卷的大小，请使用 `lvextend` 命令。当扩展逻辑卷时，可以指定您想要增大的量，或者指定扩展它需要达到的大小。

**前提条件**

1. 您有一个现有逻辑卷(LV)，其中包含一个文件系统。使用 `df -Th` 命令确定文件系统类型。
   
   有关创建 LV 和文件系统的更多信息，请参阅[创建 LVM 逻辑卷](#53-创建-lvm-逻辑卷)。

2. 卷组中有足够的空间来扩展 LV 和文件系统。使用 `vgs -o name`,`vgfree` 命令确定可用空间。

**流程**

1. 可选：如果卷组没有足够的空间来扩展 LV，请使用以下命令向卷组中添加一个新物理卷：
   
   ```
   # vgextend myvg /dev/vdb3
   Physical volume "/dev/vdb3" successfully created.
   Volume group "myvg" successfully extended
   ```
   
    如需更多信息，请参阅[创建 LVM 卷组](#53-创建-lvm-逻辑卷)。

2. 现在卷组足够大，根据您的要求执行以下步骤：
   
   1. 要使用提供的大小扩展 LV，请使用以下命令：
      
      ```
      # lvextend -L 3G /dev/myvg/mylv
      Size of logical volume myvg/mylv changed from 500.00 MiB (125 extents) to 3.00 GiB (768 extents).
      Logical volume myvg/mylv successfully resized.
      ```
      
      > **注意**
      > 
      > 您可以使用 `lvextend` 命令的 `-r` 选项扩展逻辑卷并通过单个命令重新定义基础文件系统大小：
      > 
      > ```
      > # lvextend -r -L 3G /dev/myvg/mylv
      > ```
      
      > **警告**
      > 
      > 您还可以使用带有相同参数的 `lvresize` 命令扩展逻辑卷，但这个命令不能保证意外收缩。
   
   2. 要扩展 *mylv* 逻辑卷使其占据 *myvg* 卷组中所有未分配的空间，请使用以下命令：
      
      ```
      # lvextend -l +100%FREE /dev/myvg/mylv
        Size of logical volume myvg/mylv changed from 3.00 GiB (768 extents) to <5.00 GiB (1279 extents).
        Logical volume myvg/mylv successfully resized.
      ```
      
      与 `lvcreate` 命令一样，您可以使用 `lvextend` 命令的 `-l` 参数来指定扩展数目，从而增大逻辑卷的大小。您还可以使用此参数指定卷组的比例或者卷组中剩余空间的比例。

3. 如果您没有在 `lvextend` 命令中使用 `r` 选项来扩展 LV 并使用单个命令重新定义文件系统大小，请使用以下命令重新定义逻辑卷上的文件系统大小：
   
   ```
   xfs_growfs /mnt/mnt1/
   meta-data=/dev/mapper/myvg-mylv  isize=512    agcount=4, agsize=65536 blks
           =                       sectsz=512   attr=2, projid32bit=1
           =                       crc=1        finobt=1, sparse=1, rmapbt=0
           =                       reflink=1
   data     =                       bsize=4096   blocks=262144, imaxpct=25
           =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
   log      =internal log           bsize=4096   blocks=2560, version=2
           =                       sectsz=512   sunit=0 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 262144 to 524288
   ```

> **注意**
> 
> 如果没有 `-D` 选项，`xfs_growfs` 将文件系统增大到底层设备支持的最大大小。

**验证**

- 使用以下命令验证文件系统是否在增长：
  
  ```
  # df -Th
  Filesystem            Type      Size  Used Avail Use% Mounted 
  /dev/mapper/myvg-mylv xfs       2.0G   47M  2.0G   3% /mnt/mnt1
  ```

## 6.2 缩小逻辑卷

您可以使用 `lvreduce` 命令来减小逻辑卷的大小。

> **注意**
> 
> GFS2 或者 XFS 文件系统不支持缩小，因此您无法缩小包含 GFS2 或者 XFS 文件系统的逻辑卷大小。

如果您要缩小的逻辑卷包含一个文件系统，为了防止数据丢失，必须确定该文件系统没有使用将被缩小的逻辑卷中的空间。因此，建议您在逻辑卷包含文件系统时使用 `lvreduce` 命令的 `--resizefs` 选项。

`当您使用这个选项时，lvreduce` 命令会在缩小逻辑卷前尝试缩小文件系统。如果缩小文件系统失败，就像文件系统已满或者文件系统不支持缩小一样，则 lvreduce 命令将失败，且不会尝试缩小逻辑卷。

> **警告**
> 
> 在大多数情况下，`lvreduce` 命令会警告可能的数据丢失，并要求进行确认。但是，您不应该依赖于这些确认提示来防止数据丢失，因为在某些情况下，您不会看到这些提示信息，比如当逻辑卷不活跃或者没有使用 `--resizefs` 选项时。
> 
> 请注意，使用 `lvreduce` 命令的 `--test` 选项不指示操作是安全的，因为此选项不会检查文件系统或测试文件系统大小。

**流程**

- 要将 `myvg` 卷组中 `mylv` 逻辑卷缩小到 64MB，请使用以下命令：
  
  ```
  # lvreduce --resizefs -L 64M myvg/mylv
  fsck from util-linux 2.37.2
  /dev/mapper/myvg-mylv: clean, 11/25688 files, 4800/102400 blocks
  resize2fs 1.46.2 (28-Feb-2021)
  Resizing the filesystem on /dev/mapper/myvg-mylv to 65536 (1k) blocks.
  The filesystem on /dev/mapper/myvg-mylv is now 65536 (1k) blocks long.
  
  Size of logical volume myvg/mylv changed from 100.00 MiB (25 extents) to 64.00 MiB (16 extents).
  Logical volume myvg/mylv successfully resized.
  ```
  
    在本例中，*mylv* 包含一个文件系统，该命令可调整逻辑卷的大小。

- 在调整大小值前指定 `-` 符号表示该值将从逻辑卷的实际大小中减小。要将逻辑卷缩小到 64MB，请使用以下命令：
  
  ```
  # lvreduce --resizefs -L -64M myvg/mylv
  ```

## 6.3 扩展条状逻辑卷

要增加条状逻辑卷的大小，基本物理卷中必须有足够的可用空间，以便让卷组支持条带。例如，如果您有一个双向条带使用了整个卷组，那么向卷组中添加一个物理卷不会让您扩展条带。反之，您必须在卷组中添加至少两个物理卷。

例如，考虑由两个基础物理卷组成的卷组 `vg`，通过以下 `vgs` 命令显示：

    ```
    # vgs
    VG   #PV #LV #SN Attr   VSize   VFree
    vg     2   0   0 wz--n- 271.31G 271.31G
    ```

您可以使用整个卷组空间创建一个条带。

    ```
    # lvcreate -n stripe1 -L 271.31G -i 2 vg
    Using default stripesize 64.00 KB
    Rounding up size to full physical extent 271.31 GB
    Logical volume "stripe1" created
    # lvs -a -o +devices
    LV      VG   Attr   LSize   Origin Snap%  Move Log Copy%  Devices
    stripe1 vg   -wi-a- 271.31G                               /dev/sda1(0),/dev/sdb1(0)
    ```

请注意：卷组现在没有剩余空间。

    ```
    # vgs
    VG   #PV #LV #SN Attr   VSize   VFree
    vg     2   1   0 wz--n- 271.31G    0
    ```

下面的命令在卷组中添加了另一个物理卷，它提供了 135GB 的额外空间。

    ```
    # vgextend vg /dev/sdc1
    Volume group "vg" successfully extended
    # vgs
    VG   #PV #LV #SN Attr   VSize   VFree
    vg     3   1   0 wz--n- 406.97G 135.66G
    ```

此时您不能将条状逻辑卷扩展到卷组的大小，因为需要两个基本设备才可以对数据进行条带处理。

    ```
    # lvextend vg/stripe1 -L 406G
    Using stripesize of last segment 64.00 KB
    Extending logical volume stripe1 to 406.00 GB
    Insufficient suitable allocatable extents for logical volume stripe1: 34480
    more required
    ```

要扩展条状逻辑卷，添加另一个物理卷，然后扩展逻辑卷。在这个示例中，在卷组中添加两个物理卷，我们可将逻辑卷扩展成卷组的大小。

    ```
    # vgextend vg /dev/sdd1
    Volume group "vg" successfully extended
    # vgs
    VG   #PV #LV #SN Attr   VSize   VFree
    vg     4   1   0 wz--n- 542.62G 271.31G
    # lvextend vg/stripe1 -L 542G
    Using stripesize of last segment 64.00 KB
    Extending logical volume stripe1 to 542.00 GB
    Logical volume stripe1 successfully resized
    ```

如果您没有足够的底层物理设备来扩展条状逻辑卷，那么即使扩展没有条带化也可能会扩展卷，这可能会导致性能下降。当在逻辑卷中添加空间时，默认操作是使用与现有逻辑卷最新片段相同的条状参数，但您可以覆盖这些参数。以下示例在初始 `lvextend` 命令失败后扩展了现有条状逻辑卷以使用剩余空间。

    ```
    # lvextend vg/stripe1 -L 406G
    Using stripesize of last segment 64.00 KB
    Extending logical volume stripe1 to 406.00 GB
    Insufficient suitable allocatable extents for logical volume stripe1: 34480
    more required
    # lvextend -i1 -l+100%FREE vg/stripe1
    ```

# 第 7 章 LVM 的自定义报告

LVM 提供了广泛的配置和命令行选项来生成自定义报告，并过滤报告输出。有关 LVM 报告功能和功能的完整描述请查看 `lvmreport(7)man` page。

您可以使用 `pvs`、`lvs` 和 `vgs` 命令生成简洁、可自定义的 LVM 对象报告。这些命令生成的报告包括每行对象的输出结果。每行包含与对象相关的属性字段的有序列表。选择要报告的对象有五种方法：按物理卷、卷组、逻辑卷、物理卷片段和逻辑卷片段。

您可以使用 `lvm fullreport` 命令报告物理卷、卷组、逻辑卷、物理卷片段和逻辑卷片段的信息。有关此命令及其功能的详情，请查看 `lvm-fullreport(8)man` page。

LVM 支持日志报告，其中包含 LVM 命令执行过程中收集的完整对象识别操作、消息和每个对象状态的日志。有关 LVM 日志报告的详情，请查看 `lvmreport(7)man` page。

## 7.1 控制 LVM 显示的格式

无论您是使用 `pvs`、`lvs` 或 `vgs` 命令，都可以确定显示的默认字段集和排序顺序。您可以使用以下参数来控制这些命令的输出结果：

- 您可以使用 `-o` 参数将显示哪些字段更改为默认字段。例如：以下命令只显示物理卷名称和大小。
  
  ```
  # pvs -o pv_name,pv_size
  PV PSize
  /dev/sdb1 17.14G
  /dev/sdc1 17.14G
  /dev/sdd1 17.14G
      ```
  ```

- 您可以在输出中附加加号(+)的字段，该字段与 -o 参数结合使用。
  
    下面的例子除默认字段外还显示物理卷 UUID。
  
  ```
  # pvs -o +pv_uuid
  PV VG Fmt Attr PSize PFree PV UUID
  /dev/sdb1 new_vg lvm2 a- 17.14G 17.14G onFF2w-1fLC-ughJ-D9eB-M7iv-6XqA-dqGeXY
  /dev/sdc1 new_vg lvm2 a- 17.14G 17.09G Joqlch-yWSj-kuEn-IdwM-01S9-X08M-mcpsVe
  /dev/sdd1 new_vg lvm2 a- 17.14G 17.14G yvfvZK-Cf31-j75k-dECm-0RZ3-0dGW-UqkCS
  ```

- 将 `-v` 参数添加到命令包含一些额外的字段。例如，`pvs -v` 命令除默认字段外还会显示 `DevSize` 和 `PV UUID` 字段。
  
  ```
  # pvs -v
  Scanning for physical volume names
  PV VG Fmt Attr PSize PFree DevSize PV UUID
  /dev/sdb1 new_vg lvm2 a- 17.14G 17.14G 17.14G onFF2w-1fLC-ughJ-D9eB-M7iv-6XqA-dqGeXY
  /dev/sdc1 new_vg lvm2 a- 17.14G 17.09G 17.14G Joqlch-yWSj-kuEn-IdwM-01S9-XO8M-mcpsVe
  /dev/sdd1 new_vg lvm2 a- 17.14G 17.14G 17.14G yvfvZK-Cf31-j75k-dECm-0RZ3-0dGW-tUqkCS
  ```

- `--noheadings` 参数会抑制标题行。这对编写脚本非常有用。
  
    以下示例将 `--noheadings` 参数与 `pv_name` 参数结合使用，该参数将生成所有物理卷的列表。
  
  ```
  # pvs --noheadings -o pv_name
  /dev/sdb1
  /dev/sdc1
  /dev/sdd1
  ```

- 分隔符参数使用 `--separator` 来分隔各个字段。
  
    以下示例使用等号(=)分隔 pvs 命令的默认输出字段。
  
  ```
  # pvs --separator =
  PV=VG=Fmt=Attr=PSize=PFree
  /dev/sdb1=new_vg=lvm2=a-=17.14G=17.14G
  /dev/sdc1=new_vg=lvm2=a-=17.14G=17.09G
  /dev/sdd1=new_vg=lvm2=a-=17.14G=17.14G
  ```
  
    要在使用 `separator` 参数时让字段保持一致，请将 `--separator` 参数与 `--aligned` 参数 结合使用。
  
  ```
  # pvs --separator = --aligned
  PV =VG =Fmt =Attr=PSize =PFree
  /dev/sdb1 =new_vg=lvm2=a- =17.14G=17.14G
  /dev/sdc1 =new_vg=lvm2=a- =17.14G=17.09G
  /dev/sdd1 =new_vg=lvm2=a- =17.14G=17.14G
  ```

您可以使用 `lvs` 或 `vgs` 命令的 `-P` 参数显示否则不会出现在输出中的失败卷的信息。

有关显示参数的完整列表，请参见 `pvs(8)`、`vgs(8)`和 `lvs(8)man` page。

卷组字段可以与物理卷（和物理卷片段）字段或逻辑卷（和逻辑卷片段）字段混合，但是无法混合物理卷和逻辑卷字段。例如：以下命令可显示每行物理卷的输出结果。

```
# vgs -o +pv_name
  VG     #PV #LV #SN Attr   VSize  VFree  PV
  new_vg   3   1   0 wz--n- 51.42G 51.37G /dev/sdc1
  new_vg   3   1   0 wz--n- 51.42G 51.37G /dev/sdd1
  new_vg   3   1   0 wz--n- 51.42G 51.37G /dev/sdb1
```

## 7.2 LVM 对象显示字段

本节提供了一系列表，它们列出了您可以使用 `pvs`、`vgs` 和 `lvs` 命令显示的 LVM 对象信息。

为方便起见，字段名称前缀如果与命令的默认名称匹配就可以省略。例如，使用 `pvs` 命令时，name 表示 `pv_name`，但是使用 `vgs` 命令时，name 将 解释为 `vg_name`。

执行以下命令等同于执行 `pvs -o pv_free`。

```
# pvs -o free
PFree
17.14G
17.09G
17.14G
```

> **注意**
> 
> `pvs`、`vgs` 和 `lvs` 输出中属性字段中的字符数可能会增加。现有字符字段不会改变位置，但可能会在末尾添加新字段。在编写搜索特定属性字符的脚本时，您应该考虑这一点，根据其相对于字段开头的位置搜索字符，而不是其与字段末尾的相对位置。例如，要在 `lv_attr` 字段的第九个位中搜索字符 p，您可以搜索字符串 "^/…​…​..p/"，但您不应该搜索字符串 "/*p$/"。

[表 7.1 pvs 命令显示字段] 列出 `pvs` 命令的 显示参数，以及它在标头显示中的字段名称和字段描述。

**表 7.1 pvs 命令显示字段**

| **参数**              | **标头**  | **描述**                            |
| ------------------- | ------- | --------------------------------- |
| `dev_size`          | DevSize | 创建物理卷的基本设备的大小                     |
| `pe_start`          | 1st PE  | 在基础设备中调整到第一个物理扩展的起始位置             |
| `pv_attr`           | Attr    | 物理卷状态：(a)llocatable 或 e(x)ported. |
| `pv_fmt`            | Fmt     | 物理卷的元数据格式（`lvm2` 或 `lvm1`）        |
| `pv_free`           | PFree   | 物理卷中剩余的可用空间                       |
| `pv_name`           | PV      | 物理卷名称                             |
| `pv_pe_alloc_count` | Alloc   | 已使用的物理扩展数目                        |
| `pv_pe_count`       | PE      | 物理扩展数目                            |
| `pvseg_size`        | SSize   | 物理卷的片段大小                          |
| `pvseg_start`       | Start   | 物理卷片段的起始物理扩展                      |
| `pv_size`           | PSize   | 物理卷的大小                            |
| `pv_tags`           | PV 标签   | 附加到物理卷的 LVM 标签                    |
| `pv_used`           | Used    | 目前物理卷中已经使用的空间量                    |
| `pv_uuid`           | PV UUID | 物理卷的 UUID                         |

pvs 命令默认显示以下字段：`pv_name`、`vg_name`、`pv_fmt`、`pv_attr`、`pv_size`、`pv_free`。显示根据 `pv_name` 进行排序。

```
# pvs
  PV         VG     Fmt  Attr PSize  PFree
  /dev/sdb1  new_vg lvm2 a-   17.14G 17.14G
  /dev/sdc1  new_vg lvm2 a-   17.14G 17.09G
  /dev/sdd1  new_vg lvm2 a-   17.14G 17.13G
```

将 `-v` 参数与 `pvs` 命令一起使用会将以下字段添加到默认显示中：`dev_size`、`pv_uuid`。

```
# pvs -v
    Scanning for physical volume names
  PV         VG     Fmt  Attr PSize  PFree  DevSize PV UUID
  /dev/sdb1  new_vg lvm2 a-   17.14G 17.14G  17.14G onFF2w-1fLC-ughJ-D9eB-M7iv-6XqA-dqGeXY
  /dev/sdc1  new_vg lvm2 a-   17.14G 17.09G  17.14G Joqlch-yWSj-kuEn-IdwM-01S9-XO8M-mcpsVe
  /dev/sdd1  new_vg lvm2 a-   17.14G 17.13G  17.14G yvfvZK-Cf31-j75k-dECm-0RZ3-0dGW-tUqkCS
```

您可以使用 `pvs` 命令的 `--segments` 参数显示每个物理卷段的信息。一个片段就是一组扩展。查看片段在想查看逻辑卷是否碎片时很有用。

`pvs --segments` 命令默认显示以下字段：`pv_name`、`vg_name`、`pv _fmt`、`pv_attr`、`pv_size`、`pv_free`、`pvseg_start`、`pvseg_size`。显示在物理卷中根据 `pv_name` 和 `pvseg_size` 进行排序。

```
# pvs --segments
  PV         VG         Fmt  Attr PSize  PFree  Start SSize
  /dev/hda2  VolGroup00 lvm2 a-   37.16G 32.00M     0  1172
  /dev/hda2  VolGroup00 lvm2 a-   37.16G 32.00M  1172    16
  /dev/hda2  VolGroup00 lvm2 a-   37.16G 32.00M  1188     1
  /dev/sda1  vg         lvm2 a-   17.14G 16.75G     0    26
  /dev/sda1  vg         lvm2 a-   17.14G 16.75G    26    24
  /dev/sda1  vg         lvm2 a-   17.14G 16.75G    50    26
  /dev/sda1  vg         lvm2 a-   17.14G 16.75G    76    24
  /dev/sda1  vg         lvm2 a-   17.14G 16.75G   100    26
  /dev/sda1  vg         lvm2 a-   17.14G 16.75G   126    24
  /dev/sda1  vg         lvm2 a-   17.14G 16.75G   150    22
  /dev/sda1  vg         lvm2 a-   17.14G 16.75G   172  4217
  /dev/sdb1  vg         lvm2 a-   17.14G 17.14G     0  4389
  /dev/sdc1  vg         lvm2 a-   17.14G 17.14G     0  4389
  /dev/sdd1  vg         lvm2 a-   17.14G 17.14G     0  4389
  /dev/sde1  vg         lvm2 a-   17.14G 17.14G     0  4389
  /dev/sdf1  vg         lvm2 a-   17.14G 17.14G     0  4389
  /dev/sdg1  vg         lvm2 a-   17.14G 17.14G     0  4389
```

您可以使用 `pvs -a` 命令查看 LVM 检测到的设备，这些设备尚未初始化为 LVM 物理卷。

```
# pvs -a
  PV                             VG     Fmt  Attr PSize  PFree
  /dev/VolGroup00/LogVol01                   --       0      0
  /dev/new_vg/lvol0                          --       0      0
  /dev/ram                                   --       0      0
  /dev/ram0                                  --       0      0
  /dev/ram2                                  --       0      0
  /dev/ram3                                  --       0      0
  /dev/ram4                                  --       0      0
  /dev/ram5                                  --       0      0
  /dev/ram6                                  --       0      0
  /dev/root                                  --       0      0
  /dev/sda                                   --       0      0
  /dev/sdb                                   --       0      0
  /dev/sdb1                      new_vg lvm2 a-   17.14G 17.14G
  /dev/sdc                                   --       0      0
  /dev/sdc1                      new_vg lvm2 a-   17.14G 17.09G
  /dev/sdd                                   --       0      0
  /dev/sdd1                      new_vg lvm2 a-   17.14G 17.14G
```

[表 7.2 vgs 显示字段] 列出 `vgs` 命令的显示参数，以及在标题显示和字段描述中显示的字段名称。

**表 7.2 vgs 显示字段**

| **参数**            | **标头**        | **描述**                                                                     |
| ----------------- | ------------- | -------------------------------------------------------------------------- |
| `lv_count`        | #LV           | 卷组包含的逻辑卷数                                                                  |
| `max_lv`          | MaxLV         | 卷组中最多允许的逻辑卷数目（如果没有限制就是 0）                                                  |
| `max_pv`          | MaxPV         | 卷组中最多允许的物理卷数目（如果没有限制就是 0）                                                  |
| `pv_count`        | #PV定义卷组的物理卷数目 |                                                                            |
| `snap_count`      | #SN           | 卷组包含的快照数                                                                   |
| `vg_attr`         | Attr          | 卷组状态：(w)riteable、(r)eadonly、resi(z)able、e(x)ported、(p)artial 和(c)lustered。 |
| `vg_extent_count` | #Ext          | 卷组中的物理扩展数目                                                                 |
| `vg_extent_size`  | Ext           | 卷组中物理扩展的大小                                                                 |
| `vg_fmt`          | Fmt           | 卷组的元数据格式（lvm2 或 lvm 1）                                                     |
| `vg_free`         | vfree         | 卷组中剩余可用空间大小                                                                |
| `vg_free_count`   | Free          | 卷组中可用物理扩展数目                                                                |
| `vg_name`         | VG            | 卷组名称                                                                       |
| `vg_seqno`        | seq           | 代表修正卷组的数                                                                   |
| `vg_size`         | VSize         | 卷组大小                                                                       |
| `vg_sysid`        | SYS ID        | LVM1 系统 ID                                                                 |
| `vg_tags`         | VG Tags       | 附加到卷组中的 LVM 标签                                                             |
| `vg_uuid`         | VG UUID       | 卷组的 UUID                                                                   |

`vgs` 命令默认显示以下字段： `vg_name`、`pv_count`、`lv_count`、`snap_count`、`vg_attr`、`vg_size`、`vg_free`。显示按 `vg_name` 进行排序。

```
# vgs
  VG     #PV #LV #SN Attr   VSize  VFree
  new_vg   3   1   1 wz--n- 51.42G 51.36G
```

将 `-v` 参数与 `vgs` 命令一起使用会将以下字段添加到默认显示中： `vg_extent_size`、`vg_uuid`。

```
# vgs -v
    Finding all volume groups
    Finding volume group "new_vg"
  VG     Attr   Ext   #PV #LV #SN VSize  VFree  VG UUID
  new_vg wz--n- 4.00M   3   1   1 51.42G 51.36G jxQJ0a-ZKk0-OpMO-0118-nlwO-wwqd-fD5D32
```

[表 7.3 LVS 显示字段] 列出 `lvs` 命令的显示参数，以及在标头显示和字段描述中显示的字段名称。

> **注意**
> 
> 在以后的 OpenCloudOS 发行版中，`lvs` 命令的输出结果可能有所不同，输出中的其他字段可能会有所不同。但是，这些字段的顺序将保持不变，所有附加字段也会在显示的末尾出现。

**表 7.3 LVS 显示字段**

| **参数**                        | **标头**      | **描述**                                 |
| ----------------------------- | ----------- | -------------------------------------- |
| * C `HUNKSIZE` * `chunk_size` | Chunk       | 快照卷的单位大小                               |
| `copy_percent`                | Copy%       | 镜像逻辑卷的同步百分比 ; 还在使用 pv_move 命令移动物理扩展时使用 |
| `devices`                     | Devices     | 组成逻辑卷的基本设备：物理卷、逻辑卷以及启动物理扩展和逻辑扩展        |
| `lv_ancestors`                | Ancestors   | 对于精简池快照，逻辑卷的祖先                         |
| `lv_descendants`              | Descendants | 对于精简池快照，逻辑卷的后代                         |
| `lv_attr`                     | Attr        | 逻辑卷的状态。逻辑卷属性字节如下：                      |

* Bit 1: 卷类型： (m)irrored, (M)irrored without initial sync, (o)rigin, (O)rigin with merging snapshot, (r)aid, ®aid without initial sync, (s)napshot, merging (S)napshot, (p)vmove, (v)irtual, mirror or raid (i)mage, mirror or raid (I)mage out-of-sync, mirror (l)og device, under (c)onversion, thin (V)olume, (t)hin pool, (T)hin pool data, raid or thin pool m(e)tadata or pool metadata spare,
* bit 2: Permissions:(w)riteable,(r)ead-only-only of non-read-only of non-read-only volume
* bit 3：分配策略：(a)nywhere,(c)相邻、(i)herited、c(l)ing、(n)ormal。如果当前根据分配更改锁定卷，这将会大写，例如在执行 `pvmove` 命令时。
* bit 4：固定(m)inor
* bit 5: State:(a)ctive,(s)uspended,(I)nvalid snapshot, invalid(S)uspended snapshot, snapshot(m)erge failed, pause snapshot(M)erge failed, mapping(d)evice 显示没有表，映射设备带有(i)nactive 表
* 位 6：设备(o)pen
* bit 7: Target type:(m)irror,(r)aid,(s)napshot,(t)hin,(u)nknown,(v)irtual.这组逻辑卷同时与同一内核目标相关。因此，如果它们使用原始的 device-mapper mirror 内核驱动程序，则镜像日志、镜像日志以及镜像本身会显示(m),而 raid 等同于使用 md raid 内核驱动程序。使用原始 device-mapper 驱动程序的快照显示为(s)，而使用精简配置驱动程序的精简卷快照则显示为(t)。
* 第 8 位：新分配的数据块在使用前被(z)eroes 块覆盖。
* 第 9 位：卷健康：(p)artial、(r)efresh 需要、(m)ismatches 存在(w)rite。(p)图代表系统中缺少一个或多个物理卷。(r)efresh 表示 RAID 逻辑卷使用的一个或多个物理卷存在写入错误。写入错误可能是由物理卷临时失败造成的，或表示它已经失败。该设备应该被刷新或替换。(m)ismatches 表示 RAID 逻辑卷有部分不一致的阵列。通过在 RAID 逻辑卷中启动 检查 操作可发现不一致的问题。（可以使用 `lvchange` 命令在 RAID 逻辑卷上执行清理操作、检查 和修复。）(w)ritely 表示 RAID 1 逻辑卷中标记的大多数设备.
* Bit 10: s(k)ip activation: 这个卷被标记为在激活的过程中跳过它。|
  |`lv_kernel_major`|KMaj|逻辑卷的真实主设备号码（如果不活跃则为 -1）|
  |`lv_kernel_minor`|KMI|逻辑卷的真实从设备号码（如果是不活跃则为 -1）|
  |`lv_major`|Maj|逻辑卷持久的主设备号码（如果未指定则为 -1）|
  |`lv_minor`|Min|逻辑卷持久的从设备号（如果未指定则为 -1）|
  |`lv_name`|LV|逻辑卷名称|
  |`lv_size`|LSize|逻辑卷的大小|
  |`lv_tags`|LV Tags|附加到逻辑卷的 LVM 标签|
  |`lv_uuid`|LV UUID|逻辑卷的 UUID。|
  |`mirror_log`|Log|镜像日志所在的设备|
  |`modules`|Modules|使用这个逻辑卷所需的相应内核设备映射器目标|
  |`move_pv`|Move|用 `pvmove` 命令创建的临时逻辑卷的源 物理卷|
  |`Origin`|Origin|快照卷的源设备|
  |* `regionsize`
* `region_size`|Region|镜像的逻辑卷的单元大小|
  |`seg_count`|#Seg|逻辑卷中片段的数|
  |`seg_size`|SSize|逻辑卷中片段的大小|
  |`seg_start`|Start|逻辑卷中片段的偏移|
  |`seg_tags`|Seg Tags|附加到逻辑卷片段的 LVM 标签|
  |`segtype`|Type|逻辑卷的片段类型（例如：镜像、条状、线性）|
  |`snap_percent`|Snap%|已使用的快照卷的比例|
  |`stripes`|#Str|逻辑卷中条带或者镜像的数目|
  |* `stripesize`
* `stripe_size`|Stripe|条状逻辑卷中条状逻辑卷的单元大小|

默认情况下，`lvs` 命令提供以下显示：默认显示按照卷组中的 `vg_name` 和 `lv_name` 进行排序。

```
# lvs
  LV     VG              Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  origin VG              owi-a-s---    1.00g
  snap   VG              swi-a-s---  100.00m      origin 0.00
```

`lvs` 命令的一个常见用途是将 设备 附加到命令，以显示组成逻辑卷的基础设备。这个示例还指定 `-a` 选项来显示作为逻辑卷组件的内部卷，如 RAID 镜像，并用括号括起来。这个示例包括 RAID 卷、条状卷和一个精简池卷。

```
# lvs -a -o +devices
  LV               VG            Attr       LSize   Pool   Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices
  raid1            VG            rwi-a-r---   1.00g                                      100.00           raid1_rimage_0(0),raid1_rimage_1(0)
  [raid1_rimage_0] VG            iwi-aor---   1.00g                                                       /dev/sde1(7041)
  [raid1_rimage_1] VG            iwi-aor---   1.00g                                                       /dev/sdf1(7041)
  [raid1_rmeta_0]  VG            ewi-aor---   4.00m                                                       /dev/sde1(7040)
  [raid1_rmeta_1]  VG            ewi-aor---   4.00m                                                       /dev/sdf1(7040)
  stripe1          VG            -wi-a-----  99.95g                                                       /dev/sde1(0),/dev/sdf1(0)
  stripe1          VG            -wi-a-----  99.95g                                                       /dev/sdd1(0)
  stripe1          VG            -wi-a-----  99.95g                                                       /dev/sdc1(0)
  [lvol0_pmspare]  opencloudos_host-083 ewi-------   4.00m                                                       /dev/vda2(0)
  pool00           opencloudos_host-083 twi-aotz--  <4.79g               72.90  54.69                            pool00_tdata(0)
  [pool00_tdata]   opencloudos_host-083 Twi-ao----  <4.79g                                                       /dev/vda2(1)
  [pool00_tmeta]   opencloudos_host-083 ewi-ao----   4.00m                                                       /dev/vda2(1226)
  root             opencloudos_host-083 Vwi-aotz--  <4.79g pool00        72.90
  swap             opencloudos_host-083 -wi-ao---- 820.00m                                                       /dev/vda2(1227)
```

将 `-v` 参数与 `lvs` 命令一起使用会将以下字段添加到默认显示中：`seg_count`、`lv_ major`、`lv_ minor`、`lv_kernel_major`、`lv_kernel_ minor`、`lv_kernel_minor`、`lv_uuid`。

```
# lvs -v
    Finding all logical volumes
  LV         VG     #Seg Attr   LSize  Maj Min KMaj KMin Origin Snap%  Move Copy%  Log Convert LV UUID
  lvol0      new_vg    1 owi-a- 52.00M  -1  -1 253  3                                          LBy1Tz-sr23-OjsI-LT03-nHLC-y8XW-EhCl78
  newvgsnap1 new_vg    1 swi-a-  8.00M  -1  -1 253  5    lvol0    0.20                         1ye1OU-1cIu-o79k-20h2-ZGF0-qCJm-CfbsIx
```

您可以使用 `lvs` 命令的 `--segments` 参数来显示重点介绍网段信息的默认列的信息。使用 `segment` 参数 时，`seg` 前缀是可选的。`lvs --segments` 命令默认显示以下字段：`lv_name`、`vg_name`、`lv_attr`、`stripes`、`segtype`、`seg_size`。默认显示按照卷组中的 `vg_name`、`lv_name` 和逻辑卷中的 `seg_start` 进行排序。如果逻辑卷有碎片化的问题，这个命令的输出会显示相关信息。

```
# lvs --segments
  LV       VG         Attr   #Str Type   SSize
  LogVol00 VolGroup00 -wi-ao    1 linear  36.62G
  LogVol01 VolGroup00 -wi-ao    1 linear 512.00M
  lv       vg         -wi-a-    1 linear 104.00M
  lv       vg         -wi-a-    1 linear 104.00M
  lv       vg         -wi-a-    1 linear 104.00M
  lv       vg         -wi-a-    1 linear  88.00M
```

将 `-v` 参数与 `lvs --segments` 命令一起使用 会将以下字段添加到默认显示中：`seg_start`、`stripesize`、`blocksize`。

```
# lvs -v --segments
    Finding all logical volumes
  LV         VG     Attr   Start SSize  #Str Type   Stripe Chunk
  lvol0      new_vg owi-a-    0  52.00M    1 linear     0     0
  newvgsnap1 new_vg swi-a-    0   8.00M    1 linear     0  8.00K
```

以下示例显示了在配置了一个逻辑卷的系统上 `lvs` 命令的默认输出，后跟指定了 `segment` 参数的 `lvs` 命令的默认输出。

```
# lvs
  LV    VG     Attr   LSize  Origin Snap%  Move Log Copy%
  lvol0 new_vg -wi-a- 52.00M
# lvs --segments
  LV    VG     Attr   #Str Type   SSize
  lvol0 new_vg -wi-a-    1 linear 52.00M
```

## 7.3 LVM 报告排序

通常，`lvs`、`vgs` 或 `pvs` 命令的完整输出都必须在内部生成并存储，然后才能对该命令进行排序，并且列可以正确一致。您可以指定 `--unbuffered` 参数，以便在生成后马上显示未排序的输出。

要指定替代的列排序顺序列表，请使用任何报告命令的 `-O` 参数。在输出中不一定要包含这些字段。

以下示例显示了显示物理卷名称、大小和可用空间的 `pvs` 命令的输出结果。

```
# pvs -o pv_name,pv_size,pv_free
  PV         PSize  PFree
  /dev/sdb1  17.14G 17.14G
  /dev/sdc1  17.14G 17.09G
  /dev/sdd1  17.14G 17.14G
```

下面的例子显示相同的输出结果，根据可用空间字段排序。

```
# pvs -o pv_name,pv_size,pv_free -O pv_free
  PV         PSize  PFree
  /dev/sdc1  17.14G 17.09G
  /dev/sdd1  17.14G 17.14G
  /dev/sdb1  17.14G 17.14G
```

以下示例显示您无需显示正在排序的字段。

```
# pvs -o pv_name,pv_size -O pv_free
  PV         PSize
  /dev/sdc1  17.14G
  /dev/sdd1  17.14G
  /dev/sdb1  17.14G
```

要显示反向排序，请在字段前加上 `-O` 参数并使用 `-` 字符指定。

```
# pvs -o pv_name,pv_size,pv_free -O -pv_free
  PV         PSize  PFree
  /dev/sdd1  17.14G 17.14G
  /dev/sdb1  17.14G 17.14G
  /dev/sdc1  17.14G 17.09G
```

## 7.4 为 LVM 报告显示指定单位

要指定 LVM 报告显示的单元，请使用 report 命令的 `--units` 参数。您可以指定(b)ytes、(k)iB、(m)egabytes、(g)igabytes、(t)erabytes、(e)xabytes、(p)etabytes 和(h)uman- 可读。默认显示是人类可读的。您可以通过设置 `/etc/lvm/lvm.conf` 文件的 `global` 部分中的 `unit` 参数来 覆盖默认值。

以下示例以 MB 为单位指定 `pvs` 命令的输出结果，而不是默认的千兆字节。

```
# pvs --units m
  PV         VG     Fmt  Attr PSize     PFree
  /dev/sda1         lvm2 --   17555.40M 17555.40M
  /dev/sdb1  new_vg lvm2 a-   17552.00M 17552.00M
  /dev/sdc1  new_vg lvm2 a-   17552.00M 17500.00M
  /dev/sdd1  new_vg lvm2 a-   17552.00M 17552.00M
```

默认情况下，单位以 2 的幂（1024 的倍数）显示。您可以使用大写字母代表单位是 1000 的倍数（B、K、M、G、T、H）。

下面的命令以默认单位（1024 的倍数）显示输出结果。

```
# pvs
  PV         VG     Fmt  Attr PSize  PFree
  /dev/sdb1  new_vg lvm2 a-   17.14G 17.14G
  /dev/sdc1  new_vg lvm2 a-   17.14G 17.09G
  /dev/sdd1  new_vg lvm2 a-   17.14G 17.14G
```

下面的命令以 1000 的倍数为单位显示输出结果。

```
#  pvs --units G
  PV         VG     Fmt  Attr PSize  PFree
  /dev/sdb1  new_vg lvm2 a-   18.40G 18.40G
  /dev/sdc1  new_vg lvm2 a-   18.40G 18.35G
  /dev/sdd1  new_vg lvm2 a-   18.40G 18.40G
```

您也可以指定y (s)ectors（定义为 512 字节）或自定义单元。

以下示例将 `pvs` 命令的输出显示为多个扇区。

```
# pvs --units s
  PV         VG     Fmt  Attr PSize     PFree
  /dev/sdb1  new_vg lvm2 a-   35946496S 35946496S
  /dev/sdc1  new_vg lvm2 a-   35946496S 35840000S
  /dev/sdd1  new_vg lvm2 a-   35946496S 35946496S
```

以下示例以 4 MB 为单位显示了 `pvs` 命令的输出结果。

```
# pvs --units 4m
  PV         VG     Fmt  Attr PSize    PFree
  /dev/sdb1  new_vg lvm2 a-   4388.00U 4388.00U
  /dev/sdc1  new_vg lvm2 a-   4388.00U 4375.00U
  /dev/sdd1  new_vg lvm2 a-   4388.00U 4388.00U
```

## 7.5 以 JSON 格式显示 LVM 命令输出

您可以使用 LVM 显示命令的 `--reportformat` 选项以 JSON 格式显示输出结果。

以下示例以标准默认格式显示了 `lvs` 的输出。

```
# lvs
  LV      VG            Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  my_raid my_vg         Rwi-a-r---  12.00m                                    100.00
  root    opencloudos_host-075 -wi-ao----   6.67g
  swap    opencloudos_host-075 -wi-ao---- 820.00m
```

在指定 JSON 格式时，以下命令显示同一 LVM 配置的输出。

```
# lvs --reportformat json
  {
      "report": [
          {
              "lv": [
                  {"lv_name":"my_raid", "vg_name":"my_vg", "lv_attr":"Rwi-a-r---", "lv_size":"12.00m", "pool_lv":"", "origin":"", "data_percent":"", "metadata_percent":"", "move_pv":"", "mirror_log":"", "copy_percent":"100.00", "convert_lv":""},
                  {"lv_name":"root", "vg_name":"opencloudos_host-075", "lv_attr":"-wi-ao----", "lv_size":"6.67g", "pool_lv":"", "origin":"", "data_percent":"", "metadata_percent":"", "move_pv":"", "mirror_log":"", "copy_percent":"", "convert_lv":""},
                  {"lv_name":"swap", "vg_name":"opencloudos_host-075", "lv_attr":"-wi-ao----", "lv_size":"820.00m", "pool_lv":"", "origin":"", "data_percent":"", "metadata_percent":"", "move_pv":"", "mirror_log":"", "copy_percent":"", "convert_lv":""}
              ]
          }
      ]
  }
```

您还可以使用 `output_format` 设置，将报告格式设置为 `/etc/lvm/lvm.conf`文件中的配置选项。但是，命令行的 `--reportformat` 设置优先于此设置。

## 7.6 显示 LVM 命令日志

如果启用了 `log/report_command_log` 配置设置，则面向报告的 LVM 命令都可以报告命令日志。您可以确定要显示的一组字段，并为此报告排序。

以下示例将 LVM 配置为为 LVM 命令生成完整的日志报告。在这个示例中，您可以看到逻辑卷 `lvol0` 和 `lvol1` 都已成功处理，包含卷的卷组 `VG` 也是如此。

```
# lvmconfig --type full log/command_log_selection
command_log_selection="all"

# lvs
  Logical Volume
  ==============
  LV    LSize Cpy%Sync
  lvol1 4.00m 100.00
  lvol0 4.00m

  Command Log
  ===========
  Seq LogType Context    ObjType ObjName ObjGrp  Msg     Errno RetCode
    1 status  processing lv      lvol0   vg      success     0       1
    2 status  processing lv      lvol1   vg      success     0       1
    3 status  processing vg      vg              success     0       1

# lvchange -an vg/lvol1
  Command Log
  ===========
  Seq LogType Context    ObjType ObjName ObjGrp  Msg     Errno RetCode
    1 status  processing lv      lvol1   vg      success     0       1
    2 status  processing vg      vg              success     0       1
```

有关配置 LVM 报告和命令日志的详情请参考 `lvmreport` man page。

# 第 8 章 配置 RAID 逻辑卷

您可以创建、激活、更改、删除、显示和使用 LVM RAID 卷。

## 8.1 RAID 逻辑卷

LVM 支持 RAID 0、1、4、5、6 和 10。

LVM RAID 卷有以下特征：

- LVM 创建和管理的 RAID 逻辑卷利用多设备（MD）内核驱动程序。
- 您可以从阵列中临时分割 RAID1 镜像，并在之后将其合并到阵列中。
- LVM RAID 卷支持快照。

**集群**

RAID 逻辑卷不是集群感知型逻辑卷。

您可以只在一台机器中创建和激活 RAID 逻辑卷，但不能在多台机器中同时激活它们。

**子卷**

当您创建 RAID 逻辑卷时，LVM 会为阵列中的每个数据或奇偶校验子卷创建一个元数据子卷。

例如，创建一个双向 RAID1 阵列会导致两个元数据子卷（`lv_rmeta_0` 和 `lv_rmeta_1` ），以及两个数据子卷（`lv_rimage_0` 和 `lv_rimage_1`）。同样，创建三向条带（加 1 隐式奇偶校验设备）RAID4 会产生 4 个元数据子卷（`lv_rmeta_0`, `lv_rmeta_1`、`lv_rmeta_2` 和 `lv_rmeta_3`）和 4 个数据子卷（`lv_rimage_0`, `lv_rimage_1`、`lv_rimage_2` 和 `lv_rimage_3`）。
`
**完整性**

当 RAID 设备失败或者发生软崩溃时，可能会丢失数据。数据存储中的软崩溃意味着，从存储设备中检索的数据与写入到那个设备中的数据不同。在 RAID LV 中添加完整性有助于缓解或防止软崩溃。要了解更多有关软崩溃以及如何在 RAID LV 中添加完整性的信息，请参阅 [第 8.6 节 使用带有 RAID LV 的 DM 完整性功能](#86-使用带有-raid-lv-的-dm-完整性功能)。

## 8.2 RAID 级别和线性支持

RAID 支持各种配置，包括 0、1、4、5、6、10 和 linear。这些 RAID 类型定义如下：

**0 级**

RAID 级别 0，通常被称为条带化数据映射技术。这意味着，要写入阵列的数据被分成条块，并在阵列的成员磁盘中写入，这样可以在成本低的情况下提供高的 I/O 性能，但不提供冗余。

许多 RAID 级别 0 的实现只在成员设备间条状分布到阵列中最小设备的大小。就是说，如果您有多个设备，它们的大小稍有不同，那么每个设备的大小都被视为与最小设备的大小相同。因此，级别 0 阵列的一般存储容量等于，硬件 RAID 中容量最小的成员磁盘的容量，或软件 RAID 中的最小成员分区的容量，在乘以阵列中的磁盘或分区的数量。

**1 级**

RAID 级别 1 或称为镜像，通过将相同数据写入阵列的每个磁盘来提供冗余，在每个磁盘上保留"镜像"副本。因为其简单且数据高度可用，RAID 1 仍然被广泛使用。级别 1 需要两个或者多个磁盘，它提供了很好的数据可靠性，提高了需要读取的应用程序的性能，但是成本相对高。

为了实现数据可靠性，需要向阵列中的所有磁盘写入相同的信息，所以 RAID 1 的成本会很高。与基于奇偶校验的其他级别（如级别 5）相比，空间的利用效率较低。然而，对空间利用率的牺牲提供了高性能：基于奇偶校验的 RAID 级别会消耗大量 CPU 资源以便获得奇偶校验，而 RAID 级别 1 只是一次向多个 RAID 成员中写入同样数据，其对 CPU 的消耗较小。因此，在使用软件 RAID 的系统中，或系统中有其他操作需要大量使用 CPU 资源时，RAID 1 可能会比使用基于奇偶校验的 RAID 级别的性能更好。

级别 1 阵列的存储容量等于硬件 RAID 中最小镜像硬盘或者软件 RAID 中最小镜像分区的容量相同。级别 1 所提供的冗余性是所有 RAID 级别中最高的，因为阵列只需要在有一个成员可以正常工作的情况下就可以提供数据。

**级别 4**

级别 4 使用单一磁盘驱动器中的奇偶校验来保护数据。奇偶校验信息根据阵列中其余成员磁盘的内容计算。然后当阵列中的一个磁盘失败时，这个信息就可以被用来重建数据。然后，在出现问题的磁盘被替换前，使用被重建的数据就可以满足 I/O 的请求。在磁盘被替换后，可以在上面重新生成数据。

因为 RAID 4 使用一个专门的偶校验磁盘，因此这个磁盘就会成为对 RAID 阵列的写入操作的一个固有的瓶颈。所以， RAID 4 较少被使用。因此，Anaconda 中并没有提供 RAID 4 这个选项。但是，如果真正需要，用户可以手动创建它。

硬件 RAID 4 的存储容量等于分区数量减一乘以最小成员分区的容量。RAID 4 阵列的性能是非对称的，即读的性能会好于写的性能。这是因为，写入会在生成奇偶校验时消耗额外的 CPU 和主内存带宽，然后在将实际数据写入磁盘时也消耗额外的总线带宽，因为您不仅是写入数据，而且是奇偶校验。读取只需要读取数据而不是奇偶校验，除非该阵列处于降级状态。因此，在正常操作条件下，读取会在计算机的驱动器和总线间产生较少的流量，以实现相同数量的数据传输。

**5 级**

这是最常见的 RAID 类型。通过在一个阵列的所有成员磁盘中分布奇偶校验，RAID 5 解除了级别 4 中原有的写入瓶颈。唯一性能瓶颈是奇偶校验计算过程本身。在使用现代 CPU 和软件 RAID 时，这通常不会成为瓶颈，因为现代 CPU 可能会非常快速地生成奇偶校验。然而，如果您的软件 RAID5 阵列中有大量成员设备，且在所有设备间有大量的数据进行传输时，就可能出现瓶颈。

和级别 4 一样，级别 5 的性能也是非对称的，读性能会高于写的性能。RAID 5 的存储容量的计算方法与级别 4 的计算方法是一样的。

**级别 6**

如果数据的冗余性和保护性比性能更重要，且无法接受 RAID 1 的空间利用率低的问题，则通常会选择使用级别 6。级别 6 使用一个复杂的奇偶校验方式，可以在阵列中出现任意两个磁盘失败的情况下进行恢复。因为使用的奇偶校验方式比较复杂，软件 RAID 设备会对 CPU 造成较大负担，同时对写操作造成更大的负担。因此，与级别 4 和 5 相比，级别 6 的性能不对称性更严重。

RAID 6 阵列的总容量与 RAID 5 和 4 类似，但您必须从设备数量中减小两个（而不是 1 个）额外奇偶校验存储空间的设备数。

**级别 10**

这个 RAID 级别将级别 0 的性能优势与级别 1 的冗余合并。它还有助于减少在有多于 2 个设备时，级别 1 阵列中的利用率低的问题。在级别 10 中，可以创建一个 3 个驱动器阵列来只存储每个数据的 2 个副本，然后允许整个阵列的大小达到最小设备的 1.5 倍，而不是只等于最小设备（与有 3 个设备的级别 1 阵列相同）。与 RAID 级别 6 相比，计算奇偶校验对 CPU 的消耗较少，但空间效率较低。

在安装过程中，不支持创建 RAID 10。您可在安装后手动创建。

**线性 RAID**

线性 RAID 是创建更大的虚拟驱动器的一组驱动器。

在线性 RAID 中，块会被从一个成员驱动器中按顺序分配，只有在第一个完全填充时才会进入下一个驱动器。这个分组方法不会提供性能优势，因为 I/O 操作不太可能在不同成员间同时进行。线性 RAID 也不提供冗余性，并会降低可靠性。如果有任何一个成员驱动器失败，则无法使用整个阵列。该容量是所有成员磁盘的总量。

## 8.3 LVM RAID 片段类型

要创建 RAID 逻辑卷，请将 raid 类型指定为 `lvcreate` 命令的 `--type` 参数。下表描述了可能的 RAID 片段类型。

对于大多数用户，指定五个可用主类型之一（`raid1`、`raid4`、`raid5`、`raid6`、`raid10`）应已足够。

**表 8.1 LVM RAID 片段类型**

| **片段类型**   | **描述**                                                          |
| ---------- | --------------------------------------------------------------- |
| `raid1`    | RAID1 镜像。当您指定 `-m` 但您没有指定条带时，这是 `lvcreate` 命令的 `--type` 参数的默认值。 |
| `raid4`    | RAID4 专用奇偶校验磁盘                                                  |
| `raid5`    | 与 raid5_ls相同                                                    |
| `raid5_la` |                                                                 |

- RAID5 左非对称。
- 轮转奇偶校验 0 并分配数据|
  |`raid5_ra`|
- RAID5 右非对称。
- 轮转奇偶校验 N 并分配数据|
  |`raid5_ls`|
- RAID5 左对称。
- 使用数据重启来轮换奇偶校验 0|
  |`raid5_rs`|
- RAID5 右对称。
- 使用数据重启轮转奇偶校验 N
  |`raid6`|与 `raid6_zr`相同|
  |`raid6_zr`|
- RAID6 零重启
- 通过数据重启轮转奇偶校验零（左至右）|
  |`raid6_nr`|
- RAID6 N 重启
- 通过数据重启轮转奇偶校验 N（左至右）|
  |`raid6_nc`|
- RAID6 N 继续
- 轮转奇偶校验 N（左至右）并延续数据|
  |`raid10`|
- 条状镜像。如果您指定了 `-m`，并且您指定了大于 1 的条带数，则这是 `lvcreate` 命令的 `--type` 参数的默认值。
- 镜像集合的条带|
  |`raid0/raid0_meta`|条带。RAID0 以条带大小的单位在多个数据子卷间分布逻辑卷数据。这可以提高性能。如果任何数据子卷失败，逻辑卷数据将会丢失。|

## 8.4 创建 RAID 逻辑卷

这部分提供了创建不同类型的 RAID 逻辑卷的命令示例。

您可以根据您为 `-m` 参数指定的值创建具有不同副本数量的 RAID1 阵列。同样，您可以使用 `-i` 参数 为 RAID 4/5/6 逻辑卷指定条带数。您还可以使用 `-I` 参数指定条带大小。

以下命令在卷组 `my_vg` 中创建了一个名为 `my_lv` 的双向 RAID1 阵列，其大小为 1GB。

```
# lvcreate --type raid1 -m 1 -L 1G -n my_lv my_vg
```

以下命令在卷组 `my_ vg` 中创建名为 `my_lv` 的 RAID5 阵列（3 条带 + 1 隐式奇偶校验驱动器），该阵列的大小为 1GB。请注意，您可以像您为 LVM 条状卷一样指定条带的数目，自动添加正确的奇偶校验驱动器数目。

```
# lvcreate --type raid5 -i 3 -L 1G -n my_lv my_vg
```

以下命令在卷组 `my_vg` 中创建名为 `my_lv` 的 RAID6 阵列（3 个条带 + 2 个隐式奇偶校验驱动器），该阵列的大小为 1GB。

```
# lvcreate --type raid6 -i 3 -L 1G -n my_lv my_vg
```

## 8.5 创建 RAID0（条状）逻辑卷

RAID0 逻辑卷以条的大小为单位，将逻辑卷数据分散到多个数据子卷中。

创建 RAID0 卷的命令格式如下。

```
lvcreate --type raid0[_meta] --stripes Stripes --stripesize StripeSize VolumeGroup [PhysicalVolumePath ...]
```

**表 8.2 RAID0 命令创建参数**

| **参数**                     | **描述**                                                                                                                                                                                                                                                   |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--type raid0[_meta]`      | 指定 `raid0` 创建一个没有元数据卷的 RAID0 卷。指定 `raid0_meta` 会创建一个带有元数据卷的 RAID0 卷。因为 RAID0 是不弹性的，所以不需要保存任何已镜像的数据块（如 RAID1/10），或者计算并保存任何奇偶校验块（如 RAID4/5/6）。因此，它不需要元数据卷来保持有关镜像或奇偶校验块重新同步进程的状态。但是，在从 RAID0 转换到 RAID4/5/6/10 时元数据卷是强制的，指定 `raid0_meta` 会预先分配这些元数据卷以防止分配失败。 |
| `--stri pes stripes`       | 指定在其中分割逻辑卷的设备数。                                                                                                                                                                                                                                          |
| `--stripe size stripeSize` | 以 KB 为单位指定每个条的大小。这是在移动到下一个设备前写入一个设备的数据量。                                                                                                                                                                                                                 |
| `VolumeGroup`              | 指定要使用的卷组。                                                                                                                                                                                                                                                |
| `PhysicalVolumePath` …​    | 指定要使用的设备。如果没有指定，LVM 会选择 Stripes 选项指定的设备数，每个条带一个。                                                                                                                                                                                                         |

这个示例步骤创建一个名为 `mylv` 的 LVM RAID0 逻辑卷，该逻辑卷可在 `/dev/sda1`、`/dev/sdb1` 和 `/dev/sdc1` 磁盘间条状分布数据。

1. 使用 `pvcreate` 命令将卷组中使用的磁盘标记为 LVM 物理卷。
   
   > **警告**
   > 
   > 此命令将销毁 `/dev/sda1`、`/dev/sdb1` 和 `/dev/sdc1` 上的任何数据。
   
   ```
   # pvcreate /dev/sda1 /dev/sdb1 /dev/sdc1
   Physical volume "/dev/sda1" successfully created
   Physical volume "/dev/sdb1" successfully created
   Physical volume "/dev/sdc1" successfully created
   ```

2. 创建卷组 `myvg`。以下命令将创建卷组 `myvg` ：
   
   ```
   # vgcreate myvg /dev/sda1 /dev/sdb1 /dev/sdc1
   Volume group "myvg" successfully created
   ```
   
    您可以使用 `vgs` 命令显示新卷组的属性。
   
   ```
   # vgs
   VG   #PV #LV #SN Attr   VSize  VFree
   myvg   3   0   0 wz--n- 51.45G 51.45G
   ```

3. 从您创建的卷组中创建 RAID0 逻辑卷。以下命令从卷组 `myvg` 创建 RAID0 卷 `mylv`。这个示例创建的逻辑卷大小为 2GB，有三个条带，条带的大小为 4KB。
   
   ```
   # lvcreate --type raid0 -L 2G --stripes 3 --stripesize 4 -n mylv myvg
   Rounding size 2.00 GiB (512 extents) up to stripe boundary size 2.00 GiB(513 extents).
   Logical volume "mylv" created.
   ```

4. 在 RAID0 逻辑卷中创建文件系统。下面的命令在逻辑卷中创建 ext4 文件系统。
   
   ```
   # mkfs.ext4 /dev/myvg/mylv
   Creating filesystem with 525312 4k blocks and 131376 inodes
   Filesystem UUID: 9d4c0704-6028-450a-8b0a-8875358c0511
   Superblock backups stored on blocks:
           32768, 98304, 163840, 229376, 294912
   
   Allocating group tables: done
   Writing inode tables: done
   Creating journal (16384 blocks): done
   Writing superblocks and filesystem accounting information: done
   ```
   
    下面的命令挂载逻辑卷并报告文件系统磁盘空间用量。
   
   ```
   # mount /dev/myvg/mylv /mnt
   # df
   Filesystem             1K-blocks     Used  Available Use% Mounted on
   /dev/mapper/myvg-mylv    2002684     6168    1875072   1% /mnt
   ```

## 8.6 使用带有 RAID LV 的 DM 完整性功能

作为系统管理员，您可以使用带有 RAID LV 的设备映射器(DM)完整性，以最大程度降低软崩溃或位轮转导致数据丢失的风险。

### 8.6.1 软数据崩溃

数据存储中的软崩溃意味着，从存储设备中检索的数据与写入到那个设备中的数据不同。错误的数据可以在存储设备中无限期存在。在检索并尝试使用此数据之前，您可能不会发现这个损坏的数据。

根据配置类型，冗余独立磁盘阵列(RAID)LV 可防止设备出现故障时数据丢失。如果包含一个 RAID 阵列的设备失败，可以从作为 RAID LV 一部分的其他设备中恢复该数据。但是 RAID 配置不能保证数据本身的完整性。软崩溃、静默崩溃、软错误和静默错误用来描述，即使系统和软件仍继续按预期工作，但数据已损坏的情况的术语。

DM 完整性与 RAID 1、4、5、6 和 10 一起使用，以帮助缓解或防止软崩溃造成数据丢失。RAID 层确保非破坏的数据副本可以修复软崩溃错误。完整性层位于每个 RAID 映像之上，而额外子 LV 存储每个 RAID 镜像的完整性元数据（数据校验和）。当您从带有完整性的 RAID LV 中检索数据时，完整性数据校验和会分析崩溃的数据。如果检测到崩溃，完整性层会返回一个错误消息，RAID 层会从另一个 RAID 镜像检索到非破坏的数据副本。RAID 层会在损坏的数据中自动重写非破坏的数据，以修复软崩溃。

当创建一个带有 DM 完整性的 RAID LV 或在现有 RAID LV 中添加完整性时，请考虑以下几点：

- 完整性元数据需要额外的存储空间。对于每个 RAID 镜像，由于添加到数据的校验和，每 500MB 数据都需要 4MB 的额外存储空间。

- 添加 DM 完整性会因为访问数时延迟而影响到性能，有些 RAID 的配置会比其他 RAID 配置受到的影响更大。RAID1 配置通常比 RAID5 或其变体提供更好的性能。

- RAID 完整性块的大小也会影响性能。配置更大的 RAID 完整块可提供更好的性能。但是，一个较小的 RAID 完整性块可以提供更好的兼容性。

- 完整性有两种模式：位图（bitmap）或日志（journal）。位图模式通常比日志模式提供更好的性能。

> **提示**
> 
> 如果您遇到性能问题，建议您使用带有完整性的 RAID1，或者测试特定 RAID 配置的性能以确保它满足您的要求。

### 8.6.2 创建带有 DM 完整性的 RAID LV

当您创建 RAID LV 后，添加 DM 完整性有助于降低因为软崩溃而丢失数据的风险。

**前提条件**

- 您必须有 root 访问权限。

**流程**

- 创建带有 DM 完整性的 RAID LV：
  
  ```
  # lvcreate --type <raid-level> --raidintegrity y -L <usable-size> -n <logical-volume> <volume-group>
  ```

其中

**`<raid-level>`**

  指定您要创建的 RAID LV 的 RAID 级别。

**`<usable-size>`**

  以 MB 为单位指定可用大小。

**`<logical-volume>`**

  指定您要创建的 LV 的名称。

**`<volume-group>`**

  指定要在其中创建 RAID LV 的卷组名称。

在以下示例中，我们在 `test-vg` 卷组中创建名为 `test-lv` 的 RAID LV，可用大小为 256M 和 RAID 1。

带有完整性的 RAID LV 示例

```
# lvcreate --type raid1 --raidintegrity y -L256M -n test-lv test-vg
Creating integrity metadata LV test-lv_rimage_0_imeta with size 8.00 MiB.
  Logical volume "test-lv_rimage_0_imeta" created.
  Creating integrity metadata LV test-lv_rimage_1_imeta with size 8.00 MiB.
  Logical volume "test-lv_rimage_1_imeta" created.
  Logical volume "test-lv" created.
```

### 8.6.3 在现有的 RAID LV 中添加 DM 完整性

您可以在现有的 RAID LV 中添加 DM 完整性，以帮助降低因为软崩溃而丢失数据的风险。

**前提条件**

- 您必须有 root 访问权限。

**流程**

- 在现有 RAID LV 中添加 DM 完整性：
  
  ```
  # lvconvert --raidintegrity y <volume-group>/<logical-volume>
  ```

其中

**`<volume-group>`**

  指定要在其中创建 RAID LV 的卷组名称。

**`<logical-volume>`**

  指定您要创建的 LV 的名称。

### 8.6.4 从 RAID LV 中删除完整性

在 RAID LV 中添加完整性限制了您可以在那个 RAID LV 上执行的一些操作。因此，您必须在执行某些操作前删除完整性。

**前提条件**

- 您必须有 root 访问权限。

**流程**

- 从 RAID LV 中删除完整性：

```
# lvconvert --raidintegrity n <volume-group>/<logical-volume>
```

其中

**`<volume-group>`**

  指定要在其中创建 RAID LV 的卷组名称。

**`<logical-volume>`**

  指定您要创建的 LV 的名称。

### 8.6.5 查看 DM 完整性信息

当您创建有完整性的 RAID LVS 或者在现有 RAID LV 中添加完整性时，使用以下命令查看有关完整性的信息：

```
# lvs -a <volume-group>
```

这里的 *<volume-group>* 是包含有完整性的 RAID LV 的卷组名称。

以下示例显示了在 `test-vg` 卷组中创建的 `test-lv` RAID LV 的信息。

```
# lvs -a test-vg
  LV                        VG      Attr       LSize   Origin                   Cpy%Sync
  test-lv                   test-vg rwi-a-r--- 256.00m                          2.10
  [test-lv_rimage_0]        test-vg gwi-aor--- 256.00m [test-lv_rimage_0_iorig] 93.75
  [test-lv_rimage_0_imeta]  test-vg ewi-ao----   8.00m
  [test-lv_rimage_0_iorig]  test-vg -wi-ao---- 256.00m
  [test-lv_rimage_1]        test-vg gwi-aor--- 256.00m [test-lv_rimage_1_iorig] 85.94
  [test-lv_rimage_1_imeta]  test-vg ewi-ao----   8.00m
  [test-lv_rimage_1_iorig]  test-vg -wi-ao---- 256.00m
  [test-lv_rmeta_0]         test-vg ewi-aor---   4.00m
  [test-lv_rmeta_1]         test-vg ewi-aor---   4.00m
```

**同步**

当您创建一个带有完整性的 RAID LVS 或者在现有 RAID LV 中添加完整性时，我们建议您在使用 LV 前等待完整性同步和 RAID 元数据完成。否则，在后台进行的初始化可能会影响 LV 的性能。`Cpy%Sync` 列指示顶级 RAID LV 和每个 RAID 镜像的同步进度。RAID 镜像在 LV 列中由 `raid_image_N` 表示。请参阅 LV 列，以确保同步进度在顶级 RAID LV 和每个 RAID 镜像中显示 `100%`。

**使用完整性的 RAID 镜像**

`Attr` 列下列出的属性中的 `g` 属性表示 RAID 镜像使用完整性。完整性校验和保存在 `_imeta` RAID LV 中。

要显示每个 RAID LV 的类型，在 `lvs` 命令中添加 `-o+s egtype` 选项：

```
# lvs -a my-vg -o+segtype
  LV                       VG      Attr       LSize   Origin                   Cpy%Sync Type
  test-lv                  test-vg rwi-a-r--- 256.00m                          87.96    raid1
  [test-lv_rimage_0]       test-vg gwi-aor--- 256.00m [test-lv_rimage_0_iorig] 100.00   integrity
  [test-lv_rimage_0_imeta] test-vg ewi-ao----   8.00m                                   linear
  [test-lv_rimage_0_iorig] test-vg -wi-ao---- 256.00m                                   linear
  [test-lv_rimage_1]       test-vg gwi-aor--- 256.00m [test-lv_rimage_1_iorig] 100.00   integrity
  [test-lv_rimage_1_imeta] test-vg ewi-ao----   8.00m                                   linear
  [test-lv_rimage_1_iorig] test-vg -wi-ao---- 256.00m                                   linear
  [test-lv_rmeta_0]        test-vg ewi-aor---   4.00m                                   linear
  [test-lv_rmeta_1]        test-vg ewi-aor---   4.00m                                   linear
```

**完整性不匹配**

有一个增量的计数器，它计算在每个 RAID 镜像上检测到的不匹配数。要查看特定 RAID 镜像的完整性所检测到的数据不匹配，运行以下命令：

`# lvs -o+integritymismatches <volume-group>/<logical-volume>_raid-image_<n>`

其中

***`<volume-group>`***

  指定要在其中创建 RAID LV 的卷组名称。

***`<logical-volume>`***

  指定您要创建的 LV 的名称。

***`<n>`***

  指定您要查看完整性不匹配信息的 RAID 镜像。

您必须为每个要查看的 RAID 镜像运行该命令。在以下示例中，我们将在 `test-vg/test-lv` 下查看 `rimage_0` 的数据不匹配。

```
# lvs -o+integritymismatches test-vg/test-lv_rimage_0
  LV                 VG      Attr       LSize   Origin                      Cpy%Sync IntegMismatches
  [test-lv_rimage_0] test-vg gwi-aor--- 256.00m [test-lv_rimage_0_iorig]    100.00                 0
```

我们可以看到完整性还没有检测到任何不匹配的数据，因此 `IntegMismatches` 计数器会显示零(0)。

**内核消息日志中的完整性不匹配**

您还可以在内核消息日志中查找数据完整性信息，如下例所示。

``dm-integrity 与内核消息日志中不匹配的示例``

```
device-mapper: integrity: dm-12: Checksum failed at sector 0x24e7
```

**dm-integrity 数据从内核消息日志中更正示例**

```
md/raid1:mdX: read error corrected (8 sectors at 9448 on dm-16)
```

## 8.7 控制 RAID 卷初始化的频率

当您创建 RAID10 逻辑卷时，使用同步操作初始化逻辑卷所需的后台 I/O 可能会对 LVM 设备排除其他 I/O 操作，如卷组元数据更新，特别是在创建大量 RAID 逻辑卷时。这会导致其它 LVM 操作速度下降。

您可以通过实施节流来控制初始化 RAID 逻辑卷的速度。您可以通过使用 `lvcreate` 命令的 `--minrecoveryrate` 和 `--maxrecoveryrate` 选项为这些操作设置最小和最大 I/O 速率，从而控制执行 同步 操作的速率。如下所示指定这些选项。

- `--maxrecoveryrate Rate[bBsSkKmMgG]`
  
    为 RAID 逻辑卷设置最大恢复率，使其不会阻断其他小的 I/O 操作。这个比率被指定为“数量/每秒/阵列中的每个设备”。如果没有给出后缀，则会假定为 kiB/sec/device。将恢复率设置为 0 表示它将不被绑定。

- `--minrecoveryrate Rate[bBsSkKmMgG]`
  
    为 RAID 逻辑卷设置最小恢复率，以确保同步操作的 I/O 获得最小吞吐量，即使存在大量 I/O。这个比率被指定为“数量/每秒/阵列中的每个设备”。如果没有给出后缀，则会假定为 kiB/sec/device。

下面的命令创建了双向 RAID10 阵列，有三个条带，大小为 10GB，最大恢复率为 128 kiB/sec/device。该数组名为 `my_lv`，位于卷组 `my_vg` 中。

```
# lvcreate --type raid10 -i 2 -m 1 -L 10G --maxrecoveryrate 128 -n my_lv my_vg
```

您还可以为 RAID 清理操作指定最小和最大恢复率。

## 8.8 将线性设备转换为 RAID 设备

您可以使用 `lvconvert` 命令的 `--type` 参数将现有线性逻辑卷转换为 RAID 设备。

以下命令将卷组 `my_vg` 中的线性逻辑卷 `my_lv` 转换为双向 RAID1 阵列。

```
# lvconvert --type raid1 -m 1 my_vg/my_lv
```

因为 RAID 逻辑卷由元数据和数据子卷对组成，因此当您将线性设备转换成 RAID1 阵列时，会生成一个新的元数据子卷，并关联到线性卷所在的同一物理卷中的原始逻辑卷之一。额外的镜像在 metadata/data 子卷对中添加。例如：如果原始设备如下：

```
# lvs -a -o name,copy_percent,devices my_vg
  LV     Copy%  Devices
  my_lv         /dev/sde1(0)
```

转换为双向 RAID1 阵列后，该设备包含以下数据和元数据子卷对：

```
# lvconvert --type raid1 -m 1 my_vg/my_lv
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            6.25   my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]        /dev/sde1(0)
  [my_lv_rimage_1]        /dev/sdf1(1)
  [my_lv_rmeta_0]         /dev/sde1(256)
  [my_lv_rmeta_1]         /dev/sdf1(0)
```

如果无法将与原始逻辑卷对的元数据映像放在同一个物理卷中，`lvconvert` 将失败。

## 8.9 将 LVM RAID1 逻辑卷转换为 LVM 线性逻辑卷

您可以通过指定 `-m0` 参数，使用 `lvconvert` 命令将现有 RAID1 LVM 逻辑卷转换为 LVM 线性逻辑卷。这会删除所有 RAID 数据子卷以及构成 RAID 阵列的所有 RAID 元数据子卷，保留顶层 RAID1 镜像作为线性逻辑卷。

下面的例子显示了现有的 LVM RAID1 逻辑卷。

```
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]        /dev/sde1(1)
  [my_lv_rimage_1]        /dev/sdf1(1)
  [my_lv_rmeta_0]         /dev/sde1(0)
  [my_lv_rmeta_1]         /dev/sdf1(0)
```

以下命令可将 LVM RAID1 逻辑卷 `my_vg/my_lv` 转换为 LVM 线性设备。

```
# lvconvert -m0 my_vg/my_lv
# lvs -a -o name,copy_percent,devices my_vg
  LV      Copy%  Devices
  my_lv          /dev/sde1(1)
```

当您将 LVM RAID1 逻辑卷转换成 LVM 线性卷，,您可以指定要删除的物理卷。以下示例显示了由两个镜像组成的 LVM RAID1 逻辑卷布局： `/dev/sda1` 和 `/dev/sdb1`。在本例中，`lvconvert` 命令指定要删除 `/dev/sda1`，并将 `/dev/sdb1` 保留为构成线性设备的物理卷。

```
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]        /dev/sda1(1)
  [my_lv_rimage_1]        /dev/sdb1(1)
  [my_lv_rmeta_0]         /dev/sda1(0)
  [my_lv_rmeta_1]         /dev/sdb1(0)
# lvconvert -m0 my_vg/my_lv /dev/sda1
# lvs -a -o name,copy_percent,devices my_vg
  LV    Copy%  Devices
  my_lv        /dev/sdb1(1)
```

## 8.10 将镜像 LVM 设备转换为 RAID1 设备

您可以通过指定 `--type raid1` 参数，使用 `lvconvert` 命令将现有镜像类型为 mirror 的 LVM 设备转换为 RAID1 LVM 设备。这会将镜像子卷(**`mimage`**)重命名为 RAID 子卷(**`rimage`**)。另外，镜像日志已被删除，并为与对应数据子卷位于同一物理卷上的数据子卷创建元数据子卷(**`rmeta`**)。

以下示例显示了镜像逻辑卷 `my_vg/my_lv` 的布局。

```
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv             15.20 my_lv_mimage_0(0),my_lv_mimage_1(0)
  [my_lv_mimage_0]        /dev/sde1(0)
  [my_lv_mimage_1]        /dev/sdf1(0)
  [my_lv_mlog]            /dev/sdd1(0)
```

下面的命令可将镜像逻辑卷 `my_vg/my_lv` 转换为 RAID1 逻辑卷。

```
# lvconvert --type raid1 my_vg/my_lv
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]        /dev/sde1(0)
  [my_lv_rimage_1]        /dev/sdf1(0)
  [my_lv_rmeta_0]         /dev/sde1(125)
  [my_lv_rmeta_1]         /dev/sdf1(125)
```

## 8.11 重新定义 RAID 逻辑卷大小

您可以使用以下方法重新定义 RAID 逻辑卷大小：

- 您可以使用 `lvresize` 或 `lvextend` 命令增加任意类型的 RAID 逻辑卷的大小。这不会改变 RAID 镜像的数量。对于条带的 RAID 逻辑卷，创建条带 RAID 逻辑卷时可使用同样的条状限制。

- 您可以使用 `lvresize` 或 `lvreduce` 命令减少 任意类型的 RAID 逻辑卷的大小。这不会改变 RAID 镜像的数量。与 lvextend 命令一样，使用与创建条状 RAID 逻辑卷相同的条状限制。

- 您可以使用 `lvconvert` 命令的 `--stripes N` 参数更改条带的数量(`raid4/5/6/10`)。这通过添加或删除的条带的容量来增加或减少 RAID 逻辑卷的大小。请注意 ，`raid10` 卷只能添加条带。这个能力是 RAID *reshaping* 功能的一部分，它允许您在保持相同的 RAID 级别的情况下修改 RAID 逻辑卷的属性。有关 RAID 重新哈希以及使用 `lvconvert` 命令重新定义 RAID 逻辑卷的示例，请参阅 `lvmraid(7)man` page。

## 8.12 更改现有 RAID1 设备中的镜像数

您可以更改现有 RAID1 阵列中的镜像数量，就如同在早期的 LVM 镜像部署中更改镜像数量。使用 `lvconvert` 命令指定要添加或删除的额外元数据/数据子卷对数量。

当您使用 `lvconvert` 命令将镜像添加到 RAID1 设备时，您可以指定生成的设备的镜像总数，或者您可以指定要添加到该设备的镜像数。您还可以有选择地指定新元数据/数据镜像对所在的物理卷。

元数据子卷（名为 **`rmeta`**）始终存在于与其数据子卷对应的 **`rimage`**相同的物理设备中。元数据/数据子卷对不会在与 RAID 阵列中另一个元数据/数据子卷对相同的物理卷上创建（除非您在 任何位置指定了 --alloc）。

在 RAID1 卷中添加镜像的命令格式如下：

```
lvconvert -m new_absolute_count vg/lv [removable_PVs]
lvconvert -m +num_additional_images vg/lv [removable_PVs]
```

例如：以下命令显示 LVM 设备 `my_vg/my_lv`，这是一个双向 RAID1 阵列：

```
# lvs -a -o name,copy_percent,devices my_vg
  LV                Copy%  Devices
  my_lv             6.25    my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]         /dev/sde1(0)
  [my_lv_rimage_1]         /dev/sdf1(1)
  [my_lv_rmeta_0]          /dev/sde1(256)
  [my_lv_rmeta_1]          /dev/sdf1(0)
```

以下命令可将双向 RAID1 设备 `my_vg/my_lv` 转换为三向 RAID1 设备：

```
# lvconvert -m 2 my_vg/my_lv
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv              6.25 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
  [my_lv_rimage_0]        /dev/sde1(0)
  [my_lv_rimage_1]        /dev/sdf1(1)
  [my_lv_rimage_2]        /dev/sdg1(1)
  [my_lv_rmeta_0]         /dev/sde1(256)
  [my_lv_rmeta_1]         /dev/sdf1(0)
  [my_lv_rmeta_2]         /dev/sdg1(0)
```

当您向 RAID1 阵列中添加镜像时，您可以指定要用于该镜像的物理卷。以下命令可将双向 RAID1 设备 `my_vg/my_lv` 转换为三向 RAID1 设备，指定物理卷 `/dev/sdd1` 用于该阵列：

```
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv             56.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]        /dev/sda1(1)
  [my_lv_rimage_1]        /dev/sdb1(1)
  [my_lv_rmeta_0]         /dev/sda1(0)
  [my_lv_rmeta_1]         /dev/sdb1(0)
# lvconvert -m 2 my_vg/my_lv /dev/sdd1
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv             28.00 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
  [my_lv_rimage_0]        /dev/sda1(1)
  [my_lv_rimage_1]        /dev/sdb1(1)
  [my_lv_rimage_2]        /dev/sdd1(1)
  [my_lv_rmeta_0]         /dev/sda1(0)
  [my_lv_rmeta_1]         /dev/sdb1(0)
  [my_lv_rmeta_2]         /dev/sdd1(0)
```

要从 RAID1 阵列中删除镜像，请使用以下命令。当您使用 `lvconvert` 命令从 RAID1 设备中删除镜像时，您可以指定所生成设备的镜像总数，或者您可以指定要从该设备中删除的镜像数量。您还可以选择指定要从中删除该设备的物理卷。

```
lvconvert -m new_absolute_count vg/lv [removable_PVs]
lvconvert -m -num_fewer_images vg/lv [removable_PVs]
```

另外，当镜像及其关联的元数据子卷被删除时，任何带有高数值的镜像都将用来填充被删除卷的位置。如果您从由 `lv_rimage_0`、`lv_rimage_1` 和 `lv_rimage_2` 组成的三向 RAID1 阵列中删除 `lv_rimage_1`，则会产生一个由 `lv_rimage_0` 和 `lv_rimage_1` 组成的 RAID1 阵列。子卷 `lv_rimage_2` 将重命名并接管空插槽，变为 `lv_rimage_1`。

以下示例显示了三向 RAID1 逻辑卷 `my_vg/my_lv` 的布局。

```
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
  [my_lv_rimage_0]        /dev/sde1(1)
  [my_lv_rimage_1]        /dev/sdf1(1)
  [my_lv_rimage_2]        /dev/sdg1(1)
  [my_lv_rmeta_0]         /dev/sde1(0)
  [my_lv_rmeta_1]         /dev/sdf1(0)
  [my_lv_rmeta_2]         /dev/sdg1(0)
```

下面的命令可将三向 RAID1 逻辑卷转换成双向 RAID1 逻辑卷。

```
# lvconvert -m1 my_vg/my_lv
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]        /dev/sde1(1)
  [my_lv_rimage_1]        /dev/sdf1(1)
  [my_lv_rmeta_0]         /dev/sde1(0)
  [my_lv_rmeta_1]         /dev/sdf1(0)
```

下面的命令可将三向 RAID1 逻辑卷转换为双向 RAID1 逻辑卷，并指定要删除的镜像的物理卷为 `/dev/sde1`。

```
# lvconvert -m1 my_vg/my_lv /dev/sde1
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]        /dev/sdf1(1)
  [my_lv_rimage_1]        /dev/sdg1(1)
  [my_lv_rmeta_0]         /dev/sdf1(0)
  [my_lv_rmeta_1]         /dev/sdg1(0)
```

## 8.13 将 RAID 镜像分离为一个独立的逻辑卷

您可以分离 RAID 逻辑卷的镜像形成新的逻辑卷。

分离 RAID 镜像的命令格式如下：

```
lvconvert --splitmirrors count -n splitname vg/lv [removable_PVs]
```

和您从现有 RAID1 逻辑卷中删除 RAID 镜像一样，当您从设备的中间部分删除 RAID 数据子卷（及其关联的元数据子卷）时，会使用数字高的镜像来填充空的位置。因此，构成 RAID 阵列的逻辑卷中的索引号将是一个不可中断的整数序列。

> **注意**
> 
> 如果 RAID1 阵列还没有同步，您就无法分离 RAID 镜像。

下面的例子将一个双向 RAID1 逻辑卷 `my_lv` 分割成两个线性逻辑卷 `my_lv` 和 `new`。

```
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv             12.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]        /dev/sde1(1)
  [my_lv_rimage_1]        /dev/sdf1(1)
  [my_lv_rmeta_0]         /dev/sde1(0)
  [my_lv_rmeta_1]         /dev/sdf1(0)
# lvconvert --splitmirror 1 -n new my_vg/my_lv
# lvs -a -o name,copy_percent,devices my_vg
  LV      Copy%  Devices
  my_lv          /dev/sde1(1)
  new            /dev/sdf1(1)
```

下面的例子将一个三向 RAID1 逻辑卷 `my_lv` 分割成一个双向 RAID1 逻辑卷 `my_lv` 和线性逻辑卷 `new`

```
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
  [my_lv_rimage_0]        /dev/sde1(1)
  [my_lv_rimage_1]        /dev/sdf1(1)
  [my_lv_rimage_2]        /dev/sdg1(1)
  [my_lv_rmeta_0]         /dev/sde1(0)
  [my_lv_rmeta_1]         /dev/sdf1(0)
  [my_lv_rmeta_2]         /dev/sdg1(0)
# lvconvert --splitmirror 1 -n new my_vg/my_lv
# lvs -a -o name,copy_percent,devices my_vg
  LV            Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]        /dev/sde1(1)
  [my_lv_rimage_1]        /dev/sdf1(1)
  [my_lv_rmeta_0]         /dev/sde1(0)
  [my_lv_rmeta_1]         /dev/sdf1(0)
  new                     /dev/sdg1(1)
```

## 8.14. 分割和合并 RAID 镜像

您可以临时分离 RAID1 阵列的镜像以进行只读使用，同时将 `--trackchanges` 参数和 `lvconvert` 命令的 `--splitmirrors` 参数结合使用 `--splitmirrors` 参数来跟踪任何更改。这可让您以后将镜像合并到阵列中，同时只重新同步那些自镜像被分割后更改的阵列的部分。

分离 RAID 镜像的 `lvconvert` 命令格式如下。

```
lvconvert --splitmirrors count --trackchanges vg/lv [removable_PVs]
```

当您使用 `--trackchanges` 参数分离 RAID 镜像时，您可以指定要分离的镜像，但您无法更改要分割的卷名称。另外，得到的卷有以下限制。

- 创建的新卷为只读。
- 不能调整新卷的大小。
- 不能重命名剩余的数组。
- 不能调整剩余的数组大小。
- 您可以独立激活新卷和剩余的阵列。

您可以通过执行后续的 `lvconvert` 命令 并使用 `--merge` 参数来合并使用 `--trackchanges` 参数分离的镜像。当您合并镜像时，只有自镜像分割后更改的阵列部分会被重新同步。

`lvconvert` 命令合并 RAID 镜像的格式如下。

```
lvconvert --merge raid_image
```

下面的例子创建了 RAID1 逻辑卷，然后在追踪剩余的阵列时从那个卷中分离镜像。

```
# lvcreate --type raid1 -m 2 -L 1G -n my_lv my_vg
  Logical volume "my_lv" created
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
  [my_lv_rimage_0]        /dev/sdb1(1)
  [my_lv_rimage_1]        /dev/sdc1(1)
  [my_lv_rimage_2]        /dev/sdd1(1)
  [my_lv_rmeta_0]         /dev/sdb1(0)
  [my_lv_rmeta_1]         /dev/sdc1(0)
  [my_lv_rmeta_2]         /dev/sdd1(0)
# lvconvert --splitmirrors 1 --trackchanges my_vg/my_lv
  my_lv_rimage_2 split from my_lv for read-only purposes.
  Use 'lvconvert --merge my_vg/my_lv_rimage_2' to merge back into my_lv
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
  [my_lv_rimage_0]        /dev/sdb1(1)
  [my_lv_rimage_1]        /dev/sdc1(1)
  my_lv_rimage_2          /dev/sdd1(1)
  [my_lv_rmeta_0]         /dev/sdb1(0)
  [my_lv_rmeta_1]         /dev/sdc1(0)
  [my_lv_rmeta_2]         /dev/sdd1(0)
```

以下示例在跟踪剩余的阵列更改时从 RAID1 卷中分离镜像，然后将该卷合并回阵列中。

```
# lvconvert --splitmirrors 1 --trackchanges my_vg/my_lv
  lv_rimage_1 split from my_lv for read-only purposes.
  Use 'lvconvert --merge my_vg/my_lv_rimage_1' to merge back into my_lv
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]        /dev/sdc1(1)
  my_lv_rimage_1          /dev/sdd1(1)
  [my_lv_rmeta_0]         /dev/sdc1(0)
  [my_lv_rmeta_1]         /dev/sdd1(0)
# lvconvert --merge my_vg/my_lv_rimage_1
  my_vg/my_lv_rimage_1 successfully merged back into my_vg/my_lv
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
  [my_lv_rimage_0]        /dev/sdc1(1)
  [my_lv_rimage_1]        /dev/sdd1(1)
  [my_lv_rmeta_0]         /dev/sdc1(0)
  [my_lv_rmeta_1]         /dev/sdd1(0)
```

## 8.15 设置 RAID 失败策略

LVM RAID 根据 `lvm.conf` 文件中的 `raid_fault_policy` 字段定义的首选项自动处理设备故障。

- 如果将 `raid_fault_policy` 字段设置为 `allocate`，系统将尝试使用卷组中的备用设备替换失败的设备。如果没有可用的备用设备，则会向系统日志报告。
- 如果将 `raid_fault_policy` 字段设置为 `warn`，系统将生成警告，日志将指示设备已失败。这使得用户能够决定采取什么行动。

只要有足够的设备支持可用性，RAID 逻辑卷将继续操作。

### 8.15.1 分配 RAID 失败策略

在以下示例中，`raid_fault_policy` 字段已设置为在 `lvm.conf` 文件中 `allocate`。按如下方式定义 RAID 逻辑卷。

```
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
  [my_lv_rimage_0]        /dev/sde1(1)
  [my_lv_rimage_1]        /dev/sdf1(1)
  [my_lv_rimage_2]        /dev/sdg1(1)
  [my_lv_rmeta_0]         /dev/sde1(0)
  [my_lv_rmeta_1]         /dev/sdf1(0)
  [my_lv_rmeta_2]         /dev/sdg1(0)
```

如果 `/dev/sde` 设备失败，系统日志将显示错误消息。

```
# grep lvm /var/log/messages
Jan 17 15:57:18 bp-01 lvm[8599]: Device #0 of raid1 array, my_vg-my_lv, has failed.
Jan 17 15:57:18 bp-01 lvm[8599]: /dev/sde1: read failed after 0 of 2048 at
250994294784: Input/output error
Jan 17 15:57:18 bp-01 lvm[8599]: /dev/sde1: read failed after 0 of 2048 at
250994376704: Input/output error
Jan 17 15:57:18 bp-01 lvm[8599]: /dev/sde1: read failed after 0 of 2048 at 0:
Input/output error
Jan 17 15:57:18 bp-01 lvm[8599]: /dev/sde1: read failed after 0 of 2048 at
4096: Input/output error
Jan 17 15:57:19 bp-01 lvm[8599]: Couldn't find device with uuid
3lugiV-3eSP-AFAR-sdrP-H20O-wM2M-qdMANy.
Jan 17 15:57:27 bp-01 lvm[8599]: raid1 array, my_vg-my_lv, is not in-sync.
Jan 17 15:57:36 bp-01 lvm[8599]: raid1 array, my_vg-my_lv, is now in-sync.
```

由于 `raid_fault_policy` 字段已设置为 `allocate`，因此失败的设备将被替换为卷组中的新设备。

```
# lvs -a -o name,copy_percent,devices vg
  Couldn't find device with uuid 3lugiV-3eSP-AFAR-sdrP-H20O-wM2M-qdMANy.
  LV            Copy%  Devices
  lv            100.00 lv_rimage_0(0),lv_rimage_1(0),lv_rimage_2(0)
  [lv_rimage_0]        /dev/sdh1(1)
  [lv_rimage_1]        /dev/sdf1(1)
  [lv_rimage_2]        /dev/sdg1(1)
  [lv_rmeta_0]         /dev/sdh1(0)
  [lv_rmeta_1]         /dev/sdf1(0)
  [lv_rmeta_2]         /dev/sdg1(0)
```

请注意，虽然替换了失败的设备，但显示仍指示 LVM 无法找到失败的设备。这是因为虽然从 RAID 逻辑卷中删除了失败的设备，但故障的设备还没有从卷组中删除。要从卷组中删除失败的设备，可以执行 `vgreduce --removemissing VG`。

如果将 `raid_fault_policy` 设置为 `allocate`，但没有备用设备，分配将失败，让逻辑卷保留原样。如果分配失败，您可以选择修复驱动器，然后使用 `lvchange` 命令的 `--refresh` 选项启动故障设备的恢复。另外，您还可以替换失败的设备。

### 8.15.2 警告 RAID 失败策略

在以下示例中，`lvm.conf` 文件中将 `raid_fault_policy` 字段设置为 `warn`。按如下方式定义 RAID 逻辑卷。

```
# lvs -a -o name,copy_percent,devices my_vg
  LV               Copy%  Devices
  my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
  [my_lv_rimage_0]        /dev/sdh1(1)
  [my_lv_rimage_1]        /dev/sdf1(1)
  [my_lv_rimage_2]        /dev/sdg1(1)
  [my_lv_rmeta_0]         /dev/sdh1(0)
  [my_lv_rmeta_1]         /dev/sdf1(0)
  [my_lv_rmeta_2]         /dev/sdg1(0)
```

如果 `/dev/sdh` 设备失败，系统日志将显示错误消息。在这种情况下，LVM 将不会自动尝试通过替换其中一个镜像修复 RAID 设备。如果设备失败，您可以使用 `lvconvert` 命令的 `--repair` 参数替换该设备。

## 8.16 在逻辑卷中替换 RAID 设备

您可以替换逻辑卷中的 RAID 设备。

- 如果 RAID 设备中没有失败，请遵循 [第 8.16.1 节 替换没有失败的 RAID 设备](#8161-替换没有失败的-raid-设备)。
- 如果 RAID 设备失败，请遵循 [第 8.16.4 节 在逻辑卷中替换失败的 RAID 设备](#8164-在逻辑卷中替换失败的-raid-设备)。

### 8.16.1 替换没有失败的 RAID 设备

要替换逻辑卷中的 RAID 设备，请使用 `lvconvert` 命令的 `--replace` 参数。

**前提条件**

- RAID 设备没有失败。如果 RAID 设备失败，以下命令将无法正常工作。

**流程**

- 替换 RAID 设备：
  
  ```
  # lvconvert --replace dev_to_remove vg/lv possible_replacements
  ```
  
  - 使用您要替换的物理卷的路径替换 *dev_to_remove*。
  - 使用 RAID 阵列的卷组和逻辑卷名称替换 *vg/lv*。
  - 使用您要用作替换的物理卷的路径替换 *possible_replacements*。

> **例 8.1 替换 RAID1 设备**
> 
> 下面的例子创建了 RAID1 逻辑卷，然后替换那个卷中的一个设备。
> 
> 1. 创建 RAID1 阵列：
>    
>    ```
>    # lvcreate --type raid1 -m 2 -L 1G -n my_lv my_vg
>    
>    Logical volume "my_lv" created
>    ```
> 
> 2. 检查 RAID1 阵列：
>    
>    ```
>    # lvs -a -o name,copy_percent,devices my_vg
>    
>    LV               Copy%  Devices
>    my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
>    [my_lv_rimage_0]        /dev/sdb1(1)
>    [my_lv_rimage_1]        /dev/sdb2(1)
>    [my_lv_rimage_2]        /dev/sdc1(1)
>    [my_lv_rmeta_0]         /dev/sdb1(0)
>    [my_lv_rmeta_1]         /dev/sdb2(0)
>    [my_lv_rmeta_2]         /dev/sdc1(0)
>    ```
> 
> 3. 替换 `/dev/sdb2` 物理卷：
>    
>    ```
>    # lvconvert --replace /dev/sdb2 my_vg/my_lv
>    ```
> 
> 4. 使用替换检查 RAID1 阵列：
>    
>    ```
>    # lvs -a -o name,copy_percent,devices my_vg
>    
>    LV               Copy%  Devices
>    my_lv             37.50 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
>    [my_lv_rimage_0]        /dev/sdb1(1)
>    [my_lv_rimage_1]        /dev/sdc2(1)
>    [my_lv_rimage_2]        /dev/sdc1(1)
>    [my_lv_rmeta_0]         /dev/sdb1(0)
>    [my_lv_rmeta_1]         /dev/sdc2(0)
>    [my_lv_rmeta_2]         /dev/sdc1(0)
>    ```

> **例 8.2 指定替换的物理卷**
> 
> 下面的例子创建了 RAID1 逻辑卷，然后替换那个卷中的一个设备，指定要用来替换哪些物理卷。
> 
> 1. 创建 RAID1 阵列：
>    
>    ```
>    # lvcreate --type raid1 -m 1 -L 100 -n my_lv my_vg
>    
>    Logical volume "my_lv" created
>    ```
> 
> 2. 检查 RAID1 阵列：
>    
>    ```
>    # lvs -a -o name,copy_percent,devices my_vg
>    
>    LV               Copy%  Devices
>    my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
>    [my_lv_rimage_0]        /dev/sda1(1)
>    [my_lv_rimage_1]        /dev/sdb1(1)
>    [my_lv_rmeta_0]         /dev/sda1(0)
>    [my_lv_rmeta_1]         /dev/sdb1(0)
>    ```
> 
> 3. 检查物理卷：
>    
>    ```
>    # pvs
>    
>    PV          VG       Fmt  Attr PSize    PFree
>    /dev/sda1   my_vg    lvm2 a--  1020.00m  916.00m
>    /dev/sdb1   my_vg    lvm2 a--  1020.00m  916.00m
>    /dev/sdc1   my_vg    lvm2 a--  1020.00m 1020.00m
>    /dev/sdd1   my_vg    lvm2 a--  1020.00m 1020.00m
>    ```
> 
> 4. 将 `/dev/sdb1` 物理卷替换为 `/dev/sdd1`:
>    
>    ```
>    # lvconvert --replace /dev/sdb1 my_vg/my_lv /dev/sdd1
>    ```
> 
> 5. 使用替换检查 RAID1 阵列：
>    
>    ```
>    # lvs -a -o name,copy_percent,devices my_vg
>    
>    LV               Copy%  Devices
>    my_lv             28.00 my_lv_rimage_0(0),my_lv_rimage_1(0)
>    [my_lv_rimage_0]        /dev/sda1(1)
>    [my_lv_rimage_1]        /dev/sdd1(1)
>    [my_lv_rmeta_0]         /dev/sda1(0)
>    [my_lv_rmeta_1]         /dev/sdd1(0)
>    ```

> **例 8.3 替换多个 RAID 设备**
> 
> 您可以通过指定多个 `replace` 参数来替换 多个 RAID 设备，如下例所示。
> 
> 1. 创建 RAID1 阵列：
>    
>    ```
>    # lvcreate --type raid1 -m 2 -L 100 -n my_lv my_vg
>    
>    Logical volume "my_lv" created
>    ```
> 
> 2. 检查 RAID1 阵列：
>    
>    ```
>    # lvs -a -o name,copy_percent,devices my_vg
>    
>    LV               Copy%  Devices
>    my_lv            100.00 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
>    [my_lv_rimage_0]        /dev/sda1(1)
>    [my_lv_rimage_1]        /dev/sdb1(1)
>    [my_lv_rimage_2]        /dev/sdc1(1)
>    [my_lv_rmeta_0]         /dev/sda1(0)
>    [my_lv_rmeta_1]         /dev/sdb1(0)
>    [my_lv_rmeta_2]         /dev/sdc1(0)
>    ```
> 
> 3. 替换 `/dev/sdb1` 和 `/dev/sdc1` 物理卷：
>    
>    ```
>    # lvconvert --replace /dev/sdb1 --replace /dev/sdc1 my_vg/my_lv
>    ```
> 
> 4. 使用替换检查 RAID1 阵列：
>    
>    ```
>    # lvs -a -o name,copy_percent,devices my_vg
>    
>    LV               Copy%  Devices
>    my_lv             60.00 my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
>    [my_lv_rimage_0]        /dev/sda1(1)
>    [my_lv_rimage_1]        /dev/sdd1(1)
>    [my_lv_rimage_2]        /dev/sde1(1)
>    [my_lv_rmeta_0]         /dev/sda1(0)
>    [my_lv_rmeta_1]         /dev/sdd1(0)
>    [my_lv_rmeta_2]         /dev/sde1(0)
>    ```

### 8.16.2 LVM RAID 中失败的设备

RAID 跟传统的 LVM 镜像不同。LVM 镜像需要删除失败的设备，或者镜像逻辑卷会挂起。RAID 阵列可在有失败设备的情况下继续运行。实际上，对于 RAID1 以外的 RAID 类型，删除设备意味着将设备转换为较低级别 RAID（例如：从 RAID6 转换为 RAID5，或者从 RAID4 或者 RAID5 转换到 RAID0）。

因此，LVM 允许您使用 `lvconvert` 命令的 `--repair` 参数替换 RAID 卷中失败的设备，而不是无条件地删除失败的设备。

### 8.16.3 在逻辑卷中恢复失败的 RAID 设备

如果 LVM RAID 设备失败是一个临时故障，或者您可以修复失败的设备，您可以初始化失败设备的恢复。

**前提条件**

- 之前失败的设备现在可以正常工作。

**流程**

- 刷新包含 RAID 设备的逻辑卷：
  
  ```
  # lvchange --refresh my_vg/my_lv
  ```

**验证步骤**

- 使用恢复的设备检查逻辑卷：
  
  ```
  # lvs --all --options name,devices,lv_attr,lv_health_status my_vg
  ```

### 8.16.4 在逻辑卷中替换失败的 RAID 设备

这个过程替换作为 LVM RAID 逻辑卷中的物理卷的失败设备。

**前提条件**

- 卷组包含一个物理卷，它有足够的可用容量替换失败的设备。
  
  如果卷组中没有足够可用区块的物理卷，请使用 `vgextend` 实用程序添加新的、足够大的物理卷。

**流程**

1. 在下面的示例中，RAID 逻辑卷布局如下：
   
   ```
   # lvs --all --options name,copy_percent,devices my_vg
   
   LV               Cpy%Sync Devices
   my_lv            100.00   my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
   [my_lv_rimage_0]          /dev/sde1(1)
   [my_lv_rimage_1]          /dev/sdc1(1)
   [my_lv_rimage_2]          /dev/sdd1(1)
   [my_lv_rmeta_0]           /dev/sde1(0)
   [my_lv_rmeta_1]           /dev/sdc1(0)
   [my_lv_rmeta_2]           /dev/sdd1(0)
   ```

2. 如果 `/dev/sdc` 设备失败，`lvs` 命令的输出如下：
   
   ```
   # lvs --all --options name,copy_percent,devices my_vg
   
   /dev/sdc: open failed: No such device or address
   Couldn't find device with uuid A4kRl2-vIzA-uyCb-cci7-bOod-H5tX-IzH4Ee.
   WARNING: Couldn't find all devices for LV my_vg/my_lv_rimage_1 while checking used and assumed devices.
   WARNING: Couldn't find all devices for LV my_vg/my_lv_rmeta_1 while checking used and assumed devices.
   LV               Cpy%Sync Devices
   my_lv            100.00   my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
   [my_lv_rimage_0]          /dev/sde1(1)
   [my_lv_rimage_1]          [unknown](1)
   [my_lv_rimage_2]          /dev/sdd1(1)
   [my_lv_rmeta_0]           /dev/sde1(0)
   [my_lv_rmeta_1]           [unknown](0)
   [my_lv_rmeta_2]           /dev/sdd1(0)
   ```

3. 替换失败的设备并显示逻辑卷：
   
   ```
   # lvconvert --repair my_vg/my_lv
   
   /dev/sdc: open failed: No such device or address
   Couldn't find device with uuid A4kRl2-vIzA-uyCb-cci7-bOod-H5tX-IzH4Ee.
   WARNING: Couldn't find all devices for LV my_vg/my_lv_rimage_1 while checking used and assumed devices.
   WARNING: Couldn't find all devices for LV my_vg/my_lv_rmeta_1 while checking used and assumed devices.
   Attempt to replace failed RAID images (requires full device resync)? [y/n]: y
   Faulty devices in my_vg/my_lv successfully replaced.
   ```
   
    可选： 要手动指定替换失败设备的物理卷，请在命令末尾添加物理卷：
   
   ```
   # lvconvert --repair my_vg/my_lv replacement_pv
   ```

4. 使用替换检查逻辑卷：
   
   ```
   # lvs --all --options name,copy_percent,devices my_vg
   
   /dev/sdc: open failed: No such device or address
   /dev/sdc1: open failed: No such device or address
   Couldn't find device with uuid A4kRl2-vIzA-uyCb-cci7-bOod-H5tX-IzH4Ee.
   LV               Cpy%Sync Devices
   my_lv            43.79    my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
   [my_lv_rimage_0]          /dev/sde1(1)
   [my_lv_rimage_1]          /dev/sdb1(1)
   [my_lv_rimage_2]          /dev/sdd1(1)
   [my_lv_rmeta_0]           /dev/sde1(0)
   [my_lv_rmeta_1]           /dev/sdb1(0)
   [my_lv_rmeta_2]           /dev/sdd1(0)
   ```
   
    在您从卷组中删除失败的设备前，LVM 工具仍然指示 LVM 无法找到失败的设备。

5. 从卷组中删除失败的设备：
   
   ```
   # vgreduce --removemissing VG
   ```

## 8.17 检查 RAID 逻辑卷中的数据一致性（RAID 清理）

LVM 提供对 RAID 逻辑卷的清理支持。RAID 清理是读取阵列中的所有数据和奇偶校验块的过程，并检查它们是否是分配的。

**流程**

1. 可选：限制清理过程使用的 I/O 带宽。
   
   当您执行 RAID 清理操作时，同步 操作所需的后台 I/O 可将其他 I/O 分离到 LVM 设备，如卷组元数据更新。这可能导致其它 LVM 操作速度下降。您可以使用节流功能控制清理操作的速度。
   
   在下一步的 `lvchange --syncaction` 命令中添加以下选项：
   
    **`--maxrecoveryrate Rate[bBsSkKmMgG]`**
   
   设置最大恢复率，以便使操作不会严重影响小的 I/O 操作。将恢复率设置为 0 表示它将不被绑定。
   
    **`--minrecoveryrate Rate[bBsSkKmMgG]`**
   
   设置最小恢复率以确保 同步 操作的 I/O 获得最小吞吐量，即使存在大量 I/O。
   
   指定比率，格式为“数量/每秒/阵列中的每个设备”。如果没有后缀，选项会假定为 kiB/每秒/每个设备。

2. 显示阵列中未修复的差异的数量，没有修复它们：
   
   ```
   # lvchange --syncaction check vg/raid_lv
   ```

3. 修正阵列中的差异：
   
   ```
   # lvchange --syncaction repair vg/raid_lv
   ```

> **注意**
> 
> `lvchange --syncaction` 修复 操作的功能与 `lvconvert --repair` 操作不同：
> 
> - `lvchange --syncaction` 修复 操作会在阵列上启动后台同步操作。
> - `lvconvert --repair` 操作修复或替换镜像或 RAID 逻辑卷中失败的设备。

4. 可选：显示 scrubbing 操作的信息：
   
   ```
   # lvs -o +raid_sync_action,raid_mismatch_count vg/lv
   ```
- `raid_sync_action` 字段显示 RAID 卷执行的当前同步操作。可以是以下值之一：
  
    **`idle`**
  
      所有同步操作完成（什么都不做）
  
    **`resync`**
  
      初始化阵列或在机器失败后恢复
  
    **`recover`**
  
      替换阵列中的设备
  
    **`check`**
  
      查找阵列的不一致
  
    **`repair`**
  
      查找并修复不一致

- `raid_mismatch_count` 字段显示 检查 操作中出现的差异数。

- `Cpy%Sync` 字段显示同步操作的进度。

- `lv_attr` 字段提供了额外的指标。这个字段中的第 9 位显示逻辑卷的健康状况，它支持以下指示：
  
  - `m` (mismatches)表示 RAID 逻辑卷存在差异。这个字符在 scrubbing 操作侦测到部分 RAID 不一致时就会显示。
  
  - `r` (refresh)表示 RAID 阵列中的某个设备出现故障，并且内核将其认为失败，即使 LVM 可以读取该设备标签，并且认为该设备正常运行。刷新逻辑卷通知内核该设备现在可用 ; 如果您怀疑设备失败，则替换该设备。

## 8.18 转换 RAID 级别（RAID 接管）

LVM 支持 Raid *takeover*（接管），这意味着将 RAID 逻辑卷从一个 RAID 级别转换成另一个 RAID 级别（比如把 RAID 5 转换到 RAID 6）。更改 RAID 级别通常是为了增加或降低设备故障或恢复逻辑卷的弹性。您可以使用 `lvconvert` 进行 RAID 接管。有关 RAID 接管以及使用 `lvconvert` 转换 RAID 逻辑卷的示例，请参阅 `lvmraid(7)man` page。

## 8.19 更改 RAID 卷的属性（RAID reshape）

RAID *reshape* 意味着更改 RAID 逻辑卷的属性，但保持相同的 RAID 级别。您可以修改的一些属性包括 RAID 布局、条状大小和条带数目。有关 RAID 重新哈希以及使用 `lvconvert` 命令重新定义 RAID 逻辑卷的示例，请参阅 `lvmraid(7)man` page。

## 8.20 在 RAID1 逻辑卷中控制 I/O 操作

您可以使用 `lvchange` 命令的 `--write behind` 参数来控制 RAID1 逻辑卷中设备的 I/O 操作。使用这些参数的格式如下。

- `--[RAID]最多写入 物理卷 [:{t|y|n}]`
  
  在 RAID1 逻辑卷中将设备标记为 最多写入。除非需要，避免对这些驱动器的所有读取。设置此参数会使驱动器中的 I/O 操作数量保持最小。默认情况下，逻辑卷中指定物理卷的 `write-mostly` 属性设置为 yes。可以通过在物理卷中附加 `:n` 或通过指定 `:t` 来 切换值来删除大多数写入 标记。在单个命令中，可以最多指定 `--write` 参数，从而一次为逻辑卷中的所有物理卷切换写属性。

- `--[raid]writebehind IOCount`
  
  指定标记为写的 RAID1 逻辑卷中设备允许的最大未处理写入数。超过这个值后，写入就变为同步行为，在阵列指示写操作完成前，所有的写操作都需要已完成。将值设为零可清除首选项，并允许系统任意选择数值。

## 8.21 在 RAID 逻辑卷中更改区域大小

当您创建 RAID 逻辑卷时，逻辑卷的区域大小将是 `/etc/lvm/lvm.conf` 文件中的 `raid_region_size` 参数的值。您可以使用 `lvcreate` 命令的 `-R` 选项覆盖此默认值。

创建 RAID 逻辑卷后，您可以使用 `lvconvert` 命令的 `-R` 选项更改卷的区域大小。以下示例将逻辑卷 `vg/raidlv` 的区域大小改为 4096K。必须同步 RAID 卷以便更改区域大小。

```
# lvconvert -R 4096K vg/raid1
Do you really want to change the region_size 512.00 KiB of LV vg/raid1 to 4.00 MiB? [y/n]: y
  Changed region size on RAID LV vg/raid1 to 4.00 MiB.
```

# 第 9 章 逻辑卷的快照

使用 LVM 快照功能，您可以在特定时刻创建卷(例如 */dev/sda*)的虚拟镜像，而不会导致服务中断。

## 9.1 快照卷的概述

当您制作快照后修改原始卷（源头）时，快照功能会对修改的数据区域创建一个副本来作为更改前的状态，以便可以重建卷的状态。当您创建快照时，需要对原始卷有完全的读写权限。

因为快照只复制创建快照后更改的数据区域，因此快照功能只需要少量的存储。例如，对于很少更新的原始卷，原始容量的 3-5% 就足以进行快照维护。它不提供备份过程的替代品。快照副本是虚拟副本，不是实际的介质备份。

快照的大小控制为存储原始卷的更改而预留的空间量。例如：如果您创建了一个快照，然后完全覆盖了原始卷，则快照应至少与原始卷大小一样方可保存更改。您应该定期监控快照的大小。例如：一个以读为主的短期快照（如 `/usr`）需要的空间小于卷的长期快照，因为它包含很多写入，如 `/home`。

如果快照满了，则快照就会失效，因为它无法跟踪原始卷的变化。但是，您可以将 LVM 配置为每当使用量超过 `snapshot_autoextend_threshold` 值时就自动扩展快照，以免快照失效。快照是完全可调整大小的，您可以执行以下操作：

- 如果您有存储容量，您可以增加快照卷的大小，以防止其被删除。
- 如果快照卷大于您的需要，您可以缩小卷的大小来释放空间，以供其他逻辑卷使用。

快照卷提供以下优点：

- 大多数情况下，当需要对逻辑卷执行备份时制作快照，而不用停止持续更新数据的活动系统。

- 您可以在快照文件系统上执行 `fsck` 命令来检查文件系统的完整性，并确定原始文件系统是否需要文件系统修复。

- 由于快照是读/写的，因此您可以通过制作快照并对快照运行测试来对生产数据测试应用程序，而无需接触真实数据。

- 您可以创建 LVM 卷来用于虚拟化 。您可以使用 LVM 快照来创建虚拟客户端镜像的快照。这些快照可方便修改现有客户虚拟机或者使用最小附加存储创建新客户虚拟机。

## 9.2 创建原始卷的快照

使用 `lvcreate` 命令和 `-s` 或 `--size` 参数，后跟所需的大小来创建原始卷的快照（源头）。卷快照是可写的。默认情况下，与精简配置的快照相比，快照卷在正常激活命令过程中是使用原始卷激活的。LVM 不支持创建大于原始卷大小和卷所需的元数据大小的总和。如果您指定了大于这个总和的快照卷，则 LVM 会创建一个原始卷的大小所需的快照卷。

> **注意**
> 
> 集群中的节点不支持 LVM 快照。您不能在共享卷组中创建快照卷。然而，如果您需要在共享逻辑卷中创建一致的数据备份，您可以单独激活该卷，然后创建快照。

以下流程创建了一个名为 *origin* 的原始逻辑卷和一个名为 *snap* 的原始卷的快照卷。

**前提条件**

- 您已创建了卷组 *vg001*。如需更多信息，请参阅[创建 LVM 卷组](#41-创建-lvm-卷组)。

**流程**

1. 从卷组 *vg001* 创建一个名为 *origin* 的逻辑卷：
   
   ```
   # lvcreate -L 1G -n origin vg001
   Logical volume "origin" created.
   ```

2. 创建 */dev/vg001/origin* 的一个名为 *snap* 的快照逻辑卷，大小为 100 MB ：
   
   ```
   # lvcreate --size 100M --name snap --snapshot /dev/vg001/origin
   Logical volume "snap" created.
   ```
   
   如果原始逻辑卷包含一个文件系统，您可以在任意目录中挂载快照逻辑卷，以便访问文件系统的内容，并在不断更新原始文件系统时进行备份。

3. 显示原始卷以及当前使用的快照卷的百分比：
   
   ```
   # lvs -a -o +devices
   LV      VG    Attr       LSize  Pool Origin Data% Meta% Move Log Cpy%Sync Convert Devices
   origin vg001  owi-a-s---  1.00g                                                  /dev/sde1(0)
   snap vg001  swi-a-s--- 100.00m     origin 0.00                                 /dev/sde1(256)
   ```
   
   您还可以使用 `lvdisplay /dev/vg001/origin` 命令显示逻辑卷 `/dev/vg001/origin` 以及所有快照逻辑卷及其状态，如 active 或 inactive。
   
   > **警告**
   > 
   > 由于快照会随原始卷的变化而增加大小，因此使用 `lvs` 命令定期监控快照卷的百分比非常重要，以确保其不会变满。使用了 100% 的快照会完全丢失，因为对原始卷中未更改的部分的写入无法在不破坏快照的情况下无法成功。

4. 您可以将 LVM 配置为在其使用量超过 `snapshot_autoextend_threshold` 值时自动扩展快照，以避免快照在 100% 满时失效。查看 `/etc/lvm.conf` 文件中 `snapshot_autoextend_threshold` 和 `snapshot_autoextend_percent` 选项的现有值，并根据您的要求编辑它们。
   
    以下示例将 `snapshot_autoextend_threshold` 选项的值设为小于 100 ，将 `snapshot_autoextend_percent` 选项的值设为您需要的值来扩展快照卷：
   
   ```
   # vi /etc/lvm.conf
   snapshot_autoextend_threshold = 70
   snapshot_autoextend_percent = 20
   ```
   
    您还可以通过执行以下命令来手动扩展这个快照：
   
   ```
   # lvextend -L+100M /dev/vg001/snap
   ```

> **注意**
> 
> 这个功能需要卷组中有未分配的空间。快照的自动扩展不会将快照卷的大小增加到超出快照所需的最大计算值。一旦快照增长到足够大来覆盖原始数据后，便不会再监控它是否发生了自动扩展。

## 9.3 将快照合并到其原始卷中

使用 `lvconvert` 命令和 `--merge` 选项，将快照合并到其原始（源头）卷中。如果您丢失了数据或文件，您可以执行系统回滚，否则的话需要将系统恢复到之前的状态。合并快照卷后，得到的逻辑卷具有原始卷的名称、次要号码和 UUID。在合并过程中，对原始卷的读取和写入将会被指向要合并的快照。当合并完成后，会删除合并的快照。

如果原始卷和快照卷都没有打开且处于活跃状态，则合并会立即开始。否则，合并会在原始卷或快照激活后，或两者都关闭后开始。您可以在原始卷激活后，将快照合并到一个不能关闭的原始卷中，如 `root` 文件系统。

**流程**

1. 合并快照卷。以下命令将快照卷 *vg001/snap* 合并到其 *origin* 中：
   
   ```
   # lvconvert --merge vg001/snap
   Merging of volume vg001/snap started.
   vg001/origin: Merged: 100.00%
   ```

2. 查看原始卷：
   
   ```
   # lvs -a -o +devices
   LV      VG    Attr       LSize  Pool Origin Data% Meta% Move Log Cpy%Sync Convert Devices
   origin vg001  owi-a-s---  1.00g     
   ```

# 第 10 章 创建和管理精简配置的卷（精简卷）

OpenCloudOS 支持精简配置的快照卷和逻辑卷。

逻辑卷和快照卷可以是精简配置的：

- 使用精简配置的逻辑卷，您可以创建大于可用物理存储的逻辑卷。

- 使用精简配置的快照卷，您可以在同一个数据卷中存储更多的虚拟设备

## 10.1 精简配置概述

很多现代存储堆栈现在提供在密集配置和精简配置之间进行选择的能力：

- 密集配置提供了块存储的传统行为，其中块的分配与其实际用途无关。

- 精简配置允许置备更大的块存储池，其大小可能大于存储数据的物理设备，从而导致过度配置。过度置备可能是因为单个块在实际使用之前没有被分配。如果您有多个共享同一池的精简置备设备，那么这些设备可以是过度配置的。

通过使用精简配置，您可以超额使用物理存储，且可以管理称为精简池的可用空间池。当应用程序需要时，您可以将这个精简池分配给任意数量的设备。当需要有效分配存储空间时，您可以动态扩展精简池。

例如，如果 10 个用户的每个用户都为他们的应用程序请求一个 100GB 的文件系统，那么您可以为每个用户创建一个 100GB 的文件系统，但其由较少的实际存储支持，仅在需要时使用。

> **注意**
> 
> 在使用精简配置时，监控存储池，并在可用物理空间耗尽时添加更多容量是非常重要的。

以下是使用精简配置的设备的一些优点：

- 您可以创建大于可用物理存储的逻辑卷。

- 您可以将更多的虚拟设备存储在相同的数据卷中。

- 您可以创建逻辑上可自动增长的文件系统，以支持数据需求，并将未使用的块返回给池，以供池中的任意文件系统使用

以下是使用精简配置的设备的潜在缺陷：

- 精简配置的卷存在耗尽可用物理存储的固有风险。如果过度配置了底层存储，您可能会因为缺少可用物理存储而导致停机。例如，如果您创建了 10T 的精简配置的存储，而只有 1T 的物理存储来支持，则卷将在 1T 耗尽后不可用或不可写。

- 如果卷在精简配置的设备后没有向层发送丢弃，那么对使用情况的统计将不准确。例如，在不使用 `-o discard mount` 选项的情况下放置文件系统，且不在精简配置的设备之上定期运行 `fstrim`，则永远不会不分配之前使用的存储。在这种情况下，随着时间的推移，即使没有真正使用它，您最终都会使用全部的置备量。

- 您必须监控逻辑和物理使用情况，以便不会用尽可用的物理空间。

- 在带有快照的文件系统上，写时复制(CoW)操作可能会较慢。

- 数据块可以在多个文件系统之间混合，导致底层存储的随机访问限制，即使它没有向最终用户展示那种方式。

## 10.2 创建精简配置的逻辑卷

使用精简配置的逻辑卷，您可以创建大于可用物理存储的逻辑卷。创建精简配置的卷集允许系统分配您所使用的卷，而不是分配所请求的全部存储。

使用 `lvcreate` 命令的 `-T` 或 `--thin` 选项，您可以创建精简池或精简卷。您还可以使用 `lvcreate` 命令的 `-T` 选项，使用单个命令创建精简池和精简卷。这个流程描述了如何创建和增大精简配置的逻辑卷。

**前提条件**

- 您已创建了一个卷组。如需更多信息，请参阅[创建 LVM 卷组](#41-创建-lvm-卷组)。

**流程**

1. 创建精简池：
   
   ```
   # lvcreate -L 100M -T vg001/mythinpool
   Thin pool volume with chunk size 64.00 KiB can address at most 15.81 TiB of data.
   Logical volume "mythinpool" created.
   ```
   
   请注意：由于您要创建物理空间池，您必须指定池的大小。`lvcreate` 命令的 `-T` 选项不使用参数；它从随命令一起添加的其他选项来确定要创建的设备类型。您还可以使用额外的参数创建精简池，如下面的例子所示：
   
   - 您还可以使用 lvcreate 命令的 --thinpool 参数创建精简池。与 -T 选项不同，--thinpool 参数要求您指定您要创建的精简池逻辑卷的名称。以下示例使用 --thinpool 参数在卷组 vg001 中创建名为 mythinpool 的精简池，大小为 100M ：
     
     ```
     # lvcreate -L 100M --thinpool mythinpool vg001
     Thin pool volume with chunk size 64.00 KiB can address at most 15.81 TiB of data.
     Logical volume "mythinpool" created.
     ```
   
   - 由于创建池时支持条带，所以您可以使用 `-i` 和 `-I` 选项来创建条带。以下命令会在卷组 *vg001* 中创建一个名为 *thinpool* 的 100M 的精简池，其中有两个 64 kB 的条带，以及一个 256 kB 的块。它还会创建一个名为 *vg001/thinvolume* 的 1T 的精简卷。
     
     > **注意**
     > 确保卷组中有两个有足够空闲空间的物理卷，否则您无法创建精简池。
     
     ```
     # lvcreate -i 2 -I 64 -c 256 -L 100M -T vg001/thinpool -V 1T --name thinvolume
     ```

2. 创建精简卷：
   
   ```
   # lvcreate -V 1G -T vg001/mythinpool -n thinvolume
   WARNING: Sum of all thin volume sizes (1.00 GiB) exceeds the size of thin pool vg001/mythinpool (100.00 MiB).
   WARNING: You have not turned on protection against thin pools running out of space.
   WARNING: Set activation/thin_pool_autoextend_threshold below 100 to trigger automatic extension of thin pools before they get full.
   Logical volume "thinvolume" created.
   ```
   
   在这种情况下，您为卷指定了虚拟大小，其大于包含它的池。您还可以使用额外的参数创建精简卷，如下面的例子所示：
   
   - 要创建精简卷和精简池，请使用 `lvcreate` 命令的 `-T` 选项，并指定大小和虚拟大小参数：
     
     ```
     # lvcreate -L 100M -T vg001/mythinpool -V 1G -n thinvolume
     Thin pool volume with chunk size 64.00 KiB can address at most 15.81 TiB of data.
     WARNING: Sum of all thin volume sizes (1.00 GiB) exceeds the size of thin pool vg001/mythinpool (100.00 MiB).
     WARNING: You have not turned on protection against thin pools running out of space.
     WARNING: Set activation/thin_pool_autoextend_threshold below 100 to trigger automatic extension of thin pools before they get full.
     Logical volume "thinvolume" created.
     ```
   
   - 要使用剩余的空闲空间来创建精简卷和精简池，请使用 `100%FREE` 选项：
     
     ```
     # lvcreate -V 1G -l 100%FREE -T vg001/mythinpool -n thinvolume
     Thin pool volume with chunk size 64.00 KiB can address at most <15.88 TiB of data.
     Logical volume "thinvolume" created.
     ```
   
   - 要将现有的逻辑卷转换成精简池卷，请使用 `lvconvert` 命令的 `--thinpool` 参数。您还必须将 `--poolmetadata` 参数与 `--thinpool` 参数结合使用，将现有的逻辑卷转换成精简池卷的元数据卷。
     
     以下示例将卷组 vg001 中的现有逻辑卷 lv1 转换为精简池卷，并将卷组 vg001 中的现有逻辑卷 lv2 转换为那个精简池卷的元数据卷：
     
     ```
     # lvconvert --thinpool vg001/lv1 --poolmetadata vg001/lv2
     Converted vg001/lv1 to thin pool.
     ```
     
     > **注意**
     > 
     > 将逻辑卷转换成精简池卷或者精简池元数据卷会破坏逻辑卷的内容，因为 `lvconvert` 不会保留设备的内容，而是覆盖其内容。
   
   - 默认情况下，`lvcreate` 命令根据以下公式设置精简池的元数据逻辑卷的大小：
     
     ```
     Pool_LV_size / Pool_LV_chunk_size * 64
     ```
     
     如果您有大量快照，或者您的精简池有小的块，并且预计以后精简池的大小会显著增长，那么您可能需要使用 `lvcreate` 命令的 `--poolmetadatasize` 参数来增加精简池的元数据卷的默认值。精简池元数据逻辑卷所支持的值在 2MiB 到 16GiB 之间。
     
     以下示例演示了如何增大精简池的元数据卷的默认值：
     
     ```
     # lvcreate -V 1G -l 100%FREE -T vg001/mythinpool --poolmetadatasize 16M -n thinvolume
     Thin pool volume with chunk size 64.00 KiB can address at most 15.81 TiB of data.
     Logical volume "thinvolume" created.
     ```

3. 查看创建的精简池和精简卷：
   
   ```
   # lvs -a -o +devices
   LV                 VG    Attr       LSize   Pool       Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices
   [lvol0_pmspare]    vg001 ewi-------   4.00m                                                           /dev/sda(0)
   mythinpool         vg001 twi-aotz-- 100.00m                   0.00   10.94                            mythinpool_tdata(0)
   [mythinpool_tdata] vg001 Twi-ao---- 100.00m                                                           /dev/sda(1)
   [mythinpool_tmeta] vg001 ewi-ao----   4.00m                                                           /dev/sda(26)
   thinvolume         vg001 Vwi-a-tz--   1.00g mythinpool        0.00
   ```

4. 可选：使用 `lvextend` 命令扩展精简池的大小。但是您无法缩小精简池的大小。
   
   > **注意**
   > 
   > 在创建精简池和精简卷的过程中，如果您使用 `-l 100%FREE` 参数，这个命令会失败。
   
    以下命令通过扩展 100M 来调整现有大小为 100M 的精简池的大小：
   
   ```
   # lvextend -L+100M vg001/mythinpool
   Size of logical volume vg001/mythinpool_tdata changed from 100.00 MiB (25 extents) to 200.00 MiB (50 extents).
   WARNING: Sum of all thin volume sizes (1.00 GiB) exceeds the size of thin pool vg001/mythinpool (200.00 MiB).
   WARNING: You have not turned on protection against thin pools running out of space.
   WARNING: Set activation/thin_pool_autoextend_threshold below 100 to trigger automatic extension of thin pools before they get full.
   
   Logical volume vg001/mythinpool successfully resized
   ```
   
   ```
   # lvs -a -o +devices
   LV                 VG    Attr       LSize   Pool       Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices
   [lvol0_pmspare]    vg001 ewi-------   4.00m                                                           /dev/sda(0)
   mythinpool         vg001 twi-aotz-- 200.00m                   0.00   10.94                            mythinpool_tdata(0)
   [mythinpool_tdata] vg001 Twi-ao---- 200.00m                                                           /dev/sda(1)
   [mythinpool_tdata] vg001 Twi-ao---- 200.00m                                                           /dev/sda(27)
   [mythinpool_tmeta] vg001 ewi-ao----   4.00m                                                           /dev/sda(26)
   thinvolume         vg001 Vwi-a-tz--   1.00g mythinpool        0.00
   ```

5. 可选：要重命名精简池和精简卷，请使用以下命令：
   
   ```
   # lvrename vg001/mythinpool vg001/mythinpool1
   Renamed "mythinpool" to "mythinpool1" in volume group "vg001"
   
   # lvrename vg001/thinvolume vg001/thinvolume1
   Renamed "thinvolume" to "thinvolume1" in volume group "vg001"
   ```
   
    重命名后查看精简池和精简卷：
   
   ```
   # lvs
   LV          VG       Attr     LSize   Pool       Origin Data%  Move Log Copy%  Convert
   mythinpool1 vg001   twi-a-tz 100.00m                     0.00
   thinvolume1 vg001   Vwi-a-tz   1.00g mythinpool1         0.00
   ```

6. 可选：要删除精简池，请使用以下命令：
   
   ```
   # lvremove -f vg001/mythinpool1
   Logical volume "thinvolume1" successfully removed.
   Logical volume "mythinpool1" successfully removed.
   ```

## 10.3 精简配置的快照卷

OpenCloudOS 支持精简配置的快照卷。精简逻辑卷的快照也会创建一个精简逻辑卷(LV)。精简快照卷具有与其它精简卷相同的特征。您可以独立激活卷、扩展卷、重新命名卷、删除卷、甚至快照卷。

> **注意**
> 
> 与所有 LVM 快照卷以及所有精简卷类似，集群中的节点不支持精简快照卷。快照卷必须在一个集群节点中完全激活。

传统快照必须为创建的每一个快照分配新的空间，其中数据作为对源的修改而被保留。但精简配置的快照与源共享相同的空间。精简 LV 的快照非常有效，因为精简 LV 及它的任何快照的公共数据块是共享的。您可以创建精简 LV 的快照或从其他精简快照创建快照。递归快照的通用块也在精简池中共享。

精简快照卷提供以下优点：

- 增加源的快照数量对性能的影响可以忽略不计。

- 精简快照卷可以减少磁盘用量，因为只有新数据被写入，且不会复制到每个快照中。

- 不需要同时激活精简快照卷和源，这是传统快照的要求。

- 当从快照恢复源时，不需要合并精简快照。您可以删除源，而使用快照。传统的快照有单独的卷，其中存储必须要复制回来的更改，即：合并到源以重置它。

- 与传统的快照相比，对允许的快照数量有很大的限制。

虽然使用精简快照卷有很多好处，但在有些情况下，传统的 LVM 快照卷功能可能更适合您的需要。您可以对所有类型的卷使用传统快照。但是，要使用精简快照则要求您使用精简配置。

> **注意**
> 
> 您不能限制精简快照卷的大小 ; 如有必要，快照将使用精简池中的所有空间。一般说来，在决定使用什么快照格式时，您应该考虑具体的要求。

默认情况下，在通常的激活命令过程中会跳过精简快照卷。

## 10.4 创建精简配置的快照卷

使用精简配置的快照卷，可以在同一数据卷上存储更多的虚拟设备。

> **重要**
> 
> 在创建精简快照卷时，不要指定卷的大小。如果指定了 size 参数，则创建的快照不会是一个精简快照卷，也不会使用精简池来存储数据。例如，命令 `lvcreate -s vg/thinvolume -L10M` 将不会创建精简快照，即使原始卷是精简卷。

可为精简配置的原始卷创建精简快照，也可针对不是精简置备的原始卷创建精简快照。以下流程描述了创建精简配置的快照卷的不同方法。

**前提条件**

- 您已创建了一个精简配置的逻辑卷。如需更多信息，请参阅 [创建精简配置的逻辑卷](#102-创建精简配置的逻辑卷)。

**流程**

- 创建一个精简配置的快照卷。下面的命令在精简配置的逻辑卷 *vg001/thinvolume* 上创建一个名为 *mysnapshot1* 的精简配置的快照卷：
  
  ```
  # lvcreate -s --name mysnapshot1 vg001/thinvolume
  Logical volume "mysnapshot1" created
  ```
  
  ```
  # lvs
  LV          VG       Attr     LSize   Pool       Origin     Data%  Move Log Copy%  Convert
  mysnapshot1 vg001    Vwi-a-tz   1.00g mythinpool thinvolume   0.00
  mythinpool  vg001    twi-a-tz 100.00m                         0.00
  thinvolume  vg001    Vwi-a-tz   1.00g mythinpool              0.00
  ```
  
  > **注意**
  > 
  > 在使用精简配置时，存储管理员务必要监控存储池，并在其被完全占用时添加更多容量。有关扩展精简卷大小的详情，请参考 [创建精简配置的逻辑卷](#102-创建精简配置的逻辑卷)。

- 您还可以为非置备的逻辑卷创建精简配置的快照。因为非精简配置的逻辑卷不包含在精简池中，因此它也被称为外部源。外部原始卷可以被很多精简置备的快照卷使用和共享，即使在不同的精简池中也是如此。在创建精简置备快照时，外部原始源必须不活跃且只读。
  
  以下示例创建一个名为 *origin_volume* 的、只读的、不活动的逻辑卷的精简快照卷。精简快照卷名为 *mythinsnap*。然后，逻辑卷 *origin_volume* 在卷组 *vg001* 中成为精简快照卷 *mythinsnap* 的精简外部源，其使用现有的精简池 *vg001/pool*。原始卷必须与快照卷位于同一个卷组中。在指定原始逻辑卷时不要指定卷组。
  
  ```
  # lvcreate -s --thinpool vg001/pool origin_volume --name mythinsnap
  ```

- 您可以通过执行以下命令，创建第一个快照卷的第二个精简配置的快照卷。
  
  ```
  # lvcreate -s vg001/mysnapshot1 --name mysnapshot2
  Logical volume "mysnapshot2" created.
  ```
  
    要创建第三个精简配置的快照卷，请使用以下命令：
  
  ```
  # lvcreate -s vg001/mysnapshot2 --name mysnapshot3
  Logical volume "mysnapshot3" created.
  ```

**验证**

- 显示精简快照逻辑卷的所有祖先和后代的列表：
  
  ```
  $ lvs -o name,lv_ancestors,lv_descendants vg001
  LV           Ancestors                           Descendants
  mysnapshot2  mysnapshot1,thinvolume              mysnapshot3
  mysnapshot1  thinvolume              mysnapshot2,mysnapshot3
  mysnapshot3  mysnapshot2,mysnapshot1,thinvolume
  mythinpool
  thinvolume                                   mysnapshot1,mysnapshot2,mysnapshot3
  ```
  
    在这里，

- *thinvolume* 是卷组 *vg001* 中的一个原始卷。

- *mysnapshot1* 是 *thinvolume* 的一个快照

- *mysnapshot2* 是 *mysnapshot1* 的一个快照

- *mysnapshot3* 是 *mysnapshot2* 的一个快照
  
  > **注意**
  > 
  > `lv_ancestors` 和 `lv_descendants` 字段显示现有的依赖关系。但是，如果从链中删除了条目，则它们不会跟踪这些条目，因为这些条目可能会破坏依赖关系链。

# 第 11 章 启用缓存来提高逻辑卷性能

您可以在 LVM 逻辑卷中添加缓存以提高性能。LVM 然后使用快速设备（如 SSD）将 I/O 操作缓存到逻辑卷中。

下面的过程会从快速设备创建一个特殊的 LV，并将这个特殊 LV 附加到原始 LV，以便提高性能。

## 11.1 LVM 中的缓存方法

LVM 提供以下缓存类型。每种模式适合逻辑卷中的不同类型的 I/O 模式。

**`dm-cache`**

这个方法可通过在更快的卷上缓存数据来加快频繁使用数据的访问速度。这个方法会缓存读写操作。

`dm-cache` 方法创建类型 `cache` 的逻辑卷。

**`dm-writecache`**

这个方法只缓存写操作。使用快速卷进行写入操作，然后将其迁移到后台较慢的磁盘中。快速卷通常是一个 SSD 或持久内存（PMEM）磁盘。

 `dm-writecache` 方法创建类型为 `writecache` 的逻辑卷。

## 11.2 LVM 缓存组件

当您为逻辑卷启用缓存时，LVM 会重新命名并隐藏原始卷，并显示由原始逻辑卷组成的新逻辑卷。新逻辑卷的组成取决于缓存方法以及您是使用 `cachevol` 还是 `cachepool` 选项。

`cachevol` 和 `cachepool` 选项会公开对缓存组件放置的不同级别的控制：

- 使用 cachevol 选项时，快速设备同时存储数据块的缓存副本以及用于管理缓存的元数据。

- 通过 cachepool 选项，单独的设备可以存储数据块的缓存副本以及用于管理缓存的元数据。
  
    `dm-writecache` 方法与 `cachepool` 不兼容。

在所有配置中，LVM 会公开一个生成的设备，它会将所有缓存组件组合在一起。得到的设备的名称与原来的较慢的逻辑卷的名称相同。

## 11.3 为逻辑卷启用 dm-cache 缓存

这个过程使用 `dm-cache` 方法启用逻辑卷中常用数据的缓存。

**前提条件**

- 系统中存在一个使用 `dm-cache` 加快速度较慢的逻辑卷。
- 包含较慢逻辑卷的卷组还包含在快速块设备中未使用的物理卷。

**流程**

1. 在快速设备中创建 `cachevol` 卷：
   
   ```
   # lvcreate --size cachevol-size --name fastvol vg /dev/fast-pv
   ```
   
    替换以下值：
   
    ***`cachevol-size`***
   
      `cachevol` 卷的大小，如 5G
   
    ***`fastvol`***
   
      `cachevol` 卷的名称
   
    ***`vg`***
   
      卷组名称
   
    **`/dev/fast-pv`**
   
      到快速块设备的路径，如 `/dev/sdf1`

2. 将 `cachevol` 卷附加到主逻辑卷中以开始缓存：
   
   ```
   # lvconvert --type cache --cachevol fastvol vg/main-lv
   ```
   
    替换以下值：
   
    ***`fastvol`***
   
      `cachevol` 卷的名称
   
    ***`vg`***
   
      卷组名称
   
    ***`main-lv`***
   
      较慢的逻辑卷名称

**验证步骤**

- 检查新创建的设备：
  
  ```
  # lvs --all --options +devices vg
  
  LV              Pool           Type   Devices
  main-lv         [fastvol_cvol] cache  main-lv_corig(0)
  [fastvol_cvol]                 linear /dev/fast-pv
  [main-lv_corig]                linear /dev/slow-pv
  ```

## 11.4 使用 cachepool 为逻辑卷启用 dm-cache 缓存

这个过程可让您单独创建缓存数据和缓存元数据逻辑卷，然后将卷合并到缓存池中。

**前提条件**

- 系统中存在一个使用 `dm-cache` 加快速度较慢的逻辑卷。
- 包含较慢逻辑卷的卷组还包含在快速块设备中未使用的物理卷。

**流程**

1. 在快速设备中创建 `cachepool` 卷：
   
   ```
   # lvcreate --type cache-pool --size cachepool-size --name fastpool  vg /dev/fast
   ```
   
    替换以下值：
   
    ***`cachepool-size`***
   
      `cachepool` 的大小，如 5G
   
    ***`fastpool`***
   
      `cachepool` 卷的名称
   
    ***`vg`***
   
      卷组名称
   
    ***`/dev/fast`***
   
      到快速块设备的路径，如 `/dev/sdf1`
   
   > **注意**
   > 
   > 您可以使用 `--poolmetadata` 选项指定创建 `cache-pool` 时池元数据的位置。

2. 将 `cachepool` 附加到主逻辑卷中以开始缓存：
   
   ```
   # lvconvert --type cache --cachepool fastpool vg/main
   ```
   
    替换以下值：
   
    ***`fastpool`***
   
      `cachepool` 卷的名称
   
    ***`vg`***
   
      卷组名称
   
    ***`main`***
   
      较慢的逻辑卷名称

**验证步骤**

- 检查新创建的设备：
  
  ```
  # lvs --all --options +devices vg
  
  LV                      Pool               Type        Devices
  [fastpool_cpool]                           cache-pool  fastpool_pool_cdata(0)
  [fastpool_cpool_cdata]                     linear      /dev/sdf1(4)
  [fastpool_cpool_cmeta]                     linear      /dev/sdf1(2)
  [lvol0_pmspare]                            linear      /dev/sdf1(0)
  main                    [fastpoool_cpool]  cache       main_corig(0)
  [main_corig]                               linear      /dev/sdf1(O)
  ```

## 11.5 为逻辑卷启用 dm-writecache 缓存

这个过程允许使用 `dm-writecache` 方法将 I/O 操作缓存到逻辑卷中。

**前提条件**

- 系统中存在一个使用 `dm-writecache` 加快速度较慢的逻辑卷。
- 包含较慢逻辑卷的卷组还包含在快速块设备中未使用的物理卷。

**流程**

1. 如果一个较慢的逻辑卷是活跃的，取消激活它：
   
   ```
   # lvchange --activate n vg/main-lv
   ```
   
    替换以下值：
   
    ***`vg`***
   
      卷组名称
   
    ***`main-lv`***
   
      较慢的逻辑卷名称

2. 在快速设备中创建未激活的 `cachevol` 卷：
   
   ```
   # lvcreate --activate n --size cachevol-size --name fastvol vg /dev/fast-pv
   ```
   
    替换以下值：
   
    ***`cachevol-size`***
   
      cachevol 卷的大小，如 `5G`
   
    ***`fastvol`***
   
      cachevol 卷的名称
   
    ***`vg`***
   
      卷组名称
   
    ***`/dev/fast-pv`***
   
      到快速块设备的路径，如 `/dev/sdf1`

3. 将 `cachevol` 卷附加到主逻辑卷中以开始缓存：
   
   ```
   # lvconvert --type writecache --cachevol fastvol vg/main-lv
   ```
   
    替换以下值：
   
    ***`fastvol`***
   
      cachevol 卷的名称
   
    ***`vg`***
   
      卷组名称
   
    ***`main-lv`***
   
      较慢的逻辑卷名称

4. 激活生成的逻辑卷：
   
   ```
   # lvchange --activate y vg/main-lv
   ```
   
    替换以下值：
   
    ***`vg`***
   
      卷组名称
   
    ***`main-lv`***
   
      较慢的逻辑卷名称

**验证步骤**

- 检查新创建的设备：
  
  ```
  # lvs --all --options +devices vg
  
  LV                VG Attr       LSize   Pool           Origin           Data%  Meta%  Move Log Cpy%Sync Convert Devices
  main-lv          vg Cwi-a-C--- 500.00m [fastvol_cvol] [main-lv_wcorig] 0.00                                    main-lv_wcorig(0)
  [fastvol_cvol]   vg Cwi-aoC--- 252.00m                                                                         /dev/sdc1(0)
  [main-lv_wcorig] vg owi-aoC--- 500.00m 
  ```

## 11.6 为逻辑卷禁用缓存

这个过程禁用了当前在逻辑卷中启用的 `dm-cache` 或 `dm-writecache` 缓存。

**前提条件**

- 在逻辑卷中启用了缓存。

**流程**

1. 取消激活逻辑卷：
   
   ```
   # lvchange --activate n vg/main-lv
   ```
   
    替换以下值：
   
    ***`vg`***
   
      卷组名称
   
    ***`main-lv`***
   
      启用缓存的逻辑卷名称

2. 分离 cachevol 或 cachepool 卷：
   
   ```
   # lvconvert --splitcache vg/main-lv
   ```
   
    替换以下值：
   
    ***`vg`***
   
      卷组名称
   
    ***`main-lv`***
   
      启用缓存的逻辑卷名称

**验证步骤**

- 检查逻辑卷不再连接在一起：
  
  ```
  # lvs --all --options +devices [replaceable]_vg_
  
  LV      Attr       Type   Devices
  fastvol -wi------- linear /dev/fast-pv
  main-lv -wi------- linear /dev/slow-pv
  ```

# 第 12 章 逻辑卷激活

处于活跃状态的逻辑卷可以通过块设备使用。激活的逻辑卷可被访问，并可能有变化。当您创建一个逻辑卷时，它会被默认激活。

有一些情况需要您单独激活逻辑卷，使其在内核中是未知的。您可以使用 `lvchange` 命令的 `-a` 选项激活或取消激活独立逻辑卷。

取消激活独立逻辑卷的命令格式如下。

```
lvchange -an vg/lv
```

激活独立逻辑卷的命令格式如下。

```
lvchange -ay vg/lv
```

您可以使用 `vgchange` 命令的 `-a` 选项激活或停用卷组中的所有逻辑卷。这等同于在卷组中的每个逻辑卷上运行 `lvchange -a` 命令。

取消激活卷组中所有逻辑卷的命令格式如下。

```
vgchange -an vg
```

激活卷组中所有逻辑卷的命令格式如下。

```
vgchange -ay vg
```

## 12.1 控制逻辑卷的自动激活

自动激活逻辑卷指的是，在系统启动时基于事件自动激活逻辑卷。当设备在系统上可用时（设备在线事件），`systemd/udev` 会运行每个设备的 `lvm2-pvscan` 服务。此服务运行 `pvscan --cache -aay device` 命令，该命令可读取命名设备。如果该设备属于卷组，则 `pvscan` 命令将检查系统中是否存在该卷组的所有物理卷。如果是这样，该命令将在那个卷组中激活逻辑卷。

您可以使用 `/etc/lvm/lvm.conf` 配置文件中的 以下配置选项来控制逻辑卷的自动激活。

- `global/event_activation`
  
    禁用 `event_activation` 时，`systemd/udev` 只会在系统启动时自动激活物理卷。如果还没有出现所有物理卷，那么可能不会自动激活一些逻辑卷。

- `activation/auto_activation_volume_list`
  
    将 `auto_activation_volume_list` 设置为空列表可完全禁用自动激活。将 `auto_activation_volume_list` 设置为特定的逻辑卷和卷组将自动激活限制为那些逻辑卷。

有关设置这些选项的详情请参考 `/etc/lvm/lvm.conf` 配置文件。

## 12.2 控制逻辑卷激活

您可以使用以下方法控制逻辑卷的激活：

- 通过 `/etc/lvm/conf` 文件中的 `activation/volume_list` 设置。这可让您指定激活哪些逻辑卷。有关使用此选项的详情请参考 `/etc/lvm/lvm.conf` 配置文件。
- 逻辑卷的激活跳过标签。当为逻辑卷设定这个标签时，会在正常的激活命令中跳过该卷。

您可以用下列方法在逻辑卷中设定激活跳过标签。

- 您可以在创建逻辑卷时关闭激活跳过标签，方法是指定 `lvcreate` 命令的 -kn 或 `--setactivationskip n` 选项。
- 您可以通过指定 lvchange 命令的 `-kn` 或 `--setactivationskip n` 选项来关闭现有逻辑卷的激活 跳过标签。
- 您可以为使用 lvchange 命令的 `-ky` 或 `--setactivationskip y` 选项关闭的卷再次打开激活 跳过标志。

要确定是否为逻辑卷设置了激活跳过标签，请运行 `lv`s 命令，该命令会显示 `k` 属性，如下例所示。

```
# lvs vg/thin1s1
LV         VG  Attr       LSize Pool  Origin
thin1s1    vg  Vwi---tz-k 1.00t pool0 thin1
```

除了标准 `-ay` 或 -`-activate y` 选项外，您还可以使用 `-K` 或 `--ignoreactivationskip` 选项激活设置 `k` 属性的逻辑卷。

默认情况下，精简快照卷在创建时将其标记为激活跳过。您可以使用 `/etc/lvm/lvm.conf` 文件中的 `auto_set_activation_skip` 设置来控制新精简快照卷上的默认激活 跳过设置。

下面的命令激活设定了激活跳过标签的精简快照逻辑卷。

```
# lvchange -ay -K VG/SnapLV
```

以下命令在没有激活跳过标签的情况下创建精简快照

```
# lvcreate --type thin -n SnapLV -kn -s ThinLV --thinpool VG/ThinPoolLV
```

下面的命令可从快照逻辑卷中删除激活跳过标签。

```
# lvchange -kn VG/SnapLV
```

## 12.3 激活共享逻辑卷

您可以使用 `lvchange` 和 `vgchange` 命令的 `-a` 选项控制共享逻辑卷的逻辑卷激活，如下所示：

| **命令**          | **激活**                                                                                                                               |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `lvchange -ay   | e`                                                                                                                                   |
| `lvchange -asy` | 以共享模式激活共享逻辑卷，允许多个主机同时激活逻辑卷。如果激活失败，如逻辑卷只在另一个主机中激活时一样，会出错。如果逻辑类型禁止共享访问，比如快照，命令将报告错误并失败。无法从多个主机同时使用的逻辑卷类型包括 thin、cache、raid 和 snapshot。 |
| `lvchange -an`  | 取消激活逻辑卷。                                                                                                                             |

## 12.4 在缺少设备的情况下激活逻辑卷

您可以通过将 `lvchange` 命令的 `activation_mode` 参数设置为以下值之一来配置在缺少设备的情况下激活的逻辑卷。

| **激活模式** | **含义**                           |
| -------- | -------------------------------- |
| complete | 只允许激活没有缺失物理卷的逻辑卷。这是限制性最强的模式。     |
| degraded | 允许激活含有缺失物理卷的 RAID 逻辑卷。           |
| partial  | 允许激活任何含有缺失物理卷的逻辑卷。这个选项只应用于恢复或修复。 |

`activation_mode` 的默认值由 `/etc/lvm/lvm.conf` 文件中的 `activation_mode` 设置决定。如需更多信息，请参阅 `lvmraid(7)man` page。

# 第 13 章 控制 LVM 设备扫描

您可以通过在 `/etc/lvm/lvm.conf` 文件中配置过滤器来控制 LVM 设备扫描。`lvm.conf` 文件中的过滤器包含一系列简单的正则表达式，这些表达式应用到 `/dev` 目录中的设备名称，以决定是否接受或拒绝找到的每个块设备。

## 13.1 LVM 设备过滤器

LVM 工具扫描 `/dev` 目录中的设备，并检查这里的 LVM 元数据的每个设备。`/etc/lvm/lvm.conf` 文件中的过滤器控制 LVM 扫描的设备。

过滤器是 LVM 通过扫描 `/dev` 目录或 `/etc/lvm/lvm.conf` 文件中的 `dir` 关键字指定的目录，应用于每个设备的模式列表。模式是正则表达式，用任何字符分隔，并在前面加上 `a` 表示接受或 `r` 表示 拒绝。匹配设备的列表中的第一个正则表达式决定了 LVM 接受还是拒绝（忽略）该设备。LVM 接受与任何模式不匹配的设备。

以下是该过滤器的默认配置，可扫描所有设备：

```
filter = [ "a/.*/" ]
```

## 13.2 LVM 设备过滤器配置示例

下面的例子显示使用过滤器控制 LVM 扫描设备的过滤器。

> **警告**
> 
> 这里给出的一些示例可能意外地匹配系统中的其他设备，且可能不适合您的具体系统。例如： `/loop/` 等同于 `a/.*loop.*/`，并匹配 `/dev/solooperation/lvol1`。

- 下面的过滤器添加所有发现的设备，这是配置文件中没有配置过滤器的默认行为：
  
  ```
  filter = [ "a/.*/" ]
  ```

- 下面的过滤器会删除 `cdrom` 设备，以避免在驱动器中没有介质时出现延迟：
  
  ```
  filter = [ "r|^/dev/cdrom$|" ]
  ```

- 下面的过滤器添加所有回送设备并删除所有其他块设备：
  
  ```
  filter = [ "a/loop/", "r/.*/" ]
  ```

- 下面的过滤器添加所有回路设备和 IDE 设备，同时删除所有其他块设备：
  
  ```
  filter = [ "a|loop|", "a|/dev/hd.*|", "r|.*|" ]
  ```

- 下面的过滤器只添加第一个 IDE 驱动器中的分区 8，同时删除所有其它块设备：
  
  ```
  filter = [ "a|^/dev/hda8$|", "r/.*/" ]
  ```

## 13.3 应用 LVM 设备过滤器配置

这个步骤更改了控制 LVM 扫描设备的 LVM 设备过滤器的配置。

**前提条件**

- 准备要使用的设备过滤器特征。

**流程**

1. 在不修改 `/etc/lvm/lvm.conf` 文件的情况下测试设备过滤器特征。
   
   使用带 `--config 'devices{ filter = [ your device filter pattern ] }'` 选项的 LVM 命令。例如：
   
   ```
   # lvs --config 'devices{ filter = [ "a|/dev/emcpower.*|", "r|.*|" ] }'
   ```

2. 编辑 `/etc/lvm/lvm.conf` 配置文件中的 `filter` 选项，以使用您的新设备过滤器模式。

3. 检查新配置是否缺少您要使用的物理卷或卷组：
   
   ```
   # pvscan
   ```
   
   ```
   # vgscan
   ```

4. 重建 `initramfs` 文件系统，以便 LVM 重启时只扫描所需的设备：
   
   ```
   # dracut --force --verbose
   ```

# 第 14 章 控制 LVM 分配

默认情况下，卷组根据常识分配物理扩展，比如不会将平行条带放在同一个物理卷中。这是 `normal` 分配策略。您可以使用 `vgcreate` 命令的 `--alloc` 参数来指定 `contiguous` 、`anywhere` 或 `cling` 的分配策略 。通常，只有在需要指定异常和非标准扩展分配的特殊情况下，才需要分配除 `normal` 以外的策略。

## 14.1 LVM 分配策略

当 LVM 操作需要为一个或多个逻辑卷分配物理扩展时，分配过程如下：

- 生成卷组中未分配的完整物理扩展集合供考虑。如果您在命令行末尾提供任意物理扩展范围，则只考虑指定物理卷中的未分配物理扩展。

- 每个分配策略都会依次尝试，从最严格的策略（`contiguous`）开始，以通过 `--alloc` 选项指定的分配策略或设置为特定逻辑卷或卷组的默认分配策略结尾。对于每个策略，使用需要填充的空逻辑卷空间的最小数值逻辑扩展进行工作，并尽量根据分配策略实施的限制分配空间。如果需要更多空间，LVM 会进入下一个策略。

分配策略的限制如下：

- 连续的分配策略 要求 任何逻辑扩展的物理位置（不是逻辑卷的第一个逻辑扩展）与它前面的逻辑扩展的物理位置相邻。
  
  当逻辑卷条状化或镜像时，相邻 分配限制将独立应用于需要空间的每个条带或镜像镜像(leg)。

- 大写的分配策略 `cling` 将用于任何逻辑扩展的物理卷添加至该逻辑卷中至少一个逻辑扩展已使用的现有逻辑卷中。如果同时定义了配置参数 `allocation/cling_tag_list`，那么如果两个物理卷上都存在列出的标签，则两个物理卷将被视为匹配。这允许对有类似属性（比如其物理位置）的物理卷组进行标记并视为分配的目的。
  
  当逻辑卷被条状化或镜像时，`cling` 分配限制将单独应用于每个需要空间的条状或镜像映像(leg)。

- `normal` 的分配策略不会选择将相同的物理卷共享为已分配给并行逻辑卷（即并行逻辑卷中相同偏移的逻辑扩展）的物理扩展。
  
  当与逻辑卷同时分配镜像日志来保存镜像数据时，`normal` 的分配策略首先会尝试为日志和数据选择不同的物理卷。如果不可能，并且 `allocation/mirror_logs_require_separate_pvs` 配置参数设置为 0，那么它将允许日志与部分数据共享物理卷。
  
  类似地，分配精简池元数据时，`normal` 的分配策略将遵循与分配镜像日志相同的注意事项，具体取决于 `allocation/thin_pool_metadata_require_separate_pvs` 配置参数的值。

- 如果有足够的可用扩展来满足分配请求，但 `normal` 分配策略不使用它们，那么 `anywhere` 分配策略都会使用它们，即使这会通过将两个条带放在同一个物理卷中来降低性能。

可以使用 `vgchange` 命令更改分配策略。

> **注意**
> 
> 如果您使用没有包括在此文档中的分配策略，应该注意，它们的行为在将来的版本中可能会改变。例如：如果您在命令行中提供两个空物理卷，它们有相同数量的可用物理扩展可用于分配，LVM 当前会以它们列出的顺序处理它们，但不保证在将来的版本中这个行为不会有变化。如果为特定逻辑卷获取特定的布局非常重要，那么您应该通过 `lvcreate` 和 `lvconvert` 步骤构建它，以便应用到每个步骤的分配策略使 LVM 无法在布局上自由裁量。

要查看分配过程在任何特定情况下的工作方式，您可以读取调试日志输出，例如，将 `-vvvv` 选项添加到命令。

## 14.2 防止在物理卷中分配

您可以使用 `pvchange` 命令防止在一个或多个物理卷的可用空间上分配物理扩展。这在出现磁盘错误或者要删除物理卷时是必需的。

以下命令不允许在 `/dev/sdk1` 中分配物理扩展。

```
# pvchange -x n /dev/sdk1
```

您还可以使用 `pvchange` 命令的 `-xy` 参数来允许在之前禁止进行分配的地方进行分配。

## 14.3 使用 cling 分配 策略扩展逻辑卷

在扩展 LVM 卷时，您可以使用 `lvextend` 命令的 `--alloc cling` 选项指定 `cling` 分配策略。这个策略将选择同一物理卷中的空间作为现有逻辑卷的最后区段。如果物理卷空间不足，并且在 `/etc/lvm/lvm.conf` 文件中定义了标签列表，LVM 将检查是否将任何标签附加到物理卷中，并尝试在现有扩展和新扩展之间匹配这些物理卷标签。

例如，如果您的逻辑卷在一个卷组中的两个站点之间镜像，您可以根据物理卷的位置标记它们，方法是使用 `@site1` 和` @site 2` 标签标记物理卷。然后您可以在 `lvm.conf` 文件中指定以下行：

```
cling_tag_list = [ "@site1", "@site2" ]
```

在以下示例中，`lvm.conf` 文件已被修改为包含以下行：

```
cling_tag_list = [ "@A", "@B" ]
```

本例中还创建了由物理卷 `/dev/sdb1`、`/dev/sdc1`、`/dev/sdd1`、`/dev/sde1`、`/dev/sde1`、`/dev/sdf1`、`/dev/sdg1` 和 `/dev/sdh1` 组成的卷组 `taft`。这些物理卷标有标签 `A`、`B` 和 `C`。这个示例没有使用 `C` 标签，但这会显示 LVM 使用标签来选择要用于镜像分支的物理卷。

```
# pvs -a -o +pv_tags /dev/sd[bcdefgh]
  PV         VG   Fmt  Attr PSize  PFree  PV Tags
  /dev/sdb1  taft lvm2 a--  15.00g 15.00g A
  /dev/sdc1  taft lvm2 a--  15.00g 15.00g B
  /dev/sdd1  taft lvm2 a--  15.00g 15.00g B
  /dev/sde1  taft lvm2 a--  15.00g 15.00g C
  /dev/sdf1  taft lvm2 a--  15.00g 15.00g C
  /dev/sdg1  taft lvm2 a--  15.00g 15.00g A
  /dev/sdh1  taft lvm2 a--  15.00g 15.00g A
```

以下命令从卷组 `taft` 创建一个 10GB 镜像卷。

```
# lvcreate --type raid1 -m 1 -n mirror --nosync -L 10G taft
  WARNING: New raid1 won't be synchronised. Don't read what you didn't write!
  Logical volume "mirror" created
```

以下命令显示使用哪些设备作为镜像分支和 RAID 元数据子卷。

```
# lvs -a -o +devices
  LV                VG   Attr       LSize  Log Cpy%Sync Devices
  mirror            taft Rwi-a-r--- 10.00g       100.00 mirror_rimage_0(0),mirror_rimage_1(0)
  [mirror_rimage_0] taft iwi-aor--- 10.00g              /dev/sdb1(1)
  [mirror_rimage_1] taft iwi-aor--- 10.00g              /dev/sdc1(1)
  [mirror_rmeta_0]  taft ewi-aor---  4.00m              /dev/sdb1(0)
  [mirror_rmeta_1]  taft ewi-aor---  4.00m              /dev/sdc1(0)
```

以下命令扩展了镜像卷的大小，使用 `cling` 分配策略指示应使用具有相同标签的物理卷扩展镜像分支。

```
# lvextend --alloc cling -L +10G taft/mirror
  Extending 2 mirror images.
  Extending logical volume mirror to 20.00 GiB
  Logical volume mirror successfully resized
```

以下显示命令显示已使用带有与图图相同标签的物理卷扩展了镜像分支。请注意，忽略了标签为 `C` 的物理卷。

```
# lvs -a -o +devices
  LV                VG   Attr       LSize  Log Cpy%Sync Devices
  mirror            taft Rwi-a-r--- 20.00g       100.00 mirror_rimage_0(0),mirror_rimage_1(0)
  [mirror_rimage_0] taft iwi-aor--- 20.00g              /dev/sdb1(1)
  [mirror_rimage_0] taft iwi-aor--- 20.00g              /dev/sdg1(0)
  [mirror_rimage_1] taft iwi-aor--- 20.00g              /dev/sdc1(1)
  [mirror_rimage_1] taft iwi-aor--- 20.00g              /dev/sdd1(0)
  [mirror_rmeta_0]  taft ewi-aor---  4.00m              /dev/sdb1(0)
  [mirror_rmeta_1]  taft ewi-aor---  4.00m              /dev/sdc1(0)
```

## 14.4 使用标签区分 LVM RAID 对象

您可以为 LVM RAID 对象分配标签分组，以便您可以按组自动控制 LVM RAID 行为（如激活）。

物理卷(PV)标签负责 LVM raid 中的分配控制，而不是逻辑卷(LV)或卷组(VG)标签，因为 lvm 中的分配根据分配策略在 PV 级别进行。为了通过不同的属性来区分存储类型，请对它们进行适当的标记（例如，NVMe、SSD、HDD）。建议您在将每个新 PV 添加到 VG 后对其进行适当的标记。

这个过程在逻辑卷中添加对象标签（假设 `/dev/sda` 是 SSD），`/dev/sd[b-f]` 是带有一个分区的 HDD。

**前提条件**

- 已安装 `lvm2` 软件包。
- 可用作 PV 的存储设备可用。

**流程**

1. 创建卷组。
   
   ```
   # vgcreate MyVG /dev/sd[a-f]1
   ```

2. 添加标签到您的物理卷。
   
   ```
   # pvchange --addtag ssds /dev/sda1
   
   # pvchange --addtag hdds /dev/sd[b-f]1
   ```

3. 创建 RAID6 逻辑卷。
   
   ```
   # lvcreate --type raid6 --stripes 3 -L1G -nr6 MyVG @hdds
   ```

4. 创建线性缓存池卷。
   
   ```
   # lvcreate -nr6pool -L512m MyVG @ssds
   ```

5. 将 RAID6 卷转换为要缓存的 RAID6 卷。
   
   ```
   # lvconvert --type cache --cachevol MyVG/r6pool MyVG/r6
   ```

# 第 15 章 使用标签对 LVM 对象进行分组

作为系统管理员，您可以为 LVM 对象分配标签分组，以便您可以按组自动控制 LVM 行为（如激活）。

## 15.1 LVM 对象标签

LVM 标签是一个单词，用于将相同类型的 LVM2 对象分组在一起。标签附加到物理卷、卷组和逻辑卷等对象，以及群集配置中的主机。

在命令行中指定标签代替 PV、VG 或 LV 参数。标签应加上 @ 前缀，以避免混淆。通过替换每个标签的所有对象，将标签替换为具有该标签（根据其在命令行中的位置所预期的标签），即可扩展各个标签。

LVM 标签是最多 1024 个字符的字符串。LVM 标签不能以连字符开头。

有效标签仅包含有限的字符范围。允许的字符是 `A-Z a-z 0-9 _ + . - / = ! : # &`。

只有卷组中的对象可以添加标记。如果从卷组中删除物理卷，物理卷会丢失标签；这是因为标签作为卷组元数据的一部分存储，并在删除物理卷时被删除。

## 15.2 列出 LVM 标签

以下示例演示了如何列出 LVM 标签。

**流程**

- 使用以下命令列出带有 `database` 标签的所有逻辑卷：
  
  ```
  # lvs @database
  ```

- 使用以下命令列出当前活跃的主机标签：
  
  ```
  # lvm tags
  ```

## 15.3 添加 LVM 对象标签

这个步骤描述了如何添加 LVM 对象标签。

**前提条件**

- 已安装 `lvm2` 软件包。
- 创建一个或多个物理卷、卷组或逻辑卷。

**流程**

- 要创建对象标签，请在 LVM 命令中添加 --addtag 选项：
  
  - 要从物理卷创建标签，可在 `pvchange` 命令中添加 选项。
  
  - 要从卷组创建标签，请将 选项添加到 `vgchange` 或 `vgcreate` 命令。
  
  - 要从逻辑卷创建标签，请在 `lvchange` 或 `lvcreate` 命令中添加 选项。

## 15.4 删除 LVM 对象标签

这个步骤描述了如何删除 LVM 对象标签。

**前提条件**

- 已安装 lvm2 软件包。
- 在物理卷、卷组或逻辑卷上创建对象标签。

**流程**

- 要删除对象标签，在 LVM 命令中添加 `--deltag` 选项：
  
  - 要从物理卷中删除标签，请在 `pvchange` 命令中添加 选项。
  - 要从卷组中删除标签，请在 `vgchange` 或 `vgcreate` 命令中添加 选项。
  - 要从逻辑卷中删除标签，请在 `lvchange` 或 `lvcreate` 命令中添加 选项。

## 15.5 定义 LVM 主机标签

这个步骤描述了如何在集群配置中定义 LVM 主机标签。您可以在配置文件中定义主机标签。

**流程**

- 在 `tags` 部分中设置 `hosttags = 1`，以使用计算机的主机名自动定义主机标签。
  
    这样，您可以使用可在所有计算机上复制的通用配置文件，以便存放该文件的相同副本，但根据主机名，计算机的行为可能会有所不同。

对于每个主机标签，读取一个额外的配置文件（如果存在）：`lvm_hosttag.conf`。如果该文件定义了新标签，则进一步的配置文件将附加到要读取的文件列表中。

例如，配置文件中的以下条目始终定义 `tag1`，如果主机名是 `host1`，则定义 `tag2` ：

```
tags { tag1 { }  tag2 { host_list = ["host1"] } }
```

## 15.6 使用标签控制逻辑卷激活

这个步骤描述了如何在配置文件中指定在该主机上只激活某些逻辑卷。

**前提条件**

- 在用户在装配之后开始之前必须满足的附带条件列表。
- 您还可以链接到其他模块或用户在启动这个组件前必须遵循的集合。
- 如果没有先决条件，请删除部分标题和要点。

**流程**

例如，以下条目充当激活请求的过滤器（如 `vgchange -ay`），并且仅激活 `vg1/lvol0` 以及该主机上元数据中的 `datbase` 标签的任何逻辑卷或卷组：

```
activation { volume_list = ["vg1/lvol0", "@database" ] }
```

特殊匹配 `@*` 仅当任何元数据标签与该计算机上的任何主机标签匹配时才会出现匹配。

再举一个例子，请考虑集群中的每台机器在配置文件中都有以下条目的情况：

```
tags { hosttags = 1 }
```

如果您只想在主机 `db2` 上激活 `vg1/lvol2`，请执行以下操作：

1. 从群集中的任何主机运行 `lvchange --addtag @db2 vg1/lvol2`。

2. 运行 `lvchange -ay vg1/lvol2`.

此解决方案涉及将主机名存储在卷组元数据中。

# 第 16 章 LVM 故障排除

您可以使用 LVM 工具排除 LVM 卷和组群中的问题。

## 16.1 在 LVM 中收集诊断数据

如果 LVM 命令没有按预期工作，您可以使用以下方法收集诊断信息。

**流程**

- 使用以下方法收集不同类型的诊断数据：
  
  - 向任何 LVM 命令添加 -v 参数，以提高命令输出的详细程度。添加更多的 v 会进一步增加输出的详细程度。最多允许 4 个这样的 v，例如 -vvvv。
  
  - 在 /etc/lvm/lvm.conf 配置文件的 log 部分中，增加 level 选项的值。这会导致 LVM 在系统日志中提供更多详情。
  
  - 如果问题与逻辑卷激活有关，请启用 LVM 在激活过程中记录信息：
    
    1. 在 /etc/lvm/lvm.conf 配置文件的 log 部分中设置 activation = 1 选项。
    
    2. 使用 -vvvv 选项执行 LVM 命令。
    
    3. 检查命令输出。
    
    4. 将 activation 选项重置为 0。
       
        如果您没有将 选项重置为 0，则系统在内存不足时可能会变得无响应。
  
  - 为诊断显示信息转储：
    
    ```
    # lvmdump
    ```
  
  - 显示附加系统信息：
    
    ```
    # lvs -v
    ```
    
    ```
    # pvs --all
    ```
    
    ```
    # dmsetup info --columns
    ```
  
  - 检查 `/etc/lvm/backup/` 目录中的最后一个 LVM 元数据备份，并在 `/etc/lvm/archive/` 目录中检查存档版本。
  
  - 检查当前的配置信息：
    
    ```
    # lvmconfig
    ```
  
  - 检查 `/run/lvm/hints` 缓存文件以获取哪些设备上具有物理卷的记录。

## 16.2 显示失败的 LVM 设备的信息

您可以显示一个失败的 LVM 卷的信息，以便帮助您确定为什么这个卷失败的原因。

**流程**

- 使用 `vgs` 或 `lvs` 实用程序显示失败的卷。
  
  > **例 16.1 失败的卷组**
  > 
  > 在本例中，组成卷组 myvg 的设备之一失败。卷组不可用，但您可以看到有关失败设备的信息。
  > 
  > ```
  > # vgs --options +devices
  > /dev/vdb1: open failed: No such device or address
  > /dev/vdb1: open failed: No such device or address
  > WARNING: Couldn't find device with uuid 42B7bu-YCMp-CEVD-CmKH-2rk6-fiO9-z1lf4s.
  > WARNING: VG myvg is missing PV 42B7bu-YCMp-CEVD-CmKH-2rk6-fiO9-z1lf4s (last written to /dev/sdb1).
  > WARNING: Couldn't find all devices for LV myvg/mylv while checking used and assumed devices.
  > 
  > VG    #PV #LV #SN Attr   VSize  VFree  Devices
  > myvg   2   2   0 wz-pn- <3.64t <3.60t [unknown](0)
  > myvg   2   2   0 wz-pn- <3.64t <3.60t [unknown](5120),/dev/vdb1(0)
  > ```
  
  > **例 16.2 逻辑卷失败**
  > 
  > 在这个示例中，其中一个设备会失败，因为卷组中的逻辑卷会失败。命令输出显示失败的逻辑卷。
  > 
  > ```
  > # lvs --all --options +devices
  > 
  > /dev/vdb1: open failed: No such device or address
  > /dev/vdb1: open failed: No such device or address
  > WARNING: Couldn't find device with uuid 42B7bu-YCMp-CEVD-CmKH-2rk6-fiO9-z1lf4s.
  > WARNING: VG myvg is missing PV 42B7bu-YCMp-CEVD-CmKH-2rk6-fiO9-z1lf4s (last written to /dev/sdb1).
  > WARNING: Couldn't find all devices for LV myvg/mylv while checking used and assumed devices.
  > 
  > LV    VG  Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices
  > mylv myvg -wi-a---p- 20.00g                                                     [unknown](0)                                                 [unknown](5120),/dev/sdc1(0)
  > ```
  
  > **例 16.3 镜像逻辑卷的失败分支（leg）**
  > 
  > 以下示例显示当镜像逻辑卷的一个分支失败时 vgs 和 lv s 实用程序的命令输出失败。
  > 
  > ```
  > # vgs --all --options +devices
  > 
  > VG    #PV #LV #SN Attr   VSize VFree Devices
  > corey 4 4 0 rz-pnc 1.58T 1.34T my_mirror_mimage_0(0),my_mirror_mimage_1(0)
  > corey 4 4 0 rz-pnc 1.58T 1.34T /dev/sdd1(0)
  > corey 4 4 0 rz-pnc 1.58T 1.34T unknown device(0)
  > corey 4 4 0 rz-pnc 1.58T 1.34T /dev/sdb1(0)
  > ```
  > 
  > ```
  > # lvs --all --options +devices
  > 
  > LV                   VG    Attr   LSize   Origin Snap%  Move Log            Copy%  Devices
  > my_mirror corey mwi-a- 120.00G my_mirror_mlog 1.95 my_mirror_mimage_0(0),my_mirror_mimage_1(0)
  > [my_mirror_mimage_0] corey iwi-ao 120.00G unknown device(0)
  > [my_mirror_mimage_1] corey iwi-ao 120.00G /dev/sdb1(0)
  > [my_mirror_mlog] corey lwi-ao 4.00M /dev/sdd1(0)
  > ```

## 16.3 从卷组中删除丢失的 LVM 物理卷

如果物理卷失败，您可以激活卷组中剩余的物理卷，并从卷组中删除所有使用该物理卷的逻辑卷。

**流程**

1. 激活卷组中剩余的物理卷：
   
   ```
   # vgchange --activate y --partial myvg
   ```

2. 检查要删除哪些逻辑卷：
   
   ```
   # vgreduce --removemissing --test myvg
   ```

3. 从卷组中删除所有使用丢失的物理卷的逻辑卷：
   
   ```
   # vgreduce --removemissing --force myvg
   ```

4. 可选：如果您意外删除了您要保留的逻辑卷，您可以撤销 `vgreduce` 操作：
   
   ```
   # vgcfgrestore myvg
   ```

> **警告**
> 
> 如果您删除了精简池，LVM 无法撤销操作。

## 16.4 查找丢失的 LVM 物理卷的元数据

如果意外覆盖或者破坏了卷组物理卷元数据区域，您会得到出错信息表示元数据区域不正确，或者系统无法使用特定的 UUID 找到物理卷。

这个过程找到丢失或者损坏的物理卷的最新归档元数据。

**流程**

1. 查找包含物理卷的卷组元数据文件。归档的元数据文件位于 `/etc/lvm/archive/volume-group-name_backup-number.vg` 路径中：
   
   ```
   # cat /etc/lvm/archive/myvg_00000-1248998876.vg
   ```
   
    使用备份号替换 *00000-1248998876*。选择该卷组最高数字最后已知的有效元数据文件。

2. 找到物理卷的 UUID。使用以下任一方法。
   
   - 列出逻辑卷：
     
     ```
     # lvs --all --options +devices
     
     Couldn't find device with uuid 'FmGRh3-zhok-iVI8-7qTD-S5BI-MAEN-NYM5Sk'.
     ```
   
   - 检查归档的元数据文件。在卷组配置的 `physical_volumes` 部分中，找到标记为 `id =` 的 UUID。
   
   - 使用 `--partial` 选项取消激活卷组：
     
     ```
     # vgchange --activate n --partial myvg
     
     PARTIAL MODE. Incomplete logical volumes will be processed.
     WARNING: Couldn't find device with uuid 42B7bu-YCMp-CEVD-CmKH-2rk6-fiO9-z1lf4s.
     WARNING: VG myvg is missing PV 42B7bu-YCMp-CEVD-CmKH-2rk6-fiO9-z1lf4s (last written to /dev/vdb1).
     0 logical volume(s) in volume group "myvg" now active
     ```

## 16.5 在 LVM 物理卷中恢复元数据

这个过程恢复被损坏或者替换为新设备的物理卷的元数据。您可以通过重写物理卷的元数据区域从物理卷中恢复数据。

> **警告**
> 
> 不要在正常的 LVM 逻辑卷中尝试这个步骤。如果您指定了不正确的 UUID，将会丢失您的数据。

**前提条件**

- 您已找出丢失的物理卷的元数据。详情请查看[查找缺少的 LVM 物理卷的元数据](#164-查找丢失的-lvm-物理卷的元数据)。

**流程**

1. 恢复物理卷中的元数据：
   
   ```
   # pvcreate --uuid physical-volume-uuid \
           --restorefile /etc/lvm/archive/volume-group-name_backup-number.vg \
           block-device
   ```
   
   > **注意**
   > 
   >  该命令只覆盖 LVM 元数据区域，不会影响现有的数据区域。
   
   > **例 16.4 在 /dev/vdb1上恢复物理卷**
   > 
   > 以下示例使用以下属性将 /dev/vdb1 设备标记为物理卷：
   > 
   > - F`mGRh3-zhok-iVI8-7qTD-S5BI-MAEN-NYM5Sk` 的UUID
   > 
   > - `VG_00050.vg` 中包含的元数据信息，它是卷组最新的好归档元数据。
   > 
   > ```
   > # pvcreate --uuid "FmGRh3-zhok-iVI8-7qTD-S5BI-MAEN-NYM5Sk" \
   >         --restorefile /etc/lvm/archive/VG_00050.vg \
   >         /dev/vdb1
   > 
   > ...
   > Physical volume "/dev/vdb1" successfully created
   > ```

2. 恢复卷组的元数据：
   
   ```
   # vgcfgrestore myvg
   
   Restored volume group myvg
   ```

3. 显示卷组中的逻辑卷：
   
   ```
   # lvs --all --options +devices myvg
   ```
   
    逻辑卷目前不活跃。例如：
   
   ```
   LV     VG   Attr   LSize   Origin Snap%  Move Log Copy%  Devices
   mylv myvg   -wi--- 300.00G                               /dev/vdb1 (0),/dev/vdb1(0)
   mylv myvg   -wi--- 300.00G                               /dev/vdb1 (34728),/dev/vdb1(0)
   ```

4. 如果逻辑卷的片段类型是 RAID，则重新同步逻辑卷：
   
   ```
   # lvchange --resync myvg/mylv
   ```

5. 激活逻辑卷：
   
   ```
   # lvchange --activate y myvg/mylv
   ```

6. 如果磁盘中的 LVM 元数据至少使用了覆盖其数据的空间，这个过程可以恢复物理卷。如果覆盖元数据的数据超过了元数据区域，则该卷中的数据可能会受到影响。您可能能够使用 fsck 命令恢复这些数据。

**验证步骤**

- 显示活跃逻辑卷：
  
  ```
  # lvs --all --options +devices
  
  LV     VG   Attr   LSize   Origin Snap%  Move Log Copy%  Devices
  mylv myvg   -wi--- 300.00G                               /dev/vdb1 (0),/dev/vdb1(0)
  mylv myvg   -wi--- 300.00G                               /dev/vdb1 (34728),/dev/vdb1(0)
  ```

## 16.6 LVM 输出中的轮询错误

LVM 命令报告卷组中的空间使用情况，将报告的编号舍入到 `2` 个十进制位置，以提供人类可读的输出。这包括 `vgdisplay` 和 `vgs` 实用程序。

因此，报告的剩余空间值可能大于卷组中物理扩展提供的内容。如果您试图根据报告可用空间的大小创建逻辑卷，则可能会遇到以下错误：

```
Insufficient free extents
```

要临时解决这个问题，您必须检查卷组中可用物理扩展的数量，即可用空间的具体值。然后您可以使用扩展数目成功创建逻辑卷。

## 16.7 防止创建 LVM 卷时出现循环错误

在创建 LVM 逻辑卷时，您可以指定逻辑卷的逻辑扩展数目以避免循环错误。

**流程**

1. 在卷组中找到可用物理扩展数目：
   
   ```
   # vgdisplay myvg
   ```
   
   > **例 16.5 卷组中可用扩展**
   > 
   > 例如：以下卷组有 8780 可用物理扩展：
   > 
   > --- Volume group ---
   >  VG Name               myvg
   >  System ID
   >  Format                lvm2
   >  Metadata Areas        4
   >  Metadata Sequence No  6
   >  VG Access             read/write
   > [...]
   > Free  PE / Size       8780 / 34.30 GB

2. 创建逻辑卷。以扩展而不是字节为单位输入卷大小。
   
   > **例 16.6 通过指定扩展数目来创建逻辑卷**
   > 
   > ```
   > # lvcreate --extents 8780 --name mylv myvg
   > ```
   
   > **例 16.7 创建逻辑卷以占据所有剩余空间**
   > 
   > 另外，您可以扩展逻辑卷使其使用卷组中剩余的可用空间的比例。例如：
   > 
   > ```
   > # lvcreate --extents 100%FREE --name mylv myvg
   > ```

**验证步骤**

- 检查卷组现在使用的扩展数目：
  
  ```
  # vgs --options +vg_free_count,vg_extent_count
  
  VG     #PV #LV #SN  Attr   VSize   VFree  Free  #Ext
  myvg   2   1   0   wz--n- 34.30G    0    0     8780
  ```

## 16.8 LVM RAID 故障排除

您可以对 LVM RAID 设备中的多个问题进行故障排除，修正数据错误、恢复设备或者替换失败的设备。

### 16.8.1 检查 RAID 逻辑卷中的数据一致性（RAID 清理）

LVM 提供对 RAID 逻辑卷的清理支持。RAID 清理是读取阵列中的所有数据和奇偶校验块的过程，并检查它们是否是分配的。

**流程**

1. 可选：限制清理过程使用的 I/O 带宽。
   
    当您执行 RAID 清理操作时，`sync` 操作所需的后台 I/O 可将其他 I/O 分离到 LVM 设备，如卷组元数据更新。这可能导致其它 LVM 操作速度下降。您可以使用节流功能控制清理操作的速度。
   
    在下一步的 `lvchange --syncaction` 命令中添加以下选项：
   
    **`--maxrecoveryrate Rate[bBsSkKmMgG]`**
   
      设置最大恢复率，以便使操作不会严重影响小的 I/O 操作。将恢复率设置为 0 表示它将不被绑定。
   
    **`--minrecoveryrate Rate[bBsSkKmMgG]`**
   
      设置最小恢复率以确保 同步 操作的 I/O 获得最小吞吐量，即使存在大量 I/O。
   
    指定比率，格式为“数量/每秒/阵列中的每个设备”。如果没有后缀，选项会假定为 kiB/每秒/每个设备。

2. 显示阵列中未修复的差异的数量，没有修复它们：
   
   ```
   # lvchange --syncaction check vg/raid_lv
   ```

3. 修正阵列中的差异：
   
   ```
   # lvchange --syncaction repair vg/raid_lv
   ```

> **注意**

`lvchange --syncaction repair` 操作的功能与 `lvconvert --repair` 操作不同：

- `lvchange --syncaction repair` 操作会在阵列上启动后台同步操作。

- `lvconvert --repair` 操作修复或替换镜像或 RAID 逻辑卷中失败的设备。
4. 可选：显示 scrubbing 操作的信息：
   
   ```
   # lvs -o +raid_sync_action,raid_mismatch_count vg/lv
   ```
   
   - `raid_sync_action` 字段显示 RAID 卷执行的当前同步操作。可以是以下值之一：
     
     **`idle`**
     
     所有同步操作完成（什么都不做）
     
     **`resync`**
     
     初始化阵列或在机器失败后恢复
     
     **`recover`**
     
     替换阵列中的设备
     
     **`check`**
     
     查找阵列的不一致
     
     **`repair`**
     
     查找并修复不一致
   
   - `raid_mismatch_count` 字段显示 `check` 操作中出现的差异数。
   
   - `Cpy%Sync` 字段显示 `sync` 操作的进度。
   
   - `lv_attr` 字段提供了额外的指标。这个字段中的第 9 位显示逻辑卷的健康状况，它支持以下指示：
     
     - `m` (mismatches)表示 RAID 逻辑卷存在差异。这个字符在 scrubbing 操作侦测到部分 RAID 不一致时就会显示。
     
     - `r` (refresh)表示 RAID 阵列中的某个设备出现故障，并且内核将其认为失败，即使 LVM 可以读取该设备标签，并且认为该设备正常运行。刷新逻辑卷通知内核该设备现在可用 ; 如果您怀疑设备失败，则替换该设备。

### 16.8.2 LVM RAID 中失败的设备

RAID 跟传统的 LVM 镜像不同。LVM 镜像需要删除失败的设备，或者镜像逻辑卷会挂起。RAID 阵列可在有失败设备的情况下继续运行。实际上，对于 RAID1 以外的 RAID 类型，删除设备意味着将设备转换为较低级别 RAID（例如：从 RAID6 转换为 RAID5，或者从 RAID4 或者 RAID5 转换到 RAID0）。

因此，LVM 允许您使用 `lvconvert` 命令的 `--repair` 参数替换 RAID 卷中失败的设备，而不是无条件地删除失败的设备。

### 16.8.3 在逻辑卷中恢复失败的 RAID 设备

如果 LVM RAID 设备失败是一个临时故障，或者您可以修复失败的设备，您可以初始化失败设备的恢复。

**前提条件**

- 之前失败的设备现在可以正常工作。

**流程**

- 刷新包含 RAID 设备的逻辑卷：
  
  ```
  # lvchange --refresh my_vg/my_lv
  ```

**验证步骤**

- 使用恢复的设备检查逻辑卷：
  
  ```
  # lvs --all --options name,devices,lv_attr,lv_health_status my_vg
  ```

## 16.8.4 在逻辑卷中替换失败的 RAID 设备

这个过程替换作为 LVM RAID 逻辑卷中的物理卷的失败设备。

**前提条件**

- 卷组包含一个物理卷，它有足够的可用容量替换失败的设备。
  
  如果卷组中没有足够可用区块的物理卷，请使用 `vgextend` 实用程序添加新的、足够大的物理卷。

**流程**

1. 在下面的示例中，RAID 逻辑卷布局如下：
   
   ```
   # lvs --all --options name,copy_percent,devices my_vg
   
   LV               Cpy%Sync Devices
   my_lv            100.00   my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
   [my_lv_rimage_0]          /dev/sde1(1)
   [my_lv_rimage_1]          /dev/sdc1(1)
   [my_lv_rimage_2]          /dev/sdd1(1)
   [my_lv_rmeta_0]           /dev/sde1(0)
   [my_lv_rmeta_1]           /dev/sdc1(0)
   [my_lv_rmeta_2]           /dev/sdd1(0)
   ```

2. 如果 `/dev/sdc` 设备失败，`lvs` 命令的输出如下：
   
   ```
   # lvs --all --options name,copy_percent,devices my_vg
   
   /dev/sdc: open failed: No such device or address
   Couldn't find device with uuid A4kRl2-vIzA-uyCb-cci7-bOod-H5tX-IzH4Ee.
   WARNING: Couldn't find all devices for LV my_vg/my_lv_rimage_1 while checking used and assumed devices.
   WARNING: Couldn't find all devices for LV my_vg/my_lv_rmeta_1 while checking used and assumed devices.
   LV               Cpy%Sync Devices
   my_lv            100.00   my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
   [my_lv_rimage_0]          /dev/sde1(1)
   [my_lv_rimage_1]          [unknown](1)
   [my_lv_rimage_2]          /dev/sdd1(1)
   [my_lv_rmeta_0]           /dev/sde1(0)
   [my_lv_rmeta_1]           [unknown](0)
   [my_lv_rmeta_2]           /dev/sdd1(0)
   ```

3. 替换失败的设备并显示逻辑卷：
   
   ```
   # lvconvert --repair my_vg/my_lv
   
   /dev/sdc: open failed: No such device or address
   Couldn't find device with uuid A4kRl2-vIzA-uyCb-cci7-bOod-H5tX-IzH4Ee.
   WARNING: Couldn't find all devices for LV my_vg/my_lv_rimage_1 while checking used and assumed devices.
   WARNING: Couldn't find all devices for LV my_vg/my_lv_rmeta_1 while checking used and assumed devices.
   Attempt to replace failed RAID images (requires full device resync)? [y/n]: y
   Faulty devices in my_vg/my_lv successfully replaced.
   ```
   
    可选： 要手动指定替换失败设备的物理卷，请在命令末尾添加物理卷：
   
   ```
   # lvconvert --repair my_vg/my_lv replacement_pv
   ```

4. 使用替换检查逻辑卷：
   
   ```
   # lvs --all --options name,copy_percent,devices my_vg
   
   /dev/sdc: open failed: No such device or address
   /dev/sdc1: open failed: No such device or address
   Couldn't find device with uuid A4kRl2-vIzA-uyCb-cci7-bOod-H5tX-IzH4Ee.
   LV               Cpy%Sync Devices
   my_lv            43.79    my_lv_rimage_0(0),my_lv_rimage_1(0),my_lv_rimage_2(0)
   [my_lv_rimage_0]          /dev/sde1(1)
   [my_lv_rimage_1]          /dev/sdb1(1)
   [my_lv_rimage_2]          /dev/sdd1(1)
   [my_lv_rmeta_0]           /dev/sde1(0)
   [my_lv_rmeta_1]           /dev/sdb1(0)
   [my_lv_rmeta_2]           /dev/sdd1(0)
   ```
   
    在您从卷组中删除失败的设备前，LVM 工具仍然指示 LVM 无法找到失败的设备。

5. 从卷组中删除失败的设备：
   
   ```
   # vgreduce --removemissing VG
   ```

## 16.9 对多路径 LVM 设备进行重复的物理卷警告进行故障排除

当将 LVM 与多路径存储搭配使用时，列出卷组或者逻辑卷的 LVM 命令可能会显示如下信息：

```
Found duplicate PV GDjTZf7Y03GJHjteqOwrye2dcSCjdaUi: using /dev/dm-5 not /dev/sdd
Found duplicate PV GDjTZf7Y03GJHjteqOwrye2dcSCjdaUi: using /dev/emcpowerb not /dev/sde
Found duplicate PV GDjTZf7Y03GJHjteqOwrye2dcSCjdaUi: using /dev/sddlmab not /dev/sdf
```

您可以排除这些警告来了解 LVM 显示它们的原因，或者隐藏警告信息。

### 16.9.1 重复 PV 警告的根本原因

当设备映射器多路径（DM 多路径）、EMC PowerPath 或 Hitachi Dynamic Link Manager(HDLM)等多路径软件管理系统上的存储设备时，到特定逻辑单元(LUN)的每个路径都会注册为不同的 SCSI 设备。

然后多路径软件会创建一个映射到这些独立路径的新设备。因为每个 LUN 在 `/dev` 目录中有多个指向同一底层数据的设备节点，所以所有设备节点都包含相同的 LVM 元数据。

**表 16.1 不同多路径软件的设备映射示例**

| **多路径软件**     | **到 LUN 的 SCSI 路径**     | **多路径设备映射到路径**                              |
| ------------- | ----------------------- | ------------------------------------------- |
| DM Multipath  | `/dev/sdb` 和 `/dev/sdc` | `/dev/mapper/mpath1` 或 `/dev/mapper/mpatha` |
| EMC PowerPath |                         | `/dev/emcpowera`                            |
| HDLM          |                         | `/dev/sddlmab`                              |

由于多个设备节点，LVM 工具会多次查找相同的元数据，并将其作为重复报告。

### 16.9.2 重复 PV 警告的情况

LVM 在以下任一情况下显示重复的 PV 警告：

**指向同一设备的单路径**

输出中显示的两个设备都是指向同一设备的单一路径。 以下示例显示一个重复的 PV 警告，

在该示例中重复的设备是同一设备的单一路径。

```
Found duplicate PV GDjTZf7Y03GJHjteqOwrye2dcSCjdaUi: using /dev/sdd not /dev/sdf
```

如果您使用 `multipath -ll` 命令列出当前的 DM 多路径拓扑，您可以在同一个多路径映射下找到 `/dev/sdd` 和 `/dev/sdf`。

这些重复的信息只是警告，并不意味着 LVM 操作失败。相反，它们会提醒您 LVM 只使用其中一个设备作为物理卷，并忽略其它设备。

如果消息显示 LVM 选择不正确的设备或者警告对用户造成破坏，您可以应用过滤器。该过滤器将 LVM 配置为仅搜索物理卷所需的设备，并丢弃多路径设备的任何基本路径。因此，不再会出现警告。

**多路径映射**

输出中显示的两个设备都是多路径映射。

以下示例显示两个设备都有重复的 PV 警告，它们是多路径映射。重复的物理卷位于两个不同的设备中，而不是位于同一设备的两个不同路径中。

```
Found duplicate PV GDjTZf7Y03GJHjteqOwrye2dcSCjdaUi: using /dev/mapper/mpatha not /dev/mapper/mpathc

Found duplicate PV GDjTZf7Y03GJHjteqOwrye2dcSCjdaUi: using /dev/emcpowera not /dev/emcpowerh
```

对于同一设备上的单一路径的设备来说，这种情形比重复的警告更为严重。这些警告通常意味着机器正在访问它不应该访问的设备：例如： LUN 克隆或镜像(mirror)。

除非您明确知道您应该从机器中删除哪些设备，否则这个情况可能无法恢复。

### 16.9.3 LVM 设备过滤器

LVM 工具扫描 `/dev` 目录中的设备，并检查这里的 LVM 元数据的每个设备。`/etc/lvm/lvm.conf` 文件中的过滤器控制 LVM 扫描的设备。

过滤器是 LVM 通过扫描 `/dev` 目录或 `/etc/lvm/lvm.conf` 文件中的 `dir` 关键字指定的目录，应用于每个设备的模式列表。模式是正则表达式，用任何字符分隔，并在前面加上 a 表示接受 或 `r` 表示 拒绝。匹配设备的列表中的第一个正则表达式决定了 LVM 接受还是拒绝（忽略）该设备。LVM 接受与任何模式不匹配的设备。

以下是该过滤器的默认配置，可扫描所有设备：

```
filter = [ "a/.*/" ]
```

### 16.9.4 防止重复 PV 警告的 LVM 设备过滤器示例

下面的例子显示 LVM 设备过滤器，可避免由到单个逻辑单元(LUN)的多个存储路径导致重复的物理卷警告。

您配置的过滤器必须包含所有 LVM 需要检查元数据的设备，比如使用根卷组以及所有多路径设备的本地硬盘。通过拒绝到多路径设备的底层路径（如 `/dev/sdb`、`/dev/sdd` 等），您可以避免这些重复的 PV 警告，因为 LVM 在多路径设备本身中找到每个唯一的元数据区域。

- 这个过滤器接受第一个硬盘中的第二个分区以及任何 DM 多路径设备，但拒绝所有其它分区：
  
  ```
  filter = [ "a|/dev/sda2$|", "a|/dev/mapper/mpath.*|", "r|.*|" ]
  ```

- 这个过滤器接受所有 HP SmartArray 控制器和任何 EMC PowerPath 设备：
  
  ```
  filter = [ "a|/dev/cciss/.*|", "a|/dev/emcpower.*|", "r|.*|" ]
  ```

- 这个过滤器接受第一个 IDE 驱动器中的所有分区以及任意多路径设备：
  
  ```
  filter = [ "a|/dev/hda.*|", "a|/dev/mapper/mpath.*|", "r|.*|" ]
  ```

### 16.9.5 应用 LVM 设备过滤器配置

这个步骤更改了控制 LVM 扫描设备的 LVM 设备过滤器的配置。

**前提条件**

- 准备要使用的设备过滤器特征。

**流程**

1. 在不修改 /etc/lvm/lvm.conf 文件的情况下测试设备过滤器特征。
   
   使用带 `--config 'devices{ filter = [ "a|/dev/emcpower.*|", "r|.*|" ] }'` 选项的 LVM 命令。例如：
   
   ```
   # lvs --config 'devices{ filter = [ "a|/dev/emcpower.*|", "r|.*|" ] }'
   ```

2. 编辑 `/etc/lvm/lvm.conf` 配置文件中的 `filter` 选项，以使用您的新设备过滤器模式。

3. 检查新配置是否缺少您要使用的物理卷或卷组：
   
   ```
   # pvscan
   ```
   
   ```
   # vgscan
   ```

4. 重建 `initramfs` 文件系统，以便 LVM 重启时只扫描所需的设备：
   
   ```
   # dracut --force --verbose
   ```