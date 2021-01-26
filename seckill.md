## 秒杀系统

> https://www.jiuzhang.com/course/77/dialog/#chapter-612_1

### 表结构设计

商品表 commodity_info
活动表 seckill_info
库存表（使用单独字段lock表示下单完毕的订单数）stock_info
订单表 order_info

商家：
select commodity_info
insert seckill_info
insert stock_info

用户：
select seckill_info & commodity_info & stock_info
insert order_info
update stock_info

扣减库存：
START TRANSACTION;

SELECT stock FROM `stock_info`
WHERE commodity_id = xx AND sexkill_id = yy FOR UPDATE;

UPDATE `stock_info` SET stock = stock -1
WHERE commodity_id = xx AND seckill_id = yy AND stock > 0;

TRANSACTION COMMIT;

使用事务解决订单表，update操作（for update）
使用stack > 0的where条件解决库存超卖的问题

### redis设计

使用redis缓存库存数
使用lua脚本解决cas（check and set）问题，即原子操作库存减少
如果此时达到mysql的压力仍然很大，考虑使用消息队列削峰

### 消息队列使用

如果消息队列投递失败，如何解决？
一个可行的解决方案：
Redis中的库存比实际库存大一些，比如1.5倍到2倍
在下单过程中double check库存数

### 库存扣减时机

下单后锁定库存，支付成功后，减库存

### 如何限购

使用Redis数据校验
使用Redis提供的集合数据结构，将扣减Redis库存的用户id写入
SADD KEY UID1
SISMEMBNER KEY UID1

Redis校验完毕后，再在Mysql中做订单uid&活动id的唯一校验

### 付款和减库存的数据一致性-分布式事务

三阶段提交，有超时机制
保证多个存在不同数据库的数据操作，要么同时成功，要么同时失败，主要用于强一致性的保证。

### Scale扩展

当库存扣减完毕后，可以直接拒绝后面的请求
前端资源静态化（CDN）
前端限流，点击一次后，按钮短时间置灰
部分请求直接跳转到繁忙页
未开始抢购时，禁用抢购按钮

如何计算倒计时？
前端轮训（Poll）服务器的时间，并获取时差

秒杀服务挂掉，怎么办？
尽量不要影响其他服务，尤其是非秒杀商品的正常购买
服务雪崩，服务A挂掉导致后面依赖的服务都不可用？
服务熔断，熔断机制是应对雪崩效应的一种微服务链路保护机制，快速返回错误响应
Netflix Hystrix
Alibaba Sentinel

防止恶意刷请求或者爬虫请求
验证码机制
限流机制 是否来自同一个ip地址，是否来自同一个用户id
黑名单机制 黑名单ip 黑名单用户id

结束时间：1.53
