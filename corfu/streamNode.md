# StreamNode 修改计划

## 1. 怎么添加？

 初步计划是从```LogUnitServer```继承出两个类：```GlobalLogServer```和```StreamLogServer```分别处理向全局日志和流日志的访问请求，将可复用的部分留在基类中，子类分别实现各自的访问处理函数。

```CorfuServer```初始化时需要初始化两个```LogUnitServer```，分别作为全局日志和流日志。

## 2. 现有可处理的CorfuMsgType

以下是之前```LogUnitServe```处理的一些```CorfuMsgType```：

|       CorfuMsgType        |                       作用                        |             修改             |
| :-----------------------: | :-----------------------------------------------: | :--------------------------: |
|       TAIL_REQUEST        |             查询结尾：全局或者各个流              |                              |
| LOG_ADDRESS_SPACE_REQUEST | 查询全局结尾和所有流的地址空间，用于sequencer启动 |                              |
|     TRIM_MARK_REQUEST     |          查询这个server中日志的开始地址           |                              |
|           WRITE           |                      写日志                       |                              |
|        RANGE_WRITE        |                    写多个entry                    |                              |
|        PREFIX_TRIM        |                  用于checkpoint                   |                              |
|       READ_REQUEST        |       读请求，调用转发到底层streamLogFiles        | 修改streamLogFiles的read函数 |
|   MULTIPLE_READ_REQUEST   |                       同上                        |             同上             |
|   KNOWN_ADDRESS_REQUEST   |       获取一个地址区间中位于本server的地址        |                              |
|      COMPACT_REQUEST      |                                                   |                              |
|        FLUSH_CACHE        |                                                   |                              |



## 3. LogMetaData类已有的可用信息

```StreamLogFiles```中存在一个```LogMetaData```类的实例，

- 全局日志的tail，由该server写入的entry信息确定，有滞后性。
- 各个流的tail以及address space，更新同上，有滞后性。

## 4. StreamLogFiles修改

- initStreamLogDirectory: 好像不用改。
- verifyLog: 检查一下，好像不用改。
- initializeLogMetadata:遍历所有文件，遍历每个文件的所有entry，读entry， 更新logMetaData。
- 