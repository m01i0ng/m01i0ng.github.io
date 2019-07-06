---
title: Elasticsearch 学习笔记
date: 2019-07-06 22:57:21
tags:
  - es
  - ElasticSearch
categories: es
---

## 概念

### 文档

| 名词 | 解释 |
| --- | --- |
| _index | 所属索引名 |
| _type | 所属类型名 |
| _id | 唯一 id |
| _source | 原始 json 数据|
| _all | 整合所有字段内容到该字段，最新版本已被废除 |
| _version | 版本信息 |
| _score | 相关性打分 |

### 索引

定义，用于定义包含文档的字段名和字段类型

| 名词 | 解释 |
| --- | --- |
| Index | 相似文档的集合，每个索引都有自己的 Mapping | 
| Shard | 体现了物理空间的概念，索引中的数据分布在 Shard 上 |
| Mapping | 定义文档字段类型 |
| Setting | 定义不同的数据分布，多少分片等等 |

相关概念与关系型数据库类比：

| 关系型 | ES |
| --- | --- |
| Table | Index |
| Row | Document |
| Column | Field |
| Schema | Mapping |
| SQL | DSL |

## 集群

### 高可用

- 服务可用性：允许有节点停止服务
- 数据可用性：部分节点丢失，不会影响数据

### 可扩展性

- 请求量提升、数据不断增长，将数据分布到所有节点上


### 节点

- 一个节点即是一个 ES 实例，生产环境建议单机一个实例
- 没一个节点都有名字，通过配置文件配置，或者启动时 -E node.name=node1 指定
- 每个节点启动之后会分配至一个 UID
- 生产环境中建议节点单一职责

#### Master-eligible 和 Master

- 每个节点启动后即是 Master-eligible 节点
- 可参加 Master 选主流程成为 Master
- 当第一个节点启动时会将自己选举成 Master
- 每个节点上都保存了集群状态，只有 Master 才能修改集群信息

#### Data 节点

可以保存数据的节点，负责保存分片数据

#### Coordinating 节点

- 负责接收客户端请求，将请求发送到合适节点，最后汇总结果
- 每个节点默认都起到了该职责

#### Hot & Warm 节点

冷热分离，热配置高，冷配置低

#### ML 节点

负责跑机器学习

#### Tribe 节点

### 分片

#### 主分片

- 用以解决数据水平扩展问题，将数据分布到集群内所有节点之上
- 一个分片是一个运行的 Lucene 实例
- 主分片数再索引创建时指定，后续不允许修改，除非 ReIndex

#### 副本

- 用以解决数据高可用问题，分片是主分片的拷贝
- 副本分片数可动态调整
- 增加副本数还可在一定程度上提高服务的可用性

#### 分片数设定

- 过小
    - 导致后续无法通过增加节点实现水平扩展
    - 单个分片数据量太大，导致数据重新分配耗时
- 过大
    - 7.0 开始默认主分片设置为 1 
    - 影响搜索结果相关性打分，影响统计结果的准确性
    - 单个节点上过多的分片会导致资源浪费，同时也会影响性能

## 倒排索引

### 正排索引

文档 ID 到文档内容和单词的关系

### 倒排索引

单词到文档 ID 的关系

组成：

- 单词词典
- 倒排列表

### 分词

#### 分词器

处理分词的组件，由三部分组成：

- CHaracter Fileter：针对原始文本处理，例如去除 html
- Tokenizer：按照规则切分为单词
- Token Filetr：将切分得单词进行加工，小写、删除 Stopwords、增加同义词等等

unicode 优化分词：icu_analyzer

中文分词：

- ik https://github.com/medcl/elasticsearch-analysis-ik
- thulac

## Api

### 搜索

#### URI 搜索

`GET /索引/_search?q=2010&df=title&sort=year:desc&from=0&size=10&timeout=1s`

参数：

| 参数 | 作用 |
| --- | --- |
| q | 查询语句 |
| df | 指定默认字段，不指定时会对所有字段进行查询 |
| sort | 排序 |
| from、size | 分页 |
| profile=true | 查看查询执行细节 |

q：

- Term、Phrase 
    - Hello World 等效 Hello OR World
    - "Hello World" 等效 Hello AND World
- 布尔运算符 AND/OR/NOT 等
    - "+" MUST 
    - "-" MUST NOT
- 范围 [] 闭区间 {} 开区间
    - year:{2019 TO 2018}
    - year:[* TO 2018]
- 算数符号
    - year:>2010
    - year:(>2012 && <=2018)
    - year:(+>2012 +<=2018)
- 通配符
    - title:mi?d
    - title:be*
- 正则
    - title:[bt]oy
- 模糊、近似
    - title:beauriful~1
    - title:"lord rings"~2

#### Request Body 搜索

`GET /索引/_search`

将上述参数全部放在 `request body` 中，`q` 换为 `query`

额外参数：

| 参数 | 作用 |
| --- | --- |
| _source | 指定获取字段数组 |
| 脚本字段 | 通过函数或其他处理生成的新字段，类似 `sql` 中的 `select x as y` |

query 参数支持的搜索方式：

- match
- query string
- simple query string

### Mapping

> https://www.elastic.co/guide/en/elasticsearch/reference/7.1/dynamic-mapping.html

概念类似数据库表的 `schema`，描述文档的字段结构

#### 数据类型

- 简单类型：
    - Text、Keyword
    - Date
    - Integer、Floating
    - Boolean
    - IP4、6
- 负责类型：
    - 对象、嵌套
- 特殊类型：
    - geo_point、geo_shape

#### Dynamic Mapping

插入文档如果索引不存在会自动创建索引，类型可以自动识别，但识别可能不准

Mapping 更新的规则：

- 新增加字段：
    - Dynamic 设为 `true` 时，一旦有新增字段的文档写入，Mapping 同时被更新
    - Dynamic 设为 `false`，Mapping 不会被更新，新增字段无法被索引，但是信息会出现在 `_source` 中
    - Dynamic 设为 `strict`，文档写入失败

![pic](https://gitee.com/alley9469/pic/raw/master/content/Mixmyi.png)

- 对已有字段：
    - 一旦已经有数据写入，就不再支持修改字段定义
    - 如果想改变字段类型必须 Reindex 重建索引
