# varnish 教程Varnish 是一个 web 加速器,被安装在 web 应用程序前面,缓存 web 应用程序,并响应 用户请求,varnish 让您的 web 应用程序运行的更快,并且 varnish 灵活好用。
这个指南没有覆盖所有的 varnish 函数和功能,它可以让您对 varnish 有个全面的认识并 且掌握如何运用它。我们假设您有一个 web 服务器和 web 应用程序正在运行,而且您想使用 varnish 给您的 web 运行程序加速。此外我们还假设您已经阅读了“varnish 安装”并且按照默认配置安装了 varnish。

教程分为多个短章，每个章节作为一个独立的章节。祝你好运~

## Backend servers(后端服务器)&emsp;&emsp;varnish 有一个概念叫做“后端服务器”或者叫“原点服务器”,一个后端服务器将 提供 varnish 加速的内容。&emsp;&emsp;我们的第一个任务就是告诉 varnish 在哪里可以找到他要的内容。使用您喜欢的文 本编辑器打开 varnish 默认的配置文件,如果您是源码安装的配置文件可能在 /usr/local/etc/varnish/default.vcl , 如 果 您 是 二 进 制 包 安 装 的 配 置 文 件 可 能 在 /etc/varnish/default.vcl.&emsp;&emsp;配置文件的最顶端如下:	```
	￼# backend default {	￼# .host = "127.0.0.1";	￼# .port = "8080";	￼#}```
&emsp;&emsp;我们取消前面的注释,并且把 8080 端口改成 80 端口。如下```
￼backend default {	￼.host = "127.0.0.1";
	￼.port = "80";}
```
&emsp;&emsp;现在,这块配置定义了一个 varnish 默认访问的后端服务器,当 varnish 需要从后端 服务器获取内容的时候,它就会访问自己(127.0.0.1)的 80 端口。&emsp;&emsp;Varnish 可以定义多个后端服务器而且您可以通过定义多个后端服务器达到负载均 衡的目的。&emsp;&emsp;现在我们完成了基本的 varnish 配置,我们可以在 8080 端口上启动 varnish,并做 一些基本的测试。
## Starting Varnish(启动 varnish)
&emsp;&emsp;假设varnishd在您的环境变量中,您可能需要运行pkill varnishd来确定varnish没有运行。然后使用 root 执行下面的命令。`￼varnishd -f /usr/local/etc/varnish/default.vcl -s malloc,1G -T 127.0.0.1:2000 -a ￼0.0.0.0:8080`&emsp;&emsp;我添加了一些选项,现在来详细分析他们:`-f /usr/local/etc/varnish/default.vcl`这个 –f 选项指定 varnishd 使用哪个配置文件。`-s malloc,1G`这个 –s 选项用来确定 varnish 使用的存储类型和存储容量,我使用的是 malloc 类型(malloc 是一个 C 函数,用于分配内存空间), 1G 定义多少内存被 malloced,1G = 1gigabyte。`-T 127.0.0.1:2000`&emsp;&emsp;Varnish 有一个基于文本的管理接口,启动它的话可以在不停止 varnish 的情况下来 管理 varnish。您可以指定管理软件监听哪个接口。当然您不能让全世界的人都能访问您的 varnish 管理接口,因为他们可以很轻松的通过访问 varnish 管理接口来获得您的 root 访问权 限。我推荐只让它监听本机端口。如果您的系统里有您不完全信任的用户,您可以通过防火 墙规则来限制他访问 varnish 的管理端口。`-a 0.0.0.0:8080`&emsp;&emsp;这一句的意思是制定 varnish 监听所有 IP 发给 8080 端口的 http 请求,如果在生产 环境下,您应该让 varnish 监听 80,这也是默认的。&emsp;&emsp;现在您的 varnish 已经在运行了,现在我们来验证它是否工作正常,在留言其中输入 http://192.168.2.2:8080/ ,您应该可以看见您的 web 程序在这里运行。&emsp;&emsp;使用 varnish 后,web 应用程序是否加速,取决于一些原因。如果您的程序的每个会话 都使用 cookies 或者您每个程序都需要三次握手认证,这样 varnish 就不能缓存更多的数据, 我们暂时忽略这个问题,等到“提高缓存命中率”这节的时候我们再继续讨论这个问题。&emsp;&emsp;想要知道 varnish 对您的网站做了什么,请查看 logs。
## Logging in varnish(记录数据)&emsp;&emsp;Varnish 一个真正的特点就是它如何记录数据的。使用内存段代替普通的日志文件, 当内存段使用完以后,又从头开始,覆盖最旧的记录。这样就可以非常快的记录数据,,并 且不需要磁盘空间。&emsp;&emsp;缺点就是您没有把数据写到磁盘上,可能会消失。(varnish 也支持将数据写到硬盘 的文件上,看您如何选择)&emsp;&emsp;Varnishlog 这个程序可以查看 varnish 记录了哪些数据。Varnishlog 给您生成原始的 日志,包括所有的事件。我一会给您演示。&emsp;&emsp;在运行了 varnish 的终端窗口上,运行 varnishlog 这个命令。 
&emsp;&emsp;您可以看见如下显示```￼0 CLI - Rd ping￼0 CLI - Wr 200 PONG 1277172542 1.0
```&emsp;&emsp;这是检查 varnish 的主进程是否正常,如果看见这就说明一切OK

