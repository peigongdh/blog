## Design twitter

> https://www.bilibili.com/video/BV1Sf4y1e7wc

1. clarify the requirements
2. capacity estimation（容量估计）
3. system apis
4. high-level system design
5. data storage
6. scalability

### clarify the requirements

- post tweets
- timeline

####  Non-Functional requirement

consistency
- 每次请求都可以获取最新的feed或者错误
- sacrifice（牺牲）达到最终一致性
availability
- 每次请求都有响应，尽量少的错误
- scalable：性能：低延迟
partition tolerance
- 少部分机器或者网络down掉不影响整体服务

### capacity estimation

- 200million DAU, 100million new tweets
- each user visit home timeline 5 times; other user timeline 3 times
- each timeline/page has 20 tweets
- each tweet has size 280 bytes, metadata 30 bytes（20% tweets hava images, 10% tweets have video, 30% videos will be watched）

storage estimate:
write size daily:
text: 100M(280 + 30) = 31GB/day
image: 100M*20%*200kb = 4TB/day
video: 100M*10%*2M = 20TB/day

bandwidth estimate
200M*(5 home visit+3 uer visit) * 20 tweets/page = 32B
text: 32B * 280 bytes / 86400 = 100MB/s
image: 32B * 20% * 200 / 86400 = 14GB/s
video: 32B * 10% * 30% * 2MB / 86400 = 20GB/s

### system apis

post tweet
delete tweet
like or unlike tweet
read home timeline (pagesize, opt pageToken) pageToken 实现翻页
read user timeline (pagesize, opt pageToken) pageToken 实现翻页

user -> loadBalancer -> tweet writer (post tweet) -> DB/CACHE
     -> loadBalancer -> timeline service -> CACHE

 visit home timeline:
 tweet writer -> (fan out on write) -> get follower info -> update timeline cache of all followers

home timeline 
naive solution: pull mode
fetch tweets from N followers from DB, merge and return
pros: write is fast: O(1)
cons: read is slow: O(N) DB reads

better solution: push mode
maintain（维持） a feed list in cache from each user
fanout on write
pros: read is fast: O(1) from the feed list in cache
cons: write need more efforts: O(N) write for each new tweet
async tasks: delay is showing latest tweets(eventual consistency)

fan out on write (局限性)
- not efficient for users with huge amount of followers
hybrid solution
- not-hot users:
fan out on write(push)
do not fanout on non-active users
- hot users:
fan in on read(pull): read during timeline request from tweets cache

### data storage

user table
tweet table
follower table

sql database:
- user table
nosql database:
- timeline
file system
- media file

### scalability

identify potential bottlenecks
discussion solutions, focusing on tradeoff
- data sharding: datastore, cache
- load balancing: user <-> application server; applcation server <-> cache server; cache server <-> db
- data caching: read heavy

#### sharding

how: break large tables into smaller shards on multiple servers
pros: horizointal shards
cons: complexity(distributed query, resharding)

option 1: shard by tweets creation time
pros: limited shards to query
cons: hot/cold data issue; new shard fill up quickly

hot/cold data issue: 在tweets这种时效性比较高的场景下，大部分访问都是最近一天甚至一小时的数据，那么大量的压力集中在了最近的表上，资源分配不均，存在浪费，应该避免

option 2: shard by hash：store all the data of a user on a single shard
pros: simple; query user timelien is straightforward
cons: 
- home timeline still needs to query multiple shards
- non-uniform distribution of storeage: user data might not be able to fit into a single shard
- hot users
- availability

option 3: shard by hash(tweetId)

pros: uniform distribution（均匀分布）; high availability
cons: need to query all shard in order to generate user / home timeline

#### caching

timeline service:
- user timeline : user_id -> {tweet_id}
- home timeline : user_id -> {tweet_id}
- tweets: tweet_id -> tweet

tweets并不能支持翻页到最后一页的设计，目前通过前端限制了翻页方式，只能往下不停翻页

topics:
- caching policy: FIFO(First In First Out) LRU(Least Recently Used) LFU(Least Frequently Used)
- sharding
- performance