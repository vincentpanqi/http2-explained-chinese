# 8. http2的世界（未校对）

那么当http2被广泛采用的时候，世界会成怎么样呢？或者它会被广泛采用吗？

**8.1. http2会如何影响普通人？**

现在http2还没被大范围部署使用，我们也无法确定到底会发生什么变化，但至少可以参考SPDY的例子和曾经做过的实验来进行大概估计。

http2减少了网络往返传输的数量，并且用多路复用和快速丢弃不需要的流的办法来完全避免了head of line blocking的困扰。

它也支持大量并行流，所以即使网站的数据分发在各处也不是问题。

合理利用流的优先级，可以让客户端尽可能优先收到更重要的数据。

所有这些加起来，我认为页面载入时间和站点相应的速度都会更快。简而言之：更好的web体验。

但到底能变得多快，到底提升有多大呢？我认为目前很难说清楚。毕竟这些技术依然在早期，我们还没发看见客户端和服务器实现这些并真正受益于新协议所提供的强大功能。

**8.2. http2会如何影响web开发？**

近年来，web开发者、web开发环境为一些HTTP 1.1的问题提供了临时解决方案。不妨回忆一下，其中一些我已在上文中简单的介绍了。

这些解决方案很多是工具和开发者默认使用的，这很可能会损害到http2的性能，或者至少让我们没法真正利用到http2新的强大威力。Spriting和内联应该是http2里面最不需要的了。因为http2更倾向于使用更少的连接，所以Sharding甚至会伤害到http2的性能。

这里的问题就是：web网站和web开发们至少在短期内需要同时支持HTTP 1.1和http2的客户端。否则的话，使所有用户获得最好的体验将是一个挑战。

考虑到这些问题，我认为距离http2的潜力被彻底发掘还有很长一段路要走。

**8.3. http2的实现**

