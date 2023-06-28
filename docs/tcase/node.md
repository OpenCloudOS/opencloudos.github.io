## **1. 快速开始**

> - 下载脚本 wget -O tcase_agent_install.sh https://tcase.opencloudos.tech/static/tcase_agent_install.sh
- 启动agent命令 sh tcase_agent_install.sh --project xx --user xx


## **2. 参数说明**

参数 | 是否 | 含义 | 支持多值
 - | - | - | -
project	|   是  |	项目id  |	-
user	| 	是	| 	启动人 	| 	-
agent	| 	否	| 	agent key, 用来标记唯一的节点，默认值是机器ip	| 	-
agent_name 	|	否	| agent名 |	-
region 	|	否	| 环境分区 	|-
master_ip	|	否	| 母机ip	|	-
node_name	|	否	|	关联节点名，默认值为node1_master_ip	|	多个节点名称使用空格分割
node_path	|	否	|	关联节点执行路径，默认值为/data/test  |	多个节点路径使用空格分割，与节点名称个数对应
tag	        |	否	|关联节点标签，默认为空，示例: 1）key1:value2,value3;key2:value4,value5 | 多组kv值以;分割，每个k下的多个v以,分隔 示例 2) value1;value2;value3 不设置key，key默认为‘未分类’，多个value以；分隔多个节点标签信息使用空格分割，与节点名称个数对应


## **3. 完整agent启动命令**

>sh tcase_agent_install.sh --project xx --agent xx --agent_name xx --region DEVNET --master_ip x.x.x.x --node_name xx --node_path /data/test --user xx


