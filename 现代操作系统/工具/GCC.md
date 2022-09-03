## gcc

### 安装

```shell
sudo apt install gcc
```

### 查看版本信息

```shell
gcc --version
```

### 编译程序

假设存在一个hello.c文件

```shell
gcc hello.c -o hello.o  # 第一个是要编译的文件  第二个是输出文件
ls # 会出现一个hello.o可执行文件
./hello.o # 便可执行
```

gcc编译C文件时分为四步：

1. 预处理 `-E` 生成.i文件，用宏定义内容和头文件替换相应的程序，此时还是c程序。
2. 汇编`-S` 生成.s文件，c语言变为汇编语言代码。
3. 编译`-c`生成.o文件，汇编语言变为机器码。
4. 链接`-o`，将每个.o文件链接起来，生成一个可执行程序。widows下生成的是.exe文件

```shell
# 具体使用方法如下所示
# 第一个阶段 预处理
gcc -E hello.c -o hello.i # 生成一个.i文件，

# 第二个阶段 汇编 可以直接用上个阶段的.i文件也可以直接使用.c文件来生成.s汇编文件。汇编语言是和平台相关的！
gcc -S hello.i -o hello.s
# 或者
gcc -S hello.c -o hello.s

# 第三个阶段 编译 将.s文件编译成机器代码
gcc -c helle.c -o hello.o
# 或者
gcc -c helle.s -o hello.o

# 第四个阶段 链接 将所有的.o文件链接起来生成一个可执行程序。
# 链接的时候分为动态链接和静态链接
#                |--静态链接：在链接的时候使用了--static选项才会进行静态链接，它在编译时会将所有用到的
#                |           库打包到可执行程序中，所以静态链接生成的程序体积大，但是有更好的兼容性，不必
#                |           依赖外部环境。
#                |--动态链接：gcc编译时的默认选项。动态是指程序在运行的时候才会加载外部的代码库，
#                           不同的程序可以共用代码库，所以动态链接生成的程序体积小，占用的内存少。
# 静态链接生成的程序占用的存储空间更大。
gcc hello.o -o hello # 动态链接
gcc hello.c -o hello

gcc hello.o -o hello_static --static #静态链接
gcc hello.c -o hello_static --static
```

如下图所示，动态链接生成的程序的大小比静态链接生成程序小了**100倍**。动态链接库在Linux系统中以.so结尾，在windows系统中是.dll文件。

![202203282032587](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/202203301042226.png)

在Linux系统中可以使用ldd指令查看动态文件依赖的库

![202203281915472](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/202203301043264.png)

### 动态库和静态库

**文件形式：**

- 动态库：.so    .dll  结尾，前者是在linux系统中，后者是widowns。

- 静态库：.a      .lib  结尾，前者是在linux系统中，后者是widowns。

### 链接

gcc的链接参数：

- -l  -- 参数表示链接哪个库，会自动检索lib开头的对应库名。比如 -lpthread , -lQt5Core。会自动检索libpthread.so，libpthread.a，libQt5Core.so，libQt5Core.a。如果动态库和静态库同时存在，会优先选择链接动态库。

- -L -- 指定去哪里找库文件，比如指定-L/home/thread/test，在编译的时候则会优先检索/home/thread/test/libpthread.so等文件
