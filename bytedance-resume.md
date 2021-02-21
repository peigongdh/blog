## system-design prepare

系统架构设计 System Design 2021 版中的剩余设计问题

- 优惠券系统设计
- - 发券接口限流保护

- 数据库扩展与一致哈希算法
- - 一致哈希算法的两个版本
- - 如何设计限流器
- - 如何设计实时数据系统

- 分布式文件系统GFS
- - master slave 的设计模式
- - 分布式文件系统的读写流程
- - 怎么处理分布式系统中的failure和recovery的问题
- - 如何做replica, check sum检查
- - 了解consistent hash和sharding的实际应用

- 文档协同编辑系统设计
- - websocket在协同编辑中的应用
- - 了解什么是协同编辑
- - 协同编辑的集中实现方案
- - 如何解决编辑冲突问题
- - 了解 OT（ Operational Transformation）原理

- 分布式数据库 Big Table
- - 通过设计分布式数据库系统Bigtable了解如下内容：
- - Big Table 的原理与实现
- - 了解NoSQL Database如何进行读写操作的,以及相应的优化
- - 了解如何建立index
- - 学习Bloom Filter的实现原理
- -  Master Slave 的设计模式

- 聊天系统 IM System
- - 聊天系统中的 Pull vs Push
- - 讲解一种特殊的 Service - Realtime Service
- - 用 channel 优化群聊
- - 如何限制多机登录
- - 用户在线状态的获取与查询 Online Status

- 视频流系统设计
- - 视频切分和断点续传如何实现
- - 如何在架构设计中节省带宽
- - 小文件存储之视频切片与缩略图存储
- - 了解视频预加载
- - 了解 CDN 的基本原理

- 基于地理位置的信息系统
- - 系统学习LBS相关系统设计的核心要点：
- - 地理位置信息存储与查询常用算法之 Geohash
- - 如何设计 Uber
- - 关键点：学会设计 Uber 以后可以轻松解决设计 Facebook Nearby 和 Yelp

- 分布式计算 Map Reduce
- - 学习Map Reduce 的应用与原理
- - 了解如何多台机器并行解决算法问题
- - 掌握Map和Reduce的原理
- - 通过三个题目掌握MapReduce算法实现：
- -  WordCount
- -  InvertedIndex
- - Anagram

- Twitter Search
- - 推特的海量推文数据如何存储
- - 如何快速搜索
- - 对搜索结果进行排名
- - 搜索系统容错能力

### master-slave主从模式

例如我们常用的读写分离，master写，slave读
Slave 与 Master 之间会有心跳数据包（一般数据库服务器会提供配置）。当 Master 有数据写入时 Master 会将数据同步至各 Slave 上。


