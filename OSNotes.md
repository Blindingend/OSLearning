#  OS Notes

[TOC]

## 1. Introduction& 8 Important Problems in
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

##2. OS Structures

Kernel 分类：Monolithic, Microkernel, Exokernel, Hybrid

- Monolithic 宏内核：OS在kernel space运行，在supervisor mode 运行 （Linux BSD)
- Microkernel 微内核：low-level address space management, thread management, and inter-process communication (IPC)等技术，（Mach  L4 kernel）
- Hybrid kernel 复合内核：Windows ，NT kernel

![1554882220939](http://ww1.sinaimg.cn/large/006tNc79ly1g3zkb0hds7j30ux07wdhi.jpg)





##3. PC Programming & Booting

PC 有不同的工作模式，不同的工作模式对应着不同的情况，不同的权限与需求，与不同的寻址模式。

- 定义：Real Mode, Protected Mode, Virtual-8086 Mode, IA-32e Mode, System Management Mode
- 名词解释：
  - PE，Protection Mode的开关
  - SMI，System Management Interrupt，系统管理中断，使系统进入SMM的特殊中断
  - SCI，System Control Interrupt，系统控制中断，专门用于ACPI电源管理的一个IRQ，需要OS支持
- Real-Address Mode是在bootloader阶段的运行模式，存在时间很短。内存寻址方式和8086相同，没有虚拟地址，由16位段寄存器（CS/SS/DS/ES）乘以0x10当做基地址，再加上16位偏移地址形成20位的物理地址，最大寻址空间1MB，最大分段64KB。
- Protected Mode是最常用的模式，内存寻址采用32位段和偏移量（现在64位），最大寻址4GB，最大分段4GB。它提供了一些增强多工和系统稳定性的设计，比如内存保护，分页系统，虚拟内存等，防止程序随意访问地址，也拥有更大的内存访问空间。
- Virtual-8086 Mode是在保护模式下运行的虚拟实模式环境，寻址方式与实模式相同。
- IA-32e Mode是64位操作系统运行的模式，具有兼容模式和64位模式两种子模式。兼容模式可以运行在32位兼容环境，但不能运行虚拟8086程序；64位模式则完全处理64位指令
- System Management Mode有独立于OS的地址空间，用来执行电源管理或系统安全方面的指令。

Protected Mode

Real Mode

![img](https://images2015.cnblogs.com/blog/809277/201601/809277-20160109142207371-383459687.png)

Logical Address : 程序运行时的逻辑地址

Linear Address(Virtual Address): 由 Logical Address 翻译过来。如果没有分页机制的话就直接是物理地址了，如果有分页机制的话就是虚拟地址，还需要经过 Page Translation 变成物理地址。

**Logical Address to Linear Address**

Logical Address(segment:offset) = 0x01:0x0000ffff

根据 segment 去相应的 DT 中查到对应的索引，然后看 table entry 中的 Flag 确认自己是否有访问权限。如果有访问权限的话取出 Base, 与 offset 进行运算之后得到 Linear Address。

而所查询的表有两种 , **GDT**(Global Discription Table) **LDT** (Local Discription Table), 作用都是存放某个运行在内存中的程序的分段信息。 GDT 全局可见，内核程序的短信息存储与此。 LDT 则是每个在内存中的程序都包含的。





##4. XV6 & VM





##5. Processes

- 逻辑地址 - 线性地址 - 物理地址的转换

  ![1556339216237](http://ww4.sinaimg.cn/large/006tNc79ly1g3zkaxmhl5j30mp05fdgh.jpg)

  ![1556339247367](http://ww3.sinaimg.cn/large/006tNc79ly1g3zke5spzjj30nq0eeabl.jpg)

- Boot阶段的主要工作：

  - 进入Protected Mode，开启Segment，配置GDT (ljmp指令)
  - 打开页表 (CR0_PG)，初始化entrypgdir，将kernel映射到高地址和低地址 (0x80100000, 0x100000)

- 一些概念

  - Process：设计进程的出发点：程序比处理器多；程序并不需要持续占用处理器；因此可以在不用的时候释放资源；进程是资源分配的最小单位，拥有一个地址空间和若干线程；包含program counter, stack, data section等数据块

  - Thread：线程的抽象定义：停止活动并在稍后某个 时刻恢复活动所需要的最小状态，通常是一些程序寄存器；线程是程序运行的最小单位

  - Context：xv6中的数据结构，包含edi、esi、ebx、ebp、eip五个field

  - Address spaces：地址空间，原则上和线程是独立的概念；可以在同一个地址空间从一个线程切换到另一个线程；也可以在不同地址空间切换

  - Process State：进程状态，一般有new，running，waiting，ready，terminated；xv6中为UNUSED, EMBRYO胚胎, SLEEPING, RUNNABLE, RUNNING, ZOMBIE

    - ENBRYO可以理解为到RUNNABLE之前的一个过渡。allocproc会在进程表中找到一个标记为UNUSED的位置。当它找到这样一个没有被使用的位置后，allocproc将其状态设置为EMBRYO，使其标记为被使用并给这个进程一个独有的pid。接下来，它尝试为进程的内核线程分配内核栈。如果分配失败了，allocproc会把这个位置的状态恢复为UNUSED并返回0来标记失败。

    ![1556342655313](http://ww4.sinaimg.cn/large/006tNc79ly1g3zkdw22y3j30kb08e0ti.jpg)

- Process Control Block，PCB

  - 每个process都会有一个PCB，内容包括Process state, Process counter, CPU registers, CPU scheduling information, Memory management information, Accounting information, I/O status information
  - xv6 中的进程数据结构

  ![1556343005224](http://ww1.sinaimg.cn/large/006tNc79ly1g3zkb1zd4wj30mr0anwgk.jpg)

- Context Switch 上下文切换

  - 调用syscall触发中断 - save当前进程的PCB - reload目标进程的PCB - 继续执行
  - 上下文切换需要保存的内容应该是存在进程对应的kernel stack中的
  - 上下文切换换栈的瞬间

  ![1556343241697](http://ww1.sinaimg.cn/large/006tNc79ly1g3zkb5l8zcj309t08ijrt.jpg)

- Process Scheduling Queue 进程调度队列

  - Job Queue：系统中所有的process
  - Ready Queue：系统中所有等待执行的process
  - Device Queue：系统中所有等待IO设备的process

- 两种Scheduler

  - Long-term scheduler / job scheduler：调度即将进入ready queue的进程，完成时间在秒/分钟级别，决定了multiprogramming的程度
  - Short-term scheduler / CPU scheduler：调度即将执行的进程，完成时间在微秒级别
  - scheduler对process的分类：IO-bound process和CPU-bound process

- Process的创建：fork，一次执行两次返回；execute，一次执行没有返回；奇妙的机制

  ![1556344453826](http://ww3.sinaimg.cn/large/006tNc79ly1g3zkdwziswj30it057mxd.jpg)





## 6. Inter Process Communication

### IPC

由于线程是同一个进程产生的，共享着堆和栈，所以共享信息比较方便，但是对于进程来说，由于并不共享堆和栈，所以需要专门的通信方式。常用的有两种方式。

- <big>**Message Passing**</big>

  

- <big>**Shared Memory**</big>

- Pipe

- Socket
- Shared memory

### LRPC - Lightweighting Remote Procedure Call



## 7. Exception



## 8. Interrupt



## 9. System Calls



## 10. I/O



## File Systems



### 11. File System XV6 ~ Ext4



### 12. FIle System Durability & Crash Recovery

crush consistancy



![fsck](https://ws1.sinaimg.cn/large/006tNc79gy1g24uxeu0j8j30v70u0aja.jpg)

#### Journaling

Using cache is good for perfermance, but how  to deal with the crash?

XV6 logging only ensure safety, and only allow one log transaction per time.

It's like a lock with a log, keep track of the file while maintianing consistancy

### 13. Journaling and ext3



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



#### Virtual Disk & VMRAM



### 14. FS:  FAT32 & NTFS

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



### 15. Flash FS



### 16. GFS & NFS

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

### 17. Scalable Lock

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




## Bug Survey



## Data Race & Deadlock

有些变量不需要被 shared 就可以定义为 thread _local 





## Virtualization:

### 22. CPU & Memory

VMM 有着不同的 Architecture

- Xen: 结构简单 
- KVM
- qemu



#### Hardware Supported CPU Virtualization

Ring 1/2 淡出历史舞台

只有 

- Ring0：kernel mode

- Ring3：user mode



#### CPU 虚拟化

##### VMCS (Virtual Machine Control Structure) 

记录 VM 的信息，类似 trap 的 trapframe，记录一组寄存器，不能直接读取寄存器，通过新的指令 vmcs read/write 来操作



#### Memory 虚拟化



### 23. I/O



## 24. Serverless Computing

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





