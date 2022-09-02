## 简介

GDB是一个强大的调试工具，支持多种语言。

## 程序怎样才能被调试

对于C程序，在编译的时候加上-g，就可以保留调试信息，才能使用gdb进行调试。

判断一个程序能否被调试有两种方法：

1. `gdb hello.o`

​				如果没有会显示：

​				`Reading symbols from hello.o...(no debugging symbols found)...done.`

2. readelf查看段信息，`readelf -S hello.o | grep debug`如果没有任何debug信息，则不能被调试。

## 调试程序

被调试程序hello.c

```C
#include <stdio.h>

int main()
{
    printf("hello world11!\n");
    printf("hello world22!\n");
    printf("hello world33!\n");
    return 0;
}
```

保留调试信息编译，得到hello.o文件。

`gcc -g hello.c -o hello.o`

### 未加断点

然后使用下面的指令

`gdb hello.o`

得到一个界面，输入run即可运行整个程序直到结束，因为还没有加入断点。

<img src="https://raw.githubusercontent.com/woshiwww/picgo/main/imgs/202203282032587.png" alt="image-20220328203247528" style="zoom:80%;" />

如果是带参数的调试，只需要在run后面添加相应的参数即可。

### 添加断点

**为什么要设置断点？**如果不设置断点，会像前面一样直接运行完整个程序，无法查看变量内容和堆栈情况。

查看一个程序是否已经设置了断点：（先使用gdb hello.o运行程序，然后输入以下指令，显示没有设置断点。）

`(gdb) info breakpoints
No breakpoints or watchpoints.`

**如何设置断点**

设置断点的方法有以下几种：

- 根据行号来设置断点。
  首先也是`gdb hello.o`
  然后输入`b 3`或者`b hello.c:2`后面的数字是行号。此时再输入`info breakpoints`就可以查看已经设置的断点。b=break
  设置好断点之后，程序运行到此处会停止。

- 根据函数名设置断点。
  `b main`此时程序运行到main函数时会停止，等待下一步的命令。

- 根据条件设置断点。
  有时程序运行到某一处会崩溃，此时就可以设置条件断点，当出现非法值时，程序停住。
  `b test.c:23 if c==0`
  当c=0时，程序会在第23行停下来。
  这个指令和condition有着相似的功能，假设上面设置的断点编号为1
  `condition 1 c==0`也可以在c=0是将程序停止。

- 根据规则设置断点。
  假设test.c中有几个函数print1()、print2()、print3()，需要对所有的print开始的函数都设置断点则可以使用
  `rbreak print*`或者`rbreak  test.c:^print`
  还可以对所有的函数都设置断点
  `rbreak .`或者`rbreak test.c:.`

- 跳过多次设置断点
  这种情况通常用于循环之中，假设某个地方前30次都是正确的，我们需要跳过这前面的30次执行。
  `ignore 1 30`
  这个1是断点的编号，通过`info breakpoints`指令可以看到这个断点会显示被忽略了30次。

- 根据表达式值变化产生断点
  要观察某个表达式或者变量，想知道发生了什么变化，可以使用watch指令，比如有个变量a
  `watch a`
  此时运行程序，a的值发生变化时，会打印相关内容。
  类似功能的还有`rwatch`和`awatch`前者是当变量值被读时断住，后者是被读或者改写时断住。

### 禁用或者启用断点

有些断点暂时不想使用又不想删除可以使用禁用和启用：

```shell
disable # 禁用所有的断点
disable 1 # 禁用编号为1的断点
enable # 启用所有断点
enable 1 # 启用编号为1的断点
enable delete 1 # 启用编号为1的断点，并且在启用后删除
```

### 断点清除

断点的清除主要使用两个指令clear 和 delete。前者加数字时对应的是行号，后者对应的是断点的编号。

```shell
clear # 删除所有的断点
clear function # 删除在函数名为function处的断点
clear test.c:function
clear line_num # 删除行号为line_num的断点
clear test.c:line_num
delete #删除所有断点、观察断点、catchpoints
delete bnum  # 删除断点编号为bnun的断点。
```

## 查看变量内容

这一步需要以前面的启动调试和设置断点为前提。

### 基本变量查看--基本类型、数组、字符串

在进入gdb调试并设置好断点之后，可以使用p后面接变量名输出内容。
`(gdb) p a`

p是print的缩写。

当存在多个文件或者函数具有相同的变量名时，可以加上文件名或者函数名来进行区分

`p 'test.c'::a`  **注意：**无论是文件名还是函数名都要加' '

`p 'main'::a` 打印main()函数中的变量a

**当打印指针指向的内容时，需要解引用，不然打印出来的是一个地址，如果这个指针指向的是一个数组还需要在*d后面加一个@和指定的长度才能打印整个数组的内容，否则只能打印一个值**。假设`d = {1,2,3,4,5}`

```shell
(gdb) p d
$1 = (int *)0x12849812  # 显示一个地址
(gdb) p *d
$2 = 1  # 只显示第一个值
(gdb) p *d@5
$3 = {1,2,3,4,5} # 解引用后在@后面加上需要打印的长度或者跟一个变量，比如如果变量a=5，--> p *d@a
```

### 按照指定格式打印变量

常见的格式控制字符有：

- x -- 以16进制的形式显示变量

- d--以十进制的形式

- u--无符号16进制的格式

- o--八进制的形式

- t--二进制的格式显示变量

- a--十六进制的格式

- c--字符格式显示

- f--浮点数格式显示 

使用方法：

假设有一个字符串`c = 'hello,world'`,并且这个程序已经使用`gcc -g hello.c -o hello.o`编译。

