# Dynamic Test Generation To Find Integer Bugs in x86 Binary Linux Programs

这篇文章由Molnar, D., Li, X., & Wagner, D. 在2009年发表于Proceedings of the 18th Conference on USENIX Security Symposiu。论文提出了基于符号执行的方法动态生成测试集以分析x86 Linux下二进制程序中的整数相关漏洞。


文章首先介绍了整数相关的漏洞，包括了整数溢出、长度转换、有符号无符号转换等产生的漏洞都会造成操作系统的安全性漏洞。


作者基于符号执行方法，提出了一种新的算法用于查找无符号和有符号转换漏洞，通过记录程序中的类型引用信息以检测无符号有符号数的使用情况。同时，作者扩展了过去符号执行的方法，将其扩展到了查找integer overflow、onderflow、width conversion和signed/unsigned conversion等漏洞，文章中也分析了在分析大的多媒体程序时的扩展性效率问题。


动态测试集的每次迭代过程中，通过选择一个较高价值测试用例，利用符号执行产生一系列约束，记录程序输入值和中间值，利用Valgrind二进制分析框架分析在约束下是否存在一个解从而判断是否存在bug。随后，文章具体的分析了查找各种漏洞的具体技巧。


作者实现了一个原型工具SmartFuzz，用于对Linux下的二进制程序进行整数bug分析，同时也建立了一个网站metafuzz.com，记录了SmartFuzz查找到的相关漏洞报告。截止文章发表的2009年时，SmartFuzz工具在EC2平台上通过2,361,595个测试集，分析了mplayer多媒体播放器、exiv2图像转换库和ImageMagick等程序，在864小时内查找到了77个不同的整数bug，成本相当于$2.24每个bug。

最后，作者提出一个完整的测试策略需要同时利用白盒和黑盒的测试方法，利用动态测试发现静态分析中的bug误报和漏报，同时本文的方法也具有一定的可扩展性。