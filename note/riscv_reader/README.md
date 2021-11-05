# RISC-V Reader

## 1 为什么要有RISC-V？

**成本**

- 原因在于目前的硅生产工艺会在晶圆上留下一些散布的小瑕疵。因此晶粒越小，有缺陷部分所占比重会越低。
- cost ≈ f(die area^2) 成本与面积约为平方关系。

**简洁性**

**性能**

![perf](./1st/perf.png)

**架构和具体实现的分离**

**程序大小**

**易于编程/编译/链接**

## 2 RV32I：RISC-V 基础整数指令集

![i_set](./2nd/i_set.png)

### 2.2 指令格式

![i_type](./2nd/i_type.png)

六种基本指令格式:
1. 寄存器-寄存器操作的 R 类型指令
2. 短立即数和访存 load 操作的 I 型指令
3. 访存 store 操作的 S 型指令
4. 条件跳转操作的 B 类型指令
5. 长立即数的 U 型指令
6. 无条件跳转的 J 型指令

**乱序执行处理器**
> 这是一种高速的、流水化的处理器。它们一有机会就执行指令，而不是在按照程序顺序。这种处理器的一个关键特性是寄存器重命名，把程序中的寄存器名称映射到大量的内部物理寄存器。条件执行的问题是不管条件是否成立，都必须给这些指令中的寄存器分配相应的物理寄存器。但内部物理寄存器的可用性是影响乱序处理器的关键性能资源。

### 2.3 寄存器

![i_reg](./2nd/i_reg.png)

### 2.4 整数计算

* 算术指令 add, sub
* 逻辑指令 and, or, xor
* 移位指令 sll, srl, sra

**利用 xor 指令进行的花式操作**
> 可以在不使用中间寄存器的情况下交换两个值！此代码交换 x1 和 x2 的值。提示：异或操作是交换的 (𝑎 ⊕ 𝑏 = 𝑏 ⊕ 𝑎)，结合的 ((𝑎 ⊕ 𝑏) ⊕ 𝑐 = 𝑎 ⊕ (𝑏 ⊕ 𝑐))，是它自己的逆操作 (𝑎 ⊕ 𝑎 = 0)，并且有一个单位元(𝑎 ⊕ 0 = 𝑎)。

~~~ asm
xor x1,x1,x2 # x1’ == x1^x2, x2’ == x2
xor x2,x1,x2 # x1’ == x1^x2, x2’ == x1’^x2 == x1^x2^x2 == x1
xor x1,x1,x2 # x1” == x1’^x2’ == x1^x2^x1 == x1^x1^x2 == x2, x2’ == x1
~~~

### 2.5 Load 和 Store

**字节序问题**
> RISC-V 选择 little endian

### 2.6 条件分支

**不使用条件码实现大位宽数据的加法**
> 在 RV32I 中是通过 sltu 计算进位来实现的
~~~ asm
add a0,a2,a4 # 加低 32 位: a0 = a2 + a4
sltu a2,a0,a2 # 若 (a2+a4) < a2 那么 a2’ = 1, 否则 a2’ = 0
add a5,a3,a5 # 加高 32 位: a5 = a3 + a5
add a1,a2,a5 # 加上低 32 位的进位
~~~

**软件检查溢出**
> RISC-V 依赖于软件溢出检查
~~~ asm
add t0, t1, t2
slti t3, t2, 0 # t3 = (t2<0)
slt t4, t0, t1 # t4 = (t1+t2<t1)
bne t3, t4, overflow # 若 ((t2<0) && (t1+t2>=t1)) || ((t2>=0) && (t1+t2<t1)) 则为溢出
~~~

### 2.7 无条件跳转

跳转并链接指令（jal）具有双重功能。

若将下一条指令 PC + 4 的地址保存到目标寄存器中，通常是返回地址寄存器 ra，便可以用它来实现过程调用。

如果使用零寄存器（x0）替换 ra 作为目标寄存器，则可以实现无条件跳转，因为 x0 不能更改。

### 2.8 杂项

控制状态寄存器指令（csrrc、csrrs、csrrw、csrrci、csrrsi、csrrwi）

`ecall` 指令用于向运行时环境发出请求，例如系统调用。调试器使用 `ebreak` 指令将控制转移到调试环境。

