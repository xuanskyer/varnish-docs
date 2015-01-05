# varnish 教程这个教程是为了系统管理员管理varnish缓存而编写的。读者应该知道怎么样配置他的网站或应用服务，并且对HTTP协议有基本了解。此外，读者还应该已经使用默认配置运行了varnish。

教程分为多个短章，每个章节作为一个独立的章节。祝你好运~

## Backend servers(后端服务器)&emsp;&emsp;varnish 有一个概念叫做“后端服务器”或者叫“源服务器”，一个后端服务器将 提供 varnish 加速的内容。&emsp;&emsp;我们的第一个任务就是告诉 varnish 在哪里可以找到他要的内容。使用您喜欢的文 本编辑器打开 varnish 默认的配置文件，如果您是源码安装的配置文件可能在 /usr/local/etc/varnish/default.vcl ， 如 果 您 是 二 进 制 包 安 装 的 配 置 文 件 可 能 在 /etc/varnish/default.vcl.&emsp;&emsp;配置文件的最顶端如下:	```
	￼# backend default {	￼# .host = "127.0.0.1";	￼# .port = "8080";	￼#}```
&emsp;&emsp;我们取消前面的注释，并且把 8080 端口改成 80 端口。如下```
￼backend default {	￼.host = "127.0.0.1";
	￼.port = "80";}
```
&emsp;&emsp;现在，这块配置定义了一个 varnish 默认访问的后端服务器，当 varnish 需要从后端 服务器获取内容的时候，它就会访问自己(127.0.0.1)的 80 端口。&emsp;&emsp;Varnish 可以定义多个后端服务器而且您可以通过定义多个后端服务器达到负载均 衡的目的。&emsp;&emsp;现在我们完成了基本的 varnish 配置，我们可以在 8080 端口上启动 varnish，并做 一些基本的测试。
## Starting Varnish(启动 varnish)
&emsp;&emsp;假设varnishd在您的环境变量中，您可能需要运行pkill varnishd来确定varnish没有运行。然后使用 root 执行下面的命令。`￼varnishd -f /usr/local/etc/varnish/default.vcl -s malloc，1G -T 127.0.0.1:2000 -a ￼0.0.0.0:8080`&emsp;&emsp;我添加了一些选项，现在来详细分析他们:* `-f /usr/local/etc/varnish/default.vcl`	这个 –f 选项指定 varnishd 使用哪个配置文件。* `-s malloc，1G`	这个 –s 选项用来确定 varnish 使用的存储类型和存储容量，malloc表示只使用内存存储方式。 1G 规定分配的内存为1G。* `-T 127.0.0.1:2000`	Varnish 有一个内置的基于文本的管理接口，启动它的话可以在不停止 varnish 的情况下来 管理 varnish。您可以指定管理软件监听哪个接口。当然您不能让全世界的人都能访问您的 varnish 管理接口，因为他们可以很轻松的通过访问 varnish 管理接口来获得您的 root 访问权 限。我推荐只让它监听本机端口。如果您的系统里有您不完全信任的用户，您可以通过防火墙规则来限制只有root用户可以访问 varnish 的管理端口。* `-a 0.0.0.0:8080`	这一句的意思是制定 varnish 监听所有 IP 发给 8080 端口的 http 请求，如果在生产环境下，您应该让 varnish 监听 80，这也是默认的。&emsp;&emsp;现在您的 varnish 已经在运行了，现在我们来验证它是否工作正常，在浏览器中输入 http://127.0.0.1:8080/ ，您应该可以看见您的 web 程序在这里运行。&emsp;&emsp;使用 varnish 后，web 应用程序是否加速，取决于一些原因。如果您的程序的每个会话都使用 cookies（很多PHP和java应用不管是否需要似乎都发送一个会话cookie）或者您使用权限认证机制，这样 varnish 就不能缓存更多的数据， 我们暂时忽略这个问题，等到“[实现高命中率](#Achieving-a-high-hitrate)”这节的时候我们再继续讨论这个问题。&emsp;&emsp;想要知道 varnish 对您的网站做了什么，请查看 logs。
## Logging in varnish(记录数据)&emsp;&emsp;Varnish一个真正的特色功能就是它如何记录数据的。使用内存段代替普通的日志文件， 当内存段使用完以后，又从头开始，覆盖最旧的记录。这样就可以非常快的记录数据，，并且不需要磁盘空间。&emsp;&emsp;缺点就是您没有把数据写到磁盘上，可能会消失。
&emsp;&emsp;Varnishlog 这个程序可以查看 varnish 记录了哪些数据。Varnishlog 给您生成原始的日志，包括所有的事件。我一会给您演示。&emsp;&emsp;在运行了 varnish 的终端窗口上，运行 varnishlog 这个命令。 
&emsp;&emsp;您可以看见如下显示```￼0 CLI - Rd ping￼0 CLI - Wr 200 PONG 1277172542 1.0
```&emsp;&emsp;这是检查 varnish 的主进程是否正常，如果看见这就说明一切OK

