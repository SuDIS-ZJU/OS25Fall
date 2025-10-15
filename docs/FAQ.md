# 常见问题及回答

在本节中会定期更新同学们的共性问题及回答。

## lab1-思考题6：如何获取ARM64/RV32/RV64/x86_64架构的系统调用表

!!! tip

    相信有很多同学在完成这个思考题时，找到了类似`*.tbl`的文件，但是恰恰RISCV目录下是没有这个文件的（其实这个RISCV使用的是linux大目录下`scripts`中的`.tbl`文件。）

    需要注意的是，`*.tbl`文件 **并不是** 思考题中要求的“宏展开后”的文件，它只是一个存放数据的文件，并没有体现出它在代码中的嵌入关系。

    而在思考题5中，我们使用交叉编译生成的`*.i`文件，才是该思考题希望同学们获取的答案。


> 由于该部分较为复杂，因此同学们在报告中该思考题的ARM和x86的部分只需要提供.tbl文件的截图即可（RISCV仍然需要体现使用交叉编译生成文件的过程），具体的交叉编译方法供有兴趣的同学参考。

首先，我们要梳理获取系统调用表的具体方法：

### 1. 找到调用了系统调用表并进行配置的`*.c`文件

在不同的架构中，这些文件可能位于不同的位置（同学们可以思考 ~~不使用GPT~~ 如何找到它们，这里直接将答案列出）：

- ARM64：位于`arch/arm64/kernel/sys.c`
- RISCV32/64：位于`arch/riscv/kernel/syscall_table.c`
- x86_64：位于`arch/x86/entry/syscall_64.c`

!!! tip

    为什么RISCV32/64使用的是同一个.c文件呢？
    
    因为这个文件会通过编译时所使用的编译器来考虑使用RISCV32/64的系统调用表，换言之，对于RV32/64的编译，生成目标（即.i文件的路径）是不变的，你只需要修改所使用的编译器即可。


### 2. 对这个`*.c`文件进行交叉编译，目标是生成`*.i`文件（实验指导一：其他架构的交叉编译中提及）

生成`*.i`文件的本质目的是： **将c文件中的宏定义进行展开，获取展开后的系统调用表** 

- 这是因为：`*.c`文件往往是通过引用多个`*.h`文件来进行构造的，而实际的系统调用表内容一般存放在`*.h`文件中。因此，只观察`*.c`文件是无法看到具体的系统调用表内容的。

对于不同的架构，你需要选择（必要时需要安装）不同的编译工具链，并指定好对应需要生成的`*.i`文件的路径：

对于编译工具链：

- ARM64：使用`gcc-aarch64-linux-gnu`，需要自行安装（指导中已给出方法）
- RISCV32：使用`riscv64-linux-gnu-`， **但是需要使用不同的config使其能够使用RV32编译方法**
- RISCV64：使用`riscv64-linux-gnu-`，在lab0中我们使用的就是这个编译器
- x86_64：使用`x86_64-linux-gnu-`，使用WSL的同学应当在Linux初始化环境时就拥有这个编译器，如果是其他方式进行实验的同学可能需要自行安装

在安装好工具链后， **需要先进行config配置** （具体方法在lab0中使用过）

- 对于ARM64、x86_64、RISCV64，使用`defconfig`配置即可
- 对于RISCV32，需要使用`rv32_defconfig`配置

之后，就可以依照lab1给出的模板生成`.i`文件了：

`make ARCH=<arch name> CROSS_COMPILE=<compiler name> <path>`

其中`<>`的内容是需要根据需求进行替换的部分，需要注意的是：
- `<path>`替换为当前目录下的相对路径，如`arch/arm64/kernel/sys.i`

!!! tip

    如果在编译过程中报错：缺失`<gelf.h>`文件：

    需要通过`sudo apt install libelf-dev`安装libelf-dev工具来补充该头文件

之后，如果你所生成的`.i`文件的末尾（约70000行）存在类似这样的对照表（不同架构生成的内容表现形式可能不同，但应当是系统调用号到系统调用名之间的一一对应关系）：

```
# 1 "./arch/riscv/include/generated/asm/syscall_table_64.h" 1
[0] = __riscv_sys_io_setup,
[1] = __riscv_sys_io_destroy,
[2] = __riscv_sys_io_submit,
[3] = __riscv_sys_io_cancel,
[4] = __riscv_sys_io_getevents,
[5] = __riscv_sys_setxattr,
[6] = __riscv_sys_lsetxattr,
[7] = __riscv_sys_fsetxattr,
[8] = __riscv_sys_getxattr,
[9] = __riscv_sys_lgetxattr,
[10] = __riscv_sys_fgetxattr,
# ...
```

这说明你成功地获取了宏展开后的系统调用表！

### 3. What About ARM32?

ARM32的情况相比于其余几种架构更为特殊，它的导入路径如下：

`arch/arm/tools/syscall.tbl -----> arch/arm/include/generated/asm/unistd-nr.h -----> arch/arm/kernel/entry-common.S`

这意味着，经过Makefile的层层处理后，最终是由一个`.S`文件来参与最终的编译和链接。遗憾的是，Makefile中并没有给出`*.S`到`*.i`的转化规则，仅有`*.S`到`*.o`的转化规则 ~~（拼尽全力无法战胜）~~ ，因此在不修改执行命令或者Makefile的前提下无法显式地查看宏展开后的系统调用表。

> 如果有同学发现了可以查看该系统调用表的方法或有补充意见及建议，欢迎与助教联系！

---

特别感谢黄同学和程同学为实验指导做出的补充！同时也鼓励同学们发掘实验文档中的问题并与助教交流！