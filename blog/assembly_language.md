# Assembly Language

## Real Mode

* In real mode, memory is limited to only one megabyte (2^20 bytes). Valid address range from (in hex) 00000 to FFFFF. These addresses require a 20-bit number. Obviously, a 20-bit number will not fit into any of the 8086’s 16-bit registers. Intel solved this problem, by using two 16-bit values determine an address. The first 16-bit value is called the selector. Selector values must be stored in segment registers. The second 16-bit value is called the offset. The physical address referenced by a 32-bit selector:offset pair is computed by the formula: `16 ∗ selector + offset`.

> Real segmented addresses have disadvantages:
> 1. A single selector value can only reference 64K of memory (the upper limit of the 16-bit offset). What if a program has more than 64K of code? A single value in CS can not be used for the entire execution of the program. The program must be split up into sections (called segments) less than 64K in size. When execution moves from one segment to another, the value of CS must be changed. Similar problems occur with large amounts of data and the DS register. This can be very awkward!
> 2. Each byte in memory does not have a unique segmented address. The physical address 04808 can be referenced by 047C:0048, 047D:0038, 047E:0028 or 047B:0058. This can complicate the comparison of segmented addresses.

## 16-bit Protected Mode

* In real mode, a selector value is a paragraph number of physical memory. In protected mode, a selector value is an index into a descriptor table.
* In protected mode, each segment is assigned an entry in a descriptor table. This entry has all the information that the system needs to know about the segment. This information includes: is it currently in memory; if in memory, where is it; access permissions (e.g., read-only). The index of the entry of the segment is the selector value that is stored in segment registers.

> One big disadvantage of 16-bit protected mode is that offsets are still 16-bit quantities. As a consequence of this, segment sizes are still limited to at most 64K. This makes the use of large arrays problematic!

## 32-bit Protected Mode

* Offsets are expanded to be 32-bits. This allows an offset to range up to 4 billion. Thus, segments can have sizes up to 4 gigabytes.
* Segments can be divided into smaller 4K-sized units called pages. The virtual memory system works with pages now instead of segments. This means that only parts of segment may be in memory at any one time. In 286 16-bit mode, either the entire segment is in memory or none of it is. This is not practical with the larger segments that 32-bit mode allows.
* Segments can be divided into smaller 4K-sized units called pages. The virtual memory system works with pages now instead of segments. This means that only parts of segment may be in memory at any one time.

## Stack Operations

Example instrution | What it does
--- | ---
pushl %eax | subl $4, %esp; movl %eax, (%esp)
popl %eax | movl (%esp), %eax; addl $4, %esp
call 0x12345 | pushl %eip; movl $0x12345, %eip
ret | popl %eip

> Stack grows down

## GCC calling conventions

* Saved %ebp is form a chain, which can help to walk stack.
* Arguments and locals at fixed offsets from %ebp.

* Prologue: pushl %ebp; movl %esp, %ebp
* Epilogue: movl %ebp, %esp; popl %ebp

~~~
            +---------------+
            |   arg n       |
            +---------------+
            |   arg 2       |
            +---------------+
            |   arg 1       |
            +---------------+
            |   ret %eip    | caller function's stack frame
            +===============+
%ebp -->    |   saved %ebp  | callee function's stack frame
            +---------------+ <-- where Prologue & Epilogue happen
            |   local       |
            |   variables,  |
%esp -->    |   etc.        |
            +---------------+
~~~

Example:

~~~ asm
int main(void) { return f(8) + 1; }
int f(int x) { return g(x); }
int g(int x) { return x + 3; }

main:
    pushl %ebp
    movl %esp, %ebp

    pushl $8
    call f
    addl $1, %eax

    movl %ebp, %esp
    popl %ebp
    ret

f:
    pushl %ebp
    movl %esp, %ebp

    pushl 8(%esp)       # arg1
    call g

    movl %ebp, %esp
    popl %ebp
    ret

g:
    pushl %ebp
    movl %esp, %ebp

    pushl %ebx          # save ebx

    movl 8(%ebp), %ebx
    addl $3, %ebx
    movl %ebx, %eax     # %eax as return value

    popl %ebx           # restore ebx

    movl %ebp, %esp
    popl %ebp
    ret
~~~

## ABI Registers