&emsp;&emsp;现在您去浏览器重新访问您的 web 程序，您将看到如下信息:```
￼11 SessionOpen 	c 127.0.0.1 58912 0.0.0.0:8080￼11 ReqStart 		c 127.0.0.1 58912 595005213￼11 RxRequest 	c GET￼11 RxURL 		c /￼11 RxProtocol 	c HTTP/1.1￼11 RxHeader 		c Host: localhost:8080￼11 RxHeader 		c Connection: keep-alive```
第一列是任意的数，它用来定义请求，相同的号码代表相同的 HTTP 请求。第二列是信息标记，所有的日志都带有一个标记(tag)，它说明什么样的活动被记录。Rx 表示 varnish 收到数据，Tx 表示 varnish 发送数据。第三列代表数据是从哪里传出或者传入的，是从 c(client)还是 b(backend)。 
第四列是被记录的数据。现在，您可以过滤掉相当多的 varnishlog，这些基本的选项，您需要知道:	-b \\只显示 varnish 和 backend server 之间的日志，当您想要优化命中率的时 候可以使用这个参数。	-c \\和-b 差不多，不过它代表的是 varnish 和 client 端的通信。	-i tag \\只显示某个 tag，比如“varnishlog –i SessionOpen”将只显示新会话，注 意，这个地方的 tag 名字是不区分大小写的。	-I \\通过正则表达式过滤数据，比如“varnishlog -c -i RxHeader -I Cookie”将 显示所有接到到来自客户端的包含 Cookie 单词的头信息。	-o \\聚合日志请求 ID现在varnish正常运行了，是时候把它调整到 80 端口上了。


## 分配内存

分配多少内存给你的varnish上一个比较棘手的任务。有很多情况要考虑：


* 你的热数据量有多大。对于一个门户或资讯站点来说，这个大小可能是：首页数据量的大小与首页所链接的所有子页面数据量大小之和。

* 生成一个数据对象的成本有多大。有时只对图片做一段时间的缓存或者不做任何缓存是更有道理的，如果从服务端直接获取数据很容易，或者你有内存大小限制的话。