`fence` 指令对外部可见的访存请求，如设备 I/O 和内存访问等进行串行化。外部可见指对处理器的其他核心、线程，外部设备或协处理器可见。`fence.i` 指令同步指令和数据流。

## 3 RISC-V 汇编语言

### 3.1 导言

![compile](./3rd/compile.png)

### 3.2 函数调用规范（Calling convention）

**保存寄存器和临时寄存器为什么不是连续编号**
> 为了支持 RV32E——一个只有 16 个寄存器的嵌入式版本的 RISC-V，只使用寄存器 x0 到 x15——一部分保存寄存器和一部分临时寄存器都在这个范围内。其它的保存寄存器和临时寄存器在剩余 16 个寄存器内。RV32E 较小，但由于和 RV32I 不匹配，目前还没有编译器支持。

### 3.3 汇编器

汇编程序的开头是一些汇编指示符（assemble directives）。它们是汇编器的命令，具有告诉汇编器代码和数据的位置、指定程序中使用的特定代码和数据常量等作用。

RISC-V 编译器支持多个 ABI，具体取决于 F 和 D 扩展是否存在。RV32 的 ABI 分别名为 ilp32，ilp32f 和 ilp32d。ilp32 表示 C 语言的整型（int），长整型（long）和指针（pointer）都是 32 位，可选后缀表示如何传递浮点参数。

**链接器松弛（linker relaxation）**

> 跳转并链接指令（jump and link）中有 20 位的相对地址域，因此一条指令就足够跳到很远的位置。尽管编译器为每个外部函数的跳转都生成了两条指令，很多时候其实一条就已经足够了。从两条指令到一条的优化同时节省了时间和空间开销，因此链接器会扫描几遍代码，尽可能地把两条指令替换为一条。每次替换会导致函数和调用它的位置之间的距离缩短，所以链接器会多次扫描替换，直到代码不再改变。这个过程称为链接器松弛，名字来源于求解方程组的松弛技术。除了过程调用之外，对于 gp 指针±2KiB 范围内的数据访问，RISC-V 链接器也会使用一个全局指针替换掉 lui 和 auipc 两条指令。对 tp 指针±2KiB 范围内的线程局部变量访问也有类似的处理。

| Directive           | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| .align n            | Align the next datum on a 2^n byte boundary.                 |
| .balign n           | Align the next datum on a n byte boundary.                   |
| .string "str"       | Store the string str in memory and null-terminate it.        |
| .byte b1, ..., bn   | Store the n 8-bit quantities in successive bytes of memory.  |
| .half h1, ..., hn   | Store the n 16-bit quantities in successive bytes of memory. |
| .word w1, ..., wn   | Store the n 32-bit quantities in successive bytes of memory. |
| .dword d1, ..., dn  | Store the n 64-bit quantities in successive bytes of memory. |
| .float f1, ..., fn  | Store the n single-precision floating-point numbers in successive memory words. |
| .double d1, ..., dn | Store the n double-precision floating-point numbers in successive memory words. |
| .option rvc         | Compress subsequent instructions.                            |
| .option norvc       | Don't compress subsequent instructions.                      |
| .option relax       | Allow linker relaxations for subsequent instructions.        |
| .option norelax     | Don't allow linker relaxations for subsequent instructions.  |
| .option pic         | Subsequent instructions are position-independent code.       |
| .option nopic       | Subsequent instructions are position-dependent code.         |
| .option push        | Push the current setting of all .option to a stack, so that a subsequent .option pop will restore their value. |
| .option pop         | Pop the option stack, restoring all .option to their setting at the time of the last .option push. |

### 3.5 静态链接和动态链接

编译器产生的代码和静态链接的代码很相似。其不同之处在于，跳转的目标不是实际的函数，而是一个只有三条指令的存根函数（stub unction）。存根函数会从内存中的一个表中加载实际的函数的地址并跳转。不过，在第一次调用时，表中还没有实际的函数的地址，只有一个动态链接的过程的地址。当这个动态链接过程被调用时，动态链接器通过符号表找到实际要调用的函数，复制到内存中，更新记录实际的函数地址的表。后续的每次调用的开销就是存根函数的三条指令的开销。

## 4 乘法和除法指令






