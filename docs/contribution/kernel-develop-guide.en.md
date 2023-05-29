# OpenCloudos Kernel Development Guide

> The purpose of this document is:
> 
> - Ensure that kernel code development and rounds comply with community license requirements.
> - Ensure that the coding style and code quality meet the quality requirements of the kernel Upstream Linus library.       
> - As the upstream kernel of the industry partner, OC KERNEL will provide the downstream partner with a forward-only kernel tree and provide the downstream partner feature selection flexibility.     

## 1-Branching Strategy

Branch names and usage processes 

- the code composition of master： LTS + upstream backport features + SIG opensource features + bugfix。    
- Each feature is formed into independent branches based on the master, and merged into the merge master after meeting the feature integration requirements and testing.    
- The master branch is merged into Tencent's internal code tree, where the features are merged into the LTS/XX stable branch and opened to downstream users.
- The stable branch of the OC kernel is in the lts/directory, and only accepts bugfix that does not affect Kabi stability. 
- Master and stable branch forward-only, force push is not allowed.

## 2-Kernel Default Configuration

`make occonfig` Provide kernel default configuration.   
`make noocconfig` The OC-specific configuration item is deleted in the current configuration as a build test case of the OC kernel.

## 3-Kernel Feature Development Requirements

In order to better track the whole process (self-developed) kernel feature development without increasing the burden on development students, the kernel features required to be added to the OC kernel must meet the following requirements :

- Clearly describe the problem that the feature is trying to solve ;

- The design solution used to solve this problem ;

- Usage and practical effects of features ;

- This feature must provide a CONFIG_ Compilation switch for mode ;

- Provide cmdline switch or sysctl runtime switch or provide code in a modular manner (optional) ;

- The impact of characteristics on KABI integrated into the kernel ;

- The features have passed those functional and performance tests. The specified testing requirements are listed below .

- The submission team must be responsible for the functionality, performance, and bugs of the submitted code. Code must be reviewed.

- The person in charge of feature submission needs to `signed-off`,  and the reviewer needs to add `Reviewed-by：`or `Acked-by: `

- If there are tests, please sign the test results, such as :
  
  ```
  Tested-by: Desheng <deshengwu@Tencent 腾讯>
  ```

## 4-Commit Specification Requirements

1. When submitting self-developed features(commit log)，require to include `Reviewed-by` or `Acked-by`。

2. When turning on the characteristics of other code trees, it is necessary to act as an independent branch and:
   
   - Maintain the original upstream commit quantity and order
   
   - Keep the original commit author
   
   - Add `Signed-off-by` of the person who took the turn: (recommended to use `git cherry-pick -s`)
   
   - The solution to the round conflict should be indicated in the commit log:`Conflicts：<refactored some_function_foo ... >`
   
   - On the first line of the commit log, indicate the commit IDs of other code trees, such as
     
     ```text
     commit 7548bf8c17d84607c106bd45d81834afd95a2edb upstream/openvz/openeuler
     ```
     
     or 
     
     ```text
     Upstream commit 8ebcc62c738f68688ee7c6fec2efe5bc6d3d7e60
     ```

## 5-Tag/Version Specification

Currently in use TAG:      

- 4.14.105-19-0015
- 5.4.119-19-0009.1

## 6-Characterization Testing Requirements

Each new kernel feature integration requires passing the following tests and providing a test report:   

1. The checkpatch.pl and each config compilation for each commit: 
   
   - The newly added kernel config needs to be closed and compiled in the kernel/configs/nooc.config file ；
   
   - As an upstream kernel code, downstream users may have different requirements for using OC Kernel.  In ARM and X86, in addition to the default `default`, `noocconfig`, `occonfig` must be built and passed, they must also compile and pass the default `defconfig`, `allyesconfig`, `allmodconfig`in the code tree.
   
   - **The minimum requirement is not to addbuild error/warning.** A self testing method is as follows:
     
     ```bash
     make allmodconfig modules all  -s ; make allyesconfig noocconfig all -s 
     ```

2. The recommendations for kernel benchmark self-testing under each config are as follows:
   
   ```bash
   make occonfig kvmconfig bzImage -s -j 32 && qemu -kernel arch/x86/boot/bzImage --nographic -m 4g -smp 4 --enable-kvm  ...
   ```

3. Provide test cases for features, such as kunit/kselftest or other use cases

4. Pass basic kernel testing
    a. LTP related testing does not increase failure.
    b. Stress-ng testing - if it can be covered.

5. Gcov or kcov code coverage>90% when executing tests.        
   （gcov's data collection method [Using gcov with the Linux kernel](https://www.kernel.org/doc/html/latest/dev-tools/gcov.html) ）

Written by:alexsshi 2023-02-17        

reference：

1. [Kernel Documentation](https://www.kernel.org/doc/html/latest/index.html#id6)      
2. [Submitting patches: the essential guide to getting your code into the kernel](https://www.kernel.org/doc/html/latest/process/submitting-patches.html)        
