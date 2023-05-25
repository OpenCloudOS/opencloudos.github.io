# OpenCloudOS Stream 版本说明

OpenCloudOS Stream 是 OpenCloudOS 社区联合伙伴共同研发的自主可控的上游版本，其内核及用户态软件均基于社区 Upstream 独立演进、自编译，自主选型和维护，不再依赖任何发行版，完全自主可控。通过内核，用户态软件的全面优化和打磨，为用户和业务提供更先进、更高性能的基础环境和服务能力，彻底解决 CentOS 断供的问题。

## 下载
载 OpenCloudOS Stream 23，请访问[链接](https://mirrors.opencloudos.org/opencloudos-stream/releases/23/images/)。

## OpenCloudOS Stream 23 新特性汇总

### 安全启动与引导
#### dracut 057
- 解决由于内核的并行设备探测导致的不稳定行为
- 添加 aarch64 uefi 支持，引入 connman 支持模块
- 支持带 .zst 后缀的 ZSTD 压缩固件

#### grub2 2.06
- 支持 Virtual LAN；
- ARM 支持 EHCI，DMA，DTB 驱动支持
- 支持 UUID，PARTUUID；
- 支持包含 Open Firmware text representation 的 NVMe 设备；
- 添加对 F2FS（flash-friendly file system）的支持；
- 支持稀疏节点的文件系统；
- 支持 RAID-5 或 RAID-6，RAID1C34 的文件系统，读取 / 启动 / 恢复；
- 全架构支持 TPM（可信平台模块）；
- 支持 RISC-V 架构；

#### systemd 251
- sd-boot 从原来的 PCR 8 升级到支持 TPM PCR 12
- 支持 cgroup v2 更多特性
- 支持 cgroup v2 BPF 相关逻辑
- 支持 GPT Discoverable Partitions Specification 相关特性
- 新增每次升级时触发所有设备 udev 规则
- 选项 CPUAffinity = 支持 numa
- udev 规则支持 PCI 和 USB 设备的 auto-suspend
- 新增 userdb 相关特性
- 新增工具 systemd-network-generator、systemd-sysext、systemd-repart 等来控制系统拓展层级、更好地支持 GPT 等
- 新增网络设备命名规则 “keep”
- 系统资源限制 RLIMIT_NOFILE 设置为 1024（soft）和 4096（hard）
- 将 / dev 的最大默认 inodes 数量从 64K 提高到 1M，将 / tmp 的从 400K 提高到 1M
- 现在在 x86_64 上偏向使用 RDRAND 指令集

### 内核
#### Kernel 6.1，带给社区最新成果体验：
- MGLRU支持，降低内存扫描与回收所占用CPU时间，同时提高内存冷热判断精度，显著提升大内存压力场景下系统性能。
- Maple Tree 支持，Cacheline 对齐的Btree实现，缓解多进程并发场景下的内存管理关键数据结构竞争问题，提升系统性能。
- Rust编程语言初始支持，提供基础功能实现。
- Folios，简易高效大页内存管理方案，带来multi-page Folios完整支持，众多真实负载场景性能提升达到10%。新增shmem Filio支持。
- io_uring，新一代异步IO框架全功能支持。新增zerocopy sendmsg、non-zerocopy sendto、async work schedule、iopoll io_uring/nvme passthrough等功能。
- Tiered memory - 分层内存系统支持，包括具有不同性能特性的多层内存系统。新增冷热页识别及层级迁移算法、层级信息用户态展示方案等。

#### 提供坚实的云原生基座支持：
- CGroupFS，简洁易用的系统级容器资源视图隔离方案，整体提升容器隔离性。
- Cgroup性能优化，增加了可靠的PSI V1支持，将大量V2专属接口适配到V1，提供BufferIO控制等功能。
- 容器级别内存细粒度分类限制回收功能，增强容器可用性。

#### 其他功能支持：
- 基于Livepatch的x86_64与ARM64平台热补丁功能支持，降低downtime，提高成功率。
- Kdump内核转储适配与能力增强等。
- 支持 RISC-V 64 架构。

#### 架构支持

##### ARM

- 支持新SoC：MTK、Qcom为主
- 联发科Helio X10 MT6795 - M4U/IOMMU支持
- perf: 内核支持通过SVE函数进行dwarf unwinding
- Yitian710：添加DDR子系统驱动PMU驱动 
- sysreg：为SVE EBF16添加hwcap 
- iommu：支持M1 Pro/Max DART 

##### x86

- bpf: x86: 在trampoline程序中支持注册内结构参数
- crypto: x86/sha512 - 基于CPU特性的负载
- intel_idle: 添加AlderLake平台支持
- iommu/amd: 增加通用IO页表框架对v2页表的支持
- mm: x86: 添加 CONFIG_ARCH_HAS_NONLEAF_PMD_YOUNG
- KVM: VMX: 支持更新的eVMCSv1版本

##### LOONGARCH

- 添加BPF JIT支持
- 添加kdump支持
- 添加kexec支持
- 添加perf event支持


### 编译器及开发环境
#### GCC 12.2.0
- 切换为 TGCC（1. 反馈编译优化, 贴合业务特性提升效果；2. LTO 链接优化；3. 基于业务运行特征实施函数重排 / 基本块重排）
- 支持 ShadowCallStack sanitizer（仅 aarch64）
- 默认编译的 c++ 版本选项为 17，实现一部分 C++23 特性
- 支持 armv9，支持 loongarch

#### Clang 14.0.5
- 支持 OpenCL 2021。
- 支持更多的 RISCV 和 ARM 处理器型号，支持 ARM V9 指令集。
- 支持 X86 的 AVX512-FP16 指令

#### LLVM 14.0.5
- 添加了对 Armv9-A、Armv9.1-A 和 Armv9.2-A 架构的支持。
- 多项 RISCV 指令从实验性变为正式。
- 增加支持 x86 AVX512-FP16 指令

#### automake 1.16.5
- 增加支持 zstd
- 支持 python3

#### vim 8.2
- 新增 bvimgrep 命令。
- 允许带有‘@’字符的 KEYMAP。

#### shell
- bash 5.1.16
    - 新增可加载内置命令：asort，mktemp、accept、mkfifo、csv、cut/lcut、seq 等。
    - shell 现在尝试在退出时取消链接所有 FIFO，无论消费过程是否已经完成。
    - `bind -x' 现在支持不同编辑模式的不同绑定和键盘映射。
    - 将历史记录写入 syslog 现在可以处理比 syslog max 更长的消息通过使用序列号写入多条消息来确定长度。

- bash-completion
    - 对 nmap、openssl、ssh-keygen、gcc 等命令提供更多的支持；新支持 gssdp-discover、xvfb-run、influx、dmypy、carton、scrub、ecryptfs-migrate-home、chmod、json_xs 等
    - 优化丰富了测试模块

#### emacs 27.2
- 裁剪了 GUI，现在只有终端编辑模式
- 内置支持任意大小的整数
- 原生支持 JSON 解析

#### ed 1.18
- 在命令选项输入有误时提示 “ed --help” 的显示内容。
- 配置选项 “--datadir” 已重命名为 “--datarootdir” 以遵循 GNU 标准。
- 提示字符串的长度限制已被删除。
- shell 转义命令（!）现在刷新标准输出，以便修改后的命令即使标准输出已完全输出，也始终在执行之前打印缓冲（例如，文件）。
- 在交互模式下，如果出现致命错误，ed 现在将最终退出状态设置为 1。


### 编程语言
#### 编程语言 - C/C++-boost 1.79.0
- 新增 C++20 支持
- 完善对于 RISC-V LP64D 支持

#### 编程语言 - JAVA 8/11/17
- 默认 JDK 发行版切换为 KonaJDK（针对大数据、云计算场景，增加大堆 GC 优化、内存资源成本优化、启动加速、性能诊断工具等特性）
- 默认版本升级为 KonaJDK11

#### 编程语言 - LUA 5.4.4
- 新的垃圾回收模式
- 常量支持
- 新增 to-be-closed 变量

#### 编程语言 - RUST 1.63.0
- 版本升级到 1.63.0

#### 编程语言 - perl 5.36.0
- perl 版本升级到 5.36.0:
- 新增 Unicode 14.0 支持
- 新增浮点异常信号支持
- 新增可变长的后行断言（lookbehind）支持
- 新增 bultin 核心模块
- 新增 try/catch/finally 异常处理机制
- C11 TLS 特性支持，线程化 perl 构建场景性能提升
- 特定场景下标量创建速度提升 30%
- 语言环境操作线程安全支持
- Hash 表 key 分配性能提升

#### 编程语言 - python 3.10
- python 版本升级到 3.10
    - python-pyparsing 支持左侧递归解释器，增加对 python `-W` 支持
    - python-asn1crypto 增加 sMIME 实现
    - python-decorator 恢复支持 Sphinx
    - python-pexpect 支持自定义 ssh 命令
    - python-pip 实现新的依赖解析器，支持 PEP600，支持安装软件包时选择其他 python 版本，支持 PEP 517（pyproject.toml 构建系统，且为默认方式）
    - python-setuptools 支持 pyproject.toml，且优先级高于 setup.py（根据 PEP 621）

- 性能提升
    - python-pexpect 处理大量数据时显著的性能提升
    - python-pip 提升解析性能

- 重要功能改善
    - python-pycparser 改善对 C11 支持
    - python-setuptools 增强离线测试功能
    - python3.10 增强 riscv 支持

- 安全加固
    - 修复 CVE-2013-0340、CVE-2021-3426、CVE-2020-14343

### 存储、文件及设备管理
#### 逻辑卷管理 lvm2 2.03.16
- 增强了大容量设备的支持能力
- 增强了缓存创建失败情形下路径恢复能力
- 部分命令进行了性能优化及功能增强

#### 文件系统工具 e2fsprogs 1.46.5
- 命令 resize2fs 支持的最大 inode 数量增大到 2^32
- 新增了对 fast_commit (COMPAT_FAST_COMMIT) 特性的支持
- 新增了对 stabe_inodes (COMPAT_STABLE_INODES) 特性的支持
- chattr 和 lsattr 新增了对'x' 属性的支持
- 部分命令进行了优化，提升了执行速度

#### 分区工具 parted 3.5
- parted 从 3.2 升级至 3.5 版本：
- 在 -script 模式中增加 -fix，以自动修复备份 GPT header 不在磁盘末端等问题
- 在标记为 MSDOS 的磁盘上增加使用交换分区标志
- 在 script 模式下设置时，允许分区名称为空字符串
- 增加 -json 命令行开关，将磁盘的细节输出为 JSON
- 增加对使用 linux-home 标志的 Linux home GUID 的支持


### 安全能力
#### SELinux
- 在 semanage port 命令中支持 “dccp” 和“sctp”协议；
- 支持 IPv4/IPv6 地址嵌入；
- 添加了对 DMTF 标准 DSP0236 定义的组件传输协议（MCTP）的支持；
- 约束表达式中允许使用列表；
- 在二进制策略中实现存储文件名转换的优化，更加节省空间；
- 提供了内核策略优化的可选支持，可为某些策略（例如 Android）节省策略编译时间；
- getconlist: 设置多级参数释放内存；
- 通过同步数据以防止 SELinux 模块存储中出现空文件；
- 对 nodecon 语句中的虚假 IP 地址或网络掩码进行警告；

#### audit 服务
- kernel syscall 版本支持：从 v5.5 升级到 v5.16；
- libev 升级：从 4.25 升至 4.33；
- 新增支持：ausyscall 支持 32/64-bit，新增支持 armv8l 架构；
- auparse 更新：优化 lib 库效率，对内核新增 syscall 和 event 类型的支持；

#### sssd 服务
- 新增 PAM 模块 pam_sss_gss，该模块使用 GSSAPI 进行身份认证
- 新增 infopipe 方法 FindByValidCertificate()，该方法接受证书作为输入，根据配置的 CA 对其进行验证，并在成功时输出用户路径
- 新增对 IPA 提供者的用户 subuid 和 subgid 范围的支持以及相应的 shadow-utils 插件

#### 安全加密
- cryptsetup
    - 支持外部 LUKS 令牌插件；
    - 新增 SSH 远程令牌支持；
    - 使用 Argon2id 作为默认的 LUKS2 PBKDF 算法实现；
    - 新增对 Blake2b 和 Black2s 哈希算法的支持。
- gnupg2
    - 支持使用 OCB 或 EAX 的 AEAD 加密模式。
    - 支持 DFN 颁发的密钥
    - 支持 Telesec ESIGN 应用程序
    - 使用 TPM 2.0 来保护私钥
    - 提升 gpg 加密，解密等过程的速度
    - 使用管道进行解密，避免内存耗竭
    - 支持使用 openpgp 进行 ssh-agent 身份验证
    - 强制为 ed448 和 cv448 算法创建版本 5 密钥
    - 支持多种类型的智能卡
- gpgme
    - 支持导出密钥和子密钥
    - 支持 WKD 查找，无需隐式导入
    - 增加撤销密钥签名的功能函数，弃用信任列表功能
    - 新增 keylist 模式，保证引擎返回 keygrip
    - 支持 Native 消息传递以及新的 Javascript 代码
- libkcapi
    - 增加对国密 SM3 和 SM4 算法的支持；
- pinentry
    - 改进了 QT，支持对密码的多种操作
    - 提升 QT pinetry 的易用性，支持 Caps Lock 提示，支持密码格式化
    - 将 Qt 5 添加到 pinentry 的可用变体列表中。
    - 添加 genpin 支持
    - 支持系统行编辑
    - 将有关 tty 模式等的信息添加到 “getinfo ttyinfo”
- nettle
    - 新增 pbkdf2_sha384 和 pbkdf2_hmac_sha512；
    - 支持 bcrypt 密码哈希算法；
    - 在 PowerPC 架构上对对称加密和圆锥曲线加密算法做了显著的性能优化；
    - 支持 Curve448 和 ED448 签名算法，支持 CFB8 对称加密模式，支持 CMAC 对称分组加密算法；
    - 支持大端 ARM 架构系统。
- libgcrypt
    - 添加 SM3 ARM/AArch64 汇编实现
    - 提升 FIPS 模式下各个特性
    - 为 aarch64 、amd64 和 i386 添加 straight-line speculation hardening
    - 优化 AES aarch64-ce 汇编实现
    - 为 GCM-SIV 添加 armv8/pmull 加速的 POLYVAL
    - 添加自动扩展 secmem 功能
    - 添加针对定时攻击的缓解措施。
    - 添加对带填充的密钥包装（KWP）的支持。
    - 添加 x86_64 VAES/AVX2 加速实现
- libgpg-error
    - 更新和升级了翻译功能
    - 支持 GNU C 库 2.34 或更高版本。
    - 添加单线程时的锁定测试。
    - 升级和改进 arg 解析器
- libxcrypt
    - 增加对 yescrypt、scrypt 和 gost-yescrypt 三种新哈希算法的支持；
    - 支持 Elbrus-2000(ek2) 架构（俄罗斯自研 CPU）；
    - 在配置脚本中支持 Python 3.11；
    - 在配置文件中添加 Fedora 和 Debian 的哈希算法集合。
#### 安全可信
- ima-evm-utils
    - 增加对椭圆曲线加密算法的支持；
    - 增加对 PKCS#11 标准的支持；
    - 支持多个 TPM bank 的 IMA 度量结果验证；
- tpm2-tss
    - 增加对国密 SM2、SM3 和 SM4 算法的支持；
    - 兼容 OpenSSL 3.0.0；
    - 支持更多新的 TPM 命令调用；
    - 增加对 SWTPM-TCTI（TPM 命令传输接口）的支持。
#### 安全基础库
- pam 1.5.2
    - 将 HMAC 算法更改为可调用 openssl 而非捆绑 sha1。
    - 新模块 pam_faillock：多重身份验证失败后用于锁定。
    - 新模块 pam_setquota：会话启动时设置或修改磁盘配额。
    - 新模块 pam_usertype：用于判断 uid 是否处于登录状态。
    - 支持 (gost-)yescrypt 哈希方法。
- gnutls 3.7.6
    - 新增 API 以实现 QUIC 传输协议；
    - 提供对一系列加密运算的加速；
    - 支持 RFC8879 定义的证书压缩标准。
- openssl 3.0.5
    - 支持内核 TLS 协议（KTLS）；
    - 引入 “提供者”（Provider）概念，支持以编程方式或配置文件指定用于给定应用程序的算法实现；
    - 基于 SHA1 的签名证书将不被认证 client 和 server 信任；

### 网络服务
#### nftables 1.0.4
- 支持集合元素的运行时自动合并；
- 规则集优化 -o/--optimize 选项的增强 允许将多个 NAT 规则合并到映射中；
- 支持 ip 和 tcp 选项以及集合中的 sctp 块
- 802.1ad (QinQ) 支持；
- cgroupsv2 支持；
#### iptables 1.8.8
- 改善在匹配 IP/MAC 时的表现，旧 iptables 不会识别掩码；
- 完整的匹配支持，包括混合 MAC 和 MAC+IP 的集合 条目
#### postfix 3.7.2
- 支持 lmdb，tls 和 pcre2 库
- 随机化初始内存中的哈希表，以防御哈希冲突
- 升级防御功能防止远程客户或者服务器控制 SMTP 和 LMTP 流量
- 支持线程反弹，这允许邮件阅读器在与原始邮件相同的电子邮件线程中显示未送达、延迟送达或成功送达通知。
#### 网络工具
- arpwatch
    - 通过增加哈希表的大小来加速 arp.dat 解析
- curl
    - 使用 msh3 添加对 QUIC 和 HTTP/3 的支持
    - 默认启用多路复用
    - 支持并行传输
    - 支持通配符主机
    - 添加 zstd 解码支持
    - 添加 SHA256 指纹支持，安全传输上支持 CURLINFO_CERTINFO
- ethtool
    - 添加对 OSFP 收发器模块的支持
    - 添加 QSFP-DD 支持
    - 在子命令中添加对统计信息的支持
    - 添加对转储统计信息的支持
- ipcalc
    - 改进 json 输出, 添加 -j/--json 输出模式以使用 JSON 格式打印信息。
    - 根据 RFC3021 设置广播地址
    - 将 IANA IPv6 专用地址注册支持更新到 2019-09-13
    - 将 IANA IPv4 专用地址注册表更新至 2020-04-06
- iputils
    - 禁用 setcap 和 setuid，防止 ARP 中毒
    - 在使用 SOCK_RAW 套接字时，对标识符字段使用随机值而不是 PID。
- tcpdump
    - 增加了对新网络协议如：Arista, Autosar SOME/IP, IEEE 802.15.9, IPoIB, SLL2, vsockmon, MACsec, OpenFlow 1.3, PTP, ZEP 等的支持
    - 增加 remote capture 特性

### 数据库支持
#### gdbm 1.23
- 桶缓存从平衡树切换到哈希表，加快刷新磁盘上更改的存储桶；
- gdbm_setopt 提供自动缓存、获取目录深度、最大密钥数等新选项；
- 在支持它的系统上预读内存映射区域；
- 在数据库重组期间保留锁定类型。
#### libtdb 1.4.7
- 提升了并行查询、高度并发的工作负载、分区表、逻辑复制和清理许多性能；
- 采用 B-tree 索引更新的管理效率更高，减少了索引膨胀；
- 实现了管道多个查询的能力，提高高延迟连接的吞吐量；
#### sqlite 3.39.0
- 内置 JSON 支持
- 数据库文件容量增加到 281 TB
- 减少 CPU 2.3% 使用
- 操作 schema 时消耗更少内存
- 针对恶意破坏数据库运行的 SQL 命令增强了稳健性。
#### mariadb-connector-c 3.3.1
- 支持 Zstd 格式
- 支持半同步复制
- 支持大于 255 字符长度密码
- 支持 openssl 3.0

### 基础库
#### glibc 2.35
- 可靠性特性
    - aarch64 架构提供了包括 BTI(branch targetidentification) 和 PAC-RET(pointer authentication for return addresses) 在内的分支保护安全加固；
    - 内存分配相关接口通过 PTRDIFF_MAX 设置了上限，避免潜在的溢出误申请过大内存的行为。提高了可靠性
    - 新增异步信号安全函数 fork 族函数_Fork，用以满足用户的异步信号安全需求
- 新功能扩展
    - 1、Unicode 编码支持到 14.0.0，支持编码范围更广
    - 2、提供新的 tunable 环境变量满足功能和性能上的灵活运用，具体为：
        - glibc.pthread.stack_cache_size 配置线程栈 cache 大小
        - glibc.rtld.dynamic_sort 选择 DSO 排序算法
        - glibc.malloc.hugetlb 通过与内核的大页配置相配合，提高系统性能
- userspace-rcu
    - 增加对 RISC-V 架构的支持
#### glib2 2.72
- 添加新 API：对齐内存分配的跨平台 API、异步文件移动 API、通知内存压力情况 API
- 升级到 Unicode 字符数据库 v12.1
#### libcap 2.64
- 引入库 libpsx，可以模拟所有 pthread 的 POSIX 系统调用
- 新的内核功能：CAP_BPF（extended Berkeley Packet Filters）
- 支持 Go 模块抽象，支持指定构建的 go 二进制文件
- libcap 和 cap 包的线程安全性的改进
#### openldap 2.6.2
- 添加了对 OpenSSL 3.0 的 slapo-autoca、slapo-otp、libldap、slapd 支持

#### 解压缩基础库
- lz4 1.9.3
    - ARM64 上解压缩速度最高提升 20%
    - BD4 压缩的数据，解压速度最高提升 70%
    - lz4frame 格式压缩的数据解压速度最高提升 40%
- zstd 1.5.2
    - 更快的压缩解压缩速度，重平衡中间压缩 level
    - 改进的 high-level 压缩率
    - long distance 模式 2 倍以上的提速，小块的解压缩速度提升，更快的目录压缩
- zlib 1.2.12
    - 加速 CRC-32 计算，ARMv8 上使用硬件 CRC-32 指令

### 软件包管理
#### 软件包管理与更新
- 新增 rpm 的增量更新能力
- 新增对 loongarch 的支持
- 更丰富的控制命令、参数，更细致的控制
- 仓库元数据读取更高效，功能更稳定
- 对 Python3.10 更好的支持

### 图形库及语言字体支持
#### gtk3 3.24.34
- 字体提供了更多灵活的设置，如设置 OpenType 字体、允许选择 OpenType 字体变体等
- emoji 功能更丰富，如支持 Emoji 的完成弹出窗口、支持删除快捷键等
- 允许从屏幕上挑选颜色
- 同时支持 gtk-text-input 和 text-input-unstable-v3 作为输入协议
#### libjpeg-turbo 2.1.3
- 在支持 AVX2 指令的 CPU 上性能优化
- 龙芯（MIPS）平台性能优化
- 优化 ARM64 性能
- java 绑定仅支持 x64
#### libpng 1.6.37
- 优化 ARM 上性能
#### freetype 2.12.1
- 支持彩色分层的轮廓字形（例如，可扩展的表情符号字形）
- 支持 WOFF 2 字体
#### harfbuzz 4.2.1
- 支持 Unicode 15 support
- 支持彩色字体
- 支持 “GDEF”、“GSUB” 和“GPOS”表中超过 65535 个字形的字体，完全支持从 4.0.0 版本开始的 “glyf” 表中超过 65535 个字形
- 加载和子集字体的速度加快，尤其是在处理 CFF 表时，对某些字体进行子集化快 3 倍

### 文档支持
#### ctags 5.9
- 全新版本的 Universal Ctags，与 Exuberant Ctags 不兼容
- 解析器上进行一系列扩展升级
- 字符解析更加严格
#### expat 2.4.8
- 升级：支持更多的命令和软件平台
- 安全：升级对网络攻击的抵御，提升软件安全性
#### hunspell 1.7.1
- 提升 suggestion 性能
- 改进 Hunspell 命令行
- 提供更多支持，合并 Weblate 翻译的功能
#### man-db 2.10.2
- 显著提升性能
- 新增为快速批量转换而设计的 man-recode 程序
- 支持更多国家语言
#### libcomps 0.1.18
- 使用新的 cmake 宏进行构建
- 解决内存泄露和资源泄露问题
#### texinfo 6.8
- 改进内部模块，提升解析速度
- 支持压缩目录
- 语言新增特性
#### json-c 0.16
- 提升解析速度
- 减少内存使用
- 增加多个新功能函数，添加对无符号 64 位整数 uint64_t 的支持

## 漏洞管理
OpenCloudOS Stream 的 bug 追踪系统：

- [http://bugs.opencloudos.tech/](http://bugs.opencloudos.tech/)

使用 OpenCloudOS Stream 版本遇到和任何问题，诚挚欢迎社区的用户、开发者朋友多提宝贵建议。我们还有很多需要改进和完善的地方。
