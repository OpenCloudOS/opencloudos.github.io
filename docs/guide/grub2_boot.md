## 1. 简介
OpenCloudOS 系统采用 GRUB2 作为引导加载程序，加载操作系统，并遵循 Boot Loader Specification（BLS）标准，为系统引导项配置独立的描述文件。同时，为了实现高效且灵活的引导项管理，OpenCloudOS 选用 Grubby 工具对 BLS 配置文件进行统一管理。通过 Grubby快速地添加、修改、删除和查看当前的引导项，从而实现对启动过程的优化和高度定制化。

接下来将详细说明各项系统工具，以便用户参考使用。

## 2. Grub2 

### 2.1 简介
GRUB2（GRand Unified Bootloader 2）是一款高效且功能丰富的启动引导程序，用于在计算机开机时管理和加载各种操作系统。作为 GRUB 的升级版，GRUB2 在原有功能基础上加入了更多创新技术，提升了其在引导领域的表现。

在 OpenCloudOS 系统中，GRUB2 负责引导操作系统的启动过程。它支持多系统引导，可以与其他操作系统共同工作，为用户提供灵活的选择。

### 2.2 配置安装 Grub2

Grub2 默认安装在 OpenCloudOS 系统中。如果需要手动安装，可以使用以下命令：

```bash
sudo dnf install grub2
```

安装完 grub2 后，可以对 grub2 进行自定义配置，主要配置文件位于 `/etc/default/grub`。要编辑此文件，请使用文本编辑器（如 `vim` ）并使用 `sudo` 权限打开文件：

```bash
sudo vi /etc/default/grub
```

完成配置后，需要更新对应的配置文件来使配置生效，`grub2-mkconfig` 命令用于根据 `/etc/default/grub` 和 `/etc/grub.d/` 目录下的配置文件生成新的 Grub2 配置文件。使用以下命令即可生成配置文件：

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 2.3 Grub2 常用命令
接下来介绍 Grub2 的常用命令，并通过示例说明命令对系统的影响。

#### 2.3.1 grub2-editenv

`grub2-editenv` 命令用于管理 Grub2 环境变量，当需要修改默认启动项的默认环境变量参数（比如 saved_entry，boot_success 等）时，可以通过`grub2-editenv`完成。以下是一些常用操作：

- 查看当前 Grub2 环境变量：
  ```bash
  sudo grub2-editenv list
  ```
此操作对系统无影响。

- 设置环境变量：
  ```bash
  sudo grub2-editenv set VARIABLE_NAME=VALUE
  ```
说明：此命令用于在 GRUB2 环境块中设置指定的环境变量及其值。将  `VARIABLE_NAME`  替换为要设置的变量名，将  `VALUE`  替换为相应的值。
系统影响：设置环境变量后，GRUB2 将在下次启动时使用新的值。

- 删除环境变量：
  ```bash
  sudo grub2-editenv unset VARIABLE_NAME
  ```
说明：此命令用于从 GRUB2 环境块中删除指定的环境变量。将  `VARIABLE_NAME`  替换为要删除的变量名。
系统影响：删除环境变量后，GRUB2 将在下次启动时不再使用该变量。

#### 2.3.2 grub2-install
`grub2-install`  是一个用于在指定设备上安装或修复 GRUB2 引导程序的命令。此命令的主要功能是将 GRUB2 引导程序写入磁盘的主引导记录（MBR），使计算机能够在启动时加载 GRUB2 菜单并引导操作系统。

在 legacy 引导模式下，当系统主引导记录损害或丢失时，可以通过 `grub2-install` 完成引导记录的重建。
例如，将 Grub2 安装到 `/dev/sda`：

```bash
sudo grub2-install /dev/sda
```
系统影响：此命令将在指定磁盘的主引导记录（MBR）上安装或修复 GRUB2 引导程序。执行此操作后，计算机将能够在启动时加载 GRUB2 菜单并引导操作系统。如果已有其他引导程序存在，可能会影响到它们的正常启动。

