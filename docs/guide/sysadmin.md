# 系统管理

## Module 包 Gitee 仓库
- rpms component 常规 rpm包gitrepo
    - https://gitee.com/organizations/src-opencloudos-modules/projects
- Modulemd 编排gitrepo
    - https://gitee.com/organizations/src-opencloudos-rpms/projects

## MBS （Module Build Service）
- **MBS 服务**: 
    - https://mbs.opencloudos.tech/module-build-service/1/module-builds/

## 模块包构建流程
### 获取rpm 组件仓库上游更新
- 通过git.centos.org 的广播消息 或者 rhel的errata邮件通告【渠道】

  module stream的rpm组件有了上游更新，更新上游的rpm组件到OpenCloudOS的rpm 仓库。
- 提交rpm组件上游更新
    - `alt-src --push oc8-stream-xxx xxx.src.rpm`
- 更新指定 rpm 组件到modulemd仓库
- fork 仓库： https://gitee.com/src-opencloudos-modules/squid.git
    ```
    git clone https://gitee.com/userxxx/squid
    cd squid
    git branch -av
    git checkout branchName
    vim squid.yaml
    ## 修改 其中rpm 下面的ref 值为 上面提交到rpm仓库的commitid
    ## 保存退出
    ```
- Push and PullRequest
    ```
    git push orign oc8-stream-4
    ## 在Gitee发起PR
    ```
- 准备json格式的提交文件（MBS需要按照这个格式提交）
    ```
    # cat squid.json 
    {
        "scmurl": "https://gitee.com/src-opencloudos-modules/squid.git?#b724a9ac46944bdb277f5a17ad640991a8b3b7fe",
        "branch": "oc8-stream-4"
    }
    ```
- 发起 module 构建给MBS
    ```
    curl -i -X POST -H "Content-Type: application/json" --negotiate -u : -d @squid.json https://mbs.opencloudos.tech/module-build-service/1/module-builds/
    ```
-  发起 module 包构建流程

    curl 提交一个post 给mbs的module-builds接口， MBS 鉴权，check modulemd的格式，然后mbs自动发起构建请求到kojihub，kojihub和mbs交互完成后面的流程。

- 参考链接：https://docs.pagure.org/modularity/


欢迎贡献OpenCloudOS！
