#  OS Notes

[TOC]

## Introduction& 8 Important Problems in
Modern Operating Systems

### What's the definition operating system?

在硬件和应用之间的都可以看做是操作系统
向上提供公共服务
向下管理资源

### 8 Important Problems

- **Scale Up**

  OS: need to evolve itself with multicore
  need to evolve for apps with multicore

  多核数据共享的开销非常的大。

- **Security & Trusty**

  Meltdown
  load key -> permission deny -> roll back, but leave key in the cache -> access cache, we may find key

- **Energy Efficiency**

  In large scale systems, power consumption is a big cost.

- **Mobility**

  Unique for Mobile OS

  - Energy Efficiency
  - Richer User Experience, But more limited resources
  - Security:More user data put into mobile OSes

- **Write Corrent (Parallel) Code**

  > *I am convinced more than ever that this type of work is very difficult, and that every effort to do it with other than the best people is doomed to either failure or moderate success at enormous cost.*
  >
  > *Edsger* *Dijkstra*
  >
  > *The Structure of the “THE” –Multiprogramming System* *1968*

- **Scale Out**

  **Distributed Systems**

  Challenges

  - System design

    What is the right interface or abstraction?

    How to partition functions for scalability?

  - Consistency

    How to share data consistently among multiple readers/writers?

  - Fault Tolerance

    How to keep system available despite node or network failures?

  - Different deployment scenarios
    Clusters
    Wide area distribution
    Sensor networks

  - Security
    How to authenticate clients or servers?
    How to defend against or audit misbehaving servers?

  - Implementation
    How to maximize concurrency?
    What’s the bottleneck?
    How to reduce load on the bottleneck resource?

    X

- **Non-Volatile Storage**

  - Needs to adapt to the new storage model

  - Non-volatility changes the way operating system manage resources

    Like crash recovery

    Recall fsck

  - Better I/O performance 

- **Virtualization**

##OS Structures



##PC Programming & Booting



##XV6 & VM



##Processes



## Inter Process Communication

由于线程是同一个进程产生的，共享着堆和栈，所以共享信息比较方便，但是对于进程来说，由于并不共享堆和栈，所以需要专门的通行方式。常用的有以下几种。

- Pipe

- Socket
- Shared memory



## Exception



## Interrupt



## System Calls



## I/O



## File Systems



### File System XV6 ~ Ext4



### FIle System Durability & Crash Recovery

crush consistancy



