# Paper Overview for "Non-scalable locks are dangerous"

"Non-scalable locks are dangerous"这篇文章发表于2012年的Ottawa Linux Symposium，针对Linux中的锁机制进行了分析测试，验证了其因为ticket spin locks等非扩展性锁机制的使用，Linux的性能在多核机器上大幅下降。

众所周知，扩展性锁机制在多核竞态时将导致系统性能下降，通过修改代码，避免出现串行化操作，就可以避免锁机制的使用，但是对于内核的复杂度来说，消除所有可能导致争用的瓶颈点几乎是不可能的。

因此，当前的Linux系统中依然存在这扩展性锁机制。然而，spin lock这种非扩展性锁机制在N个处理器环境下，一个处理器进行unlock释放该锁，将导致其余所有等待处理器的cache被invalidate，并同时开始读取该spin lock，因为进行串行化读取，需要几百个周期的延迟，其penality到与CPU核数成正比，复杂度O(N)。

本文分析了非扩展性锁机制，并与扩展性锁机制进行对比，其主要贡献在于：
- 利用FOPS/MEMPOP/PFIND/EXIM四种benchmark，对锁机制在Linux下进行性能评估，证明了即使如只包含较短代码段的spin lock，虽然关键代码段很短，但是依然容易造成多核下性能衰减，随着cpu核数的增加，锁机制的代价迅速增加。
- 基于硬件缓存一致性协议，建立了一个基于马尔科夫链的评估模型，解释了非扩展性锁机制在系统运行中的具体表现；
- 通过将Linux中的spin lock体会为MCSlocks可扩展性锁机制，其性能在多核处理器上得到了极大提升；同时对不同的可扩展性锁机制MCS/GLH/HCLH等进行了评测，发现不同的扩展性锁机制之间，系统性能差异不大。

