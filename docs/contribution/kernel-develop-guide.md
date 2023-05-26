# OpenCloudOS内核开发指南

> 本文档目的：
>
> - 保证内核代码开发和回合符合社区License要求。       
> - 保证Coding style 和代码质量达到内核Upstream Linus 库的质量要求。       
> - OC KERNEL 作为行业伙伴的上游内核， 将提供给下游伙伴一个代码稳定的（forward-only）内核树， 并且提供下游伙伴特性选择灵活性。        

## 1-分支策略

分支名和使用流程   

- master 代码组成： LTS + upstream backport features + SIG opensource features + bugfix。    
- 各特性基于 master 形成各个独立分支，在达到特性合入要求和测试后合入merge master。    
- master 分支会合入到腾讯内部代码树， 在其中充分测试验证后，特性合入lts/xx 稳定分支，开放给下游用户使用。    
- OC 内核的稳定分支在 lts/ 目录下， 仅接受不影响kabi稳定的bugfix。     
- master 和 稳定分支forward-only, 不得force push。 

## 2-内核默认配置

`make occonfig` 提供内核默认配置。   
`make noocconfig` 则在当前配置中删除OC特有的配置项，作为OC内核的一个build 测试用例。   

## 3-内核特性开发要求

为了在不增加开发同学的负担的同时更好的全流程跟踪(自研)内核特性研发，要求加入OC kernel的内核特性必须符合以下要求：

- 清楚描述特性要解决的问题；
- 解决该问题使用的设计方案；
- 特性的使用方法和实际效果；
- 该特性必须提供CONFIG_ 模式的编译开关；
- 提供cmdline 开关or sysctl runtime 开关 or 以模块方式提供代码 （可选）
- 特性对合入内核的KABI的影响；
- 特性通过了那些功能和性能测试。指定的测试要求见下文。
提交团队必须对提交代码的功能，性能和BUG负责。代码必须经过评审
- 特性提交负责人要 `signed-off`, 并且需要加签评审者 `Reviewed-by：`or `Acked-by: `
- 如果有测试则请加签测试结果， 例如：
``` 
Tested-by: Desheng <deshengwu@Tencent 腾讯>
```

## 4-Commit 规范要求

1. 遵循upstream/LTS commit 规范, 例如 commit log 必须有 `Signed-off-by`， 尽量通过scripts/check_patch.pl 检查。
2. 自研特性提交时commit log，要求包含 `Reviewed-by` or `Acked-by`。
3. 回合其他代码树的特性时，需要作为独立分支，并且：
    - 保持原upstream commit 数量和次序
    - 保留原commit author
    - 打上回合者的`Signed-off-by`: (建议使用 `git cherry-pick -s`)
    - 回合conflict的解决办法要注明在commit log:`Conflicts：<refactored some_function_foo ... >`
    - 在commit log 第一行标明其他代码树的commit ID，例如 
```text
commit 7548bf8c17d84607c106bd45d81834afd95a2edb upstream/openvz/openeuler
```
or 
```text
Upstream commit 8ebcc62c738f68688ee7c6fec2efe5bc6d3d7e60
```

## 5-Tag/版本 规范

目前在用TAG:      

- 4.14.105-19-0015
- 5.4.119-19-0009.1

## 6-特性测试要求

每个新的内核特性合入时要求通过以下测试, 并提供测试报告：    
1. 每个commit的 checkpatch.pl检查和各config 编译：   

- 新加入的内核config, 要在 kernel/configs/nooc.config  文件中关闭并编译通过；
- OC Kernel 作为上游内核代码，下游用户会有不同的，任意的config使用要求，在ARM 和X86中，除了默认的`default`, `noocconfig`, `occonfig` 必须build通过外， 还必须编译通过代码树中默认的 `defconfig`, `allyesconfig`, `allmodconfig`。
- **最小要求是不能增加build error/warning.**  一个自测方法如下：
``` bash
make allmodconfig modules all  -s ; make allyesconfig noocconfig all -s 
```

2. 各config下内核基准测试自测建议如下：
```bash
make occonfig kvmconfig bzImage -s -j 32 && qemu -kernel arch/x86/boot/bzImage --nographic -m 4g -smp 4 --enable-kvm  ...
```

3. 提供特性的测试用例，例如kunit/kselftest 或者其他用例
4. 通过基本内核测试    
    a. ltp 相关部分测试不增加failure。        
    b. stress-ng 测试–如果可以覆盖到。        
5. 执行测试时的gcov or kcov 代码覆盖率 > 90%。        
（gcov的数据收集方法 [在Linux内核里使用gcov做代码覆盖率检查](https://www.kernel.org/doc/html/latest/translations/zh_CN/dev-tools/gcov.html) ）
       
撰写人：alexsshi 2023-02-17        

参考：

1. [内核文档](https://www.kernel.org/doc/html/latest/translations/zh_CN/index.html#id6)      
2. [提交补丁：如何让你的改动进入内核](https://www.kernel.org/doc/html/latest/translations/zh_CN/process/submitting-patches.html)        