在这样一篇文章中详细说明每个实现细节注定乏味且毫无意义，我将用更广泛的术语解释实际场景，给大家提供一个http2的[实现列表](https://github.com/http2/http2-spec/wiki/Implementations)作为参考。

在http2的早期就已经有大量的实现。并且在http2标准化工作期间，这个数量还持续增长。截至我写这篇文档的时候，共有30种实现记录在案，他们中的大多数都实现了最新的草案。

Firefox一直紧跟最新的协议，Twitter也紧追不舍提供了基于http2的服务。2014年4月期间，Google在少数测试服务器上提供http2支持。从同年5月开始，开发版的Chrome支持http2。Microsoft也在他们的产品预发布会上展示了支持http2的下一代浏览器。

curl和libcurl支持未加密的http2，同时借助某些TLS库支持了TLS。

draft-17是当前最新的草案版本，它二进制兼容draft-14。第14版是最新一个被标示为实现/互操作的草案。<!-- 这段需要review -->

**8.3.1. 缺失的实现**

现有的实现列表中仍然有明星品牌缺席。目前尚未听到Apple官方有让Safari支持http2的计划。Apache HTTPD和Nginx这两大流行的服务器都提供SPDY的支持，但却没有对提供http2的支持进行任何表态。

Nginx表示“我们计划于2015年末发布带有HTTP/2支持的nginx和NGINX Plus”。而Apache已经有一个非常早期的HTTP/2模块，叫作[mod_h2](https://icing.github.io/mod_h2/)。

**8.4. 对http2的常见批评**

在制定协议的讨论过程中有许多充满争议的地方，甚至会有不少人认为这样的协议最终会以失败告终。这里我想提一些对协议常见的批评和我的解释：

**8.4.1. “这个协议是Google设计制定的”**

江湖上有太多传言暗示着这个世界越来越被Google所控制，但事实显然不是这样。这个协议是IETF制定的，就跟过去30年间很多其他协议一样。但不得不承认，SPDY是Google非常出色的成果。它不仅仅证明了开发一个新协议的可行性，还充分展现了新协议能带来的好处。

Google公开[声明](http://blog.chromium.org/2015/02/hello-http2-goodbye-spdy-http-is_9.html)了他们会在2016年移除Chrome里对SPDY和NPN的支持，并且极力推动服务器迁移至HTTP/2。

**8.4.2. “这个协议只在浏览器上有用”**

在某种程度上，这是对的。开发http2的一个主要动机就是修复HTTP pipelining。如果在你的应用场景里本来就不需要pipelining，那么确实很有可能http2对你没有太大帮助。虽然这并不是唯一的提升，但显然这是非常重要的一个。

一旦当某些服务意识到在一个连接上建立多路复用流的强大威力时，我认为会有越来越多的程序采用http2。

小规模的REST API和采用HTTP 1.x的简单程序可能不会认为迁移到http2能有多大的优势。但至少http2对绝大部分用户来讲，是几乎没有坏处的。

**8.4.3. “这个协议只对大型网站有用”**

完全不是这样。因为缺乏内容分发网络，小网站的网络延迟往往较高，而多路复用的能力可以极大的改善在高网络延迟下的体验。大型网站往往已经将内容分发到各处，所以速度其实更快。

**8.4.4. “TLS让速度变得更慢”**

这个评价在某种程度上是对的。虽然TLS的握手确实增加了额外的开销，但也有越来越多的方案来减少TLS往返的时间。使用TLS而不是纯文本带来的开销是显著的，有可观证据表明，和传输同样的流量相比，TLS会消耗更多的CPU和其他资源。具体影响有多大以及怎么影响是一个和具体测量有关的课题。更多的例子可以参看[istlsfastyet.com](http://istlsfastyet.com)。

Telecom和一些其他网络服务商，例如ATIS开放网络联盟，表示为了为卫星、飞机等提供的快速网络体验，他们需要一些[不加密的流量](http://www.atis.org/openweballiance/docs/OWAKickoffSlides051414.pdf )来提供caching，压缩和其他技术。

http2并不强制要求使用TLS，所以我们不应该为此担心。

如今，很多互联网使用者已经更希望TLS能被更广泛的使用来保护用户隐私。

实验也证明了通过使用TLS能比用在80端口实现一个新的基于文本的协议更容易成功。因为当前已经有太多中间商使用该方案，所以凡是基于80端口的协议，都很可能被理所当然的当作HTTP 1.1。

最后，得益于http2在单一连接上提供的多路复用流，普通浏览器的正常使用也可以减少TLS握手的次数，所以使用HTTPS仍然会比HTTP 1.1更快。

**8.4.5. “不基于ASCII是没法忍受的”**

是的，当我们可以直接“读”到协议的时候，调试和追踪都会变得更简单。但基于文本的协议更容易产生错误，造成更多解析的问题。

如果你真的无法接受二进制协议，那么你也无法在HTTP 1.x中处理TLS和压缩。而这些其实已经被使用了很久了。

**8.4.6. “它根本没有比HTTP/1.1快”**

当然，到底该如何定义和测量“快”就是另外一个话题了，但在SPDY的时代，已经有一些实验证明了该协议会让浏览器载入页面更快（例如华盛顿大学的“[SPDY有多快？](https://www.usenix.org/system/files/conference/nsdi14/nsdi14-paper-wang_xiao_sophia.pdf)”和Hervé Servy的“[评估启用SPDY的Web服务器性能](http://www.neotys.com/blog/performance-of-spdy-enabled-web-servers/)”）。同样，这些实验也被用来证明http2。我期待能有越来越多的测试实验发布。[httpwatch.com](http://blog.httpwatch.com/2015/01/16/a-simple-performance-comparison-of-https-spdy-and-http2/)也有进行一个简单的测试来证明HTTP/2名副其实。

http2在很多的场景下都证明了自己更快，包含非常多资源的高延迟的连接上。而正如之前的章节中提到，目前的趋势就是每个网站包含越来越多的资源和数据。

**8.4.7. “它违反了网络分层”**

你是认真的么？网络分层并不是不可侵犯的。如果我们在指定http2的时候已经踏入了灰色地带，那我们当然可以尝试在限制内制定出更好更高效的协议。


**8.4.8. “它没有修复一些HTTP/1.1的问题”**

确实是这样。兼容HTTP/1.1的范式是我们的目标之一，所以一些老的HTTP功能仍然被保留。利用一些常用的协议头、可怕的cookies、验证头等等。但保留这些范式的好处就是我们在升级到新协议的时候少掉很多工作，也不需要重写很多底层的东西。Http2更多是一个新的帧层。

**8.5. http2会被广泛部署吗？**

现在还言之尚早，但我仍然要在这里做出我的预估。

很多怀疑论者会以“看看IPv6现在的德性”为例子来试图说明这个经历了10多年才开始慢慢被采用的协议。但http2毕竟不是IPv6。他是一个建立在TCP之上的使用HTTP更新机制、端口号和TLS等的协议。大部分路由器或者防火墙不需要为此而进行更改。

Google向世界证明了他们的SPDY，证明了像这样的新协议也能在足够短的时间内拥有多种实现并且能被浏览器和服务所采用。虽然如今支持SPDY服务器端数量在1%以内，但这些服务器所交换的数据却要大很多。一些最流行的网站现在也有提供SPDY支持。

我认为建立在SPDY的基本范式之上的http2会被更广泛的部署，毕竟它是IETF制定的协议。因为SPDY背负了“它是Google的协议”这个恶名，所以它的发展总是畏首畏脚。

在首次发布的幕后有很多大型浏览器。来自Freifox，Chrome和IE的代表宣布了他们会发布支持http2特性的浏览器，并且他们已经演示了一些能正常运作的实现。

也有很多公司，像Google，Twitter和Facebook希望尽快支持http2，所以我们可以期待着很快能有支持http2的服务器，例如Apache HTTP Server和nginx。[H2o](https://github.com/h2o/h2o)作为一个极有潜力的新生HTTP服务器，也同样支持http2。

那些大型代理服务器开发商，例如HAProxy、Squid和Varnish也表示出了他们对支持http2的兴趣。

我也相信一旦规范被RFC批准，会有更多的实现雨后春笋般的涌现出来。

在2015年1月下旬，默认启用HTTP/2的Firefox 35发布后，Google也宣布Chrome 40对2%的用户启用了该功能。虽然他没有告诉我们具体的数字，但HTTP/2已经差不多占到了他们全球流量的5%。与此同时，Firefox 35记录到了9%的相应都是HTTP/2。