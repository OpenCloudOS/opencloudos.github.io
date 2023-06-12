# CentOS Discontinued Zone

## CentOS Discontinued Background

CentOS 8 has been out of maintenance on January 1, 2022, and will not be able to receive any software maintenance and support including bug fixes and feature updates thereafter, see [CentOS Official Announcement]( https://blog.centos.org/2020/12/future-is-centos-stream/?spm=a2c4g.11174386.n2.3.348f4c07hk46v4).

## OpenCloudOS Solution to Service Downtime

The suspension of Centos8 has caused great inconvenience to the original users, and many solutions have been proposed by the major communities, and the migration solution of the OpenCloudOS community has also emerged, and users can choose different migration strategies according to business needs.

### Migration Strategy

According to the operational situation of the business, different migration strategies are selected, mainly divided into the following two types：

#### Redeployment：

1.When the business is brand new, use a new operating system.   
2.The business node is already running and needs to be expanded to use a new operating system

#### In-place Migration：

1.The business node has been running for a period of time and replaces the original operating system with a new operating system.

### Migration process

#### Preparation before migration：

System backup: Before migration, a system backup is required to ensure the success rate of the migration.

Business evaluation: Before migration, it is necessary to determine the business type, dependent components, whether there is a highly available architecture, and whether it is sensitive.

System evaluation: differences in system components, system configuration, and system kernel.

#### Migration Execution:

User Migration Guide: [CentOS8 Migration to OpenCloudOS8 User Manual](migrate.en.md)

#### Post migration check:

Business check: Whether the original business can continue to run stably. 

System check: whether the kernel is the latest, whether the system version is OpenCloudOS, and whether the yum source has been replaced.

## OpenCloudOS Platform Features and Guarantees

The OpenCloudOS operating system open source community is an operating system community project initiated by operating system, software and hardware manufacturers, and individuals. It provides users with the next generation cloud native operating system that is self controllable, green, energy-saving, safe, reliable, and high-performance. Currently, there are 31 community council members and over 500 ecological partners connected, The OpenCloudOS operating system will work with numerous ecological partners to create an open and neutral operating system open source ecosystem for the future.

As of now, the OpenCloudOS operating system supports X86_ 64. ARM64, RISC-V architecture, and improved compatibility with chips such as LongArch, Feiteng, Haiguang, Zhaoxin, and Kunpeng. At the same time, it provides support for full stack state secrets and confidential computing, with a download and installation volume of tens of millions of nodes. Additionally, more than 300 enterprise products have been adapted to the OpenCloudOS operating system.

### Compatibility Guarantee For Software and Hardware manufacturers：

#### Hardware Compatibility

OpencloudOS has a complete set of standard hardware compatibility testing system, continuously testing and updating the hardware compatibility list, please refer to [Hardware Compatibility List](https://docs.opencloudos.org/adaptation/adaptation_hw/) .

#### Software Compatibility

Mainly verify the compatibility of mainstream open source software and third-party commercial software with OpenCloudOS, actively promote compatibility mutual certification with various manufacturers, and continuously update[SoftwareCompatibility List](https://docs.opencloudos.org/adaptation/adaptation_sw/) .
