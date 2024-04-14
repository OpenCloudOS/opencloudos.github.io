
## **1. 概述**

TCase平台上对自动化的用例发起执行，可以识别用例执行的结果。要将自动化用例接入TCase做执行，用例需要符合平台的标准。  
目前平台支持三种模式的用例：<font color="#0000cb">**pytest模式、类gtest模式和nose模式**</font>。  
不同模式的用例运行方式一致，都是由平台拉起一个新的进程运行；不同模式的主要区别在于结果解析不一致。

## **2. 用例执行结果**

用例执行结果分为成功、跳过、失败、异常、超时和终止六种。

> - **成功**：用例执行成功，所有用例逻辑中的检查点全部检查通过
- **跳过**：用例标记为跳过，未实际执行。
- **失败**：用例执行过程中，因有检查点未通过（如实际值和预期值不符）
- **异常**：用例执行过程中发生异常而中止，没有正常结束
- **超时**：用例执行超时
- **终止**：用例执行过程中被终止

## 3. pytest模式

需要Python3环境，并下载pytest依赖。详细可以查看[pytest参考文档](https://docs.pytest.org/en/7.3.x/getting-started.html)

### 3.1 简单示例

pytest用例函数以test开头即可识别，采用【python3 -m pytest example.py】调用

```python
# example.py

# pytest用例函数以test开头即可识别，python3 -m pytest example.py
def test_example():
    assert 1 == 1
```

### 3.2 用例结果判断

上面示例输出如下所示，用例结果为passed即为【成功】
```
============================================================== test session starts ===============================================================
platform linux -- Python 3.8.2, pytest-7.1.2, pluggy-1.0.0
rootdir: /data/tcase/test, configfile: pytest.ini
plugins: anyio-3.6.1
collected 1 item                                                                                                                                 

example.py::test_example PASSED                                                                                                            [100%]

================================================================ warnings summary ================================================================
../../../usr/local/lib/python3.8/site-packages/_pytest/config/__init__.py:1252
  /usr/local/lib/python3.8/site-packages/_pytest/config/__init__.py:1252: PytestConfigWarning: Unknown config option: thread_id
  
    self._warn_or_fail_if_strict(f"Unknown config option: {key}\n")

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
========================================================== 1 passed, 1 warning in 0.01s ==========================================================
```

## **4. 类gtest模式**

天然适用于c++测试框架gtest编写的用例。类gtest模式下测试用例编写的形式没有限制，只要是命令行可以执行的程序，有日志输出结果通过终端输出即可。  
平台对于用例结果的判断，完全基于用例程序执行后的输出的日志内容。也就是说，无论用例是shell脚本还是可执行程序，只要输出的文本满足以下要求，就可以被平台识别为合法用例

### 4.1 用例结果判断

> - 1、如果用例结束时<font color=red>没有输出</font>"**Global test environment tear-down**"，则认为用例是异常结束的，结果为【<font color=red>**异常**</font>】
- 2、如果用例结束时输出了"**[  FAILED  ]**"（采用英文方括号，方括号与FAILED前后都有2个空格），则认为用例失败，结果为【<font color=orange>**失败**</font>】
- 3、如果用例结束没有输出"[  FAILED  ]"，且输出了"**[  PASSED  ]**"（括号与FAILED、PASSED前后都有2个空格），则认为用例成功，结果为【<font color=#21e24f>**成功**</font>】
- 4、如果用例输出"**[  SKIPED  ]**"，则用例结果会被标记为【<font color=#21e24f>跳过</font>】
- 5、用例可以设置超时时间，当用例执行时间超过设置的时间时，用例结果会被标记为【<font color=#21e24f>超时</font>】


## **5. nose模式**

nose框架的使用可以查看[nose参考文档](https://nose.readthedocs.io/en/latest/)

在用例执行命令时需要指定with-xunit参数，并且指定xunit-file参数指定保存结果的路径。如：nosetests -s --with-xunit --xunit-file=TestAllocBlob_not_exist_space.xml --verbose case/allocator/test_alloc_blob.py:TestAllocBlob.not_exist_space --nologcapture


### 5.1 用例结果判断

>如下所示为nose用例的结果输出  
>如果errors为非零则用例会被标记为【<font color=orange>**失败**</font>】；  
>用例errors为零被标记为【<font color=#21e24f>**成功**</font>】。  
>没有产生xml文件或者无法解析到对应结果时用例被标记为【<font color=red>**异常**</font>】。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<testsuite name="nosetests" tests="1" errors="0" failures="0" skip="0">
<testcase classname="test.case.engine.test_put_shard_slice.TestPutShardSlice" name="test_delay_to_put_slice" time="69.420">
<system-out><![CDATA[10.160.131.87[]]]>
</system-out>
</testcase>
</testsuite>
```