* 使用`varnishstat`或其他工具观察LRU的数量。如果有很多[LRU](http://baike.baidu.com/link?url=__Xq3RHZoTSg4UPrEL_aOF-UK38yd9M66pYEwDgGBAXZni1IshhHK0ZI6KFMDo52fEiYwWePAQvw1GekXJNuFK)活动导致varnish一直在将缓存内容置换出内存，这时你应该考虑增加你的内存大小。

## 设置varnish监听80端口&emsp;&emsp;如果您的程序正常运行，没有问题，我们就可以把 varnish 调整到 80 端口运行。先关闭 varnish`pkill varnishd`&emsp;&emsp;然后停止您的 web 服务器，修改 web 服务器配置，把 web 服务器修改成监听 8080 端口，然后修改 varnish 的 default.vcl 和改变默认的后端服务器端口为 8080.&emsp;&emsp;启动您的 web 服务器，然后启动 varnish:`varnishd -f /usr/local/etc/varnish/default.vcl -s malloc，1G -T 127.0.0.1:2000` 
&emsp;&emsp;我们取消了-a 选项，这样 varnish 将监控默认端口，启动后，检查您的 web 程序是否正常。
## varnish配置语言-VCL
&emsp;&emsp;Varnish 有一个很棒的配置系统，大部分其他的系统使用配置指令，让您打开或者关闭 一些开关。Varnish 使用区域配置语言，这种语言叫做“VCL”(varnish configuration language)，当有请求时，varnish就把 VCL 转换成二进制代码并执行。&emsp;&emsp;VCL 文件被分为多个子程序，不同的子程序在不同的时间里执行，比如一个子程序在接到请求时执行，另一个子程序在接收到后端服务器传送的文件时执行。&emsp;&emsp;varnish 将在不同阶段执行它的子程序代码，因为它的代码是一行一行执行的，不存在优先级问题。随时可以调用这个子程序中的功能并且当他执行完成后就退出。 
&emsp;&emsp;如果到最后您也没有调用您的子进程中的功能，varnish 将执行一些内建的 VCL 代码，这些代码就是 default.vcl 中被注释的代码。 
&emsp;&emsp;你的99%的配置工作会在vcl_recv 和 vcl_fetch 这两个子进程中完成。 
### vcl_recv&emsp;&emsp;vcl_recv(当然，我们在字符集上有点不足，它适用在unix)在请求的开始被调用， 在接收、解析后，决定是否响应请求，怎么响应，使用哪个后台服务器。&emsp;&emsp;在 vcl_recv 中，您可以修改请求，比如您可以修改 cookies，添加或者删除请求的 头信息。&emsp;&emsp;注意 vcl_recv 中只有request对象`req`是可用的。 ### vcl_fetch 
&emsp;&emsp;vcl_fetch在一个文件成功从后台获取后被调用，通常他的任务就是改变 response headers，触发 ESI 进程，在请求失败的时候轮询其他服务器。&emsp;&emsp;在 vcl_fetch 中一样有request对象`req`可用，同时还有一个backend的response对象`beresp`可用。beresp 将会包含后端服务器的 HTTP 的头信息。### actions最常用的action动作如下：* **pass** 
	当调用pass动作后，这个请求和接下来的响应将直接转发到后端服务器，而不会被缓存。pass 可以在 vcl_recv 和 vcl_fetch 中使用。* **lookup** 
	当一个请求在 vcl_recv 中被 lookup 后，varnish 将从缓存中提取数据，如果缓存中没有数据，将被设置为 pass，不能在 vcl_fetch 中设置 lookup。* **pipe** 
	pipe 和 pass 相似，都要访问后端服务器，不过当进入 pipe 模式后，在 此连接未关闭前，后续的所有请求都发到后端服务器(这句是我自己理解后简化的，有能力的朋友可以看看官方文档，给我提修改建议)。* **deliver** 
	请求的目标被缓存，然后发送给客户端* **esi** 
	ESI-process the fetched document(我理解的就是 vcl 中包换一段html代码)### Requests，Responses and objects 
在 VCL 中，有 3 个重要的数据结构。来自客户端的请求，来自服务端的响应，存储在内存中的内容对象。其中，你需要知道以下结构：* `req` 请求目标，当 varnish 接收到一个请求时，req 对象就被创建了并常驻， 你在 vcl_recv 中的大部分工作，都是在 req 对象上展开的。	
* `beresp` 后端服务器响应对象，它包含服务端返回的头信息，你在 vcl_fetch 中的 大部分工作都是在 beresp 对象上开展的。	
* `obj` 被缓存对象，大多数情况下，它是只读的，常驻在内存中，`obj.ttl `的值可写，其他的是只读的。### 运算符

VCL 支持以下运算符，请阅读下面的例子:* `=`	赋值运算符* `==`	对比* `~` 	匹配，在 ACL 中和正则表达式中都可以用 
* `!` 	非* `&&` 	逻辑与
* `||` 	逻辑或### 示例1 – 操作HTTP头
我们想要移除我们服务器上/images 目录下的所有对象的cookie:```
sub vcl_recv {
  if (req.url ~ "^/images") {
    unset req.http.cookie;
  }
}
```现在，当这个请求在操作后端服务器时，将不会有 cookie 头，这里有趣的行是 if语句，它匹配 request对象中的URL，如果匹配这个操作，那么头信息中的 cookie 就会被删除。 
### 示例 2 – 操作响应对象`beresp`如果req对象属性`req.url`匹配特定的规则，我们就修改来自backend端的对象属性`beresp.ttl`的值:```sub vcl_fetch {
   if (req.url ~ "\.(png|gif|jpg)$") {
     unset beresp.http.set-cookie;
     set beresp.ttl = 3600s;
  }
}
```
###  EXAMPLE3-ACLs通过关键字`vcl`创建一个访问控制列表。通过匹配操作，你可以配置一个客户端的 IP 地址对应一个vcl配置：```
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

