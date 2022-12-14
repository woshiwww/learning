## 一、概述

作用：编译和连接。

好处：自动化编译，一旦写好后，只需要一个make命令。

## 二、编译和连接

编译：将.c 或.cpp文件转换为.o文件。在unix下是.o ，在windows下是.obj文件，这些是可重定向文件。编译时使用`-c`

链接：把大量的.o文件合成一个执行文件。编译时使用`-o`

一个源文件会对应一个.c文件，但是如果源文件数量很多，会生成很多.o文件，在链接的时候需要明显的指出中间目标文件名，这对于编译很不方便，所以需要给中间文件打个包，生成一个静态库.a文件。

**主要过程：**源文件-->经过编译-->生成目标文件-->经过链接-->生成可执行文件

## 三、Makefile介绍

makefile文件是用来告诉make如何编译和链接.c文件和.h文件，主要由以下几个规则：

1. 如果这个工程文件没有编译过，那么所有的.c文件将会被编译并链接。
2. 如果有一些.c文件被修改过，那么被修改的.c文件将会被编译，并链接到目标程序。
3. 如果这个工程的头文件被修改了，那么需要编译引用了这个头文件的.c文件，并链接到目标程序。

**Makefile的规则：**

target：require files

​				command

target：指一个目标文件，也可以是可执行文件，还可以是一个标签。

require files： 指生成这个target所需要的.c文件或目标文件，target依赖于这些文件。如果这些文件中有一个比target文件新，下面的command就会被执行。

command ：指make需要执行的命令，主要是一些shell命令。**makefile中的命令之前，必须使用tab键**

```makefile
#makefile 有自动推导,他看到一个.o文件就会将.c文件加在依赖关系中,并且会自动推导命令。
# 比如
main.o:main.c main.h
	cc -c main.c
# 可以写成
main.o:main.h

# 每个makefile文件都应该写一个清空目标文件的规则，这样有利于重新编译，也利于保持文件清洁。
clean:
	rm main.o

# 或者
.PHONY:clean
clean:
	-rm main.o
.PHONY表示clean是一个伪目标，在rm前面加了一个-号，表示当某些文件出现问题时，不用管，继续执行后面的事情。clean规则不能放在文件的开头，不然会变成make默认的目标，一般都是放在最后面。
```

## 四、Makefile总述

### Makefile主要包括了五个东西

1. 显示规则。显示规则说明了如何生成一个或多个目标文件。由书写者明显指出要生成的文件、依赖的文件、生成命令等。
2. 隐晦规则。由make自动推导出依赖的.c文件和生成命令。
3. 定义变量。Makefile中定义的变量一般都是字符串，像c中的宏定义，其中的变量会被展开到引用的位置。
4. 文件指示。主要包括三个部分。
   - 一个makefile中引用另一个makefile，像c中的include。使用include关键字就能将别的makefile文件包含进来。`include filename1 filename2`，makefile文件之间可以用空格隔开。make会优先在当前目录下寻找这些文件，如果没有找到，那么make会去以下几个目录找。
     1. 如果make执行时，有`-I`或者`--include-dir`参数，那么make就会去这个参数指定的目录下寻找。
     2. 也会去`/usr/local/bin`或者`/usr/include`下寻找。如果也没有找到，就会产生错误。
   - 根据某些情况进行条件判断，指定makefile中的有效部分，就像c中的#if。
   - 定义一个多行命令。
   
5. 注释。makefile中只有行注释，使用#， 如果要使用#字符，可以使用\#进行转义。

### make的工作方式

1. 读入所有的Makefile文件。
2. 读入被include的其他makefile文件。
3. 初始化文件中的变量。
4. 推导隐晦规则，分析所有规则。
5. 为所有的目标文件创建依赖关系链。
6. 根据依赖关系决定哪些目标需要重新生成。
7. 执行生成命令。

### 文件搜寻
一个大工程项目中，通常含有很多的源文件在不同的目录下，当make去搜寻文件依赖关系时，可以在文件前面加上路径，但最好的方法是把一个路径告诉make，让它自己去找。Makefile中的特殊变量VPATH就具有这个功能，make只会在当前目录下去寻找文件，如果找不到，就会去指定的目录寻找。

`VPATH = src:../headers`此处定义了两个目录，不同的目录之间使用：分开，在最前面的目录具有最高优先级。

也可以通过`vpath`关键字来设置`vpath %.h src:../headers`在当前目录下没有找到文件，就去src或headers目录下寻找.h文件，同样在前面的目录具有更高的优先级，会优先去这个目录下寻找。

### 伪目标

之前提到的clean就是一个伪目标，通过make clean来使用伪目标。伪目标不是一个文件，而是一个标签。由于其不是文件，所以make无法生成它的依赖关系和决定它是否要执行，只能通过显示的指明这个目标，它才能生效。伪目标的名字不能和文件名字重复，不然就失去了伪目标的意义，比如伪目标clean和可执行文件clean。

