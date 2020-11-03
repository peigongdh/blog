在从事网络应用的开发过程中会经常遇到一些问题，有些问题是涉及到所依赖的软件或框架的实现，这类问题我们可以通过阅读该软件或框架的说明文档来解决问题；有些则是涉及到网络通信中使用的协议，那我们应该找到该协议的说明文档，即该协议定制的原始出处，这篇文章记录了我探寻这一过程的历程。

这个过程起源于我在wiki上查询http-header字段时发现的一个彩蛋：
<blockquote><span class="reference-text">“引导者”（“referrer”）这个单词，在RFC中被拼错了，因此在大部分的软件实现中也拼错了，以至于，错误的拼法成为了标准的用法，还被当成了正确的术语。</span></blockquote>
检验referrer头部可以用于防止CSRF(跨站请求伪造)，我曾经困惑于它的拼写但未深究，原来它是RFC文档中的一个错误拼写，那么这里的RFC是否就是HTTP协议的标准文档？带着这个问题我查询了RFC的相关信息。
<blockquote><b>征求意见稿</b>（<span class="LangWithName">英语：<span lang="en" xml:lang="en">Request For Comments</span></span>，缩写为RFC），是由互联网工程任务组（IETF）发布的一系列备忘录。文件收集了有关互联网相关信息，以及UNIX和互联网社区的软件文件，以编号排定。目前RFC文件是由互联网协会（ISOC）赞助发行。

RFC始于1969年，由斯蒂芬·克罗克用来记录有关ARPANET开发的非正式文档，最终演变为用来记录互联网规范、协议、过程等的标准文件。基本的互联网通信协议都有在RFC文件内详细说明。RFC文件还额外加入许多的论题在标准内，例如对于互联网新开发的协议及发展中所有的记录。

例如常见互联网协议的RFC编号：IP：791, TCP：793, HTTP1.1：2616</blockquote>
可以看到RFC(征求意见稿，Request for Comments)制定了我们常用的一些互联网协议，RFC是由IETF<strong>（互联网工程任务组，Internet Engineering Task Force）</strong>发布的，那么这个IETF又是谁？进一步查询可知它是目前TCP/IP协议族的制定者。
<blockquote>The <b>Internet Engineering Task Force</b> (<b>IETF</b>) develops and promotes voluntary Internet standards, in particular the standards that comprise the Internet protocol suite (TCP/IP). ...

The IETF started out as an activity supported by the U.S. federal government, but since 1993 it has operated as a standards development function under the auspices of the Internet Society, an international membership-based non-profit organization.</blockquote>
与它在同一个层面的组织有ISO<strong>（国际标准化组织，International Organization for Standardization）</strong>和ITU<strong>（国际电信联盟，International Telecommunication Union）</strong>等，回忆起学生时代计算机网络中提及的OSI模型，它被制定在ISO的标准文件中，定义于ISO/IEC 7498-1，这是一个理论上的模型，也是RFC的所参考的对象。

为什么我们现在使用的是RFC制定的TCP/IP协议族而不是OSI模型，这一点要结合当时的时代背景来看，在1980年代早期，个人计算机产业快速发展，客户和服务商之间对通讯技术的要求也急剧增加，这使得他们急于开发并使用一些新的通信技术规范，即使这种规范是并未被标准化的。因此，标准制定机构必须加快标准制定的进程来适应这一需求，否则他们将不得不面对并承认一些事实标准。不幸的是，这时候像ISO和CCITT（ITU前身）这样的机构并不足以应付这样的改变。所以公众更倾向于从另外一类能更快的对公众的要求作出反馈的标准制定组织来获取标准，这样的组织包括非正式的、非政府的组织，此时像IETF和W3C<strong>（万维网联盟，World Wide Web Consortium）</strong>这样的组织便诞生了。其中W3C也定制了大量影响深远的协议，例如CSS，XML，HTML等。

那么IETF定制的RFC标准除了比别的标准化组织快，还有什么其他的原因使得它流行起来？主要还是得益于当时的历史进程，例如美国国防部的支持；IBM，AT&T等大厂的支持；AT&T所研发的UNIX中的使用了符合TCP/IP标准的代码。

需要注意的一点是，我们普遍提及的概念TCP/IP全称是Internet protocol suite，它是一个与硬件无关的协议，所以它的最底层组件是链接（Link）而不是OSI模型中的数据链路层（Data link）或物理层（Physical），也就是说TCP/IP可以在几乎任何硬件网络技术的基础上实现。我们经常混淆TCP/IP4层模型中的链接为OSI模型中的数据链路层，这曾经也很让我困惑，既然TCP/IP与硬件无关，为何要参与设计数据链路层。我们做出了进一步的查询。
<blockquote>The link is treated as a black box. The IETF explicitly does not intend to discuss transmission systems, but practical alternative to the OSI model.</blockquote>
TCP/IP中的链接被视为黑盒，在现实中，它实际对应的是OSI模型中的物理层或数据链路层，或两者都包括。

我们熟悉的数据链路层协议是由IEEE<strong>（电气电子工程师学会，Institute of Electrical and Electronics Engineers）</strong>制定的LLC（逻辑链路控制，Logical Link Control），它被制定在IEEE 802.2标准中。这个在电气和通信领域如雷贯耳的组织，它也参与定制了计算机网络协议中的一部分，主要是在数据链路层和物理层，例如我们使用最为广泛的，基于铜缆双绞线或光纤的以太网（IEEE 802.3），Wi-Fi（IEEE 802.11 ）等。同时IEEE在计算机硬件也制定了很多影响广泛的标准，例如POSIX（IEEE Std 1003）。

另一个例子是我们生活中常用的网线即超5类双绞线（CAT-5e），它使用的是TIA/EIA-568-B标准，而它的组织也是赫赫有名的ANSI<strong>（美国国家标准学会，American National Standards Institute）</strong>。有趣的是在最新的CAT-7使用的是ISO的标准（ISO/IEC 11801 ）。

到这里，我们没有必要去继续探究这些标准制定的组织和协会，只需要在使用协议的时候再去查询到它们，可以推测，他们之间并不是愉快的合作关系而是激烈的竞争关系，标准的制定权代表的是组织背后的行业（企业）的整体实力，能拿到这个话语权无疑是站在了食物链的顶端，一个新鲜的例子是华为在2017年拿下了5G（第五代移动通信系统）的标准制定权。

回到我们的出发点，我们可以得出一个结论，即我们目前广泛使用的TCP/IP协议族是由IETF所制定的标准，作为网络程序的开发者，我们所集中关注的应用层和传输层的协议有很多都是他们所制定的，他们出版的RFC文档即现实意义上的标准文档。

他们的标准文件可以从官方网站上免费获得<a href="https://tools.ietf.org/rfc/index">https://tools.ietf.org/rfc/index</a>。