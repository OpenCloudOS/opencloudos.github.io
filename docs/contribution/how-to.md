# 如何贡献

欢迎贡献 OpenCloudOS！

## 贡献 OpenCloudOS

## 贡献文档

1. Fork [文档仓库](https://github.com/OpenCloudOS/opencloudos.github.io/fork)
2. 将您 Fork 后的文档仓库 clone 至本地

    ```sh
    git clone git@github.com:yourname/opencloudos.github.io.git # (1)
    ```

    1.（你需要将 `yourname` 更换为你自己的 GitHub 用户名）

3. 安装环境
    - 安装 Python 3.x
    - 安装 [mkdocs-material](https://squidfunk.github.io/mkdocs-material/) 及多语言插件

    ``` sh
    pip install mkdocs-material mkdocs-static-i18n
    ```

    - 在本地运行预览服务器

    ``` sh
    mkdocs serve
    ```

4. 可以开始贡献啦！
    - 具体 markdown 及本文档站支持的显示特性可查看 mkdocs-material 的[说明文档](https://squidfunk.github.io/mkdocs-material/reference/)。
    - 请注意遵守本文档站的 [格式手册](docs-format-guide.md)。

5. 在本地通过预览服务器确认内容与格式正确后，commit 您的修改。
6. 向 [文档仓库](https://github.com/OpenCloudOS/opencloudos.github.io) 提交 Pull Request，待维护者审核后即可合并。
