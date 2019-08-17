# 编译 Squid
## 目录
# 1、我应该下载那个文件获取 Squid？
这取决于你选择尝试的鱿鱼的版本。发布的当前版本列表可在[Squid Version](http://www.squid-cache.org/Versions/)上找到。每个版本都有一个发布包页面。通常您需要列出为最新版本的发布包。

必须下载Squid-x.y.tar.gz或Squid-x.y.tar.bz2格式的源文件（例如，Squid-2.6.stable14.tar.bz2）。

我们建议您首先尝试我们的镜像站点之一进行实际下载。它们通常更快。

或者，在Squid的主站点[Squid WWW site ](http://www.squid-cache.org) 和[FTP站点](ftp://www.squid-cache.org/pub/) 下载源代码文件。

上下文差异通常可用于升级到新版本。这些可以应用于补丁程序（可从[GNU FTP站点](ftp://ftp.gnu.org/gnu/patch) 或您的发行版获得）。
# 2、有预编译的二进制文件吗？
请看[SquidFaq/BinaryPackages](https://wiki.squid-cache.org/SquidFaq/BinaryPackages)
# 3、如何编译 Squid
在运行make之前，必须自己运行配置脚本。我们建议您首先调用./configure--help，并记下您需要的配置选项，以便支持您打算使用的功能。不要在您认为不需要的特性中编译。
```Shell
% tar xzf squid-2.6.RELEASExy.tar.gz
% cd squid-2.6.RELEASExy
% ./configure --with-MYOPTION --with-MYOPTION2 etc
% make
```
... 最后安装 ...
```Shell
% make install
```
Squid将默认安装到/usr/local/squid中。如果您希望安装到其他地方，请参阅--prefix选项进行配置。
## 3.1 我需要哪种编译器？
- 要编译 Squid V3，任何合适的C++编译器都可以。几乎所有的现代UNIX系统都配有预先安装好运行良好的C++编译器。
- 要编译 Squid V4和以后的版本，您需要一个兼容C++ 11的编译器。最近的UNIX发行版带有支持C++ 11的预装编译器。
如果./configure检测到这样的支持，SQUID V3.4和V3.5在编译器中自动启用C++ 11的支持。以后的 Squid 版本需要C++ 11的支持，而较早的Squid版本不支持C++11编译器，强行编译会失败。

如果您对系统的C编译器不确定，那么GNUC编译器在几乎所有操作系统中都广泛可用并提供。它也用鱿鱼做了很好的测试。如果您的操作系统没有GCC,您可以从GNU FTP站点下载。除了gcc 和 g++，您可能还需要或需要安装binutils包和一些库，这取决于您想要启用的功能集。

Clang是GCC的一个流行替代品，特别是在BSD系统上。一般来说，它也可以很好地建造鱿鱼。过去的测试和测试的其他选择是英特尔的C++编译器和Sun的SunStudio。微软Visual C++是SQUID开发人员瞄准的另一个目标，但在撰写本文（2014年4月）的时候，仍然有相当一段路要走。
> 请注意，由于clang 对原子操作的支持中有一个bug，Squid不会建立在超过3.2的clang之上。
## 3.2 我还需要什么来编译鱿鱼？
- 您将需要automake工具集来通过Makefiles编译。
- 您需要在系统上安装Perl。
- 您选择启用的每个功能也可能需要额外的库或工具来构建。
## 3.3 如何交叉编译Squid？
使用./configure选项--host 为安装squid的计算机指定交叉编译元组。Autotools手册中有一些简单的文档说明了这个和其他交叉配置选项——特别是它们的含义是非常有用的细节。此外，Squid是使用几个自定义工具创建的，这些工具本身是在构建过程中创建的。这需要一个C++编译器来生成可以在构建平台上运行的二进制文件。HOSTCXX=参数需要提供此编译器的名称或路径。
## 3.4 如何应用补丁程序或差异?
你需要补丁程序。在应用补丁之前，您可能应该复制整个目录结构。例如，如果您要从squid-2.6.stable13升级到2.6.stable14，您将运行以下命令：
```Shell
cp -rl squid-2.6.STABLE13 squid-2.6.STABLE14
cd squid-2.6.STABLE14
zcat /tmp/squid-2.6.STABLE13-STABLE14.diff.gz | patch -p1
```
- Squid-2 patches require the -p1 option.
- Squid-3 patches require the -p0 option.
补丁运行完后，你需要重新构建Squid
```Shell
make distclean
./configure [--option --option...]
make
make install
```
如果您的补丁程序不正常工作，您应该从GNU FTP站点获得一个更新的版本。
理想情况下，您应该使用操作系统附带的patch命令。
## 3.5 配置选项
配置脚本可以选择几个选项。最常用的是预先将程序安装在不同的目录：--prefix。默认安装目录是/usr/local/squid/。修改默认目录，可按下面方式：
```Shell
cd squid-x.y.z
% ./configure --prefix=/some/other/directory/squid
```
某些操作系统要求文件安装在特定目录，具体情况参见下面的操作系统特定说明，./configure选项需要正确的参数。
```Shell
--prefix=PREFIX         install architecture-independent files in PREFIX
                        [/usr/local/squid]
--enable-dlmalloc[=LIB] Compile & use the malloc package by Doug Lea
--enable-gnuregex       Compile GNUregex
--enable-xmalloc-debug  Do some simple malloc debugging
--enable-xmalloc-debug-trace
                        Detailed trace of memory allocations
--enable-xmalloc-statistics
                        Show malloc statistics in status page
--enable-async-io       Do ASYNC disk I/O using threads
--enable-icmp           Enable ICMP pinging and network measurement
--enable-delay-pools    Enable delay pools to limit bandwidth usage
--enable-useragent-log  Enable logging of User-Agent header
--enable-kill-parent-hack
                        Kill parent on shutdown
--enable-cachemgr-hostname[=hostname]
                        Make cachemgr.cgi default to this host
--enable-htpc           Enable HTCP protocol
--enable-forw-via-db    Enable Forw/Via database
--enable-cache-digests  Use Cache Digests
                        see http://www.squid-cache.org/Doc/FAQ/FAQ-16.html
```
Squid-2通常也需要这些，但现在在Squid-3中是默认的。
```Shell
--enable-carp           Enable CARP support
--enable-snmp           Enable SNMP monitoring
--enable-err-language=lang
                        Select language for Error pages (see errors dir)
```
# 4、开始构建
## 4.1 Ubantu，Debian
Ubuntu和Debian的许多版本都是常规的构建测试和单元测试，也是我们[构建厂](https://wiki.squid-cache.org/BuildFarm)的一部分，所以编译时没问题的。

Linux系统布局和Squid的默认设置是明显不同的，要将squid安装到debian/ubuntu标准文件系统位置，需要./configure选择如下设置：
```Shell
--prefix=/usr \
--localstatedir=/var \
--libexecdir=${prefix}/lib/squid \
--datadir=${prefix}/share/squid \
--sysconfdir=/etc/squid \
--with-default-user=proxy \
--with-logdir=/var/log/squid \
--with-pidfile=/var/run/squid.pid
```
当然，您可以设置您需要的自定义选项。
- 对于Debian Jesse（8）、Ubuntu Oneiric（11.10）或更旧的Squid3软件包，上述Squid标签应附加3个。
- 记住，这些只是默认值。修改squid.conf，您可以将日志指向正确的路径，而无需解决方法或修补。
同样，如果你需要构建你想要的功能，则需要第三方库的支持，请选择下面方法安装默认的包依赖项：
```Shell
aptitude build-dep squid
```
这只需要sources.list包含deb-src存储库来提取源包信息。分发包不支持的功能需要进行调查，以发现依赖包并安装它。
>通常请求的是libssl dev 以支持ssl。但是，请注意，squid-3.5与openssl v1.1+不兼容。从Debian Squeeze或Ubuntu Zesty开始，必须改用libssl1.0-dev包。这在Squid-4包中解决。
## 4.2 初始化脚本 Init Script
init.d脚本是官方debain/ubuntu包的一部分。它不直接和Squid一起出现，所以你需要从[https://alioth.debian.org/plugins/scmgit/cgi-bin/gitweb.cgi?p=pkg-squid/pkg-squid3.git;a=blob_plain;f=debian/squid.rc]()下载一份复制到/etc/init.d/squid
## 4.2 Compiling with Cygwin