&emsp;&emsp;现在您去浏览器通过 varnish 重新访问您的 web 程序,您将看到如下信息:```
￼11 SessionOpen 	c 127.0.0.1 58912 0.0.0.0:8080￼11 ReqStart 		c 127.0.0.1 58912 595005213￼11 RxRequest 	c GET￼11 RxURL 		c /￼11 RxProtocol 	c HTTP/1.1￼11 RxHeader 		c Host: localhost:8080￼11 RxHeader 		c Connection: keep-alive```
第一列是任意的数,它用来定义请求,相同的号码代表相同的 HTTP 传输。第二列是信息标记,所有的日志都带有一个标记(tag),标记对应相对的操作。Rx 表示 varnish 收到数据,Tx 表示 varnish 发送数据。第三列代表数据是从哪里传出或者传入的,是从 c(client)还是 b(backend)。 
第四列是被记录的数据。现在,您可以过滤掉相当多的 varnishlog,这些基本的选项,您需要知道:	-b \\只显示 varnish 和 backend server 之间的日志,当您想要优化命中率的时 候可以使用这个参数。	-c \\和-b 差不多,不过它代表的是 varnish 和 client 端的通信。	-i tag \\只显示某个 tag,比如“varnishlog –i SessionOpen”将只显示新会话,注 意,这个地方的 tag 名字是不区分大小写的。	-I \\通过正则表达式过滤数据,比如“varnishlog -c -i RxHeader -I Cookie”将 显示所有接到到来自客户端的包含 Cookie 单词的头信息。	-o \\聚合日志请求 ID如果 varnish 一切运行 OK,我们就可以把它调整到 80 端口上。


## Sizing your cache

Picking how much memory you should give Varnish can be a tricky task. A few things to consider:

* How big is your hot data set. For a portal or news site that would be the size of the front page with all the stuff on it, and the size of all the pages and objects linked from the first page.

* How expensive is it to generate an object? Sometimes it makes sense to only cache images a little while or not to cache them at all if they are cheap to serve from the backend and you have a limited amount of memory.

* Watch the n_lru_nuked counter with :ref:`varnishstat`_ or some other tool. If you have a lot of LRU activity then your cache is evicting objects due to space constraints and you should consider increasing the size of the cache.

