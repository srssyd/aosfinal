# Using Crash Hoare Logic for Certifying the FSCQ File System阅读笔记

文件系统是一个庞大，而又非常重要的系统。文件系统的bug，会导致数据丢失，信息不安全等重大问题。目前，有很多工作致力于解决文件系统的bug。有一些方法，如Symbolic execution，致力于减少文件系统中的bug，但是这些系统只能辅助减少bug，并不能彻底消灭文件系统中的bug。还有一些工作，如BilbyFS,试图证明文件系统的正确性。然而，它们的证明并没有考虑crash的情况，当机器crash时，这会使得数据不一致。

于是，在该论文中，作者提出了CHL(Crash Hoare Logic)，一种基于Hoare Logic的方法，能够证明一个文件系统在任意的crash 序列下的的正确性。作者基于CHL，实现了FSCQ文件系统，并证明了它的正确性。

### Crash Hoare Logic的设计

Crash Hoare Logic包含以下几个部分：``PRE``,``POST``,``CRASH``。其中，``PRE``代表在一个操作之前数据所满足的条件，``POST``代表在该操作成功以后数据所满足的条件，而``CRASH``代表当机器CRASH的瞬间数据的状态。

对于每个断言，一般用``a→<v,vs>``来表示，其中，a代表地址，v代表最后一次写入的值，vs是之前写入值的一个集合。由于IO是异步的，写操作的顺序也有可能被来打乱，所以要用vs来维护之前写入的值的集合。如果IO操作是同步的，可以直接用``a→ v``来表示。

文件系统的操作，实际会很复杂。比如pwrite操作，包含block的分配，inode的扩张，还有修改一些已有的block。如果直接进行用specification进行描述，会变得非常复杂。于是，Crash Hoare Logic引入了Logical address space的概念。通过抽象出由底层的日志系统所提供的逻辑磁盘，我们可以假设底层能够提供一个事物系统，进而降低了specification编写的复杂性。

每次系统从CRASH中恢复的时候，会进入recover中进行恢复。当然，恢复的时候也有可能CRASH。于是一个操作的``POST``，实际上有可能进入两种状态，一种是该操作恢复，还有一种是该操作成功。

### CHL的自动证明

CHL的证明分为以下两个步骤：

首先，在第一个步骤中，对于一个过程p的每一步，CHL都会产生两个证明，第一个是当前状态满足这一步的pre condition,还有一个证明就是当前的crash condition满足p的crash condition。当p执行完之后，CHL就会去证明当前的状态满足p的post condition。

在第二个步骤中，CHL会自动的去证明一些非常显然的结论。但是，有一些结论是无法自动证明的，需要开发者手动证明。

### FSCQ文件系统

FSCQ文件系统是作者利用CHL证明了其正确性的一个文件系统。

FSCQ支持和POSIX类似的文件接口。但是，FSCQ不支持硬链接，而且FSCQ也不支持FD。FSCQ依赖于FUSE来实现inode和FD之间的转化。FSC实现了FscqLog系统，用于支持transaction操作。不过，FSCQLOG只能允许同时执行一条transaction。整个FSCQ的文件系统的恢复就是基于FscqLog来实现的。FSCQ使用位图来分配，管理各种资源。FSCQ一共包含约31000行代码。

### FSCQ文件系统的性能

作者对比了在各种情况下Fscq文件系统的性能。经过对比，Fscq文件系统拥有和XV6文件系统相类似的性能，是Ext4文件系统的一半左右。

### FSCQ文件系统的缺陷
FSCQ文件系统拥有如下的缺陷：

* 正确性依赖于Haskell编译器的正确性
* 不能保证多线程的正确性
* FSCQ假设内存空间足够大。当内存不够大时会出错
* 日志使用的空间没有限制，会导致出现错误
* 性能还需要进一步的优化
