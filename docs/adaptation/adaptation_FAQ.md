# OC 生态认证-FAQ

## 1-系统安装
Q1：在选择源的时候出现error setting up base repository    
A：制作安装镜像时建议：
使用U盘作为引导介质
- linux 上用dd 命令    

```bash
dd if=xxx.iso  of=/dev/xxx  bs=2M
```
(可以使用`fdisk -l`查看U盘路径)
- macos 或者windows 建议用 [https://www.balena.io/etcher/](https://www.balena.io/etcher/)

## 2-测试相关
Q1：测试工具编译时，`ts_common.c`缺少`common.h`头文件    
A：使用如下命令下载测试工具    

```bash
git clone --recurse-submodules https://gitee.com/opencloudos-stream/oc-hct.git
```
不要使用直接下载的方式。

Q2：在使用`yum install kernel-devel-$(uname -r)`安装kernel devel包时提示找不到对应内核devel包     
例如：    
![image](../assets/install_devel_error.png)      
A：更换命令为  

```bash
# TS3
yum install kernel-devel-$(uname -r).tl3

# OC8
yum install kernel-devel-$(uname -r).oc8
```
