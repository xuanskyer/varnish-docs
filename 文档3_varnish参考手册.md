# VCL

## varnish configuration language
Author:	Dag-Erling Smørgrav

Author:	Poul-Henning Kamp

Author:	Kristian Lyngstøl

Author:	Per Buer

Date:	2010-06-02

Version:	1.0

Manual section:	7

### 描述
VCL语言是一种限定域语言，它被设计用来为varnish的HTTP加速器定义请求处理和文档缓存策略。
当一个新的配置文件被加载,varnish 管理进程把 vcl 转换成 c 代码,然后编译成动态共享库连接到服务器进程。
### 语法
VCL 的语法相当简单,和 c,perl 相似。使用花括号做界定符,使用分号表示声明 结束。注释和 C、C++、perl 语法一样,你可以自己选择。除此之外还类似 c 语法,比如赋值(=)、比较(==)、和一些布尔值(!、&&、||), VCL 支持正则表达式,ACL 匹配使用 ~ 操作。不同于 C 和 perl 的地方,反斜杠(\)在 VCL 中没有特殊的含义。只是用来匹配 URLs, 所以没有反斜线,请大家自由使用正则表达式。字符串的连接仅仅只需要把它们一个个放在一起，而并不需要任何连接操作符。通过关键字`set`进行赋值，VCL 没有用户定义的变量，只能直接给服务端、请求端、文档体的变量赋值，这些内容大部分是人工设置，而且给这些变量分配值的时候, 必须有一个 VCL 兼容的单位
你可以用`set`关键字设置任意的HTTP头。可以用`remove`或`unset`关键字来移除HTTP头，这两个是同义词。
关键词`return`可以在程序结束的时候返回一个子程序。子程序可以是下列中的一个：
* deliver
* error
* fetch
* hash
* lookup
* pass
* pipe
* restart

请看下上面的这个子程序，搞清楚哪些返回动作是可用的。

你可以通过`log`关键字，记录任何信息到共享内存日志里。

你可以通过关键字`inclucde`后面跟一个用双引号（`“”`）引起来的文件名的形式，将该VCL文件的内容引入到当前代码中。
VCL 有 if 用法，但是没有循环。
### backend 声明
一个backend声明即创建初始化一个以`backend`命名的对象：

```
backend www {
  .host = "www.example.com";
  .port = "http";
}
```
这个backend对象，可以被用在随后的一个请求中：

```
if (req.http.host ~ "^(www.)?example.com$") {
  set req.backend = www;
}
```
为了避免后端服务器过载，`.max_connections` 可以用来设置后端服务器的最大连接数。

超时参数可以在backend声明内进行覆盖设置。参数`.connect_timout`表示服务器连接等待的时间，`.first_byte_timeout`表示第一个字符从服务端传过来的时间，`.between_bytes_timeout`表示每个字符之间的接收间隔时间。
这些都可以在backend声明内设置：

```
backend www {
  .host = "www.example.com";
  .port = "http";
  .connect_timeout = 1s;
  .first_byte_timeout = 5s;
  .between_bytes_timeout = 2s;
}
```
标记一个backend为不健康状态后，这些不健康backend的统计数会被记录到神圣模式列表中，`.saintmode_threshold`可以用来设置这个列表记录的最大值。
设置这个值为0则不做任何记录。这个参数可以在backend声明内重新设置。

### 管理器(Directors)
管理器是一个多backend服务簇的逻辑组合，以此实现冗余。它扮演的角色就是，当一个backend服务端挂掉时，让varnish在这些服务中选择其他的来使用。

管理器的类型有很多种。不同的管理器用不同的算法来选择哪个管理器被使用。

配置一个管理器大概像这样：

```
director b2 random {
  .retries = 5;
  {
    // We can refer to named backends
    .backend = b1;
    .weight  = 7;
  }
  {
    // Or define them inline
    .backend  = {
      .host = "fs2";
    }
  .weight         = 3;
  }
}
```
#### random管理器 （random director）
每个random管理器有个参数`.retries`。它指定为了查找到一个可用的backend服务端而重连backend的次数。默认每个backend服务端的次数相同。

管理器内，每个backend服务有个参数：`.weight`，它设置每个backend的权重。
#### THE round-robin 管理器 （round-robin director）Round-robin 没有什么选项。
#### 客户端管理器（client director）
客户端管理器基于客户端标识来选择backend服务端。你可以通过设置VCL变量`client.identity`为session cookie或其他类似的值，来标识一个客户端。

注意：在2.1版本中没有`client.identity`变量，管理器使用`client.ip`来分发客户端到backend服务。

客户端管理器有一个配置项：`retries`，它设置了为了查找到一个健康的backend服务端而需要重backend服务端的次数。
#### hash管理器（hash director）
hash管理器基于URL的hash值来选择backend服务端。

This is useful is you are using Varnish to load balance in front of other Varnish caches or other web accelerators as objects won’t be duplicated across caches.

hash管理器有一个配置项：`retries`，它设置了为了查找到一个健康的backend服务端而需要重backend服务端的次数。
#### DNS管理器（DNS director）
DNS管理器有三种不同的方式来选择backend服务端。像random管理器一样使用，或者像round-robin管理器一样使用，或者使用.list方式：

```
director directorname dns {
        .list = {
                .host_header = "www.example.com";
                .port = "80";
                .connect_timeout = 0.4;
                "192.168.15.0"/24;
                "192.168.16.128"/25;
        }
        .ttl = 5m;
        .suffix = "internal.example.net";
}
```
这段代码会指定384个backend端（192.168.15.0 - 192.168.15.255 ，192.168.16.128 - 192.168.16.255 ？），都绑定80端口，连接超时为0.4s。`.list`中的参数必须在IP列表前面设置。

`.ttl`定义DNS查找的缓存时间。

上面的例子，会先追加`internal.example.net`到客户端提交过来的主机报文头中，然后在去匹配backend服务端。所有的参数都是可选的。

# varnishadm

# varnishd

# varnishhist

# varnishlog

# varnishncsa

# varnishreplay

# varnishsizes

# varnishstat

# varnishtest

# varnishtop

