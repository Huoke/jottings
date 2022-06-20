<h1 align="center">Windows Driver Model(WDM)</h1>

>注意：
本节是对WDM驱动程序指导，WDM驱动程序不再是推荐的驱动程序模型。有关选择驱动模型相关指导请参见: Choosing a driver model

微软为了支持驱动开发者编写对所有Windows系列操作系统源码兼容的设备驱动程序，引入了Windows 驱动程序模型(WDM). 因此遵循WDM规则的内核模式驱动程序都称为WDM驱动程序。

所有WDM驱动程序必须执行一下操作:
- include Wdm.h, 而不是Ntddk.h (请注意，Wdm.h是Ntddk.h的子集)
- 被设计成为一个总线(bus) 驱动程序、函数驱动程序或过滤器驱动程序，就像Types of WDM Drivers中所描述的那样。
- 创建设备对象
- 支持即插即用
- 支持电源管理
- 支持 Windows Management Instrumentation(WMI)

# 1.1、你应该编写WDM驱动程序吗?
如果你正在编写新的驱动程序，请考虑使用内核模式驱动程序框架(KMDF). KMDF提供了比WDM接口更容易使用的接口。

如果要讲WDM驱动程序插入非WDM驱动程序堆栈，请不要使用WDM模式。请阅读特定于设备的Microsoft提供的驱动程序的文档，以确定新驱动程序必须如何与Microsoft提供的驱动程序交互。详细信息参见：Device and Driver Technologies

<h1 align="center">Types of WDM Drivers</h1>
这里有三种类型WDM驱动：bus driver、function driver 和filter driver。

- 总线驱动程序驱动单个I/O总线设备，并提供独立于设备的每个插槽功能。总线驱动程序还检测并上报连接到总线的子设备。
- 函数驱动程序驱动单个设备。
- 过滤器驱动程序为设备、一类型设备或总线过滤I/O请求。


To develop Device and Driver Installation Reference , need these headers:
- cfg.h
- cfgmgr32.h
- newdev.h

这项技术的开始指导书：
Device and Driver Installation:
这部分解释了设备和驱动怎么样在windows系统上安装。如果你不熟悉设备和驱动安装的process，我建议你先看看 Roadmap for Device and Driver Installation。
你可能也想读一读 Overview of Device and Driver Installation，它是这一process跟更高的概览and its components。


<h2 align="center">Roadmap for Device and Driver Installation</h1>
在windows上安装设备和驱动需要一下步骤：

- 1、学习和了解windows中设备和驱动安装的基本原理。
   你需要理解windows操作系统体系中驱动和设备安装的原理。它将帮助你做出适当的设计决策，使您能够简化开发过程。
   参见：Overview of Device and Driver Installation。
- 2、学习了解驱动 packages and their components。
   一个驱动包包含的所有组件，在windows下安装并支持设备所需要的组件。它包含了INF文件和INF文件所有的依赖文件。
   参见：Driver Package and INF Files。
- 3、为你的设备和驱动创建一个驱动包。
   你的驱动包需要提供一个INF文件和额外的驱动文件或者一些软件组件。
   更多的知识了解请参见：Createing a Driver Package。
   案例请参见：Toaster Sample。
- 4、在开发和测试阶段Test-sign你的驱动包。
   Test-signing 参考用一个测试证书来签名一个预发布版本的驱动包在测试机上测试，特别的地方是，它还支持开发者使用自签名证书来签名驱动包，比如使用MakeCert这样的证书生成工具。此功能允许开发人员在启用驱动程序签名验证的 Windows 中安装和测试驱动程序包。
   详细的知识参见：Signing Drivers during Development and Test。
- 5、使用 Secure Boot测试你的 Preproduction-sign驱动包
   Preproduction-signing 参考使用WHQL/WHCP 预生产证书签名一个驱动包的预发布版本，以便在零售/生产系统上使用，而无需启动测试签名。
   这个功能支持开发者在一个开启了签名认证的Windows上安装测试驱动包。
   更多知识参见：Preproduction Driver Signing and Testing。
- 6、部署你的 Release-sign 驱动包。
   在测试和验证驱动程序包后，就要release-sign驱动程序包。发布签名标识驱动程序包的发布者，虽然此步骤是可选的，但出于一下原因，驱动程序包应该经过发布签名：
   - 确保驱动程序包的真实性、完整性和可靠性。Windows使用数字签名来验证发布者的身份，并验证驱动程序自发布以来没有被修改过。
   - 可以方便的自动化地安装驱动程序，为用户提供最佳的用户体验。
   - 在64位版本的Windows-vista和更高版本的Windows上运行内核模式驱动程序。
   - Playback 某些类型的下一代高级内容。 
   
   驱动程序包通过一下方式进行发布签名：
       - 通过Windows硬件兼容性程序，(适用于Windows 10/11) 或 Windows硬件认证程序(适用于Windows 8或者更早版本的操作系统)获得的WHQL版本签名。
       - 通过软件发布者证书(SPC)创建的发布签名。
   更多相关信息参见：Signing Drivers for Public Release。
- 7、部署你的驱动包。
   最后就是部署驱动包。如果你的驱动包满足Windows Hardware Compatibility Program(针对Windows 10/11)或者Windows Hardware Certification Program(针对win8或者老的windows系统)中定义的质量标准，你就能通过Microsoft Windows Update program来发布驱动包。
   有关这方便的详细知识参见： Publishing a driver to Windows Update。

这些事基本步骤，根据你个人设备和驱动程序的安装需要，可能需要执行其他步骤。

<h2 align="center">Overview of Device and Driver Installation</h2>
当Windwos操作系统将设备作为现有设备报告给操作系统时，Windows操作会自动安装没有安装驱动程序包的设备。
所有"现有设备"都会发生这种情况, 当系统启动时，当用户插入(或手动安装)一个即插即用(PnP)设备时就会发生。
例如：ACPI驱动程序和其他PnP总线驱动程序等驱动程序可以帮助Windows确定存在哪些设备。

一下步骤更详细地介绍了此过程的各个部分：
- 1、识别新设备。
- 2、选择设备的驱动程序。
- 3、安装设备的驱动程序
- 驱动选择过程概览
# 1、识别新设备



  
