## 1. journald服务

### 1.1 查看系统日志

在 Linux 中，您可以使用以下命令查看系统日志：

- 查看所有日志：

 ```
 sudo journalctl
 ``` 
该命令将显示所有系统日志。

- 查看特定服务的日志：

 ```
 sudo journalctl -u service-name
 ``` 
将 "service-name" 替换为您要查看的服务名称。该命令将显示特定服务的日志。

- 查看特定时间段的日志：

 ```
 sudo journalctl --since "2022-01-01 00:00:00" --until "2022-01-02 00:00:00"
 ``` 
将日期和时间替换为您要查看的时间段。该命令将显示指定时间段内的日志。

- 查看内核日志

 ```
 sudo journalctl -k
 ``` 

- 查看所有启动日志

当前系统中journald默认只会保存当次启动的日志，存储位置在`/run/log/journal/`下，如果需要记录所有的启动日志，可以手动创建`/var/log/journal`目录，创建后journald服务会把启动日志存储到这里。再次重启后，`journalctl --list-boots`就能查看到之前的启动记录了。
```
# mkdir -p /var/log/journal/
# reboot
...
# journalctl --list-boots
IDX BOOT ID                          FIRST ENTRY                 LAST ENTRY                 
 -1 506feea161494cff925010ea7599ab5d Fri 2023-05-26 15:57:28 CST Fri 2023-05-26 16:00:10 CST
  0 59f4624ce2d94299be2afd80d3650c1a Fri 2023-05-26 16:00:18 CST Fri 2023-05-26 16:03:20 CST
```

### 1.2 管理系统日志

在 Linux 中，您可以使用以下命令管理系统日志：

- 清除所有日志：

 ```
 sudo journalctl --vacuum-time=1s
 ``` 
该命令将清除所有日志。

- 清除特定时间段的日志：

 ```
 sudo journalctl --vacuum-time=1s --since "2022-01-01 00:00:00" --until "2022-01-02 00:00:00"
 ``` 
将日期和时间替换为您要清除的时间段。该命令将清除指定时间段内的日志。

- 将日志输出到文件：

 ```
 sudo journalctl > log.txt
 ``` 
该命令会将所有日志输出到名为 "log.txt" 的文件中。

- 将特定服务的日志输出到文件：

 ```
 sudo journalctl -u service-name > service-log.txt
 ``` 
将 "service-name" 替换为您要输出的服务名称。该命令会将特定服务的日志输出到名为 "service-log.txt" 的文件中。

## 2. rsyslog 的安装和配置

### 2.1 rsyslog 简介
rsyslog (Reliable System Logging) 是一个强大的系统日志处理引擎，它提供了一套完整的工具集，可以轻松传输、后处理和存储各种来源的系统与应用日志。rsyslog 对 Syslog 协议进行了改良与扩展，包括更高的可靠性、性能以及强大的插件架构。

### 2.2 安装 rsyslog

在 OpenCloudOS 中，rsyslog 通常已经预装。同时可以使用包管理器（ dnf ）来安装：

```bash
sudo dnf update
sudo dnf install rsyslog
```

### 2.3 配置 rsyslog

rsyslog 的配置文件通常位于 `/etc/rsyslog.conf`。在这个文件中定义日志消息的来源、格式和目的地。例如，配置 rsyslog 从特定的 Unix 套接字读取日志，从 journald 读取日志或者将日志消息发送到远程服务器。

rsyslog 的配置文件使用了一种特殊的语法，其中包括模块、模板和规则。例如，以下是一个简单的配置：

```bash
# 加载模块
module(load="imuxsock") # 提供 Unix 套接字支持
module(load="imklog")  # 提供内核日志支持

# 提供 UDP syslog 接收
module(load="imudp")
input(type="imudp" port="514")

# 提供 TCP syslog 接收
module(load="imtcp")
input(type="imtcp" port="514")

# 指定日志文件的位置
auth,authpriv.*                 /var/log/auth.log
*.*;auth,authpriv.none          -/var/log/syslog
daemon.*                        -/var/log/daemon.log
kern.*                          -/var/log/kern.log

```
 修改配置文件后，需要重启 rsyslog 服务：
 
