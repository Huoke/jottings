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
！\请注意，由于clang对原子操作的支持中有一个bug，Squid不会建立在超过3.2的clang之上。


## 3.2 我还需要什么来编译鱿鱼？
## 3.3 如何交叉编译Squid？
## 3.4 如何应用补丁程序或差异?
## 3.5 配置选项
