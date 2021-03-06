# Mars Code
这篇发表于ACM通信的文章详细介绍了"好奇号"火星车的代码设计原则，正如文章副标题所写，冗余软件（及硬件）确保好奇号顺利抵达目的地并安装设计师要求正常运行。

由于好奇号火星车的特殊任务，因此其软件开发团队采取了一套不常见的设计措施：

- 基于风险的代码编写规则：在开发前分析该领域内的设计错误，根据其原因进行分类，并生成一份主要问题清单。在此基础上制定代码编写规则，其中，仅包含风险相关的规则，而非代码风格规则。同时，代码中所有assert在飞行期间都是启用状态，而非常见的测试后禁用的方式。

- 基于代码的审查工具：传统的代码审查方式是同行代码审核。但传统的同行代码审查之适用于数量较少的代码，若代码量到了百万行，则这种方法将不可行。作者应用了四种代码分析器，并利用脚本对输出统一格式化，利用开发的Scrub工具进行审查。代码审查和意见、报告回复等均离线完成，每一模块的代码审核仅需要一次会议来解决争议并达成一致意见。

- 模型检查：作者利用了逻辑模型检查，因为火星车代码大量使用了多线程，容易发生争用并导致异常。Spin模型检查器对程序断言的有效性、零死锁情况等进行检查，并验证以线性时序逻辑表述的代码是否可行等。在大多数情况下，模型检查都可以成功标识出软件中的并发性错误。

为了保证好奇号火星车顺利登陆火星并执行任务，作者也采用了硬件和软件冗余措施，关键软硬件都有备份副本。如对于着陆功能，其主CPU上运行着软件的主版本，备份CPU上运行着软件简化版。若主CPU在着陆过程中发生意外，备份CPU就能自动接管。当然，最后的实践登陆过程中，次备份机制未被调用。

