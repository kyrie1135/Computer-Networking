# 抓包实践

## HTTP 请求

### Request headers
- GET / HTTP/1.1
  - 使用 GET 方法
  - 获取 / 路径上的文件，这里应该是有个 HTTP 查找规则的，默认是寻找路径下的 index.html
  - 使用 HTTP 协议，版本是 1.1
- Host
  - 主机名（可能是域名也可能是 ip）
  - RFC 协议规定所有的 HTTP 请求必须携带 Host 头，即使 Host 没有值，也必须带上这个 Host 头附加一个空串，如果不满足，应用服务器应该抛出 400 Bad Request。协议虽然这样规定，不过大部分网关或者服务器都比较仁慈，既然没有指定 Host 字段，那就给你默认加上一个。网关代理可以根据不同的 Host 值转发到不同的 upstream 服务节点，它常用于虚拟主机服务业务。
- Pragma	no-cache
  - 禁止缓存；
  - 在前端开发模式下经常会加上这个头部，当网关收到一个带有这样请求的头部时，即使内部存在该请求资源的缓存并且有效也不可以直接发送给客户端，而必须转发给后面的 upstream 进行处理。不过如果真的所有的网关都遵循这个协议的话，攻击是很容易伪造的，所以它一般仅用于开发模式，防止静态资源修改后前端得不到即时更新。
- Cache-Control	no-cache
  - 通知服务器，不使用缓存资源；
  - 这个头既可以用于请求，也可以用于响应，在请求和响应的取值不一样，分别代表了不同的意思。
  - no-cache：如果 no-cache 没有指定值，那就表示不允许缓存。对于请求来说，服务器不得使用缓存内容直接返回。对于响应来说，客户端不得缓存响应的资源内容。如果 no-cache 指定了值，那就表示对应的头信息不得使用缓存，其他的信息还是可以缓存的。
  - no-store：告知对方不要持久化请求/响应数据到其他地方，这种信息是敏感的，要保持它的易失性。告知对方存在内存（memory）可以，但是不能写在磁盘（disk）中。
  - no-transform：告知对方不要转换数据。比如客户端上传了 raw 图像数据，服务器一般都会选择性压缩图像数据进行存储。no-transform 告知对方保留原始数据信息，不要进行任何转换。
  - only-if-cached：用于请求头，告知服务器只要那些已经缓存的内容，如果没有缓存内容就返回 504 Gateway Timeout 错误。（只要缓存，不要新数据）
  - max-age：用于请求头。限制缓存内容的年龄，如果超过 max-age 年龄的，需要服务器去 reload 内容资源。
  - max-state：用于请求头。客户端允许服务器返回缓存已过期的资源内容，但是限定了最大过期时间。
  - min-fresh：用于请求头。客户端限制服务器不要那些即将过期的资源内容。
  - public：用于响应头。表示允许客户端缓存响应信息，并可以给别人使用。比如代理服务器缓存静态资源供素有代理用户使用。
  - private：用于响应头。表示仅允许客户端缓存响应信息给自己使用，不得分享给别人。这样是为了禁止代理服务器进行缓存，而允许客户端自己缓存资源内容。
- WWW-Authenticate
  - WWW-Authenticate 是 401 Unauthorized 错误码返回时必须携带的头，该头会携带一个问题 Challenge 给客户端，告知客户端需要携带这个问题的答案来请求服务器才可以继续访问目标资源。这种问题 Challenge 可以自定义，比较常见的是 Basic 认证。
  - WWW-Authenticate: Basic realm=xxx
  - Basic 指代 base64 加密算法(不安全)，realm 指代认证范围 /场合 /情景名称。
- Authorization: Basic YWRtaW46YWRtaW4xMjM=
  - value = base64(user_name:password)
  - 对于某些需要特殊权限才能访问的资源需要客户端在请求里提供用户名密码的认证信息。它是对 WWW-Authenticate 的应答。
- Upgrade-Insecure-Requests：1
  - Upgrade-Insecure-Requests 是一个请求首部，用来向服务器端发送信号，表示客户端优先选择加密及带有身份验证的响应，并且它可以成功处理 upgrade-insecure-requests CSP 指令。
- User-Agent	Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36
  - 携带当前的用户代理信息，一般包括浏览器、浏览器内核和操作系统的版本型号信息。它是 Server 头是对应的，一个是表达服务器信息，一个是表达客户端信息。服务器可以根据用户代理信息统计出网页服务的浏览器、操作系统的使用占比情况，服务器也可以根据 UA 的信息来定制不一样的内容。
- Accept	text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
  - 表示客户端期望服务器返回的媒体格式，客户端期望的资源类型服务器可能没有，所以客户端会期望多种类型，并且设置优先级，服务器根据优先级寻找响应的资源返回给客户端。
- Accept-Encoding	gzip, deflate
  - HTTP 请求头 Accept-Encoding 会将客户端能够理解的内容编码方式——通常是某种压缩算法——进行通知。通过内容协商的方式，服务端会选择一个客户端提议的方式，使用并在响应报文首部 Content-Encoding 中通知客户端该选择。