为了避免重名的情况，可以使用特殊标记.PHONY来显示指明一个伪目标，向make说明，不管是否有这个文件，这个目标就是伪目标。

`.PHONY:clean`

伪目标一般没有依赖文件，但是也可以为伪目标指定所依赖的文件。当伪目标放在第一个时，也可以被当作默认目标。比如如果要生成多个可执行文件，但只想使用一次make，并且所有的目标文件写在一个makefile中，可以使用伪目标的特性。由于伪目标是肯定会被执行的，所有后面的依赖项永远没有all新，所以他们也会被执行。

```makefile
all:prog1 prog2 prog3
.PHONY:all
prog1:prog1.o 2.o
	cc -o prog1 prog1.o 2.o
prog2:prog2.o
	cc -o prog2 prog2.o
prog3:prog3.o 2.o 3.o
	cc -o prog3 prog3.o 2.o 3.o
```

除此之外，伪目标也可以成为依赖项。分别清楚不同种类的文件

```makefile
.PHONY:cleanall cleanobj cleandiff
cleanall:cleanobj cleandiff
	rm program
cleanobj:
	rm *.o
cleandiff:
	rm *.diff
```

### 静态模式

静态模式可以更容易的定义多目标的规则。

```makefile
#在这个例子中&(objects)是一个目标集，它后面的%.o指明了目标集的模式，指&(objects)集合中都是以.o结尾的，再后面的%.c指明了目标依赖的模式，对%.o所形成的目标集进行二次定义，去掉.o为其加上.c，形成新的集合。下面的这个例子说明了我们的目标从&(objects)中获取，取其中以%.o结尾的目标形成一个新的目标集，然后再形成一个依赖集以.c结尾。&<和&@是自动化变量。分别表示依赖目标集(foo.c bar.c)和目标集(foo.o bar.o)。
objects = foo.o bar.o
all : &(objects)
&(objects):%.o:%.c
	&(CC) -c &(CFLAGS) &< -o &@

# 上面等价于
foo.o : foo.c
	&(CC) -c &(CFLAGS) foo.c -o foo.o
bar.o : bar.c
	&(CC) -c &(CFLAGS) bar.c -o bar.o
```

### 输出依赖关系

```shell
#使用  -M 输出所有的依赖项包括系统中的.h
gcc -M example.c
example.o: example.c /usr/include/stdc-predef.h /usr/include/stdio.h \
 /usr/include/x86_64-linux-gnu/bits/libc-header-start.h \
 /usr/include/features.h /usr/include/x86_64-linux-gnu/sys/cdefs.h \
 /usr/include/x86_64-linux-gnu/bits/wordsize.h \
 /usr/include/x86_64-linux-gnu/bits/long-double.h \
 /usr/include/x86_64-linux-gnu/gnu/stubs.h \
 /usr/include/x86_64-linux-gnu/gnu/stubs-64.h \

#使用 -MM输出不包括系统文件的.h
gcc -MM example.c
example.o: example.c json/cJson.h
```

## 五、 书写命令

每条规则中的命令和操作系统中的shell命令是相同的，make会一条一条的去执行，每条命令必须以tab键开头。通常make会把将要执行的命令输出在屏幕上显示出来，如果想要屏蔽掉这个命令可以在命令之前使用@字符。

比如，在一个makefile文件中有如下信息

![image-20220903234516229](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/UNIX%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B2/202209032345289.png)

执行结果：

![image-20220903234545039](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/UNIX%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B2/202209032345073.png)

也可以在使用make的时候带上参数

```shell
make -n  #只显示命令
make -s #只显示禁止显示命令
make -w #进入一个目录和退出一个目录时会输出一些文件目录的信息
```

结果如下：

![image-20220903234827084](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/UNIX%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B2/202209032348117.png)

如果想要上一条的命令的结果应用在下一条时，应该使用分号隔开这两条命令，不然不会应用到下一条命令。

![image-20220903235419166](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/UNIX%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B2/202209032354198.png)

![image-20220903235428326](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/UNIX%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B2/202209032354357.png)

每次运行完一个命令之后，make会检查每个命令的返回码，如果命令返回成功，make继续执行下一条命令，返回失败则终止当前的规则，这很有可能终止所有的规则的执行。有时候命令出错并不代表错误，比如mkdir命令，如果要创建的目录存在那么将会返回失败，但我们使用这个命令的初衷就是要有一个这样的目录，于是我们就希望mkdir不要因为返回失败而终止规则的执行。可以通过-来忽略命令的错误，这样不管命令是否成功都会返回成功。

```makefile
clean:
	-mkdir test
```

还有一种全局的办法就是在使用make时加一个-i参数，`make -i`所有命令的错误都会被忽略。还有一个是`make -k`如果某个规则中命令出错了，那么终止这个规则的执行，但是继续执行其他规则。

## 六、嵌套执行make