```bash
sudo systemctl restart rsyslog
```

### 2.4 rsyslog 的基本使用

一旦 rsyslog 安装并配置好，它就会开始自动运行，并处理日志消息。使用 `systemctl` 命令来管理 rsyslog 服务，例如启动、停止或重启服务：

```bash
sudo systemctl start rsyslog
sudo systemctl stop rsyslog
sudo systemctl restart rsyslog
```

设置 rsyslog 随系统默认启动：
```bash
sudo systemctl enable rsyslog
```

同时可以查看 rsyslog 的状态，以确保它正在正常运行：

```bash
sudo systemctl status rsyslog
```

### 2.5 配置日志转发规则

rsyslog 的强大之处体现在其便捷的日志文件转发功能，用户可以通过 rsyslog 的配置文件，方便地发送本机日志文件至远程服务器，或者接收来自远程服务器的日志信息。

#### 2.5.1 转发本机日志消息

用户配置 /etc/rsyslog.conf 文件，从而将本机所有日志消息通过 rsyslog 转发到其他服务器：

```bash
*.* @@192.168.1.1:514
```

在这个示例中，`*.*` 表示所有设施和所有优先级的日志消息，`@@` 表示使用 TCP 连接，`192.168.1.1:514` 是远程服务器的 IP 地址和端口。

修改配置文件后，请重新启动 rsyslog 服务使更改生效：

```bash
sudo systemctl restart rsyslog
```

#### 2.5.2 接收远程日志消息

用户配置 /etc/rsyslog.conf 文件，加载特定模块并打开特定端口，接收来自其他服务器的日志消息：

```bash

# 提供 UDP syslog 接收
module(load="imudp")
input(type="imudp" port="514")

# 提供 TCP syslog 接收
module(load="imtcp")
input(type="imtcp" port="514")

```

在这个示例中，rsyslog 打开了 TCP 和 UDP 接收模块，接收来自其他服务器的 syslog 消息。

修改配置文件后，请重新启动 rsyslog 服务使更改生效：

```bash
sudo systemctl restart rsyslog
```

### 2.6 结合使用 rsyslog 和 journal

rsyslog 和 journal 都是用于处理和管理系统日志的工具，但它们在设计和实现上有一些主要的区别。rsyslog 使用的是传统的文本文件格式，可以直接阅读和处理，而 journal 使用的是二进制格式，需要通过特定的工具进行处理。此外，rsyslog 更加注重网络日志收集，而 journal 则更加注重本地日志的结构化和索引。

#### 2.6.1 如何在 rsyslog 中使用 journal

rsyslog 可以配置为从 journal 中读取日志。这可以通过在 rsyslog 配置文件中加载 imjournal 模块来实现。加载此模块后，rsyslog 将从 journal 中读取日志，而不是从传统的 syslog UNIX 套接字。这样可以使 rsyslog 利用 journal 提供的结构化日志数据。

在 rsyslog 的配置文件 `/etc/rsyslog.conf` 中，添加以下行来加载 imjournal 模块（在 OpenCloudOS 中，通常已默认配置）：

```bash
module(load="imjournal" StateFile="imjournal.state")
```

这将使 rsyslog 从 journal 中读取日志。

#### 2.6.2 如何在 journal 中使用 rsyslog

虽然 journal 本身并不直接使用 rsyslog，但它可以配置为将日志消息转发给 rsyslog。这可以通过在 journal 的配置文件中设置 ForwardToSyslog 选项来实现。设置此选项后，journal 将把所有日志消息都发送到 rsyslog，这样 rsyslog 就可以像处理传统的 syslog 消息一样处理这些消息。

在 journal 的配置文件 `/etc/systemd/journald.conf` 中，添加或修改以下行来启用 ForwardToSyslog 选项 (默认选项为 no )：

```bash
ForwardToSyslog=yes
```

然后重启 systemd-journald 服务以使更改生效：

```bash
sudo systemctl restart systemd-journald
```

一般情况下，如果需要将 journal 的日志消息转发至 rsyslog 时，建议用户同时打开 rsyslog 的 imjournal 模块和 journald 的 ForwardToSyslog 选项，这样能够确保所有的日志消息都能正确地被 rsyslog 捕获。

