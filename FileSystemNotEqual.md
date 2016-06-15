# All File Systems Are Not Created Equal: On the Complexity of Crafting Crash-Consistent Applications 阅读笔记

故障恢复是目前系统领域的一个重要问题。其中，文件系统做了很多工作来保障它的metadata在crash的时候保持一致性。目前，很多重要的系统软件，如数据库，分布式文件系统，都是基于文件系统实现的。这些系统，往往需要保证这些应用在crash之后，也能能够保证数据的一致性。于是，这些系统依赖于文件系统所提供的一致性来实现用户态的一致性。然而，不同的文件系统，甚至同一个文件系统的不同的参数，也会对一致性有不同的定义与行为。例如，有的文件系统会交换两次写的顺序，有的文件系统不能保证重命名操作是原子的。

为了解决这些问题，作者首先提出了BOB，该工具用来测试文件系统所满足的可持久化特性。它能够测试出来一个文件系统在各种情况下所满足的一致性与原子性。在这个基础上，作者又提出了Alice，一个用于分析软件中的故障脆弱性(Crash Vulnerability)的工具，并用它找到了60个现有的广泛使用的系统的bug。

### 文件系统的可持久化属性

作者研究了ext2,ext3,ext4,btrfs,xfs,reiserfs这六个文件系统以及它们在不同的参数下的原子性和顺序保证。作者开发了BOB(Block Order Breaker)。BOB首先在用户态进行一些IO操作，然后对于一些block，选择性的写入一些数据，产生新的合法的crash后磁盘的状态。之后，BOB会执行文件系统的修复工具，并检查文件系统的各个可持久化属性是否满足。

在这之后，作者使用这些工具总结出了一个表。该表描述了各种文件系统的各种原子性和顺序方面的特征。

### ALICE(Application Level Intelligent Crash Explorer)

ALICE是一个能够用来构建多种不同crash时磁盘状态的工具，通过这个工具，可以验证一个应用程序能否在各种crash下都能狗正常工作。

ALICE的使用方法比较简单。应用程序提供一段测试脚本用来验证该应用程序是否正常工作。ALICE则被用来模拟在系统CRASH后，文件系统中的文件的所有可能的状态。测试脚本则测试在这些情况下应用程序是否都能通过测试。

### APM(Abstract Persistence Models)

APM是一个用于定义文件系统中各种操作的原子性和顺序的模型，通过APM我们可以知道到底哪种CRASH时的状态是合法的。

APM把对文件的各种操作分为5种微操作:
* write_block
* change_file_size
* create_dir_entry
* delete_dir_entry
* stdout

通过BOB所得出的各种文件系统的性质，ALICE把每一个实际操作，如write(),pwrite()等划分为很多个微操作。然后，ALICE构建了这些微操作之间的依赖图（因为文件系统有可能把操作的顺序打乱），来选出一个应用微操作的顺序，进而把这些操作应用到磁盘上。


### 软件脆弱性

作者利用Alice，分析了包括git，HDFS,ZooKeeper,LevelDB,SQLite,VMWare等软件，并发现了一共60个新的问题。这些软件，很多声称自己有完整的满足ACID的实现，如SQLite。但是Alice发现这些软件其实在Crash的时候很脆弱，并不能满足其所声称的性质。
