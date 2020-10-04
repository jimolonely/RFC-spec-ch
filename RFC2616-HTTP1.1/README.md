[https://tools.ietf.org/html/rfc2616](https://tools.ietf.org/html/rfc2616)

1999年

**摘要**

-----

Hypertext Transfer Protocol(HTTP) 是应用层的协议，用在分布式、协作、超媒体信息系统。
是一个通用的、无状态的协议，可以用在很多任务，不只是超文本，比如名称服务，分布式对象管理系统，
只需要扩展其request方法，错误码和header。HTTP的特点是允许系统和数据传输相互独立，因为HTTP不
影响数据本身，只是数据的类型和协调。

从1990年开始，HTTP已经被用于万维网。 

# 介绍

## 1.1 目的

第一版的HTTP/0.9协议，只是个简单的裸数据传输，HTTP/1.0允许数据以MIME格式传输。
然而1.0没有考虑分层的代理、缓存、持久化连接或虚拟主机的影响，所以需要进化了。

HTTP/1.1更加严格。

应用需要除了检索外的更多功能，比如搜索，前后端更新和注释。
HTTP开放式的method和header定义了请求的目的。在URI上构建，用URL或URN表名资源位置。

HTTP也可用作通用协议，包括SMTP、NNTP、FTP、Gopher、WATS。

## 1.2 规约

"MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT","SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL"
这些术语的定义要搞清楚。

## 1.3 术语

* connection（连接）
    * 2个程序为了通信而建立的传输层的虚拟电路
* message（消息）
    * HTTP交流的基础单元，由第4章定义的结构化的字节序列，通过连接传输
* request（请求）    
    * 第5章定义，HTTP request message
* response（响应）
    * 第6章定义，HTTP response message
* resource（资源）
    * 可以用过URI确定的网络数据对象或服务，3.2节定义。资源可以有多种表征。
* entity（实体）
    * 一个request或response的传输负载，由entity-header和entity-body组成，第7章描述。
* representation（表示）
    * 包含在响应中的实体，需要进行内容协商，12章描述。对于一个响应状态，可能有多种表示。
* content negotiation（内容协商）
    * 请求时，选择合适的表示的机制，12章。任何响应中的实体表示都可被协商。
* variant（变体）
    * 一个资源在每个瞬间都可能有一个或多个表示，每个表示叫一个变体。
* client（客户端）
    * 为了发送请求而建立连接的程序
* user agent（用户代理）
    * 初始化请求的客户端，通常是浏览器、编辑器、爬虫或自定义工具。
* server（服务端）
    * 接收连接并响应请求的应用程序。很多程序既是客户端也是服务端，比如网关，代理，隧道等。
* origin server（源服务器）
    * 资源所存在或被创建的服务器
* proxy（代理）
    * 一个既是服务端又是客户端的程序，负责转发请求和响应。透明的代理不会修改请求和响应。
* gateway（网关）
    * 类似代理，区别是客户端感觉不到这是个代理，网关就像是原始服务器一样
* tunnel（隧道）
    * 在2个连接之间的传递，一旦激活，就不被认为是HTTP通信的一部分。当两边的连接关闭时，隧道也不存在了。
* cache（缓存）
    * 程序本地对响应资源的存储和控制、检索、删除。缓存存储可被缓存的响应来减少带宽，隧道不能有缓存。
* cacheable（可被缓存）
    * 在13章定义。
* first-hand（一手）
    * 对一个响应，没有经过代理等转发耽误的，直接从源服务器过来的响应
* explicit expiration time（显示过期时间）
    * 过期了就不能使用缓存了
* heuristic expiration time（启发式过期时间） 
    * 没有明确过期时间
* age（年龄）
    * 一个响应，从源服务器发出到现在的时间
* freshness lifetime（生存时间）
    * 一个响应，从产生到过期的时间
* fresh（新鲜）
    * age小于生存时间就是新鲜的响应
* stale（过时）
    * age大于生存时间
* semantically transparent（语义上透明）
    * 对缓存来说，缓存和服务器是一样的
* validator（校验器）
    * 一个元素（实体tag或Last-Modified时间）表名缓存是否合理
* upstream/downstream(上下游)
    * 数据的流向，总数从上往下
* inbound/outbound（出入边界）
    * 相对于源服务器来说，inbound就是收到请求，outbound就是响应请求

## 1.4 整体运行

请求响应协议：v（单个连接），UA（User Agent）， O（Origin Server）
```shell script
          request chain ------------------------>
       UA -------------------v------------------- O
          <----------------------- response chain
```

复杂点的请求：A(gateway)

```shell script
          request chain -------------------------------------->
       UA -----v----- A -----v----- B -----v----- C -----v----- O
          <------------------------------------- response chain
```

当B有缓存时：
```shell script
          request chain ---------->
       UA -----v----- A -----v----- B - - - - - - C - - - - - - O
          <--------- response chain
```

# 2.符号和语法说明

## 2.1 增强的BNF（巴克斯洛尔范式）

认识下面的范式：

* name = definition 
* "literal"
    * 双引号文本，大小写不敏感    
* rule1 | rule2
    * 可选其一
* (rule1 rule2)
    * 表示单个元素
* `*rule`
    * 表示重复，`<n>*<m>element`表示n-m遍
* `[rule]`
    * 表示可选的元素
* N rule
    * 等价于 `<n>*<n>element`
* `#rule`
    * 表示可被逗号分开的多个元素
* `; 注释`
    * 逗号后面的注释
* `implied *LWS`
    * 就是任何单词之间是由制表符的（Linear white space）

## 2.2 基础规则

```shell script
       OCTET          = <any 8-bit sequence of data>
       CHAR           = <any US-ASCII character (octets 0 - 127)>
       UPALPHA        = <any US-ASCII uppercase letter "A".."Z">
       LOALPHA        = <any US-ASCII lowercase letter "a".."z">
       ALPHA          = UPALPHA | LOALPHA
       DIGIT          = <any US-ASCII digit "0".."9">
       CTL            = <any US-ASCII control character
                        (octets 0 - 31) and DEL (127)>
       CR             = <US-ASCII CR, carriage return (13)>
       LF             = <US-ASCII LF, linefeed (10)>
       SP             = <US-ASCII SP, space (32)>
       HT             = <US-ASCII HT, horizontal-tab (9)>
       <">            = <US-ASCII double-quote mark (34)>
```

一些符号约定：

```shell script
CRLF           = CR LF

LWS            = [CRLF] 1*( SP | HT )

TEXT           = <any OCTET except CTLs,
            but including LWS>

HEX            = "A" | "B" | "C" | "D" | "E" | "F"
              | "a" | "b" | "c" | "d" | "e" | "f" | DIGIT

token          = 1*<any CHAR except CTLs or separators>
separators     = "(" | ")" | "<" | ">" | "@"
              | "," | ";" | ":" | "\" | <">
              | "/" | "[" | "]" | "?" | "="
              | "{" | "}" | SP | HT

comment        = "(" *( ctext | quoted-pair | comment ) ")"
ctext          = <any TEXT excluding "(" and ")">

quoted-string  = ( <"> *(qdtext | quoted-pair ) <"> )
qdtext         = <any TEXT except <">>

quoted-pair    = "\" CHAR
```

# 3.协议参数

## 3.1 协议版本

HTTP的版本格式为： `<major>.<minor>`.

* 当消息格式变化了，才升大版本
* 增加功能时，只升小版本

```shell script
HTTP-Version   = "HTTP" "/" 1*DIGIT "." 1*DIGIT
```

HTTP/2.4 比 HTTP/2.13版本小。

关于版本不会向下兼容，网关和代理一定不能在代理后升版本。

## 3.2 统一资源定位符

URI有很多名字：WWW地址，UDI，URI，URL，URN。

对于HTTP来讲，只是通过名称、地址或其他字符来确定一个资源而已。

### 3.2.1 通用语法

URI可以是相对的和绝对的，具体参考[规范2396](https://tools.ietf.org/html/rfc2396)

另外，当URI长度超过服务端能接收的范围，SHOULD返回414.

### 3.2.2 http URL

http的格式：

```shell script
http_URL = "http:" "//" host [ ":" port ] [ abs_path [ "?" query ]]
```

端口没给，默认80，尽量避免直接用IP。

### 3.2.3 比较URI

比较URI是否匹配时，应该用大小写敏感的字节对字节的比较，除了下面的例外：
* 端口没写整默认的
* 主机名一定是大小写不敏感的
* abs_path为空等价于 "/"

转义字符相等，例如：下面的URI都相等

* http://abc.com:80/~smith/home.html
* http://ABC.com/%7Esmith/home.html
* http://ABC.com:/%7esmith/home.html

## 3.3 Date/Time格式
