&emsp;&emsp;现在您的 varnish 已经正常运行，我们来看一下 varnish 是如何工作的，这里有些工具可以帮助您。### Varnishtop&emsp;&emsp;Varnishtop 工具读取共享内存的日志，实时显示最频繁更新的日志列表。合理使用 –I，-i，-X 和-x 参数选项，它可以按照您的要求显示请求的内容，客户端，浏览器等其他日志里的信息。	varnishtop -i rxurl \\您可以看到客户端请求的 url。	Varnishtop -i txurl \\您可以看到哪个backend后端服务器的访问最频繁 。	Varnishtop -i Rxheader –I Accept-Encoding \\可以看见从客户端接收到的头信息中那个 Accept-Encoding头最多。 
### Varnishhist&emsp;&emsp;Varnishhist 工具读取 varnishd 的共享内存段日志，生成一个实时的柱状图，显 示最后 N 个请求的处理情况。这个 N 的值和纵坐标尺度会显示在左上角，横坐标尺度是log对数。“`|`”表示缓存命中，“`#`”表示没有命中。### Varnishsizes&emsp;&emsp;Varnishsizes 和 varnishhist 相似，不同的是它显示的是缓存对象的大小，而不是了完成 请求的时间。这可以让你更好的从整体上观测您的服务当前的缓存有多大。### Varnishstat&emsp;&emsp;Varnish 有很多计数器，我们计数丢失率，命中率，存储信息，创建线程，删除对 象等，几乎所有的操作。Varnishstat 展现这些计数器，这在调试 varnish 的时候很有用。&emsp;&emsp;有一个程序可以定期轮询 varnishstat 的数据并生成好看的图表。其中一个项目叫做 Munin。Munin 可以在 http://munin-monitoring.org/找到。在 varnish 的源码中有 munin 插件。
## [实现高命中率](id:Achieving-a-high-hitrate)
&emsp;&emsp;现在 varnish 已经正常运行了，您可以通过 varnish 访问到您的 web 应用程序。如果 您的 web 程序在设计时候没有考虑到加速器的架构，那么您可能有必要修改您的应用程序 或者 varnish 配置文件，来提高 varnish 的命中率。varnish只在绝对安全的情况下才会缓存你的数据。因此为了让您理解varnish究竟何时缓存，如何缓存一个页面，我会提供一些工具为你做引导，你会发现它们很有用。
&emsp;&emsp;既然这样，您就需要一个工具用来观察在您和 web 服务器之间 飞奔的HTTP 头信息。如果你可以直接操作varnish，那最简单的 varnish 工具是使用 varnishlog 和 varnishtop，但有时客户端的工具也是有用的，下面是我经常使用的工具。### Varnistop&emsp;&emsp;您可以使用 varnishtop 确定哪些 URL 经常命中后端。`varnishtop –i txurl` 就是一个基本的命令。您可以通过阅读“统计”章节了解其他示例。### Varnishlog&emsp;&emsp;当您需要鉴定哪个 URL 被频繁的发送到后端服务器，您可以通过 varnishlog 对请求 做一个全面的分析。`varnishlog –c –o /foo/bar` 这个命令将告诉您所有来自客户端(-c)的请求中(-o)包含”/football/bar” 字段的请求。### Lwp-request&emsp;&emsp;Lwp-request 是 www 库的一部分，使用 perl 语言编写。它是一个真正的基本程序， 它可以执行 HTTP 请求，并给您返回结果。我主要使用两个程序，GET 和 HEAD。&emsp;&emsp;Vg.no 是第一个使用 varnish 的站点，他们使用 varnish 相当完整，所以我们来看看 他们的 HTTP 头文件。我们使用 GET 请求他们的主页:```
￼￼￼$ GET -H 'Host: www.vg.no' -Used http://vg.no/￼GET http://vg.no/￼Host: www.vg.no￼User-Agent: lwp-request/5.834 libwww-perl/5.834￼200 OK￼Cache-Control: must-revalidate￼Refresh: 600￼Title: VG Nett - Forsiden - VG Nett￼X-Age: 463￼X-Cache: HIT￼X-Rick-Would-Never: Let you down￼X-VG-Jobb: http://www.finn.no/finn/job/fulltime/result?keyword=vg+multimedia￼￼Merk:HeaderNinja￼X-VG-Korken: http://www.youtube.com/watch?v=Fcj8CnD5188￼X-VG-WebCache: joanie￼X-VG-WebServer: leon
```&emsp;&emsp;OK，我们来分析它做了什么。GET 通过发送 HTTP 0.9 的请求，它没有主机头，所以我使用-H 选项添加一个主机头，-U 打印请求头信息，-s 打印返回状态，-e 答应返回头信息，-d 丢弃正文内容部分。这里我们不关心正文内容，只关心头信息。&emsp;&emsp;如您所见 VG 的头文件中有相当多的信息，比如 X-RICK-WOULD-NEVER 是 vg.no 定 制的信息，他们有几分奇怪的幽默感。其他的内容，比如 X-VG-WEBCACHE 是用来调试错误的。&emsp;&emsp;因此检查一个站点是否对特定的URL设置了cookies，可以使用下面的命令:`GET -Used http://example.com/ | grep ^Set-Cookie`
### Live HTTP Headers&emsp;&emsp;这是一个 firefox 的插件，live HTTP headers 可以查看您发送的和接收的 http 头。软 件在 https://addons.mozilla.org/en-US/firefox/addon/3829/下载。或者 google“Live HTTP headers”。### The Role of HTTP headers
&emsp;&emsp;随着HTTP请求和响应头一起发送的，还有一组源数据（metadata）头信息。varnish会根据这些头信息决定是不是缓存、缓存多久这些内容。

要注意的是，在理解这些头信息时varnish实际上是把自己当做是web服务器的以一部分。The ratinonale being that both are under your control.

**IETF**和**RFC 2616**  中对 术语**surrogate origin cache** 的定义都不够完善，所有有很多varnish的用法可能会你期望的不一样。

让我们来看看一些你需要注意的重要头信息：

* **Cache-Control**	Cache-control 指示缓存如何处理内容，varnish 关心 max-age 参数，并使用这个参数 计算每个对象的 TTL 值。	“cache-control:nocache” 这个参数已经被忽略，不过您可以很容易的使它生效。所以要确保在设置头信息中 cache-control 的时候带上 max-age，您可以参照下面内容管理框架**Varnish Softwares drupal**的例子:


	`$ GET -Used http://www.varnish-software.com/|grep ^Cache-Control`

	`Cache-Control: public， max-age=600`

* **Age**	Varnish 添加了一个 age 头信息，用来指示对象已经被保存在 varnish 中多长时间了。 您可以在 varnish 中找到 Age 信息:	`varnishlog -i TxHeader -I ^Age`
* **Pragma**

	HTTP 1.0 服务允许发送头信息“Pragma: nocache”。Varnish 默认忽略这个头信息。你可以很容易通过VCL添加这个头信息支持。

	在vcl_fetch内添加：

	```
if (beresp.http.Pragma ~ "nocache") {
   pass;
}
```

