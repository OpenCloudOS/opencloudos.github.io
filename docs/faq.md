# 常见问题

Q1: 请问 OpenCloudOS 有什么特点，适合什么场景使用？
A1: OpenCloudOS 是由操作系统、云平台、软硬件厂商与个人共同倡议发起的操作系统社区项目。成立之初，即决定成为完全开放中立的开源社区。社区将打造全面中立、开放、安全、稳定易用、高性能的 Linux 服务器操作系统为目标，与成员单位共同构建健康繁荣的国产操作系统生态。OpenCloudOS 适用于物理机，虚拟机，docker 子机，适用于金融，互联网，政务，桌面等各类场景。

Q2: OpenCloudOS 有哪些自研的特性？
A2: 请参考 [这个页面](http://www.opencloudos.org/?p=537)。

Q3: OpenCloudOS 8.5 和 CentOS 8.5 兼容吗？怎样从 CentOS 迁移到 OpenCloudOS？
A3: OpebCloudOS 8.5 和 CentOS 8.5 保持用户态二进制兼容。参考 [官方文档中的迁移指引](https://docs.opencloudos.org/guide/migrate/?h=%E8%BF%81%E7%A7%BB) 进行迁移。

Q4: 迁移工具支持的哪些操作系统版本？
A4: 支持除了 Centos Stream 8 外的所有 Centos 8 版本，更多版本在持续更新中。

Q5: 迁移之前是否需要备份数据？
A5: 需要，备份数据请参考云主机 [创建快照](https://cloud.tencent.com/document/product/362/5755)，物理机需手动备份。

Q6: 是否支持物理机迁移？
A6: 迁移工具同时支持虚拟机和物理机的迁移。

Q7: 迁移过程出现异常如何解决？
A7: 请在 [OpenCloudOS 社区 Bugzilla](https://bugs.opencloudos.tech/) 上提交 bug，我们的工程师会为您解答。

Q8: OpenCloudOS 兼容性做的如何？
A8: 我们在社区内持续更新软件和硬件兼容性列表，同时我们有一套标准社区适配流程，详情参见 [生态认证流程](https://docs.opencloudos.org/adaptation/adaptation_process/)，软件兼容性可参考 [商业软件兼容列表](https://docs.opencloudos.org/adaptation/adaptation_sw/) 以及 [开源软件兼容列表](https://docs.opencloudos.org/adaptation/adaptation_oss/)，硬件兼容性可参考 [硬件兼容列表](https://docs.opencloudos.org/adaptation/adaptation_hw/)。

Q9: 迁移过程有哪些需要注意？
A9: 迁移过程注意事项请参考 [官方文档](https://docs.opencloudos.org/guide/migrate/?h=%E8%BF%81%E7%A7%BB#_2)。

Q10: CentOS 8 已于 2021 年底停止更新，OpenCloudOS 受影响吗？
A10: OpenCloudOS 8 用户态软件包基于 RHEL 8 源码包重编译，内核基于社区 5.4 lts 内核版本定制开发，不受 CentOS 停更的影响。

Q11: 在哪里可以查看当前版本的生命周期以及后续版本的发布计划？
A11: 请参考 [该文档](https://doc.weixin.qq.com/doc/w3_AIgAbAbdAFw8Wuk5xl6RzWkDNb0QQ?scode=AJEAIQdfAAoFSP1qO1AIgAbAbdAFw)。
