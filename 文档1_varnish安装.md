# varnish 安装

>varnish 是一款最新型的网站加速器，他的任务是在web服务器之前设置内容缓存。 这可以让您的网站响应更快。我们建议您先阅读 “varnish 安装”。如果您的 varnish 已经成功 启动，您可以去阅读“varnish 指南”。

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

&emsp;&emsp;作为开源软件，你有2种安装方式：通过二进制安装包、源码编译安装，选择哪一种看个人习惯。如果您不知道选择哪种方式，可以先通读整个文档，然后再选择一种您喜欢的方式。

在大多数操作系统上，都可以通过系统自带的包管理器来安装varnish。比较典型的如：

### FreeBSD

* 源码安装：

    `cd /usr/ports/varnish && make install clean`
* 安装包安装：

    `pkg_add -r varnish`

###CentOS/RedHat
&emsp;&emsp;我们尽力维护最近的版本提供 RPMS(EL4&EL5) on [SourceForge](http://sourceforge.net/projects/varnish/files/)。

&emsp;&emsp;Varnish 被包含在 [EPEL](http://fedoraproject.org/wiki/EPEL) 资源库，不幸的是 varnish2.0.6→varnish2.1.X 有语法改变。这意味着我们不能更新 [EPEL](http://fedoraproject.org/wiki/EPEL) 中的 varnish 版本，因为EPEL中最后的版本是 2.0.6.

&emsp;&emsp;在EPEL6发布时，varnish会变成2.1的版本。
### DEBIAN/UBUNTU
&emsp;&emsp;varnish 已经发布了 DEBINA 和 UBUNTU 的包，只需要使用一下命令`sudo apt-get install varnish`就可以安装，注 意：这样安装可能不是最新的版本。


### 其他系统&emsp;&emsp;您最好使用源码安装，参照“[源码编译安装](#compiling-varnish-from-source)”如果您已经完成了安装，您跳过文档剩下的部分，开始阅读更有意思的“varnish 指南”部分。
￼## [通过源码包编译安装](id:compiling-varnish-from-source)&emsp;&emsp;如果没有您系统适用的二进制包，或其他原因您想要通过源码编译，请参考以下步骤:* 首先需要使用 svn 命令下载源码。如果您没有这个命令，您需要先在您的系统上安装[subversion](http://subversion.tigris.org/) 软件，笔者使用的是二进制安装的 subversion。
* 下载当前的 2.1 分支版本	`svn co http://varnish-cache.org/svn/branches/2.1 `
* 下载当前的开发版源码	`svn co http://varnish-cache.org/svn/trunk`
	### DEBIN/UBUNTU 系统环境下的依耐关系&emsp;&emsp;要在 DEBIN/UBUNTU 系统下成功编译安装 varnish，需要先安装以下软件包: 

* autotools-dev
* automake1.9
* libtool
* autoconf
* libncurses-dev
* xsltproc
* groff-base
* libpcre3-dev

&emsp;&emsp;使用以下命令安装上面所有的包
	
&emsp;&emsp;`￼￼￼sudo apt-get install autotools-dev automake1.9 libtool autoconf libncurses-dev xsltproc ￼groff-base libpcre3-dev`
### RedHat/CentOS 系统环境下的依耐关系&emsp;&emsp;如果您是 RedHat/CentOS 系统想安装 varnish，您需要安装以下软件包: 
* automake
* autoconf
* libtool
* ncurses-devel
* libxslt
* groff
* pcre-devel
* pkgconfig

&emsp;&emsp;以下是在配置好 yum 包管理器的情况下运行
	
&emsp;&emsp;`yum install automake autoconf libtool ncurses-devel libxslt groff pcre-devel pkgconfig`
	### 配置和编译
&emsp;&emsp;下一步，配置，配置时会检查软件的依耐关系是否满足。
  ```
cd varnish-cachesh autogen.sh
sh configure
make
```
&emsp;&emsp;这里的 configure 命令可以使用一些参数，但是大多数情况下和上面一样不使用参数。您可以忘记这一步，因为几乎所有的参数都可以在 varnish 运行的时候添加。&emsp;&emsp;在您安装 varnish 前，您可以运行回归测试，然后去喝杯茶，因为它会花费您几分钟。


&emsp;&emsp;`(cd bin/varnishtest && ./varnishtest tests/*.vtc)`

&emsp;&emsp;如果您发现有一两个失败的时候，请不用担心，某些测试项目对时间比较敏感(您可以告诉我们，我们来解决这个问题)。如果出现很多错误，特别是 b00000.vtc 测试也失败的话，应该是发生了某些可怕的错误，如果您不解决这个错误的话，将无法进展下去。### 安装  最后的考验，您是否有一颗勇敢的心:
  `make install`
&emsp;&emsp;varnish 将被安装到/usr/local 目录，varnishd 可执行文件将被安装到/usr/local/sbin/， 默认配置文件被安装到/usr/local/etc/varnish/default.vcl。&emsp;&emsp;现在你可以进入到下一个文档：[文档2_varnish教程.md](https://github.com/xuanskyer/varnish-docs/blob/master/%E6%96%87%E6%A1%A32_varnish%E6%95%99%E7%A8%8B.md)