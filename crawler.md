## 设计一个爬虫

将原先队列的方式优化为使用数据库的方式

表结构:
url
status
priority
available_time

### 解决问题

1. 慢查询，sharding数据库
2. 链接错误， available_time 设置翻倍的间隔时间
3. 如果网页频繁变化，我们把abailable_time 设置减倍的间隔时间
4. 巨型网页吃掉了大量的资源，设置分配的比例限制
