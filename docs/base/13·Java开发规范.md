# 13·Java开发规范

- [13·Java开发规范](#13java开发规范)
  - [函数命名](#函数命名)
    - [命名参考](#命名参考)
  - [POJO使用方式](#pojo使用方式)

## 函数命名
优雅的命名函数：
1. 函数命名不应过于追求简单，而是尽可能清晰表达出函数的作用。
2. 高质量的函数命名是可以让函数体本身或是被调用时无需任何注释的。
3. 函数命名应该偏重于函数的功能而非执行的过程。

### 命名参考
动词选取要精准，动词决定了函数的具体动作，而名词决定了函数具体的操作对象。
对于名词，尽量使用领域词汇，不要使用生僻或者大家很少使用的词语。

常用动词表：
| 类别 | 单词 |
| --- | --- |
| 添加/插入/创建/初始化/加载 | add、append、insert、create、initialize、load |
| 删除/销毁 | delete、remove、destroy、drop |
| 打开/开始/启动 | open、start |
| 关闭/停止 | close、stop |
| 获取/读取/查找/查询 | get、fetch、acquire、read、search、find、query |
| 设置/重置/放入/写入/释放/刷新 | set、reset、put、write、release、refresh |
| 发送/推送 | send、push |
| 接收/拉取 | receive、pull |
| 提交/撤销/取消 | submit、cancel |
| 收集/采集/选取/选择 | collect、pick、select |
| 提取/解析 | sub、extract、parse |
| 编码/解码 | encode、decode |
| 填充/打包/压缩 | fill、pack、compress |
| 清空/拆包/解压 | flush、clear、unpack、decompress |
| 增加/减少 | increase、decrease、reduce |
| 分隔/拼接 | split、join、concat |
| 过滤/校验/检测 | filter、valid、check |

## POJO使用方式
![图15-POJO使用图解](/docs/images/图15-POJO使用图解.png)