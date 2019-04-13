# Homework 2

鄢新

515110910013

---

#### What is the difference between user-level ISA and system-level ISA?

System-level ISA 的指令需要在 ring 0 模式下使用，比如 load cr3，是特权指令。而 user-level ISA 是普通运行指令。

#### What is memory addressing mode? How many modes are there (please describe them)? Why not just use one?

Memory addressinng mode 是指从虚拟地址映射到物理地址的方式。

- Real Mode
  - :  physicaladdr = 16 \* segment + offset
- External address to 20-bits
  - Segment +IP
- Protected Mode
  - 32-bits & 64-bits
  - Page table

计算机发展的过程中内存逐渐增大，需要的位数越来越多，在不同的位数之下需要有不同的寻址模式（历史原因）。


#### Use your own word (and figures, if you want) to describe the process from power-on to BIOS end (just before kernel starts)

Power-on -> BIOS searching at the bigining of disk sector for boot code -> when searched, load them to the memory at 0x7c00 -> run boot code -> change from real mode to protected mode with ljmp -> enter main.c

##### 	–What is the usage of "ljmp  \$(SEG_KCODE<<3),  $start32"?

jmp to start32 and set code segment to 0 at a single line

##### 	–What do you learn from the A20 problem?

1. 写代码的时候不能为了省事儿钻空子，这样在环境更新之后就很容易出问题。
2. 计算机发展的过程中因为历史原因，需要做很多的向后兼容需要做很多的妥协。