## urlencode

> https://www.zhihu.com/question/19673368/answer/71537081

这个问题涉及到URL的定义。
我们知道URL是为了 统一的命名网络中的一个资源（URL不是单单为了HTTP协议而定义的，而是网络上的所有的协议都可以使用）。
所以这就要求URL有一些基本的特性：
URL是可移植的。（所有的网络协议都可以使用URL）
URL的完整性。（不能丢失数据，比如URL中包含二进制数据时，如何处理）
URL的可阅读性。（希望人能阅读）
因为一些历史的原因URL设计者使用US-ASCII字符集表示URL。（原因比如ASCII比较简单；所有的系统都支持ASCII）为了满足URL的以上特性，设计者就将转义序列移植了进去，来实现通过ASCII字符集的有限子集对任意字符或数据进行编码。
URL转义表示法包含一个百分号，后面跟上两个表示字符ASCII码的十六进制数值。现在URL转义表示法比较常用的有两个：

RFC 2396 - Uniform Resource Identifiers (URI): Generic Syntax
RFC 3986 - Uniform Resource Identifier (URI): Generic Syntax

## 对比base64与urlencode

对比urlencode和base64都希望能够具备一定的可读性，严格来说base64只是要求便于使用文本传输
区别在于，base64是二进制的映射，基本丢失了可读性，而urlencode的英文字符则不变，具备一定的可读性

## Base64URL

```go
const encodeStd = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
const encodeURL = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_"
const xxxxxxxxx = "tYsvKxjwABSD9-rhp_P7Ty6ZuWLamn2cOlJGg13oCVHQb5RIqief4N8UMkF0dEXz"
```

JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。
Base64 有三个字符+、/和=，在 URL 里面有特殊含义，所以要被替换掉：=被省略、+替换成-，/替换成_ 。这就是 Base64URL 算法。