## 设置varnish监听80端口&emsp;&emsp;如果您的程序正常运行,没有问题,我们就可以把 varnish 调整到 80 端口运行。先 关闭 vernish`pkill varnishd`&emsp;&emsp;然后停止您的 web 服务器,修改 web 服务器配置,把 web 服务器修改成监听 8080 端口,然后修改 varnish 的 default.vcl 和改变默认的后端服务器端口为 8080.&emsp;&emsp;先启动您的 web 服务器,然后在启动 varnish:`varnishd -f /usr/local/etc/varnish/default.vcl -s malloc,1G -T 127.0.0.1:2000` 
&emsp;&emsp;我们取消了-a 选项,这样 varnish 将监控默认端口,启动后,检查您的 web 程序是否正常。
## varnish配置语言-VCL
&emsp;&emsp;Varnish 有一个很棒的配置系统,大部分其他的系统使用配置指令,让您打开或者关闭 一些开关。Varnish 使用区域配置语言,这种语言叫做“VCL”(varnish configuration language),在执行 vcl 时,varnish 就把 VCL 转换成二进制代码。&emsp;&emsp;VCL 文件被分为多个子程序,不同的子程序在不同的时间里执行,比如一个子程序在接到请求时执行,另一个子程序在接收到后端服务器传送的文件时执行。&emsp;&emsp;varnish 将在不同阶段执行它的子程序代码,因为它的代码是一行一行执行的,不存在优先级问题。随时可以调用这个子程序中的功能并且当他执行完成后就退出。 
&emsp;&emsp;如果到最后您也没有调用您的子进程中的功能,varnish 将执行一些内建的 VCL 代码,这些代码就是 default.vcl 中被注释的代码。 
&emsp;&emsp;99%的几率您需要改变 vcl_recv 和 vcl_fetch 这两个子进程。 
### vcl_recv&emsp;&emsp;vcl_recv(当然,我们在字符集上有点不足,应为它是 unix)在请求的开始被调用, 在接收、解析后,决定是否响应请求,怎么响应,使用哪个后台服务器。&emsp;&emsp;在 vcl_recv 中,您可以修改请求,比如您可以修改 cookies,添加或者删除请求的 头信息。&emsp;&emsp;注意 vcl_recv 中只有请求的目标,req is available。 ### vcl_fetch 
&emsp;&emsp;vcl_fetch在一个文件成功从后台获取后被调用,通常他的任务就是改变 response headers,触发 ESI 进程,在请求失败的时候轮询其他服务器。&emsp;&emsp;在 vcl_fetch 中一样的包含请求的 object,req,available,他们通常是 backend response,beresp。beresp 将会包含后端服务器的 HTTP 的头信息### actions主要有以下动作	pass \\当一个请求被 pass 后,这个请求将通过 varnish 转发到后端服务器,但是它不会被缓存。pass 可以放在 vcl_recv 和 vcl_fetch 中。	lookup \\当一个请求在 vcl_recv 中被 lookup 后,varnish 将从缓存中提取数据,如果缓存中没有数据,将被设置为 pass,不能在 vcl_fetch 中设置 lookup。	pipe \\pipe 和 pass 相似,都要访问后端服务器,不过当进入 pipe 模式后,在 此连接未关闭前,后续的所有请求都发到后端服务器(这句是我自己理解后简化的,有能力的朋友可以看看官方文档,给我提修改建议)。	deliver \\请求的目标被缓存,然后发送给客户端	esi \\ESI-process the fetched document(我理解的就是 vcl 中包换一段html代码)### Requests,Responses and objects 
在 VCL 中,有 3 个重要的数据结构 
* 请求（request） 从客户端进来* 响应（responses） 从后端服务器过来 

* 实体（object） 存储在 cache 中在 VCL 中,你需要知道以下结构	req \\请求目标,当 varnish 接收到一个请求,这时 req object 就被创建了, 你在 vcl_recv 中的大部分工作,都是在 req object 上展开的。	
	beresp \\后端服务器返回的目标,它包含返回的头信息,你在 vcl_fetch 中的 大部分工作都是在 beresp object 上开展的。	
	obj \\被 cache 的目标,只读的目标被保存于内存中,obj.ttl 的值可修改,其 他的只能读。### 运算符