* **Authorization**

	如果varnish检测到一个授权报文头将通过请求，不做处理。如果这不是你想要的，你可以取消这个设置。


* **Overriding the time-to-live(ttl) **

	有时候后端服务器会误操作。通过设置，可以更容易在varnish中对TTL重新赋值，以此解决后端backend的麻烦。	您需要在 VCL 中使用 beresp.ttl 定义您需要修改的对象的 TTL:
	```
sub vcl_fetch {
    if (req.url ~ "^/legacy_broken_cms/") {
        set beresp.ttl = 5d;
    }
}
```* Normalizing your namespace	有些站点访问的主机名有很多，比如 http://www.varnish-software.com， http://varnish-software.com，http://varnishsoftware.com 所有的地址都指向同一站点。但是 varnish 不知道，varnish 会缓存每个主机名的每个页面。您可以减少这种情况， 通过修改 web 配置文件或者通过以下 VCL:	```
if (req.http.host ~ "^(www.)?varnish-?software.com") {
  set req.http.host = "varnish-software.com";
}
```


### 更多提高命中率的方式
下面的章节应该可以给你提供一些方法来进一步提高你的命中率，特别是在cookies的章节。

#### CookiesVarnish 接收到后端服务器返回的头信息中有 Set-Cookie 信息的话，将不缓存。 同样的当客户端发送一个 Cookie 头的话，varnish 将不做缓存，直接发送到后端服务器。 
这样的话有点过度的保守，很多站点使用 Google Analytics(GA)来分析他们的流量。GA 设置一个 cookie 跟踪您，这个 cookie 只被客户端上的一个 js 脚本使用，而对服务器没用。对于一个 web 站点来说，忽略一般 cookies 是有道理的，除非您是访问一些关键部分。这个 VCL 的 vcl_recv 片段将忽略除/admin/访问之外的所有cookies：```
if ( !( req.url ~ ^/admin/) ) {
  unset req.http.Cookie;
}
```
这很简单。

不过，如果您需要做更复杂的，比如删除多个 cookies中的某一个，这就变得有点困难。很不幸，varnish没有比较好的工具来处理。我们只能使用正则表达式来完成这个工作。如果您熟悉正则表达式，您很容易理解下面的工作，否则我建议您找找相关资料学习一下。我们来看看 内容管理框架**Varnish Softwares drupal**是怎么工作的，我们有一些cookies供 GA 或类似的工具使用。这些 cookies 只被js脚本设置和使用。Varnish 和内容管理框架**Varnish Softwares drupal**不需要这些cookies，而 varnish 会因为这些 cookies 而不做页面缓存，所以我们使用VCL取消这些多余的 cookies。下面的 VCL 将会丢弃所有以下划线“`_`”开头的cookies。
```
// Remove has_js and Google Analytics __* cookies.
set req.http.Cookie = regsuball(req.http.Cookie， "(^|;\s*)(_[_a-z]+|has_js)=[^;]*"， "");
// Remove a ";" prefix， if present.
set req.http.Cookie = regsub(req.http.Cookie， "^;\s*"， "");
```
下面的例子将删除所有名字叫 COOKIE1 和 COOKIE2 的 cookies:

```
sub vcl_recv {
  if (req.http.Cookie) {
    set req.http.Cookie = ";" req.http.Cookie;
    set req.http.Cookie = regsuball(req.http.Cookie， "; +"， ";");
    set req.http.Cookie = regsuball(req.http.Cookie， ";(COOKIE1|COOKIE2)="， "; \1=");
    set req.http.Cookie = regsuball(req.http.Cookie， ";[^ ][^;]*"， "");
    set req.http.Cookie = regsuball(req.http.Cookie， "^[; ]+|[; ]+$"， "");

    if (req.http.Cookie == "") {
        remove req.http.Cookie;
    }
}
```
这个例子是来自 varnish wiki 的，在那里你还可以找到更神奇的VCL的例子。
#### Varyweb服务器发送的**vary**头决定，根据何种形式生成多样化的 HTTP 对象。这非常有用，比如Accept-Encoding 头。当一个服务器分发一个“Vary:Accept-Encoding”给 varnish。这就告诉Varnish 需要 对来自客户端的每个不同的 Accept-Encoding各自进行缓存。因此，如果客户端只接收 gzip 编码，varnish 将不会对deflate编码的页面做响应。问题就是，Accept-Encoding 域包含很多编码方式，如果一个浏览器发送的:
`Accept-Encodign: gzip，deflate`
另一个浏览器发送的:`Accept-Encoding:: deflate，gzip`
Varnish 会对两个不同的 accept-enconding 头编码做不同的缓存，规范accept-enconding头，可以统一缓存的版本。下面的 VCL 代码可以是 accept-encoding 头标准化:
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
这段代码设置客客户端发送的 accept-encoding 头只有 gzip 和 default 两种编码，gzip 优先。

#### Vary的坑:User-Agent一些应用或者一些应用服务器发送不同 user-agent 头信息，这让 varnish 为每个不同的访问源（一般指浏览器）保存一个单独的信息。这样会有大量冗余。一个版本相同的浏览器在不同的操作系统 上也会产生最少 10 种不同的 user-agent 头信息。
所以如果您确实要使用vary的 user-agent，请确保规范好这个头，否则您的缓存命中率将受到严重的打击。上面的代码可以做为一个规范模板。#### Purging and banning提高命令率最好的一个好方法是增加缓存对象的 TTL 值，但是，正如你意识到的那样，在这个属于Twitter的时代，提供过期的内容对您的公司业务不是什么好事。解决方法就是当有新内容提供的时候通知 varnish。可以通过两种机制 **HTTP purging** 和 **bans**。首先，我们来解释HTTP purges。* **HTTP purges**
HTTP purges 和 HTTP GET 请求相似，只不过请求方式是 **PURGE** 。事实上您可以在任何时候使用这个方法，但是大多数人使用它来做数据清除（purging）。Squid 支持相同的机制，为了让 varnish 支持 purging，您需要在 VCL 中做如下配置:
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
如你所见，我们使用了新的 VCL 子程序，**vcl_hit** 和 **vcl_miss**。当您调用 lookup 时将在 缓存中查找目标，结果只会是 miss 或者 hit，然后对应的子程序就会被调用，在 vcl_hit 中缓存对象是可用的，我们可以修改 TTL 值。所以为了使 vg.no 的首页缓存失效，他们使用 varnish 做如下处理:
```
PURGE / HTTP/1.0
Host: vg.no
```
varnish 会丢弃主页缓存，即使有对同一 URL 有多个版本的缓存，也只有匹配的版本才会被清除。要清除页面的 gzip 版本缓存，可以使用下面命令:
```
PURGE / HTTP/1.0
Host: vg.no
Accept-Encoding: gzip
```
* **Bans**

这是另外一种清空无效内容的方法，bans。您可以认为 bans 是一种过滤方法，您可以禁用缓存某些特定的数据。您也可以禁用其他任何我们有的源数据（metadata）的缓存。Varnish 内置的 CLI 接口就是支持 bans 的。禁止 vg.no 网站上的所有 png 缓存的目标代码如下:


`purge req.http.host == "vg.no" && req.http.url ~ "\.png$"`

这真的很强大！ bans检测在缓存命中后，数据返回之前被调用。一个目标对象只会使用最新的 bans 操作。如果您有很多长 TTL 的目标在缓存中，您需要知道执行很多的 Bans 对性能照成潜在的影响。您也可以在 varnish 中通过HTTP.xxx添加 bans，这样做需要一点 VCL:
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
这是一个实用 varnish 的 VCL 处理 HTTP的ban方式请求。在URL的host部分添加一个 ban 操作。     

#### Edge Side Includes
Edge Side Includes 是一种将一段网页嵌入其他网页的语言。可以把它想象成是在HTTP上工作的HTML包含语句。

在大多数网站上，很多内容是在网页之间共享的。在每个页面上重复生成这些内容太浪费了，ESI试图做到让您为每个片段语句单独设置缓存策略。

在varnish中，我们仅仅实现了ESI的一小部分。在2.1版本中，我们有三种ESI表示方式：

* esi:include
* esi:remove
* <!–esi ...–>

Content substitution based on variables and cookies is not implemented but is on the roadmap.

**esi:include示例：**

让我们来看看它如何使用。下面是一个简单的输出日期的CGI脚本：

```
#!/bin/sh
echo 'Content-type: text/html'
echo ''
date "+%Y-%m-%d %H:%M"
```
现在我们创建一个HTML文件，并包含一个ESI嵌套语句：

```
<HTML>
	<BODY>
		The time is: <esi:include src="/cgi-bin/date.cgi"/>
		at this very moment.
	</BODY>
</HTML>
```
为了让ESI起作用，你需要在VCL中采用如下方式激活ESI进程：

```
sub vcl_fetch {
    if (req.url == "/test.html") {
       esi;                      /* Do ESI processing               */
       set obj.ttl = 24 h;       /* Sets the TTL on the HTML above  */
    } elseif (req.url == "/cgi-bin/date.cgi") {
       set obj.ttl = 1m;         /* Sets a one minute TTL on        */
                                 /*  the included object            */
    }
}
```
**esi remove示例：**

关键词“**remove**”可以让你清除输出。当ESI无效时，你可以用它作为一个备用，就像这样：

```
<esi:include src="http://www.example.com/ad.html"/>
<esi:remove>
  <a href="http://www.example.com">www.example.com</a>
</esi:remove>
```
**<!–esi ... –>示例：**
这是一个特别的结构，允许HTML用ESI标记而不做渲染，当页面被处理时，ESI进程会移除开始（“<!-esi”）和结束（“->”）标签，同时处理里面的内容。如果页面没有ESI进程处理，则会变成一个HTML/XML的注释标签：

`<!--esi`

`<p>Warning: ESI Disabled!</p>`

`</p>  -->`

这样就确保ESI标记如果没被处理，也不会妨碍最终的HTML的渲染。

## 后端服务器高级配置
在某些时刻您需要 varnish 从多台服务器上缓存数据。您可能想要 varnish 映射所有的 URL 到一个单独的主机或者不到这个主机。这里很多选项。我们需要引进一个 java 程序进出 php 的 web 站点。假如我们的 java 程序使用的 URL 开 始于/JAVA/我们让它运行在 8000 端口，现在让我们看看默认的 default.vcl:
```
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}
```
我们添加一个新的 backend:

```
backend java {
    .host = "127.0.0.1";
    .port = "8000";
}
```

现在我们需要说明这个特定的 URL 被发送到哪里:
```
sub vcl_recv {
    if (req.url ~ "^/java/") {
        set req.backend = java;
    } else {
        set req.backend = default.
    }
}
```
这真的很简单，让我们停下来并思考一下。正如您所见，可以通过任意形式来定义如何选择要使用的backend后端。您想发送移动设备的请求到不同的后端?没问题：`if (req.User-agent ~ /mobile/)...`，应该是这样。。。

## Directors（分发器）

您可以把多台 backends 聚合成一个组，这些组被叫做 分发器（directors）。这样可以增强性能和容错率。您可以定义多个 backends 并分组在同一个 分发器。
```backend server1 {
    .host = "192.168.0.10";
}
backend server2{
    .host = "192.168.0.10";
}
```
现在我们创建一个 director:
```
director example_director round-robin {
	{
        .backend = server1;
	}
	# server2
	{
        .backend = server2;
	}
	# foo
}
```
这个 分发器 是一个循环遍历的 分发器。它的含义就是 分发器 使用循环遍历的方式把请求分发到 backend。还有一中随机发送的分发器，它会以一种，你应该猜到了，随机的方式分发请求。但是如果您的一个服务器挂掉了呢？varnish 能否指导所有的请求发送到健康的后端？当然可以，这时健康检查（Health checks）就登场了。
## Health checks
让我们设置一个包含两个backend和 health checks的分发器。
首先定义backends：
```
backend server1 {
  .host = "server1.example.com";
  .probe = {
         .url = "/";
         .interval = 5s;
         .timeout = 1 s;
         .window = 5;
         .threshold = 3;
    }
  }