#### 2.6.3 结合使用 rsyslog 和 journal

结合使用 rsyslog 和 journal 可以充分利用两者的优点。通过 journal获得结构化和索引的日志，这使得日志查询和分析更加方便。而通过 rsyslog将日志消息发送到网络上的其他服务器，这对于集中式日志管理非常有用。此外，由于 rsyslog 可以处理传统的 syslog 消息，所以它可以与那些仍然使用这种消息格式的旧应用程序兼容。

例如，配置 /etc/rsyslog.conf 文件，从而将本机所有 journal 日志消息通过 rsyslog 转发到其他服务器：

```bash
*.* @@192.168.1.1:514
```

#### 2.6.4 常见问题和解决方案

##### rsyslog 常见问题

- **rsyslog 服务无法启动**
   如果 rsyslog 服务无法启动，首先检查 rsyslog 的配置文件是否有语法错误。使用 `rsyslogd -N1` 命令来检查配置文件。如果这个命令没有输出任何错误，那么配置文件应该是正确的。
   另一个可能的原因是 rsyslog 服务没有足够的权限来读取或写入日志文件。检查日志文件的权限确保 rsyslog 服务有权访问它们。

- **rsyslog 不接收远程日志**
   如果 rsyslog 不接收远程日志，首先检查防火墙设置，确保 rsyslog 的端口（默认是 514）没有被阻塞。也可以检查 rsyslog 的配置文件，确保已经加载了 imudp 或 imtcp 模块，并且已经启动了 UDP 或 TCP 服务器。

##### journal 常见问题

- **journal 日志丢失**
   如果 journal 的日志丢失，可能的原因是 journal 的存储空间不足。在 journal 的配置文件中设置 `SystemMaxUse` 选项，以限制 journal 可以使用的最大磁盘空间。
   另一个可能的原因是系统崩溃或意外关机。因为 journal 默认在内存中存储日志，所以在这些情况下，日志可能会丢失。在 journal 的配置文件中设置 `Storage=persistent` 选项，以在磁盘上持久化存储日志。

- **journal 日志过大**
   如果 journal 的日志过大，使用 `journalctl --vacuum-size` 或 `journalctl --vacuum-time` 命令来清理旧的日志。也可以在 journal 的配置文件中设置 `SystemMaxUse` 和 `SystemKeepFree` 选项，以自动管理日志的大小。

## 3. 日志转储

### 3.1 安装logrotate

logrotate是一个Linux系统中常用的日志转储工具，可以通过以下命令来安装：
```
sudo dnf install logrotate
```
### 3.2 配置logrotate

logrotate的配置文件通常位于`/etc/logrotate.conf`和`/etc/logrotate.d/`目录下，可以通过编辑这些文件来配置日志转储的规则。系统中目前的`/etc/logrotate.conf`配置为：
```
weekly
rotate 4
create
dateext
include /etc/logrotate.d

```
该文件指定了

- 日志转储的时间为每周

- 最多存在4个备份文件

- 每次转储时，都创建新的文件

- 使用日期作为转储的文件的前缀

- 最后包括了/etc/logrotate.d目录下其他的配置文件

当软件包组件安装的时候，部分有日志的包也会默认安装其日志转储的配置文件，可以看到在 /etc/logrotate.d 目录下看到
```
# ls /etc/logrotate.d
btmp  chrony  dnf  httpd  iptraf-ng  iscsiuiolog  libvirtd  libvirtd.qemu  mariadb  numad  openstack-trove  psacct  rabbitmq-server  wtmp
```
以下面一个配置文件为例，详细说明一下logrotate配置文件中的基本规则

```
/var/log/myapp.log {
    rotate 7
    daily
    missingok
    notifempty
    compress
    delaycompress
    postrotate
        /usr/bin/systemctl restart myapp.service > /dev/null 2>&1 || true
    endscript
}
```

上述配置文件中，我们定义了对/var/log/myapp.log文件进行日志转储的规则，具体说明如下：

-  `rotate 7` ：保留7个旧的日志文件，超过7个则自动删除。

-  `daily` ：每天执行一次日志转储。

-  `missingok` ：如果日志文件不存在，则忽略错误。