- Accept-Language	zh-CN,zh;q=0.9,en;q=0.8
  - Accept-Language 请求头允许客户端声明它可以理解的自然语言，以及优先选择的区域方言。借助内容协商机制，服务器可以从诸多备选项中选择一项进行应用，并使用 Content-Language 应答头通知客户端它的选择。浏览器会基于其用户界面语言来为这个请求头设置合适的值，即便是用户可以进行修改，但是这种情况极少发生。
- Cookie
  - Cookie 是一个请求首部，其中含有先前服务器通过 Set-Cookie 首部投放并存储到客户端的 HTTP cookies；
- Connection	keep-alive
  - Connection 头决定当前的事务完成后，是否会关闭网络连接。如果该值是 “keep-alive”，网络连接就是持久的，不会关闭，使得对同一个服务器的请求可以继续在该连接上完成。
  - close：表明客户端或服务器想要关闭该网络连接，这是 HTTP/1.0 请求的默认值。
- Referer
  - Referer 首部包含了当前请求页面的来源页面的地址，即表示当前页面是通过此来源页面里的链接进入的。服务端一般使用 Referer 头部识别访问来源，可能会以此进行统计分析、日志记录以及缓存优化等。

### Response Headers
- HTTP/1.1 200 OK
  - HTTP 协议版本 1.1，状态码为 200，信息为 OK
- Date	Wed, 04 Sep 2019 23:00:19 GMT
  - Date 是一个通用首部，其中包含了报文创建的日期和时间。
- Content-Type	text/html; charset=utf-8
  - Content-Type 实体头部用于指示资源的 MIME 类型 media type；
  - 在响应中，Content-Type 标头告诉客户端实际返回的内容的内容类型，浏览器会在某些情况下进行 MIME 查询，并不一定遵循此标题的值；为了防止这种行为，可以将标题 X-Content-Type 设置为 nosniff；
  - 在请求中（如 POST 或 PUT），客户端告诉服务器实际发送的数据类型。
- Transfer-Encoding	chunked
  - Transfer-Encoding 消息首部指明了将 entity 安全传递给用户所采用的编码形式；
  - Transfer-Encoding 是一个逐跳传输消息首部，即仅用于两个节点之间的消息传递，而不是所请求的资源本身。一个多节点连接中的每一段都可以应用不同的 Transfer-Encoding 值。如果你想要把压缩后的数据应用于整个连接，那么奇怪使用端到端传输消息首部 Content-Encoding
  - chunked：数据以一系列分块的形式发送。Content-Length 在这种情况下不被发送。在每一个分块的开头需要添加当前分块的长度，以十六进制的形式表示，后面紧跟着'\r\n'，之后是分块本身，后面也是'\r\n'。终止块是一个常规的分块，不同之处在于其长度为 0.终止块后面是一个挂载，由一系列（或者为空）的实体消息首部构成。
  - compress：压缩算法，因为专利问题已被弃用；
  - deflate：采用 zlib 结构，和 deflate 压缩算法。
  - gzip：表示采用 Lempel-Ziv coding（LZ77）压缩算法，以及 32 位 CRC 校验的编码方式。这个编码最初由 UNIX 平台式的 gzip 程序采用。
  - identity：用于指代自身（例如：未经过压缩和修改）。
- Vary	Accept-Encoding
  - Vary 是一个 HTTP 响应头信息，它决定了对于未来的一个请求头，应该用一个缓存的回复还是向源服务器请求一个新的回复。它被服务器用来表面在内容协商算法中选择一个资源代表的时候应该使用哪些头部信息。
- ETag	W/"141fb-8bBJz9vV9Eti0oCDTknCy6CH8/Y"
  - ETag HTTP 响应头是资源的特定版本的标识符。这可以让缓存更高效，并节省带宽，因为如果内容没有改变，Web 服务器不需要发送完整的响应。而如果内容发送了改变，使用 ETag 有助于防止资源的同时更新相互覆盖。
  - 如果给定 URL 中的资源更改，则一定要生成新的 ETag 值。因此 Etags 类似于指纹，也可能被某些服务器用于跟踪。比较 etags 能快速确定此资源是否变化，但也可能被跟踪服务器永久存留。
  - 客户端发送 If-None-Match 字段，值为服务器返回的 E-Tag，然后服务器将客户端的 Etag 与其当前版本的资源的 ETag 进行比较，如果两个值匹配（即资源未更改），服务器将返回不带任何内容的 304 修改状态，告诉客户端缓存版本可用。 
- Content-Encoding
  - Content-Encoding 是一个实体消息首部，用于对特定媒体类型的数据进行压缩。当这个首部出现的时候，它的值表示消息主体进行了何种方式的内容编码转换。这个消息首部用来告知客户端应该怎样解码才能获取在 Content-Type 中标示的媒体类型内容。
- X-CDN-Edge
- X-Cache
  - 这两个字段是 CDN 加速
- Proxy-Connection
  - 与 Connection 类似，只不过这是代理服务器返回的响应头部字段。