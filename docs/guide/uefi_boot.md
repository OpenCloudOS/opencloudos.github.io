efibootmgr（EFI Boot Manager）是一个用于管理操作系统启动项的命令行工具。它允许用户在 UEFI 引导启动的系统上创建、删除、编辑和重新排序启动项。efibootmgr 使用户能够在 **用户态环境** 下轻松管理多个操作系统的启动顺序、添加新的启动项或修改现有启动项的配置，而无需进入 BIOS 进行设置。 

## 1. 确保系统以 UEFI 模式启动

在开始使用 efibootmgr 之前，请确保系统是由 UEFI 模式引导启动。通过以下命令检查系统是否以 UEFI 模式启动：

```bash
ls /sys/firmware/efi
```

如果命令返回 EFI 相关的文件和目录，则表示系统已经以 UEFI 模式启动。

## 2. 安装 efibootmgr

在开始使用 efibootmgr 之前，确保已经在系统上安装了它。如果尚未安装，请通过以下命令进行安装：

```bash
sudo dnf install efibootmgr
```

接下来，就可以使用 efibootmgr 命令对 UEFI 启动项进行管理。以下将对常见操作进行详细说明。
## 3. efibootmgr 常见命令说明
### 3.1 显示当前启动项

要查看当前系统的启动项列表，可以使用以下命令：

```bash
sudo efibootmgr
```

这将显示类似以下内容的输出：

```
BootCurrent: 0005
Timeout: 3 seconds
BootOrder: 0005,0002,0000,0001,0003,0004
Boot0000* UiApp
Boot0001* UEFI Misc Device
Boot0002* UEFI Misc Device 2
Boot0003* UEFI Non-Block Boot Device
Boot0004* EFI Internal Shell
Boot0005* OpenCloudOS
```

### 3.2 创建新的启动项

要创建新的启动项，请使用以下命令格式：

```bash
sudo efibootmgr -c -d /dev/sdX -p Y -L "Label" -l "\EFI\path\to\bootloader.efi"
```

- `-c`：创建新的启动项
- `-d`：指定磁盘设备（例如：`/dev/sda`）
- `-p`：指定分区号（例如：`1`）
- `-L`：为新启动项指定标签
- `-l`：指定启动加载程序的路径

例如，要在 `/dev/sda1` 分区上创建一个名为 "OpenCloudOS" 的新启动项，使用以下命令：

```bash
sudo efibootmgr -c -d /dev/sda -p 1 -L "OpenCloudOS" -l "\EFI\opencloudos\grubx64.efi"
```

### 3.3 修改启动项

要修改现有启动项，可以使用以下命令格式：

```bash
sudo efibootmgr -b XXXX -u -d /dev/sdX -p Y -L "New Label" -l "\EFI\new\path\to\bootloader.efi"
```

- `-b`：指定要修改的启动项的编号（例如：`0001`）
- `-u`：更新现有启动项
- 其他选项与创建新启动项时相同

例如，要修改启动项 `0001` 的标签和加载程序路径，请使用以下命令：

```bash
sudo efibootmgr -b 0001 -u -L "Updated Label" -l "\EFI\updated\path\to\bootloader.efi"
```

### 3.4 删除启动项

要删除现有启动项，请使用以下命令格式：

```bash
sudo efibootmgr -b XXXX -B
```

- `-b`：指定要删除的启动项的编号（例如：`0001`）
- `-B`：删除指定的启动项

例如，要删除启动项 `0001`，请使用以下命令：

```bash
sudo efibootmgr -b 0001 -B
```

### 3.5 更改启动顺序

要更改启动顺序，请使用以下命令格式：

```bash
sudo efibootmgr -o XXXX,YYYY,ZZZZ
```

- `-o`：指定新的启动顺序，使用逗号分