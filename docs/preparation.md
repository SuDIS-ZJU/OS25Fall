# OS 实验导读

欢迎来到《操作系统》课程实验。按照培养方案，你已经完成了《计算机系统原理》硬件课程的学习，了解了指令集架构基本知识和计算机系统的基本组成。现在，我们将进入软件部分，动手实现一个运行在 RISC-V 架构下的简易操作系统内核。

OS 实验主要考察工程实践能力，核心要点是 **按照 RISC-V 指令集规范，以 Linux 为参考** 完成自己的操作系统内核。这需要你 **具备阅读大型项目的英文文档和源代码，理解相关内容并动手实现** 的能力，下列资料将陪伴你的整个实验过程：

- [RISC-V Ratified Specifications](https://riscv.org/specifications/ratified/)
    - [ISA Specifications](https://github.com/riscv/riscv-isa-manual/)
        - 非特权级手册：The RISC-V Instruction Set Manual Volume I: Unprivileged ISA
        - 特权级手册：The RISC-V Instruction Set Manual Volume II: Privileged Architecture
    - Non-ISA Hardware Specifications
        - SBI 规范：[RISC-V Supervisor Binary Interface Specification](https://github.com/riscv-non-isa/riscv-sbi-doc/)
        - 汇编手册：[RISC-V Assembly Programmer's Manual](https://github.com/riscv-non-isa/riscv-asm-manual)
- Linux 内核源码：
    - [torvalds/linux: Linux kernel source tree](https://github.com/torvalds/linux)
    - [Linux source code - Bootlin Elixir Cross Referencer](https://elixir.bootlin.com/)

后续的实验文档会指引同学们去阅读相应章节，但不会复述标准中的内容，避免产生歧义、过时等情况造成误导。

!!! tip

    如果在课程实验过程中遇到了困难，请积极寻求AI、助教、同学的帮助。
    
    如果你发现课程文档、代码有值得进一步完善的地方，请向我们反馈。

接下来为同学们导读 RISC-V 架构规范、Linux 内核源码。

## RISC-V 架构规范

RISC-V 因其开源、模块化的特点，拥有丰富的扩展指令集。**其中 Zicsr 扩展主要用于操作特权级寄存器，但在非特权级也有用法**，因此没有放置在特权级手册中。其余章节描述的扩展指令对我们来说并不重要。

在《操作系统》课程中，我们的任务是在 Linux 内核的接口约束下，向下对接 SBI 规范和 RISC-V 指令集，实现操作系统。

在某些实验中，我们可能会用到 **RISC-V指令集** 对应的汇编语言进行编程，因此可以先适当~~复习~~预习汇编语言、指令集架构等基本知识，再来学习RISC-V的指令集架构。

需要注意的是，**实验主要涉及 RISC-V 特权级手册中的下列内容** ：

- Chapter 2. Control and Status Registers (CSRs)
- Chapter 3. Machine-Level ISA
    - M-Mode 的 Trap 委托（Lab1）
    - 特权指令（Lab1、4）
- Chapter 12. Supervisor-Level ISA
    - S-Mode 的中断、异常处理（Lab1），上下文切换（Lab2）
    - Sv32 虚拟内存系统（Lab3）
- 其余扩展对我们来说并不重要。

## Linux 内核源码和课程仓库结构

详见 [Linux source code layout — The Linux Kernel documentation](https://linux-kernel-labs.github.io/refs/heads/master/lectures/intro.html#linux-source-code-layout)。你可以打开本文开头提到的 Linux 源码链接，边读边看。

课程仓库目录的组织结构与 Linux 源码保持一致。

- `arch`：特定架构的代码，实现启动、异常处理等核心功能，都受到指令集架构的约束
- `arch/riscv/kernel/head.S`：内核最开始执行的代码
- `lib`：内核无法使用标准库，`printk` 等辅助函数实现在这里

其中 [`arch/riscv`](https://github.com/torvalds/linux/tree/master/arch/riscv) 目录最为重要，课程实验大部分的代码工作都将发生在这里。

## 实验与考试的关联

考试重在原理而不会考察技术细节。Lab 的整体流程希望你通过实验牢牢掌握，重点不是细枝末节而是逻辑关系。

!!! example

    24 秋冬《操作系统》考试的最后一道大题是：手工完成 Sv32 虚拟内存到物理内存的翻译流程。如果你认真实现了 Lab3，那么这道题非常简单。