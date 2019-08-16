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
