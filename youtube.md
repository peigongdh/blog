## design youtube

> https://www.bilibili.com/video/BV1s5411p7dR

1. clarify the requirements
2. capacity estimation（容量估计）
3. system apis
4. high-level system design
5. data storage
6. scalability

### clarify the requirements

- upload
- view
- share
- like/dislike
- comment
- search
- recommend

#### Non-Functional requirement

consistency
- 每次请求都可以获取最新的feed或者错误
- sacrifice（牺牲）达到最终一致性
availability
- 每次请求都有响应，尽量少的错误
- scalable：性能：低延迟
partition tolerance
- 少部分机器或者网络down掉不影响整体服务

### capacity estimation

assumptions:
- 2 billion total users, 150 million daily active user
- each content creator: 1 video pre week, 1% among all users are content creators
- each content consumer: avg watching time: 50 mins
- each video: avg length: 10 mins; avg upload resolution 1080p; bandwidth requirement for 1080p playback: 5Mbps

storage estimation:
new videos:
2B * 1% * 1 video / 7 days == 33 videos / s
file size per minute:
1080p + ... = 100MB/min
daily write size:
33 videos / s * 86400 s /day * 10min * 100MB / min = 2.7PB / day
replication:
redundancy: same-region replication * 3
availabliity: cross-regin replication * 3

bandwidth esimation:
daily updaload bandwidth:
- 33 videos/s * (10*60s/video) * 5Mps = 99Gbps
daily download/ongoing bandwidth:
- concurrent users: 150M DAY * 50 mins / (24 * 60mins /day) = 5.2M users
- bandwidth: 5.2M * 5Mbps = 26Tbps
- read / write ratio: 26 Tbps / 99Gbps = 263:1 

根据上行下行带宽比值得出，此类视频网站是读多写少的平台

### system apis

- uploadVideo
- streamVideo(videoId, offSet, codec, resolution) offSet可以约定从什么时候开始播放

### hight-level system design

#### uploadVideo

user -> load balancer -> upload service -> processing queue -> video processing service
                                        -> distributed media storeage
                                        -> database (metadata user)
video processing service (async) -> completion queue -> video distributing service -> CDN

video processing service
- breakdown video into chunks
- transcoding(multiple codec & resolutions) decode&encode
- generate thumbnails and previews
- video understanding ML

CDN
热门视频process给用户，冷门视频直接由源传给用户

#### watchVideo

user -> load balancer -> video playback service -> distributed media storage 
                                                -> database (metadata/user)
                      -> host identify service -> CDN

### data storage

user table
video metadata table
comment table

- sql: relational database for user table, video metadata
- nosql: store unstructured data for thumbnails in big table
- file system / blob storage: video
  file system: HDFS / GlusterFS
  blob storage: netflix used amazon S3

### scalability

bottlenecks & solutions

read heavy system
- distribute/replicate data across servers/regions
- data sharding
- data replication

latency
- caching
- CDN

optimization 1: data sharding
- sharding by video_id via consistent hashing
- hot video can make shards experience very traffic: solution: replicate the hot videos to more servers

optimization 2: data replication
- primary-secondary configuration: write to primary and then propagate to all secondaries
- read from secondary

optimization 3: caching
- CDN
- server <-> database
- how many to cache: cache 20% of daily read videos
- how to scale: consistent hashing

optimization 4: CDN
- predict locations where people would prefer to watch a video