backend server2 {
   .host = "server2.example.com";
   .probe = {
         .url = "/";
         .interval = 5s;
         .timeout = 1 s;
         .window = 5;
         .threshold = 3;
   }
 }
 ```
 这里新增了**probe**配置。varnish会用这个probe来检测每一个backend是否健康。它的内部参数有：

参数字段 |说明
--- | ---
url | 需要varnish监测的URL
interval | 轮询的周期
Timeout | 等待多长时间探针超时
Window | varnish会运行一个动态变化的窗口，里面显示5个监测结果
Threshold | 设置最新的监测结果中多少次是正常的，则表示这个backend是健康的
initial | How many of the of the probes a good when Varnish starts - defaults to the same amount as the threshold.
    	现在我们定义 分发器:
```
director example_director round-robin {
	{
    	.backend = server1;
    }
    # server2
    {
        .backend = server2;
    }
}
```
您的站点在您需要的时候使用这个 分发器，varnish 将不会发送请求流量给标志为不健康的主机。如果所有的 backends都挂了，varnish仍可以使用旧的内容提供服务。参照“Misbehaving servers”获得更多的信息。

请注意，varnish会保持所有加载的VCLs是有效的。varnish会合并相同的probe，所以如果你做很多VCL加载，那要小心不要改变probe的配置。关闭VCL配置，将使probe被丢弃。

## 服务器异常（ Misbehaving servers ）

Varnish 的一个关键特色就是将你从 web 和应用服务器异常的灾难中解救出来。
### 优雅模式（Grace mode）当几个客户端请求同一个页面的时候，varnish 只发送一个请求到后端服务器， 然后让那个其他几个请求挂起等待返回结果，返回结果后，复制请求的结果发送给客户 端。如果您的服务每秒有成千上万的点击率，那么这个队列是庞大的，没有用户喜欢等待服务器响应。有一种解决的办法是使用过期的 缓存内容 给用户提供服务，为此我们需要告诉varnish保存缓存内容即使在 TTL的时间之后， 保存所有 cache 中的内容在 TTL 过期以后 30 分钟内不删除，使用以下 VCL:
```
sub vcl_fetch {
  set beresp.grace = 30m;
}
```
这样Varnish 还不会使用过期的目标给用户提供服务，为了真正的使用这么过期内容提供服务，所以我们在request中启用，我们配置在cache 过期后的 15 秒内，使用旧的内容提供服务:
```
sub vcl_recv {
  set req.grace = 15s;
}
```
你会考虑为什么要多保存过去的内容 30 分钟?当然，如果你启用了`Health checks`，并检测到backend是健康的，就可以设置更长保存时间:

```
if (! req.backend.healthy) {
   set req.grace = 5m;
} else {
   set req.grace = 15s;
}
```

### 神圣模式（saint mode）
有时候，服务器很古怪，他们发出随机错误，您需要通知 varnish 使用更加优雅的 方式处理它，这种方式叫神圣模式(saint mode)。神圣模式允许您抛弃一个后端服务器返回的页面缓存，然后尝试从另一个后端服务器或者 旧的缓存数据中获取数据。

让我们看看 VCL 中如何开启这个 功能的:```
sub vcl_fetch {
  if (beresp.status == 500) {
    set beresp.saintmode = 10s;
    restart;
  }
  set beresp.grace = 5m;
}
```
当我们设置 beresp.saintmode 为 10 秒，varnish 在 10 秒内将不会为这个URL访问后端服务器。如果有一个备用列表，当重新执行此请求时，列表有其他的后端有能力提供此服务内容，varnish 会尝试请求他们，当您没有可用的后端服务器，varnish 将使用它过期的 cache 提供服务内容。它真的是一个救生员。
### 上帝模式（god mode）
还没实现。 :-)
## 进阶篇
上述教程已经涵盖了基本的varnish内容。如果你通读了上面的内容，那你应该已经掌握了使用varnish的技能。
这里是对教程中没有涵盖到的内容的一个简短概述章节。
### 更多VCL
相比较我们目前讨论的VCL内容，VCL有着更加复杂功能。还有更多的子程序和功能操作我们没有讨论到。关于完整的VCL用法，可以参考帮助页：ref:reference-vcl.

### 使用嵌入式C扩展varnish
你可以使用嵌入式C扩展varnish。如果你这么做了，当心别玩砸了~。c代码运行在 varnish缓存进程内部，如果您的代码出现错误，缓存将会崩溃。我看到的第一个使用嵌入C的用法是记录日志到 syslog:

```
# The include statements must be outside the subroutines.
C{
        #include <syslog.h>
}C

