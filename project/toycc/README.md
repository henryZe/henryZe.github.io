# Toycc Compiler

## 序言

本编译器项目是参考chibicc项目，改编以RISC-V指令集为目标机器语言的C编译器。

本文档是在Toycc项目完结以后，针对项目的每个提交，作分析讲解。

在开始之前，这里先介绍一个网站，可以实时将对象语言编译为目标语言，方便读者对照：[godbolt](https://godbolt.org/)。

## 1 创建编译整数的语言

* 提交 1 使用指定数值作为程序退出
* 提交 2 支持加减法运算符
* 提交 3 引入分词器（tokenizer）概念，以空格区隔
* 提交 4 改进错误信息，定位错误位置

## 2 创建一种可以执行四种算术运算的语言

### 2.1 语法描述方法与递归下降解析

想要在语言中添加乘法*，除法/和优先括号()，需要实现运算符优先级，可以使用树结构表示语法结构。

使用生成式规则定义语法，这里介绍 EBNF 规则：

 表达式 | 含义
-------|-----------------------------------
ε      | epsilon 作为符号，表示没有任何东西
A*     | A出现0次或多次重复
A?     | A或者ε
A \| B | A或者B
 (...) | 分组

简单的生成式规则表达：

~~~
expr = num ("+" num | "-" num)*
~~~

使用生成式规则表达运算符优先级：

~~~
expr = mul ("+" mul | "-" mul)*
mul  = num ("*" num | "/" num)*
~~~

涉及递归的生成式规则：

~~~
expr    = mul ("+" mul | "-" mul)*
mul     = primary ("*" primary | "/" primary)*
primary = num | "(" expr ")"
~~~

递归降序解析：

只提前读取一个标记的递归下降解析器被称为1解析器

### 2.2 堆叠机

计算 2*3 + 4*5 可以用堆叠机表示为：

~~~
push 2
push 3
mul

push 4
push 5
mul

add
~~~

转化为x86上的堆栈为：

~~~
push 2
push 3

pop rdi
pop rax
mul rax, rdi
push rax

push 4
push 5

pop rdi
pop rax
mul rax, rdi
push rax

pop rdi
pop rax
add rax, rdi
push rax
~~~

### 2.3 提交解析

* 提交 5 支持运算符 * / ()
* 提交 6 支持单目运算符 + -

~~~
expr    = mul ("+" mul | "-" mul)*
mul     = unary ("*" unary | "/" unary)*
unary   = ("+" | "-")? primary
primary = num | "(" expr ")"
~~~

## 3 支持比较运算符

### 3.1 比较运算符优先级

优先级从低到高：

~~~
==, !=
<, <=, >, >=
+, -
*, /
unary +, unary -
()
~~~

~~~
expr       = equality
equality   = relational ("==" relational | "!=" relational)*
relational = add ("<" add | "<=" add | ">" add | ">=" add)*
add        = mul ("+" mul | "-" mul)*
mul        = unary ("*" unary | "/" unary)*
unary      = ("+" | "-")? primary
primary    = num | "(" expr ")"
~~~

### 3.2 提交解析

* 提交 7 支持比较运算符 '==' '!=' '<' '<=' '>' '>='

## 4 单独编译和链接

减少因文件变动而导致的重复编译。头文件包含公共的声明。

* 提交 8 源文件分隔和 Makefile 适配

## 5 函数和局部变量

### 5.1 栈上变量区

### 5.2 解析器的变化

~~~
program    = stmt*
stmt       = expr ";"
expr       = assign
assign     = equality ("=" assign)?
equality   = relational ("==" relational | "!=" relational)*
relational = add ("<" add | "<=" add | ">" add | ">=" add)*
add        = mul ("+" mul | "-" mul)*
mul        = unary ("*" unary | "/" unary)*
unary      = ("+" | "-")? primary
primary    = num | ident | "(" expr ")"
~~~

### 5.3 左值和右值

现今的语言中，只有变量是左值，所有其他值都是右值。

左值是将给定节点指向变量时，计算变量的地址并将其压入堆栈。

当使用变量作为右值时，首先将其计算为左值，然后将堆栈顶部的计算结果视为地址，并从该地址加载值。

~~~
void gen_lval(Node *node)
{
    printf("  mov rax, rbp\n");
    printf("  sub rax, %d\n", node->offset);
    printf("  push rax\n");
}

void gen(Node *node)
{
    switch (node->kind) {
    case ND_LVAR:
        gen_lval(node);

        printf("  pop rax\n");
        printf("  mov rax, [rax]\n");
        printf("  push rax\n");
        return;
    }
}
~~~

### 5.4 提交解析

* 提交 9 接受多个使用 ";" 分隔的语句
* 提交 10 支持单字符的局部变量

## 6 多字符局部变量

解析器必须确定每个标识符以前是否见过，如果是新的，则自动在堆栈区域中分配一个变量。

* 提交 11 支持多字符的局部变量

## 7 多字符局部变量

### 7.1 新增语法

~~~
program = stmt*
stmt    = "return" expr ";" | expr ";"
...
~~~

解析器：

~~~
Node *stmt()
{
    Node *node;

    if (consume(TK_RETURN)) {
        node = calloc(1, sizeof(Node));
        node->kind = ND_RETURN;
        node->lhs = expr();
    } else {
        node = expr();
    }

    assert (!consume(';'));
    return node;
}
~~~

代码生成器：

~~~
void gen(Node *node)
{
    if (node->kind == ND_RETURN) {
        gen(node->lhs);

        printf("  pop rax\n");
        printf("  mov rsp, rbp\n");
        printf("  pop rbp\n");
        printf("  ret\n");
        return;
    }
}
~~~

### 7.2 提交解析

* 提交 12 支持return语句

> 语法层次结构：
>
> 存在语法层次结构：正则表达式 < 上下文无关语法 BNF/EBNF < 可判定 < 图灵可识别。

## 8 块结构

块具有将多个语句组合成单个语句的作用。

~~~
program = stmt*
stmt    = expr ";"
        | "{" stmt* "}"
        | ...
...
~~~

* 提交 13 支持 {} 复合语句
* 提交 14 支持 NULL 语句

## 9 控制结构

添加 if, while, for 的新语法：

~~~
program = stmt*
stmt    = "return" expr ";"
        | expr ";"
        | "if" "(" expr ")" stmt ("else" stmt)?
        | "while" "(" expr ")" stmt
        | "for" "(" expr? ";" expr? ";" expr? ")" stmt
...
~~~

if (A) B 编译成这样的程序集：

~~~
    if (A == 0)
        goto end;
    B;
end:
~~~

if (A) B else C 编译成这样的程序集：

~~~
    if (A == 0)
        goto els;
    B;
    goto end;
els:
    C;
end:
~~~

while (A) B 编译如下：

~~~
begin:
    if (A == 0)
        goto end;
    B;
    goto begin;
end:
~~~

for (A; B; C) D 编译如下：

~~~
    A;
begin:
    if (B == 0)
        goto end;
    D;
    C;
    goto begin;
end:
~~~

> 以 .L 开头的标签会被汇编器特别识别，并自动成为文件范围。文件范围标签可以从同一文件内引用，但不能从其他文件引用。

* 提交 15 支持 if 语句
* 提交 16 支持 for 语句
* 提交 17 支持 while 语句

## 10 指针和字符串文字

### 10.1 一元&和一元*

更新解析器：

~~~
unary = "+"? primary
      | "-"? primary
      | "&" unary
      | "*" unary
~~~

更新代码生成器：

~~~
    case ND_ADDR:
        gen_lval(node->lhs);
        return;

    case ND_DEREF:
        gen(node->lhs);

        printf("  pop rax\n");
        printf("  mov rax, [rax]\n");
        printf("  push rax\n");
        return;
~~~

* 提交 19 解析器中增加定位信息用以改善错误信息
* 提交 20 增加 */& 一元操作符
* 提交 21 支持指针算术操作

### 10.2 消除隐式变量定义并引入 int 关键字

* 提交 22 增加int关键字，强制性变量定义

### 10.3 引入指针类型

**定义一个表示指针的类型**

使用 struct Type 代表数据类型：

~~~
struct Type {
    enum { INT, PTR } ty;
    struct Type *ptr_to;
};
~~~

![pointer_ty](./images/pointer_ty.svg)

### 10.4 sizeof运算符

sizeof虽然它看起来像一个函数，但从语法上来说它是一个一元运算符。

~~~
unary = "sizeof" unary
      | ("+" | "-")? primary
~~~

* 提交 30 支持 sizeof 单目运算符

### 10.5 实现数组

**定义数组类型**

使用 struct Type 代表数组数据类型：

~~~
struct Type {
    enum { INT, PTR, ARRAY } ty;
    struct Type *ptr_to;
    size_t array_size;
};
~~~

* 提交 27 支持一维数组
* 提交 28 支持数组的数组

**实现从数组到指针的隐式类型转换**

在 C 语言实现中，x[y]被定义为等价于*(x+y)。以此，实现数组下标。

* 提交 29 支持 [] 操作符

### 10.6 实现全局变量

在 C 中，字符串是一个字符数组，但不同的是字符串不是存在于堆栈上的值。字符串驻留在内存中的固定位置，而不是堆栈上。

仅当变量名称无法解析为局部变量时，才会尝试将其解析为全局变量。这允许以自然的方式让局部变量隐藏同名的全局变量。

* 提交 31 合并函数和全局变量到 globals 链表上
* 提交 32 支持全局变量

### 10.7 实现字符串文字

* 提交 33 添加字符类型
* 提交 34 添加字符串类型

## 11 响应函数调用

~~~
primary = num
        | "(" expr ")"
        | ident ("(" ")")?
~~~

这里还未实现函数定义，先实现函数调用，需要先额外通过编译器编译输出后再与链接器输出链接。

x86-64 函数调用 ABI 很简单，但有一个警告。在调用函数之前，RSP 必须是 16 的倍数。push由于pop RSP以8字节为单位改变，因此call发出指令时RSP不一定是16的倍数。确保在调用该函数之前调整 RSP，使其成为 16 的倍数。

* 提交 23 支持0参数函数调用
* 提交 24 支持最高6参数函数调用

## 12 解决函数定义问题

被调用方需要能够按名称访问参数，要做的就是将函数参数存在局部变量一样进行编译，并在函数的序言中将寄存器值写入该局部变量的堆栈区域。之后，您应该能够毫无区别地处理参数和局部变量。

* 提交 25 支持0参数函数定义
* 提交 26 支持最高6参数函数定义

> ABI 二进制级接口
>
> ABI 除了规定如何传递参数和返回值之外，还包含：
>
> * 被调用函数改变的寄存器和没有改变的寄存器（RBP等在返回前都恢复到原来的值，但有些寄存器不需要恢复到原来的值）
> * 类型字长，例如 int,long
> * 结构体布局规则（结构成员在内存中实际排列的规则）
> * 位域布局规则（例如，位域应该从最低有效位开始排列还是从最高有效位开始排列？）

## 13 整数的表示

### 13.1 有符号整数

具体来说，如果我们考虑一个4位二进制数，则每个数字及其代表的数字如下表所示：

  符号位 |  4 | 3 | 2 | 1
---------|----|---|---|---
unsigned |  8 | 4 | 2 | 1
signed   | -8 | 4 | 2 | 1

即 -2 = 1110

### 13.2 符号扩展

对有符号整数进行扩展时，如果符号位为1，则新的高位必须全1填充，如果符号位为0，则新的高位必须全0填充。这种操作称为“符号扩展”。

**反转和补码**

0b0011_0011 可以按如下方式反转：

~~~
  1111 1111
- 0011 0011
= 1100 1100
~~~

故补码能以下方式转换：

~~~
-n = (-1 - n) + 1
~~~

## 14 行注释和块注释

* 提交 43 支持行注释和块注释

## 15 初始化表达式语法

初始化表达式似乎只是一个赋值表达式，但实际上，初始化表达式和赋值表达式在语法上有很大不同，并且有一些特殊的写法只允许初始化表达式。

~~~
int x[3] = {0, 1, 2};
~~~

如果给出初始化表达式，则可以省略数组的长度，因为可以通过查看右侧元素的数量来确定数组的长度。

~~~
int x[] = {0, 1, 2};
~~~

如果显式给出了数组长度并且仅给出了部分初始化表达式，则其余元素必须初始化为零。

~~~
int x[5] = {1, 2, 3, 0, 0};
int x[5] = {1, 2, 3};
~~~

char作为数组初始化表达式的特殊语法，允许使用以下使用文字字符串作为初始化表达式的编写方式。

~~~
char msg[] = "foo";
char msg[4] = {'f', 'o', 'o', '\0'};
~~~

## 16 C 类型语法

### 16.1 如何读取C类型

C 类型从头开始可以分解为四个部分：

* 基本类型
* 星号代表指针
* 括号中的标识符或嵌套类型
* 括号代表函数和数组

C 类型表示法   | 意义
--------------|-----------
int x         | int
int *x        | * int
int **x	      | * * int

当有函数括号时，基类型和代表指针的星号代表返回类型：

C 类型表示法   | 意义
--------------|------------------
int x()       | func() int
int *x()      | func() * int
int **x(int)  | func(int) * * int

当您看到数组括号时，它的意思是“数组”：

C 类型表示法   | 意义
--------------|------------------
int x[5]      | [5] int
int *x[5]     | [5] * int
int **x[4][5] | [4] [5] * * int

### 16.2 如何读取嵌套类型

~~~
void (*signal(int, void (*)(int)))(int);
~~~

> func(int, * func(int) void) * func(int) void

signal是一个带有两个参数的函数：

1. int
2. 指向接受int并返回void函数的指针

signal的返回类型是一个指针，它指向接受int并返回void的函数。
