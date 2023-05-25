# OpenCloudOS文档库

欢迎来到OpenCloudOS文档库！

OpenCloudOS 操作系统开源社区是由操作系统、软硬件厂商与个人共同倡议发起的操作系统社区项目，为用户提供自主可控、绿色节能、安全可靠、高性能的下一代云原生操作系统，目前社区理事单位已达 31家，联接生态伙伴达 500+家，OpenCloudOS 操作系统将与众多生态伙伴一起打造面向未来开放中立的操作系统开源生态。

截止目前 OpenCloudOS 操作系统已支持 X86_64、ARM64、RISC-V 架构，完善适配 LoongArch、飞腾、海光、兆芯、鲲鹏等芯片。同时提供支持全栈国密和机密计算，下载量和装机量已达千万节点，另有 600 余家企业产品与 OpenCloudOS 操作系统完成适配。

欢迎访问：[https://www.opencloudos.org/](https://www.opencloudos.org/)

## 如何贡献

欢迎贡献OpenCloudOS！

### 贡献OpenCloudOS

### 贡献文档

1. Fork [文档仓库](https://github.com/OpenCloudOS/opencloudos.github.io/fork)
2. 将您Fork后的文档仓库clone至本地

    ```sh
    git clone git@github.com:yourname/opencloudos.github.io.git # (1)
    ```

    （你需要将n`yourname` 更换为你自己的 GitHub 用户名）

3. 安装环境
    - 安装Python 3.x
    - 安装[mkdocs-material](https://squidfunk.github.io/mkdocs-material/)及多语言插件

    ``` sh
    pip install mkdocs-material mkdocs-static-i18n
    ```

    - 在本地运行预览服务器

    ``` sh
    mkdocs serve
    ```

4. 可以开始贡献啦！
    - 具体markdown及本文档站支持的显示特性可查看mkdocs-material的[说明文档](https://squidfunk.github.io/mkdocs-material/reference/)。
    - 请注意遵守本文档站的 [格式手册](https://github.com/OpenCloudOS/opencloudos.github.io/blob/main/docs/contribution/docs-format-guide.md)。

5. 向 [文档仓库](https://github.com/OpenCloudOS/opencloudos.github.io) 提交 Pull Request，待维护者审核后即可合并。