![fsck](https://ws1.sinaimg.cn/large/006tNc79gy1g24uxeu0j8j30v70u0aja.jpg)

#### Journaling

Using cache is good for perfermance, but how  to deal with the crash?

XV6 logging only ensure safety, and only allow one log transaction per time.

It's like a lock with a log, keep track of the file while maintianing consistancy

### Journaling and ext3



**Recovery approach**

- Synchronous meta-data update + fsck
- **Logging**
- Soft Update:基于文件系统的语义，寻求 system call 之间的关系

Journal = Log

JFS Journaling FS ≠ LFS log-structured file system

LFS 数据存储方式固定， JFS 只是多加了一个 Journal 而已

同时 open 的 transition 只能有一个，一个 transaction 里可能有多个写操作。

Journaled data 代表着 log 区域包含着 meta data 和 data，每次都要写两遍，很浪费。

Copy on write 让后继的 transaction 能够写前面写过但是尚未提交的 block

每个 syscall 会告诉 log 自己大概需要多少 space

先更新 data 再更新 metadata

Ordered Mode（只写一次 data，metadata 写两次）

1. data -> home
2. Meta -> Journal
3. Commit -> Jounal
4. meta -> home

先写 data，再把 metadata 写入 Journal，再单独 commit metadata。

how to solve conflict for overwrite olddata?

when creating new files, it's not a big deal since we can allocate new disk blocks. But when modify old data, it get some risk. 但是可以写到新的 block 再改 inode 里的指针。这样也不会对旧数据产生影响。会对 locality 产生影响，但是不如正确性重要。

It's a trade off.



All or nothing 的根源是磁盘写 4k 能够保证一次写入 All or nothing



数据 reuse， metadata 的 block 删了之后再被 reuse，crash 之后 roll back，删除的操作能 roll back，但是更改过的数据已经回不来了。



### Virtual Disk & VMRAM



### FS:  FAT32 & NTFS

#### FAT32

FAT表可以理解为一堆 link list，每个节点是一个 cluster

与 inode 相比数据结构本身就不一样，如果是跳着读性能会比较差，所以适合顺序读的应用场景



FAT32 长文件名的 support 很蛋疼，找长文件名的文件的性能很差

ExFAT 改进了，使用了文件名的哈希 



#### NTFS

NTFS 将很小的文件直接存储在 MFT 的对应的 record 里

MFT 本身最少占用 12.5% 的空间

Homework：NTFS 的 hard link 到底是怎么记的



summary blcok 是用来记录反向映射，类似于 NTFS 的 MFT 里的 Entry 里记录 Parent Directory 一样。目的是提高性能，但是带来的问题就是需要保证一致性，多处同步的更新。



### Flash



### GFS & NFS

GFS 是谷歌针对自己应用的需求设计的文件系统



Feature：

1. Component **failures** are more **normal** 
2. Files are **huge**
3. Most **appending** rather than overwrite







Large sequential writes

高并发，所以bandwidth 比 latency 更重要



chunk 就是很大的 Block



Client ask Single Master for metadata, and then ask chunk server with metadata for chunkdata



Heartbeat 的 Message 里同时包含着 chunk 的 metadata 的更新，因为短消息和长消息的 overhead  差不多，索性多发一点信息。

GFS `/foo/bar` 整个都是文件名，总共只有一层，没有 inode，好处是找起来方便。坏处是修改目录名或者删目录都得遍历一边

Operation log，存取 metadata 的 log

dsfdsfGFS 是少有的大规模的使用三备份的

Chunk size: 64 MB





## Lock

### Scalable Lock

#### Non-scalable locks

Non-scalable locks leads to performance collapse when adding a few more cores

Ticket lock can guarantees lock is given in order, but still non_scalable

请求的时候拿到 next ticket 的号并把号码+1，每次之鞥呢拿到符合自己号码的，每次 release 的时候会吧 current++，给自己下一个号码的 acquire 到。



多核 CPU 每个 CPU 有自己的 cache，将多个 CPU cache 上的内容保持同步花费巨大。

#### Scalable locks

MCS 锁则是每个 CPU 维护一个锁的链表，每次申请的时候看一下自己前面有没有，如果有的话就把自己加到链表尾部，waiting = 1；如果没有的话就直接那所。

释放的时候看看自己是不是最后一个，是的话就直接将 mcs_lock 置 NULL，不是的话就把后一个 waiting 置 0，把锁给后一个。



### Read-Write Lock

为了提高读写操作的并发量而设计出来的锁，基本逻辑是保证同时只有一个 Writer 或者是多个 Reader。

Writer 拿锁的时候必须确保没有任何 Reader 和 Writer

Reader  拿锁的时候必须确保没有 Writer, 但是可以有多个 Reader，具体的上限看 CPU 核数。

#### RW lock scalability

Problem:

- Need to wait for message (current #reader)
- Need to send message (next reader)

Ideal 1 **GOLL**： Reduce time of waiting for massages

![image-20190531144201607](http://ww2.sinaimg.cn/large/006tNc79ly1g3khu69wsqj30dk0ek0ue.jpg)

Waiting domain is split into pieces. Reduce remote messages.

Idea 2 **BR Lock**: Reduce meesage

![image-20190531144417845](http://ww4.sinaimg.cn/large/006tNc79ly1g3khwi53kej30d20cwgma.jpg)

No reader messages if there's no writer

### 实时协同编辑和 OT 算法



#### 一、实时协同编辑的概念和原理

实时协同编辑，通俗来讲，是指多人同时在线编辑一个文档，且当一个参与者编辑文档的某处时，这个修改会立即同步到其他参与者的计算机上。归纳起来，需要下面几个步骤：

1. 计算出当前参与者对文档做出的修改，并发送到服务器
2. 在服务器端，对所有参与者的修改进行合并以及冲突处理
3. 将合并之后的结果返回到所有参与者的计算机上
4. 将光标移动到正确的位置

由于没有锁的机制，当多个参与者在编辑同一处内容时，便可能出现冲突，这个时候就需要通过一定的算法来自动地解决这些冲突。最后，当所有改变都同步后，每个参与者计算机上所看到的文档内容应该是完全一致的。

#### 二、Operational Transformation 算法

Operational Transformation（简称 OT）是一个用于在协同编辑中自动合并多个修改的算法，它最早发表于 1989 年，随后因 Google 在其在线协作产品 Google Wave 中大量使用而流行。

在 OT 算法中，可以将对文档的修改转义成三种类型的操作（Operation）：

1. `retain(n)`：保持 n 个字符
2. `insert(s)`：插入字符串 s
3. `delete(s)`：删除字符串 s

例如，假设用户 Alice 将 `hello, tom!` 修改成 `hello, world!`，相当于产生了如下的操作序列 `operationA`：

```pseudocode
retain(7)
delete('tom')
insert('world')
retain(1)
```

注意上面代码中最后的一个 Retain 操作。实际上，Retain 操作像一个虚拟的光标，从文档的开始位置遍历，在需要修改的地方进行 Insert 或 Delete 操作，直到文档末尾。

在 Alice 编辑文档的同时，用户 Bob 也对原文档进行了修改，并产生了如下的操作序列 `operationB`：

```pseudocode
delete('hello')
insert('hi')
retain(6)
```

对应地，Bob 修改后的文档应该是 `hi, tom!`。此时，Alice 和 Bob 的修改便会产生冲突，于是我们需要对这组操作进行一定的转换（Transform）：

```pseudocode
var [operationAPrime, operationBPrime] = OT.transform(operationA, operationB)
var strAB = operationAPrime.apply('hi, tom!')       // 'hi, world!'
var strBA = operationBPrime.apply('hello, world!')  // 'hi, world!'
```

这样在经过转换操作后，Alice 和 Bob 两个人最终看到的文档便会是一致的。




## Bugs



## Data Race & Deadlock

有些变量不需要被 shared 就可以定义为 thread _local 





## Virtualization

VMM 有着不同的 Architecture

- Xen: 结构简单 
- KVM
- qemu



### Hardware Supported CPU Virtualization

Ring 1/2 淡出历史舞台

只有 

- Ring0：kernel mode

- Ring3：user mode



### CPU 虚拟化

##### VMCS (Virtual Machine Control Structure) 

记录 VM 的信息，类似 trap 的 trapframe，记录一组寄存器，不能直接读取寄存器，通过新的指令 vmcs read/write 来操作



### Memory 虚拟化







## Serverless Computing

什么叫 Serverless?

Serverless 是相对于以前的 Server 相关的

Before：用户向厂商

After：



Scale out 一台变多台

Scale up 一核变多核



Advantages

- Event-driven: stateless 

- Auto-scale

- Easier dev-ops

- Fine-grained billing



Serverless 和 Microservice 的区别，Serverless 自动 scale，短时，免维护



初始化很慢，可以把刚初始化完的保存成一个 snapshot，然后加载这个 snapshot，减少时间

Android Zygote 机制，启动之后会初始化一个 JVM，然后每个应用启动的时候都 fork 这个 JVM。



## OS Review



### 8 Important Problems

1. Scale Up:

​	Spin Lock -> MCS Lock

2. Scale Out:

​	Serverless, 和 Scale Up 相同的地方：共享的东西越少越好，不同的地方，Scale Up 是在 Cycle 这个级别，不会太多，但是 Scale Out 是毫秒级，扩展量级不一样，可能会很多。Stateless 对 Scale Out 很重要。

​	步骤，拆分成 microservice，然后让每个 microservice 尽量的 stateless. 

3. Virtualization & Mobility 

   手机的虚拟化需要考虑更多的硬件



### Kernels

kernels

micro kernel

exokernel

如何结合 exokernel 和 hypervisor，用 SRIOV 虚拟出硬件提供给 exokernel 上面的 libOS 来调度

 LRPC:一个线程下来多个 Address Space， caller 切换到 calle 只需要改 CR3 和 IP，性能好

![image-20190611112138316](http://ww2.sinaimg.cn/large/006tNc79gy1g3x1v26y5bj310m0twju5.jpg)

虚拟内存：洞(640KB-1MB, 无法访问 被浪费)

32：10 10 12

**解释why2M? **36bit

#### 两个细节

1. P2V V2P不能用在用户态，用户态有自己的页表（用户 和 内核 2个虚拟地址对应一个PA）

2. kernel不能拿用户态指针V2P

switch：换esp，switch 和 trap 和 trap frame 之间的关系？



#### 中断：

Top half, bottom half 的区别



### FS





