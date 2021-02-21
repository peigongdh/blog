## 设计typeahead

> https://www.jianshu.com/p/c7fc9092d9fe
> http://www.noteanddata.com/interview-problems-system-design-learning-notes-autocomplete.html
> https://www.youtube.com/watch?v=uIqvbYVBiCI&t=1s

比较粗暴的方式

实时log，记录所有单词的出现频率，然后用SQL抓取Top多少的词with a prefix

keyword   hit_count
"amazon"  20b

SELECT * FROM hit_stats
WHERE keyword LIKE '${key}%'
ORDER BY hit_count DESC
LIMIT 10

缺陷：like查询较慢，是一种range query

优化思路：使用Trie树
联想到在Gin中使用的前缀路由匹配

> https://geektutu.com/post/gee-day3.html

```go
type node struct {
	pattern  string // 待匹配路由，例如 /p/:lang
	part     string // 路由中的一部分，例如 :lang
	children []*node // 子节点，例如 [doc, tutorial, intro]
	isWild   bool // 是否精确匹配，part 含有 : 或 * 时为true
}
```

```java
class TrieNode {
    Map<Character, TrieNode> children;
    List topWords;
}
```

如何做sharding，使用一致哈希，这样机器增多时，还是会map的原来的key

如何reduce log file
按照1/10000等频率采样

时间和空间复杂度是多少？
trie多久更新一次？ 具体如何更新trie?
trie通常的实现有用数组的，有用hashmap的， 这里用哪种？
在leetcode里面我们经常使用数组， 但是这里用户搜索输入词范围肯定是unicode， 那一定非常稀疏， 所以必须要用hashmap
单机装不下一个完整的trie怎么办？搜索词的prefix分布很不均衡怎么办？
如何结合trending query?
当前发生热门突发事件， 用户的搜索日志还没有来得及处理这些搜索词， 但是需要加入结果，怎么办？
如何做用户个性化的结果？
存储trie的节点如果挂了怎么办？ 如何提高可用性？
整个trie是直接放在内存里面嘛？ 还是放在文件里面？ 还是放在其他类似redis或者memcache的系统里面？
online的service需要做cache嘛?