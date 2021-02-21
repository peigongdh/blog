## consistent hashing

> https://www.youtube.com/watch?v=lm6Zeo3tqK4

传统的哈希函数：
what is a hash function
it maps data of arbitrary size to fixed-size value

keys -> hashes

properties of a good hash function
- uniformity（均匀）
- deterministic(确定)
- efficiency（高效）

当我们在服务器上使用hash取余，当服务器水平扩展的时候
大部分的key都remapped

一致哈希：
a special kind of hashing
only k/n keys are remapped when resizing (k - keys count, n - bucket size)

basic technique:
- ring-based
- evenly place servers onto a ring
...

当新增一个节点时，只有部分相邻区域的key被remapped
删除一个节点时，下一个节点的负载可能存在翻倍

### 对比

operation   classic hash table   consistent hash table
add a node         O(k)                 O(K/N+logN)
remove a node      O(k)                 O(K/N+logN)
add a key          O(1)                 O(logN)
remove a key       O(1)                 O(logN)

通常N较小