VCL 支持以下运算符,请阅读下面的例子:* = \\赋值运算符* == \\对比* ~ \\匹配,在 ACL 中和正则表达式中都可以用 
* ! \\否定* && \\逻辑与
*  || \\逻辑或### 示例1 – manipulation headers 
我们想要取消我们服务器上/images 目录下的所有缓存:```
sub vcl_recv {
  if (req.url ~ "^/images") {
    unset req.http.cookie;
  }
}
```现在,当这个请求在操作后端服务器时,将不会有 cookie 头,这里有趣的行是 if-statement,它匹配 URL,如果匹配这个操作,那么头信息中的 cookie 就会被删除。 
### 示例 2 – manipulation beresp从后端服务器返回对象的值满足一些标准,我们就修改它的 TTL 值:```sub vcl_fetch {
   if (req.url ~ "\.(png|gif|jpg)$") {
     unset beresp.http.set-cookie;
     set beresp.ttl = 3600s;
  }
}
```
###  EXAMPLE3-ACLs你创建一个 VCL 关键字的访问控制列表。你可以配置客户端的 IP 地址```
# Who is allowed to purge....
acl local {
    "localhost";
    "192.168.1.0"/24; /* and everyone on the local network */
    ! "192.168.1.23"; /* except for the dialin router */
}

sub vcl_recv {
  if (req.request == "PURGE") {
    if (client.ip ~ local) {
       return(lookup);
    }
  }
}

sub vcl_hit {
   if (req.request == "PURGE") {
     set obj.ttl = 0s;
     error 200 "Purged.";
    }
}

sub vcl_miss {
  if (req.request == "PURGE") {
    error 404 "Not in cache.";
  }
}
```

## 统计

&emsp;&emsp;现在您的 varnish 已经正常运行,我们来看一下 varnish 在做什么,这里有些工具可 以帮助您做到。### Varnishtop&emsp;&emsp;Varnishtop 工具读取共享内存的日志,然后连续不断的显示和更新大部分普通日志。适当的过滤使用 –I,-i,-X 和-x 选项,它可以按照您的要求显示请求的内容,客 户端,浏览器等其他日志里的信息。	varnishtop -i rxurl \\您可以看到客户端请求的 url 次数。	Varnishtop -i txurl \\您可以看到请求后端服务器的 url 次数。	Varnishtop -i Rxheader –I Accept-Encoding \\可以看见接收到的头信息中有有多少次包含 Accept-Encoding。 
### Varnishhist&emsp;&emsp;Varnishhist 工具读取 varnishd 的共享内存段日志,生成一个连续更新的柱状图,显 示最后 N 个请求的处理情况。这个 N 的值是终端的纵坐标的高度,横坐标代表的是对数, 如果缓存命中就标记“|”,如果缓存没有命中就标记上“#”符号。### Varnishsizes&emsp;&emsp;Varnishsizes 和 varnishhist 相似,除了 varnishsizes 现实了对象的大小,取消了完成 请求的时间。这样可以大概的观察您的服务对象有多大。### Varnishstat&emsp;&emsp;Varnish 有很多计数器,我们计数丢失率,命中率,存储信息,创建线程,删除对 象等,几乎所有的操作。Varnishstat 将存储这些数值,在优化 varnish 的时候使用这个命令。&emsp;&emsp;有一个程序可以定期轮询 varnishstat 的数据并生成好看的图表。这个项目叫做 Munin。Munin 可以在 http://munin-monitoring.org/找到。在 varnish 的源码中有 munin 插件。
## 实现高命中率
&emsp;&emsp;现在 varnish 已经正常运行了,您可以通过 varnish 访问到您的 web 应用程序。如果 您的 web 程序在设计时候没有考虑到加速器的架构,那么您可能有必要修改您的应用程序 或者 varnish 配置文件,来提高 varnish 的命中率。&emsp;&emsp;既然这样,您就需要一个工具用来观察您和 web 服务器之间 HTTP 头信息。服务器端您可以轻松的使用 varnish 的工具,比如 varnishlog 和 varnishtop,但是客户端的工具需要 您自己去准备,下面是我经常使用的工具。### Varnistop&emsp;&emsp;您可以使用 varnishtop 确定哪些 URL 经常命中后端。Varnishtop –i txurl 就是一个基 本的命令。您可以通过阅读“Statistics”了解其他示例。### Varnishlog&emsp;&emsp;当您需要鉴定哪个 URL 被频繁的发送到后端服务器,您可以通过 varnishlog 对请求 做一个全面的分析。varnishlog –c –o /foo/bar 这个命令将告诉您所有(-o)包含”/football/bar” 字段来自客户端(-c)的请求。### Lwp-request&emsp;&emsp;Lwp-request 是 www 库的一部分,使用 perl 语言编写。它是一个真正的基本程序, 它可以执行 HTTP 请求,并给您返回结果。我主要使用两个程序,GET 和 HEAD。&emsp;&emsp;Vg.no 是第一个使用 varnish 的站点,他们使用 varnish 相当完整,所以我们来看看 他们的 HTTP 头文件。我们使用 GET 请求他们的主页:```
￼￼￼$ GET -H 'Host: www.vg.no' -Used http://vg.no/￼GET http://vg.no/￼Host: www.vg.no￼User-Agent: lwp-request/5.834 libwww-perl/5.834￼200 OK￼Cache-Control: must-revalidate￼Refresh: 600￼Title: VG Nett - Forsiden - VG Nett￼X-Age: 463￼X-Cache: HIT￼X-Rick-Would-Never: Let you down￼X-VG-Jobb: http://www.finn.no/finn/job/fulltime/result?keyword=vg+multimedia￼￼Merk:HeaderNinja￼X-VG-Korken: http://www.youtube.com/watch?v=Fcj8CnD5188￼X-VG-WebCache: joanie￼X-VG-WebServer: leon
```&emsp;&emsp;OK,我们来分析它做了什么。GET 通过发送 HTTP 0.9 的请求,它没有主机头,所 以我需要添加一个主机头使用-H 选项,-U 打印请求的头,-s 打印返回状态,-e 答应返 回状态的头,-d 丢弃当前的连接。我们正真关心的不是连接,而是头文件。&emsp;&emsp;如您所见 VG 的头文件中有相当多的信息,比如 X-RICK-WOULD-NEVER 是 vg.no 定 制的信息,他们有几分奇怪的幽默感。其他的内容,比如 X-VG-WEBCACHE 是用来调试 错误的。&emsp;&emsp;核对一个站点是否使用 cookies,可以使用下面的命令:`GET -Used http://example.com/ |grep ^Set-Cookie`
### Live HTTP Headers&emsp;&emsp;这是一个 firefox 的插件,live HTTP headers 可以查看您发送的和接收的 http 头。软 件在 https://addons.mozilla.org/en-US/firefox/addon/3829/下载。或者 google“Live HTTP headers”。### The Role of HTTP headers
&emsp;&emsp;Varnish 认为自己是真正的 web 服务器,因为它属于您控制。IETF 没有真正定义 surrogate origin cache 角色的含义,(The role of surrogate origin cache is not really well defined by the IETF so RFC 2616 doesn’t always tell us what we should do.不知如何翻译) 