在大型项目中，会将不同的模块或功能的源文件放在不同的目录中，我们可以在不同的目录中都编写一个该目录的makefile文件，这样有利于让makefile变得简洁，而不至于将所有的东西全都写在一个makefile中，导致makefile很难维护。在这种情况下一般会有一个总控makefile，总控中的显示声明的变量可以传递到下一级makefile文件中。假设现在有一个子目录subdir，其下有一个makefile文件。

```makefi
subsystem:
	cd subdir && $(MAKE)
	
#等价于
subsystem:
	$(MAKE) -C subdir   #当有-C时，后面跟的是一个目录，表示在执行任何指令之前先进入到这个目标下，
						#此时下层makefile 的-w会被自动打开
#这两个例子的意思都是进入sundir然后执行make命令
```

总控中的变量可以传递到下一级，但是不会覆盖下一级makefile中的变量，除非指定了-e参数，如果想传递变量到下一级makefile，可以这样声明，下面三条声明都是等价的。如果没有export声明的变量是不会传递到下一层makefile的。

1--`export variable = value`  

等价于：

`variable = value`

`export variable`

2--`export variable += value`

3--`export variable ：= value`

如果要传递所有的变量只需要一个export就行，后面什么也不用跟，如果不想让某些变量传递到下一级makefile中可以使用`unexport variable`

## 七、变量基础

变量大小写敏感`foo Foo FOO`代表三个不同的变量，makefile中的变量一般都是全部大写，但是推荐使用驼峰法，这样可以有效的和系统变量避免冲突，减少发生意外的事情。

在makefile中定义的变量就像c中的宏一样，表示一个文本串，会在使用的地方展开。与c中的宏定义不同的是，makefile中变量的值是可以改变的。变量可以用在目标、依赖、命令三个地方，或者是makefile中的其他部分。如下：

```makefi
Objects = foo.o utils.o
program : $(Objects)   #依赖中
	cc -o program $(Objects)  # 命令中
$(Objects):def.h  #目标中

#变量也会在相应的地方精确展开，比如
foo = c
prog.o : prog.$(foo)
	$(foo)$(foo) -$(foo) prog.$(foo)
```

### 变量中的变量

在定义变量的时候，可以使用其他变量来构造变量的值，右侧的变量不一定是定义好的值，也可以在稍后进行定义。

```makefile
foo = &(bar)
bar = uff

#但是如果使用的是 :=操作符，这种方法让前面的变量不能使用后面的变量，只能使用前面已经定义好的变量
x := foo
y := $(x) bar
x := later
#等价于
y := foo bar
x := later

#那如果想定义一个变量的值是一个空格怎么办呢？
nullstring := 
space := $(nullstring) #定义一个空格值
#也可以这样
space := $(nullstring) $(nullstring)
# 这里nullstring变量是一个空值，什么也没有，而space的值是一个空格，先使用一个空变量来表明变量的值开始了，然后再使用#注释符来表示变量定义的终止，这样就能定义出一个空格变量。

# 如果定义了一个这个变量
dir := /foo/bar    #定义了一个文件路径
# 在使用这个变量的时候，这个变量的值是"/foo/bar    "后面还有四个空格
```

在变量赋值中还有一个操作符`?=`

`FOO ?= bar`表示如果FOO没有被定义过，那么它的值等于bar

### 追加变量的值

追加变量的值使用`+=`操作符,如果变量之前没有被定义过那么`+=`就会变成`=`。

```makefile
objects = 1.o 2.o
objects += 3.o
#此时objects的值就成为了1.o 2.o 3.o,其实它就等价于
objects = 1.o 2.o
objects := &(objects) 3.o
```

## 八、函数的使用

函数的调用和变量的使用有点像`&(<function> <arguments>)`参数之间以`,`隔开，函数和参数之间以空格隔开。

### 字符串处理函数

示例：

```makefile
comma := ,
empty := 
space := $(empty) $(empty)
foo := a b c
bar := &(subst $(space),$(comma),$(foo))
```

这个函数的意思是将foo中的空格替换成`,`，subst是一个替换函数，第一个参数是被替换的字符串，第二个是替换操作用的字符，第三个是替换操作用的字符串。即`a b c`变成了`a,b,c`

```makefile
$(subst <form>,<to>,<text>)
	功能：把字符串<text>中的<form>替换成<to>
	返回值：返回被替换过的字符串
```

## 九、make的运行

最简单的方式是直接输入`make`，他会找当前目录的makefile文件来执行。

make命令有三个退出码：

1. 0--表示执行成功
2. 1--make运行时出现任何错误，都会返回1
3. 2--make时使用了-q选项，make使得一些目标不更新，那么将会返回2

指定执行特定的makefile文件:

`make -f filename`

## 十、命令中的变量

AR

- 函数库打包程序，默认命令是“ar”

CC

- C语言编译程序，默认命令是“cc”

CXX

- C++语言编译程序，默认命令是g++

RM

- 删除文件命令。默认命令是“rm -f”

ARFLAGS

- 函数库打包程序AR命令的参数，默认值是“rv”

CFLAGS

- C语言编译器参数

CXXFLAGS

- C++语言编译器参数
