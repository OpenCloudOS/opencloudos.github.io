# 提取用例

## **1. 概述**

用例git仓库根目录下需要有【**extract_case.py**】文件，平台通过执行【**python extract_case.py**】提取用例，支持**python2**和**python3**。通过生成tcase.json文件，将用例信息导入到TCase平台。

## **2. 提取用例信息**

通过解析注释中的用例信息，详情请参考[示例](https://gitee.com/OpenCloudOS/packages-testing/blob/master/extract_case.py#L137)

```py
test_case():
    """
    '用例名': '', # 必填，不能为空，普通用例填写用例名，模版用例、测试套则填写相应目录名称
    '用例/目录类型': 0, # 必填，不能为空，0为普通用例，1为模版用例，2为测试套
    '测试场景': '', # 如果无，填''
    '测试步骤': '', # 如果无，填''
    '预期结果': '', # 如果无，填''
    '备注': 'git_test1', # 如果无，填''
    '用例标签': "{\"协议": [\"abc\"]}", # 如果无，填''，支持v、kv两种形式： 1）多个v：value1,value2,value3等 多个标签使用;,、，符号中的一个分隔 2)kv形式，key表示分类: "{\"key1": [\"value1\",\"value2\"]}" 
    '用例目录': 'git_test1', # 上级目录，普通用例、模版用例上级目录不能为空，测试套上层目录可以为空，example: A/B/C
    '用例级别': 0, # 必填，0 or 1 or 2 or 3
    '代码git路径': 'git_test1', # 必填
    '执行命令': 'git_test1', # 必填，example: python -m auto_test.projects.tgw.testcases.tgw_cases.hb.test_2048_rs_udp
    '关联tapd需求单': 'git_test1', # 如果无，填''
    '创建人': 'git_test1', # 必填，英文名
    '修改人': 'git_test1' # 如果无，填''
    '超时时间': 1, # 单位：秒
    '是否自动化': 1 # 0为否，1为是，不填则为是
	'自定义用例ID': 'asdlfk', # 必须为项目内唯一，定义‘自定义用例ID’后，则以ID作为用例唯一标识，可以修改‘用例名’
	'用例排序数值': 0, # 从小到大排序，不指定的用例默认数值99999
    '测试套setup命令': '', # example: python -m auto_test.projects.tgw.testcases.tgw_cases.hb.test_2048_rs_udp
    '测试套teardown命令': '' # example: python -m auto_test.projects.tgw.testcases.tgw_cases.hb.test_2048_rs_udp
    """
    pass
```
## **3. json示例**

参考[OpenCloudOS/packages-testing示例](https://gitee.com/OpenCloudOS/packages-testing/blob/master/extract_case.py)生成用例文件[tcase.json]，其中key如下所示。

```json
[ 
    {
            'name': 'case1-testsuite-realcase334',  # 不能为空
            'is_template_case': 0,  # 必填，不能为空，0为普通用例，1为模版用例，2为测试套
            'test_scene': '测试场景1',  # 如果无，填''
            'test_step': '上传文件',  # 如果无，填''
            'expect_result': '上传成功',  # 如果无，填''
            'note': '正常场景',  # 如果无，填''
            'signs': '{"协议1": ["https", "http", "http"]}',  # 如果无，填''，支持v、kv两种形式： 1）多个v：'value1,value2,value3' 多个标签使用;,、，符号中的一个分隔 2)kv形式，key表示分类: '{"协议1": ["https", "http", "http"]}'
            'directory': 'master/上传/testsuite/case1-testsuite',  # 不能为空，example: A/B/C
            'level': 0,  # 不能为空，取值范围:[0,1,2,3]
            'git_path': '',
            'run_cmd': 'python -m auto_test.projects.tgw.testcases.tgw_cases.hb.test_2048_rs_udp',  # 不能为空。
            'tapd_url': '',  # 如果无，填''
            'create_person': 'conniezheng',  # 不能为空，英文名
            'update_person': 'conniezheng',  # 如果无，填''
            'timeout': '',  # 超时时间，单位：秒
            'auto': 1,  # 是否自动化，0为否，1为是，不填则为是
            'u_id': 'case1_testsuite-realcase_asdlfk',  # 必须为项目内唯一，定义‘自定义用例ID’后，则以ID作为用例唯一标识，可以修改‘用例名’
            'series': '',  # 从小到大排序，不指定的用例默认数值99999
            'setup_cmd': '',  # 测试套setup命令, example: python -m auto_test.projects.tgw.testcases.tgw_cases.hb.test_2048_rs_udp
            'teardown_cmd': '',  # 测试套teardown命令, example: python -m auto_test.projects.tgw.testcases.tgw_cases.hb.test_2048_rs_udp
    }
]
```