```shell
gdb hello.o
(gdb) p c
$1 = "hello,world"

# 如果想以十六进制的格式打印
(gdb) p/X c
$2 = {0x123,0x12···} # 数字是随便写的具体，理解即可

# 如果想以二进制的格式打印一个浮点数，会先将浮点数转换为整数，再输出这个整数的二进制形式 e = 8.3
(gdb) p e
$3 = 8.2
(gdb) p/t e
$3 = 1000
```

### 查看内存内容

使用examine（简写为x）可以查看内存地址中的值。语法格式为：

`x/[n][f][u] addr`

- n代表要显示的内存单元数
- f是前面的打印格式控制符
- u是要打印的单元长度--b字节        h半字(双字节)                  w字(四字节)                g八字节
- addr是内存地址

例：将浮点是变量e用二进制方式打印，打印单位为字节，浮点数的长度为32位

```shell
(gdb)x/4tb &e    # &取地址运算符
0x5fffff34ffff2222: 0000000 00000000 01203123 0321423
```

### 自动显示变量内容

想自动显示一个变量可以使用display指令，这样程序每次断住就会显示e的值。

```shell
(gdb) display e
1: e = 8.2
(gdb) info display  # 可以显示哪些变量被设置为dispaly
(gdb) delete display num # 清除编号为num的显示变量，如果不加num则为清除所有标记
(gdb) disable display num # 暂时不使用这个编号的变量，不加num为暂时不显示所有，但是标记还在
```

### 查看寄存器的内容

````shell
(gdb) info registers
```
```
```
``
# 会显示所有的寄存器中的内容
````

## 调试方法

list（可简写为l）能够将源码列出来。一般显示10行源码

### 单步执行-next

next命令，可以简写为n，可以在设置断点的地方继续执行下一条语句，后面可以+数字，表示执行这个命令多少次.**这种方式不会进入被调用的函数。**

```shell
(gdb) next  # 执行1条
(gdb) next 3 # 执行3条
```

### 单步进入-step

**如果想进入某个被调用的函数**，则可以使用step指令，简写为s。但是前提是要有源码信息，比如不能进入printf()函数，此时可以使用finish指令继续执行下一条指令。在没有函数调用的情况下s和n具有相同的功能。step指令还有一个可选项，用来设置遇到没有调试信息的函数时，s指令是否跳过该函数，执行后面的，默认情况下，这个是off

```shell
(gdb) show step-mode
Mode of the step operation if off
(gdb) set step-mode on  # 打开
(gdb) set step-mode off  # 关闭
```

stepi指令简写为si，它的作用是每次执行一条机器指令。

### 执行到下一个断点-continue

当有多个断点时，或者在一个内循环打了断点，此时可使用continue指令，简写为c或者fg，运行到下一个断点处。后面可以＋数字，表示遇到后几个断点才断住，默认是1。

### 继续运行到指定位置-until

使用until指令，简写为u，可以使程序直到运行到哪一行才停止。

```shell
(gdb) until 4 # 直到运行到第四行才停止
```

### 跳过执行-skip

使用skip指令跳过某个函数或者文件之后，使用step指令将不会进入改函数或者文件。skip指令后面可以加函数或者文件。

```shell
(gdb) skip function add # 跳过add函数
(gdb) info skip   # 可以查看将被跳过的函数或者文件
(gdb) skip file hello.c # 跳过这个文件

# 删除这些跳过的函数或者文件可以使用delete 或者disable指令 后面不加num，则针对所有的skip
(gdb) skip delete num  # num 是通过info skip 显示出来的编号，不是行数，
(gdb) skip disable num # 暂时不用编号为num的跳过
(gdb) skip enable num  # 启用
```

## 查看源码

如何在gdb调试模式下查看源码和对源码进行编辑？

- **列出源码：** list，简写为l。后面可以跟+或者-。
                          |                                                |-----    跟+表示列出上一次列出源码的后面部分。
                          |                                                |-----    跟-表示列出上一次源码的前面部分。
                          |
                          |------    list后面加数字，数字表示的是行号，可以列出该行附近的源码。

  ​                        |------    list后面加函数名，表示列出指定函数附近的源码 list add       只需要函数名即可
  ​                        |------    set listsize 20        指令可以设置每次显示的行数     show listsize可以显示当前的设置
  ​                        |------    list first，last   指令可以显示指定行之间的源码

  ​                        |------    前面的list列出的默认文件是main.c文件中的源码，如果想查看指定文件，可以在list后加文                            										件名 。` list hello.c:1     list hello.c:add  ` 可以指定行数、函数、指定行之间
  ​                                        `list hello.c:1, hello.c:3`     

- **编辑源码：**适用于已经启动了gdb调试，需要编辑源码但是又不想退出。此时默认的编辑器是/bin/ex，  					 bin文件下的ex编辑器，可以使用以下设置来使用自己熟悉的编辑器。
                          				                 ` $ EDITOR=/usr/bin/vim` 
                                               ` $ export EDITOR` 如果不知道自己想要的编辑器在哪，可以使用whereis或者witch指令
                                              `  $ whereis vim `     或者  `witch vim`

​                                    设置好之后，就可以在gdb调试模式下进行源码编辑，使用edit指令。

```shell
(gdb)edit 3  # 编辑当前文件的第三行
(gdb)edit add  # 编辑add函数
(gdb)edit hello.c:3 # 编辑hello.c文件的第三行

# 编辑完保存后，要重新编译程序
(gdb)shell gcc -g main.c hello.c -o main.o  # 在gdb模式下运行shell指令 前面要加shell，这样就可在不                                               退出程序的情况下编译程序了。
```

除此之外，如果想在多个窗口呈现调试界面，可以在启动调试的时候使用`gdb main.o -tui`。
