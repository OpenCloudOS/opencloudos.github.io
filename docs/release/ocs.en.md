# OpenCloudOS Stream version description

OpenCloudOS Stream is an independent and controllable Upstream version jointly developed by OpenCloudOS community partners. Its kernel and user-mode software are based on upstream's independent evolution, self-compilation, independent selection and maintenance. It is no longer dependent on any distribution and is completely controllable. Through the comprehensive optimization and polishing of kernel and user-mode software, it provides users and businesses with more advanced and higher performance basic environment and service capabilities, and completely solves the problem of interrupted supply of CentOS.
## Download
Download OpenCloudOS Stream 23, please visit[link](https://mirrors.opencloudos.org/opencloudos-stream/releases/23/images/).

## OpenCloudOS Stream 23 new feature summary

### Safe startup and boot
#### dracut 057
- Resolve erratic behavior due to parallel device probing of the kernel
- Add aarch64 uefi support and introduced the connman support module
- Support the ZSTD compression firmware with the.zst suffix

#### grub2 2.06
- Support Virtual LAN;
- ARM supports EHCI, DMA, and DTB drivers
- Support UUID, PARTUUID;
- Support NVMe devices with Open Firmware text representation;
- Added support for F2FS (flash-friendly file system);
- File systems that support sparse nodes;
- Support RAID-5 or RAID-6, RAID1C34 file system, read/start/recover;
- The whole architecture supports TPM (Trusted Platform Module);
- Support RISC-V architecture;

#### systemd 251
- Sd-boot has been upgraded from PCR 8 to TPM PCR 12
- Support more features of cgroup v2
- Support cgroup v2 BPF logic
- Support GPT Discoverable Partitions Specification
- Added the udev rule for all devices triggered during each upgrade
- CPUAffinity = numa is supported
- The udev rule supports auto-suspend for PCI and USB devices
- Added UserDB-related features
- Added tools such as systemd-network-generator, systemd-sysext, systemd-repart to control system hierarchy expansion and better support for GPT
- New network device naming rule "keep"
- System Resource Limit RLIMIT_NOFILE Set to 1024 (soft) and 4096 (hard)
- Increase the maximum default inodes for/dev from 64K to 1M and for/tmp from 400K to 1M
- Now prefers the RDRAND instruction set on x86_64

### Kernel
#### Kernel 6.1, to bring the community experience of the latest achievements:
- MGLRU reduces the CPU time occupied by memory scanning and reclamation, improves the accuracy of memory hot and cold judgment, and significantly improves system performance in scenarios with high memory pressure.
- Maple Tree support, Cacheline aligned Btree implementation, ease the memory management of the key data structure competition in the multi-process concurrent scenario, improve system performance.
- Initial support for the Rust programming language, providing basic functionality implementation.
- Folios, a simple and efficient large-page memory management solution, provides complete support for multi-page Folios and improves performance by 10% in real load scenarios. Added shmem Filio support.
- io_uring: fully functional support for the new generation asynchronous I/O framework. Added zerocopy sendmsg, non-zerocopy sendto, async work schedule, and iopoll io_uring/nvme passthrough functions.
- Tiered memory - Tiered memory system support, including multiple tiers with different performance characteristics. New hot and cold page recognition and hierarchical migration algorithm, hierarchical information user mode display scheme.

#### Provide solid cloud native base support:
- CGroupFS, a simple and easy-to-use view isolation solution for system-level container resources, improves container isolation.
- Cgroup performance optimization, increased the reliable PSI V1 support, a large number of V2 exclusive interface to V1, providing BufferIO control and other functions.
- Container-level memory fine-grained classification limits reclamation and enhances container availability.

#### Other functions support:
- Support for hot patches on x86_64 and ARM64 platforms based on Livepatch reduces downtime and improves success rate.
- Kdump Enhances kernel dump adaptation and capability.
- Support the RISC-V 64 architecture.

#### Architecture support

##### ARM

- Support for new SoCs: MTK and Qcom
- Mediatek Helio X10 MT6795-M4U /IOMMU Support
- perf: The kernel supports dwarf unwinding through the SVE function
- Yitian710: Adds a DDR subsystem driver and a PMU driver
- sysreg: adds the hwcap to SVE EBF16
- iommu: supports M1 Pro/Max DART

##### x86

- bpf: x86: registers internal structure parameters in the trampoline program
- crypto: x86/sha512 - Loads based on CPU features
- intel_idle: Adds AlderLake platform support
- iommu/amd: Adds support for v2 page tables in the common IO page table framework
- mm: x86: CONFIG_ARCH_HAS_NONLEAF_PMD_YOUNG is added
- KVM: VMX: supports the newer eVMCSv1 version

##### LOONGARCH

- Added BPF JIT support
- Added kdump support
- Added kexec support
- Added perf event support


### Compilers and development environments
#### GCC 12.2.0
- Switch to TGCC (1. Feedback compilation and optimization, fit with business characteristics to improve the effect; 2. LTO link optimization; 3. Function rearrangement/basic block rearrangement based on service operation characteristics)
- ShadowCallStack sanitizer Support (aarch64 only)
- The default compiled c++ version option is 17, which implements some C++23 features
- Support armv9 and loongarch

#### Clang 14.0.5
- OpenCL 2021 is supported.
- Support for more RISCV and ARM processor models, support for ARM V9 instruction set.
- Support the X86 AVX512-FP16 instruction

#### LLVM 14.0.5
- Added support for Armv9-A, Armv9.1-A, and Armv9.2-A architectures.
- Multiple RISCV instructions changed from experimental to formal.
- Added support for x86 AVX512-FP16 instruction

#### automake 1.16.5
- Added support for zstd
- Python3 is supported

#### vim 8.2
- Added the bvimgrep command.
- Allow keymaps with '@' characters.

#### shell
- bash 5.1.16
    - Added loadable built-in commands: asort, mktemp, accept, mkfifo, csv, cut/lcut, seq, etc.
    - shell now attempts to unlink all FIFOs at exit, whether the consumption process is complete or not.
    - 'bind-x' now supports different bindings and keyboard mappings for different editing modes.
    - Writing history to syslog can now process messages longer than syslog max by writing multiple messages using serial numbers to determine the length.

- bash-completion
    - More support for commands such as nmap, openssl, ssh-keygen, and gcc; influx, dmypy, carton, scrub, ecryptfs-migrate-home, chmod, json_xs, and other new support include gssdp-discover, xvfb-run, Influx, dmypy, carton, Scrub, ecryptfs-migrate-home, chmod, and json_xs
    - Optimize and enrich test modules

#### emacs 27.2
- GUI cropped, now only terminal edit mode
- Built-in support for integers of any size
- Native support for JSON parsing

#### ed 1.18
- "ed --help" is displayed when incorrect command options are entered.
- The configuration option "--datadir" has been renamed to "--datarootdir" to comply with the GNU standard.
- Indicates that the length limit of the string has been removed.
- shell escape command (!) Now refresh the stdout so that the modified command always prints the buffer (for example, the file) before execution, even if the stdout is fully printed.
- In interactive mode, ed now sets the final exit status to 1 if a fatal error occurs.



### Programming language
#### Programming language -C /C++-boost 1.79.0
- Added C++20 support
- Improved support for RISC-V LP64D

#### Programming Language - JAVA 8/11/17
- Switch the default JDK release to KonaJDK (for big data and cloud computing scenarios, add features such as heap GC optimization, memory resource cost optimization, startup acceleration, and performance diagnostic tools)
- The default version is upgraded to KonaJDK11

#### Programming language - LUA 5.4.4
- New garbage collection mode
- Constant support
- Added the to-be-closed variable

#### Programming language - RUST 1.63.0
- Version is upgraded to 1.63.0

#### Programming language - perl 5.36.0
- perl upgrade to 5.36.0:
- Added Unicode 14.0 support
- Added support for floating point exception signals
- Added variable length lookbehind support
- Added the bultin core module
- Added a try/catch/finally exception handling mechanism
- C11 TLS feature support, improve the performance of threaded perl construction scenarios
- Scalars are created 30% faster in certain scenarios
- Thread safety support for locale operations
- Hash key allocation performance improved

#### Programming language - python 3.10
- python was upgraded to 3.10
-python-pyparsing supports the left recursive interpreter, with added support for python '-W'
    - python-asn1crypto adds sMIME implementations
    - python-decorator restore support for Sphinx
    - python-pexpect supports custom ssh commands
    - python-pip implements new dependency parser, supports PEP600, supports other python versions when installing packages, supports PEP 517 (pyproject.toml builds the system, and is the default)
    - python-setuptools supports pyproject.toml and has a higher priority than setup.py (according to PEP 621)

- Performance Improvement
    - python-pexpect significantly improves performance when handling large amounts of data
    - python-pip improves parsing performance

- Improved key functions
    - python-pycparser improves C11 support
    - python-setuptools enhances offline testing
    - python3.10 Enhanced riscv support

- Security Hardening
    - Fixed CVE-2013-0340, CVE-2021-3426, and CVE-2020-14343

### Storage, file and device management
#### lvm2 Logical Volume Management 2.03.16
- Enhanced support for large capacity devices
- Enhanced path recovery capability when cache creation fails
- Some commands have performance optimization and function enhancement

#### File system tool e2fsprogs 1.46.5
- The resize2fs command increases the maximum number of inodes supported to 2^32
- Added support for the fast_commit (COMPAT_FAST_COMMIT) feature
- Added support for the stabe_inodes (COMPAT_STABLE_INODES) feature
-chattr and lsattr added support for the 'x' attribute
- Some commands are optimized to improve the execution speed

#### Partitioning tool parted 3.5
- parted upgraded from 3.2 to 3.5:
- Added -fix in -script mode to automatically fix issues such as backup GPT headers not being at the end of the disk
- Added the use of swap partition flag on disks labeled MSDOS
- Allow partition names to be empty strings when set in script mode
- Added the -json command line switch to output the disk details as JSON
- Added support for Linux home Guids that use the Linux-home flag


### Safety capability
#### SELinux
- The semanage port command supports the dccp and sctp protocols.
- Support IPv4/IPv6 address embedding;
- Added support for Component Transport Protocol (MCTP) defined in DMTF standard DSP0236;
- Lists are allowed in constraint expressions;
- Optimized storage file name conversion in binary strategy to save more space;
- Provide optional support for kernel policy optimization, which can save policy compilation time for certain policies (such as Android);
- getconlist: sets multiple parameters to release memory.
- Prevent empty files from being stored in SELinux module by synchronizing data;
- Warning false IP addresses or network masks in nodecon statements;

#### audit service
- kernel syscall supports: Upgrade from v5.5 to v5.16;
- libev upgrade: from 4.25 to 4.33.
- New support: ausyscall support 32/64-bit, new support for armv8l architecture;
- auparse update: optimized lib library efficiency and added syscall and event types to the kernel.

#### sssd service
- Added PAM module pam_sss_gss, which uses GSSAPI for identity authentication
- Added infopipe method FindByValidCertificate(), which accepts the certificate as input, validates it against the configured CA, and outputs the user path on success
- Added support for user subuid and subgid ranges for IPA providers and the corresponding shadow-utils plug-in

#### Secure encryption
- cryptsetup
    - Support for external LUKS token plug-ins;
    - Added SSH remote token support;
    - Use Argon2id as the default LUKS2 PBKDF algorithm implementation;
    - Added support for Blake2b and Black2s hashing algorithms.
- gnupg2
    - Support the AEAD encryption mode of OCB or EAX.
    - DFN issued keys are supported
    - Telesec ESIGN applications are supported
    - Secure private keys using TPM 2.0
    - Improved the speed of gpg encryption and decryption
    - Use pipes for decryption to avoid memory exhaustion
    - Support ssh-agent authentication using openpgp
    - Force version 5 keys to be created for ed448 and cv448 algorithms
    - Support multiple types of smart cards
- gpgme
    - Support exporting keys and subkeys
    - Support WKD search without implicit import
    - Added the function of revoking the key signature and deprecating the trust list function
    - New keylist mode to ensure that the engine returns keygrip
    - Support for Native messaging and new Javascript code
- libkcapi
    - Increased support for state secret SM3 and SM4 algorithms;
- pinentry
    - Improved QT to support multiple operations on passwords
    - Improved QT pinetry ease of use, supports Caps Lock prompts, and support password formatting
    - Added Qt 5 to the list of available variants for pinentry.
    - Added genpin support
    - Support system line editing
    - Added information about tty mode, etc to "getinfo ttyinfo"
- nettle
    - Added pbkdf2_sha384 and pbkdf2_hmac_sha512;
    - Support bcrypt password hashing algorithm;
    - Improved the performance of symmetric encryption and conical encryption algorithms on PowerPC architecture;
    - Support Curve448 and ED448 signature algorithms, CFB8 symmetric encryption mode, and CMAC symmetric block encryption algorithm;
    - Support big-endian ARM architecture.
- libgcrypt
    - Added SM3 ARM/AArch64 assembly implementation
    - Improved FIPS features
    - Added straight-line speculation hardening to aarch64, amd64, and i386
    - Optimized AES aarch64-ce assembly implementation
    - Added armv8/pmull accelerated POLYVAL to GMM-SiV
    - Added automatic extension secmem functionality
    - Added mitigation measures against timed attacks.
    - Added support for key wrapping (KWP) with padding.
    - Added x86_64 VAES/AVX2 acceleration implementation
- libgpg-error
    - Updated and upgraded the translation function
    - Support GNU C library 2.34 or later.
    - Lock test when adding a single thread.
    - Upgrade and improve the arg parser
- libxcrypt
    - Added support for three new hashing algorithms, yescrypt, scrypt and gost-yescrypt;
    - Support Elbrus-2000(ek2) architecture (Russian developed CPU);
    - Support Python 3.11 in configuration scripts;
    - Added a collection of Fedora and Debian hashing algorithms to the configuration file.
#### Safe and trusted
- ima-evm-utils
    - Added support for elliptic curve encryption algorithm;
    - Added support for PKCS#11 standard;
    - Support IMA measurement results validation for multiple TPM banks;
- tpm2-tss
    - Increased support for national secret SM2, SM3, and SM4 algorithms;
    - Compatible with OpenSSL 3.0.0;
    - Support more new TPM command invocations;
    - Added support for SWTPM-TCTI (TPM Command Transfer interface).
#### Security infrastructure
- pam 1.5.2
    - Change the HMAC algorithm to call openssl instead of tying sha1.
    - New module pam_faillock: Used to lock after multiple authentication failures.
    - New module pam_setquota: Sets or modifies disk quota when session starts.
    - New module pam_usertype: Used to determine whether the uid is in login state.
    - Support the (gost-)yescrypt hash method.
- gnutls 3.7.6
    - New API to implement QUIC transport protocol;
    - Provide acceleration of a range of encryption operations;
    - Support the certificate compression standard defined in RFC8879.
- openssl 3.0.5
    - Support the kernel TLS protocol (KTLS);
    - Introducing the concept of "providers" to enable algorithmic implementations to be specified programmatically or in configuration files for a given application;
    - The SHA1-based signature certificate is not trusted by the client and server.

### Network service
#### nftables 1.0.4
- Support runtime automatic merging of collection elements;
- Rule set optimization enhancements to the -o/--optimize option allow multiple NAT rules to be merged into a map;
- Support for ip and tcp options and sctp blocks in collections
- 802.1ad (QinQ) support;
- cgroupsv2 Supported.
#### iptables 1.8.8
- Improved performance when matching IP/MAC, old iptables does not recognize masks;
- Full matching support, including hybrid MAC and MAC+IP collection entries
#### postfix 3.7.2
- Support lmdb, tls and pcre2 libraries
- Randomize the hash table in the initial memory to prevent hash collisions
- Upgrade the defense function to prevent remote clients or servers from controlling SMTP and LMTP traffic
- Support for thread bounce, which allows mail readers to display notifications of non-delivery, delayed delivery, or successful delivery in the same email thread as the original message.
#### Web tools
- arpwatch
    - Speed up arp.dat resolution by increasing the hash table size
- curl
    - Use msh3 to add support for QUIC and HTTP/3
    - Enable multiplexing by default
    - Support parallel transmission
    - Support wildcard hosts
    - Added zstd decoding support
    - Added fingerprint support for SHA256, CURLINFO_CERTINFO for secure transmission
- ethtool
    - Added support for OSFP transceiver module
    - Added QSFP-DD support
    - Added support for statistics in subcommands
    - Added support for dumping statistics
- ipcalc
    - Improved json output by adding -j/--json output mode to print information in JSON format.
    - Set the broadcast address based on RFC3021
    - IANA IPv6 private address registration support updated to 2019-09-13
    - Update the IANA IPv4 Private Address Registry to 2020-04-06
- iputils
    - Disable setcap and setuid to prevent ARP poisoning
    - When using SOCK_RAW sockets, use random values for identifier fields instead of PID.
- tcpdump
    - Added support for new network protocols such as Arista, Autosar SOME/IP, IEEE 802.15.9, IPoIB, SLL2, vsockmon, MACsec, OpenFlow 1.3, PTP, ZEP, etc
    - Added remote capture

### Database support
#### gdbm 1.23
- Bucket cache switches from balanced tree to hash table to speed up flushing of changed buckets on disk;
- gdbm_setopt Provides new options such as automatic cache, directory depth, and maximum number of keys.
- Preread memory mapped regions on systems that support it;
- Lock types are retained during database reorganization.
#### libtdb 1.4.7
- Improved performance for parallel queries, highly concurrent workloads, partitioned tables, logical replication, and cleanup;
- The management efficiency of B-tree index update is higher and index inflation is reduced;
- Implement the ability to channel multiple queries, improving throughput for high-latency connections;
#### sqlite 3.39.0
- Built-in JSON support
- The database file capacity increased to 281 TB
- Reduced CPU usage by 2.3%
- Consumes less memory when manipulating schemas
- SQL commands run against malicious database corruption enhance robustness.
#### mariadb-connector-c 3.3.1
- Support Zstd format
- Semi-synchronous replication is supported
- The password length is longer than 255 characters
- Support openssl 3.0

### Base library
#### glibc 2.35
- Reliability characteristics
    - The aarch64 architecture provides branch protection security hardening including branch targetidentification (BTI) and pointer authentication for return addresses (PAC-RET).
    - The memory allocation interface sets the upper limit through PTRDIFF_MAX to prevent potential overflow from mistakenly applying for too much memory. Improved reliability
    - Added asynchronous signal security function fork family function _Fork to meet users' asynchronous signal security requirements
- New function extension
    - 1. Unicode encodings are supported up to 14.0.0, with a wider range of encodings supported
    - 2. Provide new tunable environment variables for flexible use of functions and performance, specifically:
    - glibc.pthread.stack_cache_size Sets the size of thread stack cache
    - glibc.rtld.dynamic_sort Specifies the DSO sorting algorithm
    - glibc.malloc.hugetlb works with large page configurations of the kernel to improve system performance
- userspace-rcu
    - Added support for RISC-V architecture
#### glib2 2.72
- Added new apis: cross-platform API for aligning memory allocation, asynchronous file movement API, notification of memory pressure API
- Upgrade to Unicode character database v12.1
#### libcap 2.64
- Import libpsx library to simulate POSIX system calls for all PThreads
- New kernel features: extended Berkeley Packet Filters (CAP_BPF)
- Support for Go module abstraction and support for specifying build go binaries
- Thread safety improvements to libcap and cap packages
#### openldap 2.6.2
- Added slapo-autoca, slapo-otp, libldap, and slapd support for OpenSSL 3.0

#### 解压缩基础库
#### Decompress the base library
- lz4 1.9.3
    - The ARM64 upper decompression speed is increased by up to 20%
    - BD4 Increases the decompression speed of compressed data by up to 70%
    - lz4frame increases the decompression speed of compressed data by up to 40%
- zstd 1.5.2
    - Faster compression and decompression speed, rebalancing the intermediate compression level
    - Improved high-level compression
    - More than twice the speed of long distance mode, small block decompression speed, faster directory compression
- zlib 1.2.12
    - Speed up CRC-32 calculations using hardware CRC-32 instructions on ARMv8
### Package Management
#### Software package management and update
- Added incremental update capability for rpm
- Added support for loongarch
- Richer control commands and parameters, and more detailed control
- Warehouse metadata reading is more efficient and stable
- Better support for Python3.10

### Graphics library and language font support
#### gtk3 3.24.34
- Fonts provide more flexible Settings, such as setting OpenType fonts and allowing you to select OpenType font variants
- More emoji features, such as support for Emoji completion pop-up window, support for deleting shortcut keys, etc
- Allow color selection from the screen
- gtk-text-input and text-input-unstable-v3 are supported as input protocols
#### libjpeg-turbo 2.1.3
- Optimized performance on cpus that support AVX2 instructions
- MIPS platform performance optimization
- Optimized ARM64 performance
- Java binding supports only x64
#### libpng 1.6.37
- Optimized performance on ARM
#### freetype 2.12.1
- Contour glyphs that support color layering (e.g., extensible emoji glyphs)
- Supports the WOFF 2 font
#### harfbuzz 4.2.1
- Unicode 15 support
- Color fonts are supported
- Support fonts with more than 65535 glyphs in the "GDEF", "GSUB", and "GPOS" tables, and fully supports more than 65535 glyfs in the "glyf" table starting from version 4.0.0
- Load and subset fonts faster, especially when working with CFF tables, subsets some fonts up to 3 times faster

### Documentation support
#### ctags 5.9
- Brand new version of Universal Ctags, incompatible with Exuberant Ctags
- A series of extensions and upgrades on the parser
- Character parsing is stricter
#### expat 2.4.8
- Upgrade: Support more commands and software platforms
- Security: Upgrades the defense against network attacks to improve software security
#### hunspell 1.7.1
- Improved the suggestion performance
- Improved the Hunspell command line
- More support to incorporate Weblate translation functionality
#### man-db 2.10.2
- Significantly improve performance
- New man-recode program designed for fast batch conversion
- Support for more national languages
#### libcomps 0.1.18
- Build with the new cmake macro
- Resolve memory leaks and resource leaks
#### texinfo 6.8
- Improved internal modules to improve parsing speed
- Support directory compression
- New language features
#### json-c 0.16
- Improved parsing speed
- Reduce memory usage
- Added several new functions adding support for unsigned 64-bit integer uint64_t

## Vulnerability management
OpenCloudOS Stream bug tracker:

- [http://bugs.opencloudos.tech/](http://bugs.opencloudos.tech/)

If you have any problems with the OpenCloudOS Stream version, we sincerely welcome more valuable suggestions from users and developers in the community. We still have a lot to improve and improve.
