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



## File System XV6 ~ Ext4



## FIle System Durability & Crash Recovery

crush consistancy



![fsck](https://ws1.sinaimg.cn/large/006tNc79gy1g24uxeu0j8j30v70u0aja.jpg)

### Journaling

Using cache is good for perfermance, but how  to deal with the crash?

XV6 logging only ensure safety, and only allow one log transaction per time.

It's like a lock with a log, keep track of the file while maintianing consistancy

## Journaling and ext3



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



