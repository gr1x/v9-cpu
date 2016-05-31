# The Art of War: Offensive Techniques in Binary Analysis

这篇文章由Shoshitaishvili, Y., Wang, R., Salls, C., Stephens, N., Polino, M., Dutcher, A., Vigna, G.等发表于2016年的IEEE S&P。文章详细分析了程序的二进制漏洞分析技巧。

查找和利用二进制程序中的漏洞一直都是一项具有挑战性的任务。由于二进制中缺乏相关的数据结构和控制结构，因此二进制分析十分困难且难于扩展。然而，二进制分析的重要性却日益增加，因为在某些情况下，二进制分析是验证目标程序的唯一方法。

这篇文章首先分析了具体的二级制分析方法，静态分析方法主要有：
1. Recovering Control Flow;
2. Vulnerability Detection with Flow Modeling
3. Vulnerability Detection with Data Modeling

动态分析方法主要有：
1. Dynamic Concrete Execution
	- Coverage-based fuzzing
	- Taint-based fuzzing
2. Dynamic Symbolic Execution
	- Classical dynamic symbolic execution
	- Symbolic-assisted fuzzing
	- Under-constrained symbolic execution


基于过去的各种二进制分析方法，本文提出了一个二进制分析框架，通过系统性的归集不同的二进制分析方法，允许其他的研究人员灵活的组合这些分析方法，比较这些方法各自的优缺点，以进一步发现新的方法。

最后，作者基于DAPRA提供的二进制数据集评估了二进制分析框架的有效性。