### Cache-Control&emsp;&emsp;Cache-control 指示缓存如何处理内容,varnish 关心 max-age 参数,并使用这个参数 计算每个对象的 TTL 值。&emsp;&emsp;“cache-control:nocache” 这个参数已经被忽略,不过您可以很容易的使它生效。在头信息中控制 cache-control 的 max-age,您可以参照下面,varnish 软件管理服务器的例子:


`$ GET -Used http://www.varnish-software.com/|grep ^Cache-Control`

`Cache-Control: public, max-age=600`

### AgeVarnish 添加了一个 age 头信息,用来指示对象已经被保存在 varnish 中多长时间了。 您可以在 varnish 中找到 Age 信息:`varnishlog -i TxHeader -I ^Age`
### Pragma

HTTP 1.0 服务允许发送头信息“Pragma: nocache”。Varnish 默认忽略这个头信息。你可以很容易通过VCL添加这个头信息支持。

在vcl_fetch内添加：

```
if (beresp.http.Pragma ~ "nocache") {
   pass;
}
```

### Authorization

如果varnish检测到一个授权报文头将通过请求。如果这不是你想要的，你可以取消这个设置。


### Overriding the time-to-live(ttl) 

有时候后端服务器会当掉,也许是您的配置问题,很容易修复。不过更简单的方法是修改您的 ttl,能在某种程度上修复难处理的后端。您需要在 VCL 中使用 beresp.ttl 定义您需要修改的对象的 TTL:
```
sub vcl_fetch {
    if (req.url ~ "^/legacy_broken_cms/") {
        set beresp.ttl = 5d;
    }
}
```### Normalizing your namespace有些站点访问的主机名有很多,比如 http://www.varnish-software.com, http://varnish-software.com,http://varnishsoftware.com 所有的地址都对应相同的一个 站点。但是 varnish 不知道,varnish 会缓存每个地址的每个页面。您可以减少这种情况, 通过修改 web 配置文件或者通过以下 VCL:```
if (req.http.host ~ "^(www.)?varnish-?software.com") {
  set req.http.host = "varnish-software.com";
}
```


### 更多提高命中率的方式
下面的章节应该可以给你提供一些方法来进一步提高你的命中率，特别是在cookies的章节。

#### Cookies现在 Varnish 接收到后端服务器返回的头信息中有 Set-Cookie 信息的话,将不缓存。 所以当客户端发送一个 Cookie 头的话,varnish 将直接忽略缓存,发送到后端服务器。 
这样的话有点过度的保守,很多站点使用 Google Analytics(GA)来分析他们的流 量。GA 设置一个 cookie 跟踪您,这个 cookie 是客户端上的一个 java 脚本,因此他们对服务器不感兴趣。对于一个 web 站点来说,忽略一般 cookies 是有道理的,除非您是访问一些关键部分。这个 VCL 的 vcl_recv 片段将忽略 cookies,除非您正在访问/admin/:```
if ( !( req.url ~ ^/admin/) ) {
  unset req.http.Cookie;
}
```
很简单,不管您需要做多么复杂的事情,比如您要删除一个 cookies,这个事情很 困难,varnish 也没有相应的工具来处理,但是我们可以使用正则表达式来完成这个工 作,如果您熟悉正则表达式,您将明白接下来的工作,如果您不会我建议您找找相关资 料学习一下。我们来看看 varnish 软件是怎么工作的,我们使用一些 GA 和一些相似的工具产生 cookies。所有的 cookies 使用 jsp 语言。Varnish 和企业网站不需要这些 cookies,而 varnish 会因为这些 cookies 而降低命中率,我们将放弃这些多余的 cookies,使用 VCL。下面的 VCL 将会丢弃所有被匹配的 cookies。
```
// Remove has_js and Google Analytics __* cookies.
set req.http.Cookie = regsuball(req.http.Cookie, "(^|;\s*)(_[_a-z]+|has_js)=[^;]*", "");
// Remove a ";" prefix, if present.
set req.http.Cookie = regsub(req.http.Cookie, "^;\s*", "");
```
下面的例子将删除所有名字叫 COOKIE1 和 COOKIE2 的 cookies:

```
sub vcl_recv {
  if (req.http.Cookie) {
    set req.http.Cookie = ";" req.http.Cookie;
    set req.http.Cookie = regsuball(req.http.Cookie, "; +", ";");
    set req.http.Cookie = regsuball(req.http.Cookie, ";(COOKIE1|COOKIE2)=", "; \1=");
    set req.http.Cookie = regsuball(req.http.Cookie, ";[^ ][^;]*", "");
    set req.http.Cookie = regsuball(req.http.Cookie, "^[; ]+|[; ]+$", "");

    if (req.http.Cookie == "") {
        remove req.http.Cookie;
    }
}
```
这个例子是来自 varnish wiki 的。
#### Vary各式各样的头被发送到 web server,他们让 HTTP 目标多样化。Accept-Encoding 头 就有这种感觉,当一个服务器分发一个“Vary:Accept-Encoding”给 varnish。Varnish 需要 cache 来自客户端的每个不同的 Accept-Encoding。如果客户端只接收 gzip 编码,varnish 不对 其他编码服务,那么就可以缩减编码量。问题就是这样的,Accept-Encoding 字段包含很多编码方式,下面是不同浏览器发送的:
`Accept-Encodign: gzip,deflate`
另一个浏览器发送的:`Accept-Encoding:: deflate,gzip`
Varnish 可以使两个不同的 accept-enconding 头标准化,这样就可以尽量减少变 体。下面的 VCL 代码可以是 accept-encoding 头标准化:
```
if (req.http.Accept-Encoding) {
    if (req.url ~ "\.(jpg|png|gif|gz|tgz|bz2|tbz|mp3|ogg)$") {
        # No point in compressing these
        remove req.http.Accept-Encoding;
    } elsif (req.http.Accept-Encoding ~ "gzip") {
        set req.http.Accept-Encoding = "gzip";
    } elsif (req.http.Accept-Encoding ~ "deflate") {
        set req.http.Accept-Encoding = "deflate";
    } else {
        # unkown algorithm
        remove req.http.Accept-Encoding;
    }
}
```
这段代码设置客客户端发送的 accept-encoding 头只有 gzip 和 default 两种编码,gzip 优先。

#### Pitfall – Vary:User-Agent一些应用或者一些应用服务器发送不同 user-agent 头信息,这让 varnish 为每个单独的用户保存一个单独的信息,这样的信息很多。一个版本相同的浏览器在不同的操作系统 上也会产生最少 10 种不同的 user-agent 头信息。
所以如果您不打算修改 user-agent,让他们标准 化,您的命中率将受到严重的打击,使用上面的代码做模板。### Purging and banning增加 TTL 值是提高命令率的一个好方法,如果用户访问到的内容是旧的,这样就会对您的商务照成影响。解决方法就是当有新内容提供的时候通知 varnish。可以通过两种机制 HTTP purging 和 bans。首先,我们来解释HTTP purges。* HTTP purges
HTTP purges 和 HTTP GET 请求相似,除了这是用来 purges 的。事实上您可以在任何 您喜欢的时间使用这个方法,但是大多数人使用它 purging。Squid 支持相同的机制,为了让 varnish 支持 purging,您需要在 VCL 中做如下配置:
```
acl purge {
        "localhost";
        "192.168.55.0/24";
}

sub vcl_recv {
        # allow PURGE from localhost and 192.168.55...

        if (req.request == "PURGE") {
                if (!client.ip ~ purge) {
                        error 405 "Not allowed.";
                }
                return (lookup);
        }
}

sub vcl_hit {
        if (req.request == "PURGE") {
                # Note that setting ttl to 0 is magical.
                # the object is zapped from cache.
                set obj.ttl = 0s;
                error 200 "Purged.";
        }
}

sub vcl_miss {
        if (req.request == "PURGE") {

                error 404 "Not in cache.";
        }
}
```
您可以看见,使用了新的 VCL 子程序,vcl_hit 和 vcl_miss。当您调用 lookup 时将在 缓存中查找目标,结果只会是 miss 或者 hit,然后对应的子程序就会被调用,如果 vcl_hit 的目标存储在缓存中,并且可用,我们可以修改 TTL 值。所以对于 vg.no 的无效首页,他们使用 varnish 做如下处理:
```
PURGE / HTTP/1.0
Host: vg.no
```
如果 varnish 想要丢弃主页,如是很多相同 URL 的变体在 cache 中,只有匹配的变体才会被清除。清除一个相同页面的 gzip 变体可以使用下面命令:
```
PURGE / HTTP/1.0
Host: vg.no
Accept-Encoding: gzip
```
* Bans

这是另外一种清空无效内容的方法,bans。您可以认为 bans 是一种过滤方法,您 可以禁止某些存在 cache 中存在的数据。您可以基于我们拥有的元数据来禁止。Varnish 内置的 CLI 接口就是支持 bans 的。禁止 vg 网站上的所有 png 目标代码如下:


`purge req.http.host == "vg.no" && req.http.url ~ "\.png$"`

是不是很强大?在没有被 bans 命中之前的 cache,是能够提供服务的。一个目标只被最新的 bans 检查。如果您有很多长 TTL 的目标在缓存中,您需要知道执行很多的 Bans 对性能 照成的影响。您也可以在 varnish 中添加 bans,这样做需要一点 VCL:
```
sub vcl_recv {
        if (req.request == "BAN") {
                # Same ACL check as above:
                if (!client.ip ~ purge) {
                        error 405 "Not allowed.";
                }
                purge("req.http.host == " req.http.host
                      "&& req.url == " req.url);

                # Throw a synthetic page so the
                # request wont go to the backend.
                error 200 "Ban added"
        }
}
```
这是一个实用 varnish 的 VCL 处理 ban 的方法。添加一个 ban 在 URL 上,包含它的 主机部分。