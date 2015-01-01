# varnish 安装

>varnish 是一款最新型的网站加速器,他的任务是在web服务器之前设置内容缓存。 这可以让您的网站响应更快。我们建议您先阅读 “varnish 安装”。如果您的 varnish 已经成功 启动,您可以去阅读“varnish 指南”。

## 基本要求
安装前您需要做如下准备：

* 任意一个（Linux、FreeBSD、Solaris）比较新的、64位的操作系统；
* 拥有这个操作系统的root权限

varnish也可以安装在其他的UNIX操作系统上，不过在这些平台上没有被全面的测试。一般情况下，varnish也可以运行在如下平台：

* 32位的上面提到的操作系统；
* OS X
* NetBSD
* OpenBSD


## 安装varnish

作为开源软件,你有2种安装方式：通过二进制安装包、源码编译安装，选择哪一种看个人习惯。如果您不知道选择哪种方式，可以先通读整个文档，然后再选择一种您喜欢的方式。

在大多数操作系统上，都可以通过系统自带的包管理器来安装varnish。比较典型的如：

### FreeBSD

* 源码安装：

    `cd /usr/ports/varnish && make install clean`
* 安装包安装：

    `pkg_add -r varnish`# varnish 安装

>varnish 是一款最新型的网站加速器,他的任务是在web服务器之前设置内容缓存。 这可以让您的网站响应更快。我们建议您先阅读 “varnish 安装”。如果您的 varnish 已经成功 启动,您可以去阅读“varnish 指南”。

## 基本要求
安装前您需要做如下准备：

* 任意一个（Linux、FreeBSD、Solaris）比较新的、64位的操作系统；
* 拥有这个操作系统的root权限

varnish也可以安装在其他的UNIX操作系统上，不过在这些平台上没有被全面的测试。一般情况下，varnish也可以运行在如下平台：

* 32位的上面提到的操作系统；
* OS X
* NetBSD
* OpenBSD


## 安装varnish

&emsp;&emsp;作为开源软件,你有2种安装方式：通过二进制安装包、源码编译安装，选择哪一种看个人习惯。如果您不知道选择哪种方式，可以先通读整个文档，然后再选择一种您喜欢的方式。

在大多数操作系统上，都可以通过系统自带的包管理器来安装varnish。比较典型的如：

### FreeBSD

* 源码安装：

    `cd /usr/ports/varnish && make install clean`
* 安装包安装：

    `pkg_add -r varnish`

###CentOS/RedHat
&emsp;&emsp;我们尽力维护最近的版本提供 RPMS(EL4&EL5) on SourceForge。
Varnish 被包含在 EPEL 资源库,不幸的是 varnish2.0.6→varnish2.1.X 有语法改变。这样 我们就不能更新 EPEL 中的 varnish 版本,EPEL 中最后的版本是 2.0.6.
### DEBIAN/UBUNTU
&emsp;&emsp;varnish 已经发布了 DEBINA 和 UBUNTU 的包,只需要使用一下命令就可以安装,注 意:这样安装可能不是最新的版本。

`apt-get install varnish`
### OTHER SYSTEMS&emsp;&emsp;您最好使用源码安装,参照“源码编译安装“如果您已经完成了安装,您可以阅读“varnish 指南“,“varnish 指南”比安装更 加有趣。
￼## 通过源码包编译安装如果没有您系统适用的二进制包,或其他原因您想要通过源码编译,请参考以下步骤:* 首先需要使用 svn 命令下载源码。如果您没有这个命令,您需要先在您的系统上安 装 subversion 软件,笔者使用的是二进制安装的 subversion。
* 下载当前的 2.1 分支版本	`svn co http://varnish-cache.org/svn/branches/2.1 `
* 下载当前的开发源码	`svn co http://varnish-cache.org/svn/trunk`
	### DEBIN/UBUNTU 系统环境下的依耐关系要在 DEBIN/UBUNTU 系统下成功编译安装 varnish,需要先安装以下软件包: 

* autotools-dev
* automake1.9
* libtool
* autoconf
* libncurses-dev
* xsltproc
* groff-base
* libpcre3-dev

	使用以下命令安装上面所有的包
	
	`￼￼￼sudo apt-get install autotools-dev automake1.9 libtool autoconf libncurses-dev xsltproc ￼groff-base libpcre3-dev`
### RedHat/CentOS 系统环境下的依耐关系如果您是 RedHat/CentOS 系统想安装 varnish,您需要安装以下软件包: 
* automake
* autoconf
* libtool
* ncurses-devel
* libxslt
* groff
* pcre-devel
* pkgconfig

	以下是在配置好 yum 包管理器的情况下运行
	
	`yum install automake autoconf libtool ncurses-devel libxslt groff pcre-devel pkgconfig`
	### 配置和编译
  下一步,配置,配置时会检查软件的依耐关系是否满足。
  ```
cd varnish-cachesh autogen.sh
sh configure
make
```
这里的 configure 命令可以使用一些参数,但是大多数情况下和上面一样不使用参 数。您可以忘记这一步,因为几乎所有的参数都可以在 varnish 运行的时候添加。在您安装 varnish 前,您可以运行自动测试,然后去喝杯茶,因为它会花费您几分钟。


`(cd bin/varnishtest && ./varnishtest tests/*.vtc)`

如果您发现有一两个失败的时候,请不用担心,某些测试项目对时间比较敏感(您可以告诉我们,我们来解决这个问题)。如果出现很多错误,或者 b00000.vtc 测试也失败的话,应该是发生了某些可怕的错误,如果您不解决这个错误的话,接下来将一事无成。### INSTALLING  最后的考验,您是否有一颗勇敢的心:
  `make install`
varnish 将被安装到/usr/local 目录,varnishd 可执行文件将被安装到/usr/local/sbin/, 默认配置文件被安装到/usr/local/etc/varnish/default.vcl。## 技术支持关于直接获取 varnish 团队的支持,我们会在时间允许的情况下尽量多的帮助大家, 并试图尽可能的简化这一过程。
但是请在联系我们前花一点时间,整理您的想法和明白表达您的问题,如果您只告 诉我们“我的 varnish 不能工作了”,而没有进一步的信息,这将是毫无意义的。### IRC CHANNEL最直接的获得我们帮助的方法就是加入我们的 IRC 通道。
`#varnish on server irc.linpro.no`
含义:时区是欧洲+美国
如果您要发表您的 VCL 或者相关文档,可以使用 http://gist.github.com/### MAILING LISTS打开或关闭邮件列表请访问 MailMan http://lists.varnish-cache.org/mailman/listinfo 
### COMMERCIAL SUPPORT商业支持,请联系 sales@varnish-software.com
## 提交BUG
Varnish 的 debug 工作就像一个棘手的禽兽,有可能一些数据的拥挤,导致成千上万的线程核心转储。
如果您发现一个 bug,花一点时间收集 bug 的相关信息是十分必要且必须的,我们可以通过您收集的信息来修复这个 bug。