此外，`grub2-install` 已不支持在UEFI引导下使用，因此如果UEFI引导记录被破坏，需要通过 `efibootmgr` 命令完成修复

#### 2.3.3 grub2-set-default

`grub2-set-default` 命令用于设置默认启动项。要设置默认启动项，请使用以下命令，其中 `N` 是启动项的索引（从 0 开始计数）：

```bash
sudo grub2-set-default N
```

系统影响：在您下次启动计算机时，GRUB2 将自动选择新设置的默认启动项 N 。请注意，要使此设置生效，您需要确保  `/etc/default/grub`  文件中的  `GRUB_DEFAULT`  设置为  `saved` ，然后执行 `grub2-mkconfig` 重新生成 GRUB2 配置文件。

#### 2.3.4 grub2-reboot

`grub2-reboot` 命令用于设置下一次启动时使用的启动项。要设置下一次启动的启动项，请使用以下命令，其中 `N` 是启动项的索引（从 0 开始计数）：

```bash
sudo grub2-reboot N
```

系统影响：在下次启动计算机时，GRUB2 将自动选择新设置的启动项 N 。请注意，`grub2-reboot` 仅影响当次启动。

## 3. BLS ( Boot Loader Specification ) 
BLS（Boot Loader Specification）是一个由 systemd 项目提出的标准，旨在简化引导加载器和操作系统之间的交互。BLS 通过定义一个统一的引导项格式，使得 GRUB2 无需理解各种不同的操作系统和内核版本。当前系统支持并默认打开了 BLS 标准，GRUB2 默认使用 BLS 标准管理系统启动项。

### 3.1 配置 BLS

在 GRUB2 中启用 BLS 需要修改 GRUB2 的配置文件。在 `/etc/default/grub` 文件中，添加以下行来启用 BLS：

```bash
GRUB_ENABLE_BLSCFG=true
```

然后运行 `grub2-mkconfig` 命令来更新 GRUB2 的配置。这个命令会生成一个新的 GRUB2 配置文件，其中包含了从 BLS 文件中读取的引导项，OpenCloudOS 默认打开了 BLS 配置。


### 3.2 BLS 基本格式

在支持 BLS 的系统中，GRUB2 会自动读取 BLS 文件来生成引导菜单。这意味着，你只需要创建或修改 BLS 文件，就可以添加或修改引导项。

打开 BLS 标准后，所有的系统引导项将位于 `/boot/loader/entries/` 目录下，文件名的格式是 `<machind-id>-<version>.conf`。在这些文件中，可以定义系统引导项，如以下的示例：

```bash
title OpenCloudOS Stream (6.1.26-2303.1.0.ocs23.x86_64) 23
version 6.1.26-2303.1.0.ocs23.x86_64
linux /boot/vmlinuz-6.1.26-2303.1.0.ocs23.x86_64
initrd /boot/initramfs-6.1.26-2303.1.0.ocs23.x86_64.img
options $kernelopts
grub_users $grub_users
grub_arg --unrestricted
grub_class opencloudos
```

在这个示例中，`title` 行定义了引导项的标题，`version` 行定义了引导项的版本，`linux` 行指定了内核的位置，`initrd` 行指定了初始 RAM 磁盘的位置，`options` 行定义了引导参数。

当你更新了 BLS 文件后，你需要运行 `grub2-mkconfig` 命令来更新 GRUB2 的配置。这个命令会读取所有的 BLS 文件，然后生成一个新的 GRUB2 配置文件。

## 4. Grubby

### 4.1 简介
Grubby 是一个用于管理内核引导加载器配置文件的命令行工具。它支持多种引导加载器，如 GRUB、GRUB2、LILO、ELILO 和 ZIPL。本文档将介绍如何使用 Grubby 管理内核启动项。

### 4.2 安装 Grubby
安装 Grubby只需运行以下命令：

```bash
sudo dnf install grubby
```