Register    | Callee Save | Description
--          | --          | --
%rax	    |no|	result register, also used in idiv and imul
%rbx	    |yes|	miscellaneous register
%rcx	    |no|	fourth argument register
%rdx	    |no|	third argument register, also idiv and imul
%rsp	    |no|	stack pointer
%rbp	    |yes|	frame pointer
%rsi	    |no|	second argument register
%rdi	    |no|	first argument register
%r8	        |no|	fifth argument register
%r9	        |no|	sixth argument register
%r10	    |no|	miscellaneous register
%r11	    |no|	miscellaneous register
%r12~%15	|yes|	miscellaneous registers

## iret Instruction

* The `iret` instruction pops the return instruction pointer, return code segment selector, and EFLAGS image from the stack to the `EIP`, `CS`, and `EFLAGS` registers, respectively, and then resumes execution of the interrupted program or procedure.
* If the return is to another privilege level, the `IRET` instruction also pops the `stack pointer` and `SS` from the stack, before resuming program execution.

~~~ c
struct Trapframe {
    struct PushRegs tf_regs;

    uint16_t tf_es;
    uint16_t tf_padding1;
    uint16_t tf_ds;
    uint16_t tf_padding2;
    uint32_t tf_trapno;

    /* below here defined by x86 hardware */
    uint32_t tf_err;
    uintptr_t tf_eip;
    uint16_t tf_cs;
    uint16_t tf_padding3;
    uint32_t tf_eflags;

    /* below here only when crossing rings, such as from user to kernel */
    uintptr_t tf_esp;
    uint16_t tf_ss;
    uint16_t tf_padding4;
} __attribute__((packed));
~~~

### packed attribute

* packed: 数据结构不作对齐处理

~~~ c
struct File {
    char f_name[5];
    int f_size;
    unsigned int f_type;
};

sizeof(struct File) == 16;

struct File {
    char f_name[5];
    int f_size;
    unsigned int f_type;
} __attribute__((packed));

sizeof(struct File) == 13;
~~~

## \__ASSEMBLER__ MACRO

* `__ASSEMBLER__` 用来告知预处理器此时是 asm 还是 gcc

## *%ecx 和 *(%ecx) 区别

* *%ecx 取`ecx`寄存器的值
* *(%ecx) 取`ecx`寄存器指向的内存的值

~~~ asm
80520d3:       ff e1                   jmp    *%ecx
80520d3:       ff 21                   jmp    *(%ecx)
~~~

## Inline Assembly Routine

* 其语法固定为: `asm volatile ("asm code": output: input: changed);`
* `__asm__` 表示后面的代码为内嵌汇编，`asm` 是 `__asm__` 的别名
* `__volatile__` 表示编译器不要优化代码，后面的指令保留原样，`volatile` 是它的别名
* **changed** 破坏描述符用于通知编译器我们使用了哪些寄存器或内存，由逗号格开的字符串组成，每个字符串描述一种情况，一般是寄存器名；除寄存器外还有"memory"，例如："%eax"，"%ebx"，"memory"等
* **Memory** :
    1. 不要将该段内嵌汇编指令与前面的指令重新排序；也就是在执行内嵌汇编代码之前，它前面的指令都执行完毕。
    2. 不要将变量缓存到寄存器，因为这段代码可能会用到内存变量，而这些内存变量会以不可预知的方式发生改变，因此GCC插入必要的代码先将缓存到寄存器的变量值写回内存，如果后面又访问这些变量，需要重新访问内存。
* 如果汇编指令修改了内存，但是 GCC 本身却察觉不到，因为在输出部分没有描述，此时就需要在修改描述部分增加"memory"，如 `__asm__ __volatile__("": : : "memory");`

~~~ asm
asm volatile("int %1\n"
            : "=a" (ret)
            : "i" (T_SYSCALL),
            "a" (num),
            "d" (a1),
            "c" (a2),
            "b" (a3),
            "D" (a4),
            "S" (a5)
            : "cc", "memory");
~~~

Qualifier            | 意义
---                  | ---
"m" "v" "o"          | 内存单元
"r"                  | 任何寄存器
"q"                  | 寄存器eax, ebx, ecx, edx之一
"i" "h"              | 直接操作数
"E" "F"              | 浮点数
"g"                  | 任意
"a" "b" "c" "d"      | 分别表示寄存器eax, ebx, ecx, edx
"S" "D"              | 寄存器esi, edi
"I"                  | 常数 (0至31)

Output Qualifier     | 描述
---                  | ---
\+                   | 可以读取和写入操作数
=                    | 只能写入操作数
%                    | 如果有必要，操作数可以和下一个操作数切换
&                    | 在内联函数完成之前，可以删除和重新使用操作数
