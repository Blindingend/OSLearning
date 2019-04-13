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



## IPC



## Exception



## Interrupt



## System Calls



## I/O



## File System XV6 ~ Ext4