### 4.3 Grubby常用命令

Grubby 通过以下常用命令实现对 BLS 启动项文件的管理。

#### 4.3.1 查看当前启动项

查看当前 Grubby 管理的启动项，请运行以下命令：

```bash
grubby --info=ALL
```

这将显示所有已配置的 BLS 启动项及其详细信息。

#### 4.3.2 添加新的内核启动项
> 一般情况下，添加内核启动项的步骤会在安装内核时自动完成，无需用户介入。这里仅介绍基本用法。

添加新的内核启动项，请使用以下命令：

```bash
sudo grubby --add-kernel=/boot/vmlinuz-<version> --initrd=/boot/initramfs-<version>.img --title="Custom Kernel <version>" --args="root=/dev/sda1"
```

`--args`: 为新内核设置默认参数。

这将新增 BLS 引导条目，并将其root参数设置为 /dev/sda1。

#### 4.3.3 修改现有内核启动项
> 修改内核启动项可能会导致系统无法启动，请谨慎操作。

修改现有内核启动项，请使用以下命令：

```bash
sudo grubby --update-kernel=/boot/vmlinuz-<version> --args="root=/dev/sda1"
```

`--args`: 更新现有内核的参数。

这将更新指定的 BLS 引导条目，并将其root参数设置为 /dev/sda1。

#### 4.3.4 删除内核启动项
> 一般情况下，删除内核启动项的步骤会在卸载内核时自动完成，无需用户介入。这里仅介绍基本用法。

要删除内核启动项，请使用以下命令：

```bash
sudo grubby --remove-kernel=/boot/vmlinuz-<version>
```

请将 `<version>` 替换为要删除的内核版本号。

#### 4.3.5 设置默认启动项
> 默认内核启动项将会被自动设置为最新安装的内核，无需用户介入。这里仅介绍基本用法。

要设置默认启动项，请使用以下命令：

```bash
sudo grubby --set-default=/boot/vmlinuz-<version>
```

请将 `<version>` 替换为要设置为默认的内核版本号。


## 5. 常见问题和解决方案

#### 5.1 GRUB2 常见问题

- **GRUB2 引导菜单不显示**
   如果 GRUB2 的引导菜单不显示，可能的原因是引导菜单的超时时间设置为 0。你可以在 `/etc/default/grub` 文件中修改 `GRUB_TIMEOUT` 选项，然后运行 `update-grub`（或 `grub2-mkconfig`）命令来更新配置。

- **GRUB2 无法引导某个操作系统**
   如果 GRUB2 无法引导某个操作系统，可能的原因是 GRUB2 的配置文件中缺少了这个操作系统的引导项。你可以检查 `/etc/grub.d/` 目录下的配置文件，或者运行 `update-grub`（或 `grub2-mkconfig`）命令来自动检测可用的操作系统。

#### 5.2 BLS 常见问题

- **BLS 文件无效**
   如果 BLS 文件无效，可能的原因是文件的格式不正确。可以检查 BLS 文件的内容，确保它遵循 BLS 的格式。特别是，需要确保 `title`，`linux`，和 `options` 行存在，并且指向正确的位置。

- **GRUB2 无法读取 BLS 文件**
   如果 GRUB2 无法读取 BLS 文件，可能的原因是 GRUB2 没有启用 BLS。可以在 `/etc/default/grub` 文件中检查 `GRUB_ENABLE_BLSCFG` 选项，确保它设置为 `true`。然后，你需要运行 `update-grub`（或 `grub2-mkconfig`）命令来更新配置。
   
#### 5.3 Grubby的常见问题

- Grubby报告“no suitable templates were found”错误。
 这可能是因为Grubby无法找到合适的模板文件。请检查`/boot/loader/entries/`目录下的文件格式是否正确。

- Grubby无法更改默认启动项。
 请确保有足够的权限来修改引导加载器配置。如果使用sudo运行Grubby命令，也请确保sudo配置允许运行Grubby。