-  `notifempty` ：如果日志文件为空，则不进行转储。

-  `compress` ：使用gzip压缩旧的日志文件。

-  `delaycompress` ：在下一次转储时压缩旧的日志文件。

-  `postrotate` ：在转储完成后执行的命令，例如重启服务等。

-  `endscript` ：postrotate命令的结束标记。

### 3.3 手动执行logrotate

可以使用以下命令手动执行logrotate：
```
logrotate -f /etc/logrotate.conf
```
这将按照配置文件中的规则对所有日志文件进行转储。
### 3.4 自动执行logrotate

logrotate服务会在系统启动时自动启动，并按照配置文件中的规则定期执行日志转储操作。
可以使用以下命令来检查logrotate服务的状态：
```
systemctl status logrotate.service
```
如果logrotate服务未启动，可以使用以下命令来启动它：
```
systemctl start logrotate.service
```
在修改配置文件后，需要重新启动logrotate服务才能使新的配置生效：
```
systemctl restart logrotate.service
```
### 3.5 通过 logrotate 转储 rsyslog 日志
结合使用 rsyslog 和 logrotate 可以带来更高效的日志管理。rsyslog 在接收、处理与存储日志方面具有丰富的功能与高性能，而 logrotate 带来灵活的日志轮换策略以避免资源浪费。通过调整配置，可以实现日志从生成、收集、处理到存储与轮换的全程自动管理，以满足日志保留、审计、故障排查等需求。

接下来将介绍如何在 OpenCloudOS 系统上使用 rsyslog 与 logrotate，从而能够高效地管理系统和应用日志。

#### 3.5.1 配置logrotate

要配置日志轮换，需要对logrotate进行设置。

在 `/etc/logrotate.d/` 目录下，可以为指定的日志设置logrotate配置文件。例如，为rsyslog创建名为 `rsyslog` 的配置文件：

```bash
sudo touch /etc/logrotate.d/rsyslog
```

用文本编辑器打开刚刚创建的文件，添加以下配置：

```
/var/log/cron
/var/log/maillog
/var/log/messages
/var/log/secure
/var/log/spooler
{
    daily
    rotate 7
    create
}
```

这行配置表示每天转储 `/var/log` 目录下的这些日志文件，保留最近7天的日志副本。

#### 3.5.2 设置旋转周期与保留策略

用户可根据需求设置 logrotat要转储的日志周期以及保留策略。以下几个选项可以实现不同的策略：

- `daily`：每天转储日志文件。

- `weekly`：每周转储日志文件。

- `monthly`：每月转储日志文件。

- `rotate N`：保留N个日志副本。N是一个整数。

至此，完成 logrotate 转储 rsyslog 系统日志的基本配置。

## 4. 查看系统日志文件

系统中的日志主要在/var/log/目录下，包括以下文件：

- `/var/log/messages`
	- /var/log/messages 文件包含了系统的大部分消息和错误信息，包括内核消息、系统启动信息、系统服务启动和停止信息等。这个文件通常是系统管理员进行故障排除和监控的主要来源。

- `/var/log/secure`
	- /var/log/secure 文件包含了系统的身份验证和安全相关的日志信息，包括用户登录和注销信息、sudo 命令使用信息、SSH 连接信息等。这个文件通常是系统管理员进行安全审计和监控的主要来源。

- `/var/log/audit/audit.log`
	- /var/log/audit/audit.log记录了系统中的安全审计信息。这个文件通常由 auditd 守护进程生成和维护，它可以帮助系统管理员监控系统中的安全事件和活动。

- `/var/log/dnf.log`
	- /var/log/dnf.log 文件记录了系统中使用 DNF 包管理器进行软件包安装、更新和卸载等操作的详细信息。

- `/var/log/boot.log`
	- /var/log/boot.log 文件包含了系统启动期间的日志信息，包括启动脚本的输出信息、服务启动和停止信息等。这个文件通常是系统管理员进行启动故障排除和监控的主要来源。

- `/var/log/lastlog`
	- 记录了系统中所有用户最后一次登录的信息。

- `/var/log/wtmp`
	- 记录了系统中所有用户的登录和注销信息，包括用户名、登录时间、注销时间、登录来源、登录 IP 地址等。