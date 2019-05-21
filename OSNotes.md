# OS Notes

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



### Read-Write Lock



## Bugs