sub vcl_something {
    C{
         syslog(LOG_INFO， "Something happened at VCL line XX.");
    }C
}
```

### Edge side Includes
Varnish 可以缓存通过不同页面聚合而来的页面，这个缓存有单独的缓存策略，如果您的网站有一个列表显示您最受欢迎的 5 篇文章。这个列表可以作为一个代码段被保存下来，并在任何其他页面中引用。使用得当，可以大大提高您的命中率，减少服务器的负载。ESI 代码如下:

```
<HTML>
	<BODY>
		The time is: <esi:include src="/cgi-bin/date.cgi"/>
		at this very moment.
	</BODY>
</HTML>
```
ESI 在 vcl_fetch 中通过 `esi` 关键字处理：
```
sub vcl_fetch {
    if (req.url == "/test.html") {
        esi;  /* Do ESI processing */
    }
}
```

## varnish故障排查
有时候 varnish 会出错，为了使您知道到底发生了什么事，有很多地方您可以检查。 varnishlog， /var/log/syslog/，var/log/messages 所有这些地方varnish可能会留下一些线索来查明 varnish 到底抽什么风~
### varnish无法启动
有些时候，varnish 不能启动。这里有很多 varnish 不能启动的原因，通常我们可以 观看/dev/null 的权限和是否其他软件占用了端口。使用 debug 模式启动 varnish，然后观看发生了什么:
启动varnish：
`# varnishd -f /usr/local/etc/varnish/default.vcl -s malloc，1G -T 127.0.0.1:2000  -a 0.0.0.0:8080 -d`
留意 `-d` 选项，它将给您更多的信息来排查错误。让我们看看如果其他程序暂用了 varnish 的端口，它将显示什么：
```

# varnishd -n foo -f /usr/local/etc/varnish/default.vcl -s malloc，1G -T 127.0.0.1:2000  -a 0.0.0.0:8080 -d
storage_malloc: max size 1024 MB.
Using old SHMFILE
Platform: Linux，2.6.32-21-generic，i686，-smalloc，-hcritbit
200 193
-----------------------------
Varnish HTTP accelerator CLI.
-----------------------------
Type 'help' for command list.
Type 'quit' to close CLI session.
Type 'start' to launch worker process.


```
现在 varnish 开始运行，在 debug 模式中，只有主程序在运行，cache 现在还没有启动，现在您在终端中使用“start”命令来让主程序开启 cache 功能```
start
bind(): Address already in use
300 22
Could not open sockets
```
在这里，我们发现一个问题。Varnish 要使用的端口被 HTTP 使用了。如果这些没有帮组，尝试strace 或 truss，或者在IRC上联系我们。
### varnish崩溃
varnish崩溃
### Varnish gives me Guru meditation
首先到varnish的日志中查找相关日志项。那会给你一些线索。
### varnish未缓存
请参考“[实现高命中率](#Achieving-a-high-hitrate)”这章。