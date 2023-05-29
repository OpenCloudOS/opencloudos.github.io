# FAQ

Q1: What are the characteristics of OpenCloudOS and what scenarios are it suitable for?
A1: OpenCloudOS is an operating system community project jointly initiated by operating system, cloud platform, software and hardware vendors and individuals. At the beginning of its establishment, it decided to become a completely open and neutral open source community. The community will aim to build a comprehensive, neutral, open, secure, stable and easy-to-use, high-performance Linux server operating system, and work with member units to build a healthy and prosperous domestic operating system ecosystem. OpenCloudOS is applicable to physical machines, virtual machines, and docker sub-machines, and is suitable for various scenarios such as finance, Internet, government affairs, and desktop.

Q2: What are the self-developed features of OpenCloudOS?
A2: Please refer to  [this page](http://www.opencloudos.org/?p=537).

Q3: Are OpenCloudOS 8 and CentOS 8 compatible? How do I migrate from CentOS to OpenCloudOS?
A3: OpenCloudOS 8 and CentOS 8 maintain binary compatibility in user-mode.Refer to  [the migration guidelines in the official documentation](https://docs.opencloudos.org/guide/migrate/?h=%E8%BF%81%E7%A7%BB) for migration.

Q4: What operating system versions are supported by the migration tool?
A4: All Centos 8 versions except Centos Stream 8 are supported, with more versions in continuous updates.

Q5: Do I need to back up my data before migrating?
A5: Backup is required. Tencent cloud server users can refer to cloud hosts to [create snapshots](https://cloud.tencent.com/document/product/362/5755), while physical machines require manual backup.

Q6: Is physical machine migration supported?
A6: The migration tool supports the migration of both virtual and physical machines.

Q7: How do I resolve an exception during the migration process?
A7: Please submit a bug on [OpenCloudOS Community Bugzilla](https://bugs.opencloudos.tech/) and our engineers will answer it for you.

Q8: How is OpenCloudOS compatibility?
A8: We continuously update the software and hardware compatibility list within the community, and we have a standard community adaptation process, see  [Eco Certification Process](https://docs.opencloudos.org/adaptation/adaptation_process/)，Software compatibility can be found in [Commercial Software Compatibility List](https://docs.opencloudos.org/adaptation/adaptation_sw/) and [Open Source Software Compatibility List]](https://docs.opencloudos.org/adaptation/adaptation_oss/)，and hardware compatibility can be referred to [Hardware Compatibility List](https://docs.opencloudos.org/adaptation/adaptation_hw/) .

Q9: What should I pay attention to during the migration process?
A9: Please refer to the [Official Documentation](https://docs.opencloudos.org/guide/migrate/?h=%E8%BF%81%E7%A7%BB#_2) for the migration process precautions..

Q10: CentOS 8 has stopped updating at the end of 2021. Will OpenCloudOS be affected?
A10: The OpenCloudOS 8 usermode package is recompiled based on the RHEL 8 source package, and the kernel is custom-developed based on the community 5.4 lts kernel version, which is not affected by the CentOS discontinuation.
