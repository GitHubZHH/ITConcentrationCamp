## 一、什么
网上介绍得很多，总之 Makefile 也是一个脚本文件。这种脚本的文件的文件名是很有讲究的，通常为 **Makefile** 与 **makefile**，不需要任何的后缀，更多的时候是 **Makefile**。
是的，就是这么的霸气，如果文件名不是这样的，name 可能在后面你就无法执行。由于文件名的唯一性，所以在执行的时候可以不带文件中，直接这样就会找到当前文件夹中的 **Makefile** 进行执行：
 > make

是的，你依然没有看错，就是这么简单。当然了，前提是在当前的文件夹中有一个正确的 **Makefile** 文件。

## 二、简单的语法
网上都是这么说的：
```
target ... : prerequisites ...
            command
            ...
            ...

```

所以我反手就写出了这样的代码：

![image.png](https://upload-images.jianshu.io/upload_images/1198135-e2a1df0449bcb234.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

运行结果：

![image.png](https://upload-images.jianshu.io/upload_images/1198135-a9fb1c89792ed840.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

是不是很简单，反手就是一个赞。

## 三、总结
我上面介绍的，仅仅是我现在的一点理解，皮毛都算不上。可以去研读一下我所参考的文章。

## 参考文章
[跟我一起写 Makefile](https://blog.csdn.net/haoel/article/details/2886)
[Makefile简易教程](https://www.jianshu.com/p/ff0e0e26c47a)


在这里引用一下[Makefile简易教程](https://www.jianshu.com/p/ff0e0e26c47a)的文章：

### 从一个简单的例子开始
我们可以用K&R C中4.5那个例子来做个说明。在这个例子中，我们会看到一份主程序代码(main.c)、三份函数代码(getop.c、stack.c、getch.c)以及一个头文件(calc.h)。通常情况下，我们需要这样编译它：

```
    gcc -o calc main.c getch.c getop.c stack.c 
```

如果没有makefile，在开发+调试程序的过程中，我们就需要不断地重复输入上面这条编译命令，要不就是通过终端的历史功能不停地按上下键来寻找最近执行过的命令。这样做两个缺陷：

一旦终端历史记录被丢失，我们就不得不从头开始；

任何时候只要我们修改了其中一个文件，上述编译命令就会重新编译所有的文件，当文件足够多时这样的编译会非常耗时。

那么Makefile又能做什么呢？我们先来看一个最简单的makefile文件：

```
    calc: main.c getch.c getop.c stack.c
        gcc -o calc main.c getch.c getop.c stack.c 
```

现在你看到的就是一个最基本的Makefile语句，它主要分成了三个部分，第一行冒号之前的calc，我们称之为目标（target），被认为是这条语句所要处理的对象，具体到这里就是我们所要编译的这个程序calc。冒号后面的部分（main.c getch.c getop.c stack.c），我们称之为依赖关系表，也就是编译calc所需要的文件，这些文件只要有一个发生了变化，就会触发该语句的第三部分，我们称其为命令部分，相信你也看得出这就是一条编译命令。现在我们只要将上面这两行语句写入一个名为Makefile或者makefile的文件，然后在终端中输入make命令，就会看到它按照我们的设定去编译程序了。

请注意，在第二行的“gcc”命令之前必须要有一个tab缩进。语法规定Makefile中的任何命令之前都必须要有一个tab缩进，否则make就会报错。
接下来，让我们来解决一下效率方面的问题，先初步修改一下上面的代码：

```
    cc = gcc
    prom = calc
    src = main.c getch.c getop.c stack.c
     
    $(prom): $(src)
        $(cc) -o $(prom) $(src)
```

如你所见，我们在上述代码中定义了三个常量cc、prom以及src。它们分别告诉了make我们要使用的编译器、要编译的目标以及源文件。这样一来，今后我们要修改这三者中的任何一项，只需要修改常量的定义即可，而不用再去管后面的代码部分了。

请注意，很多教程将这里的cc、prom和src称之为变量，个人认为这是不妥当的，因为它们在整个文件的执行过程中并不是可更改的，作用也仅仅是字符串替换而已，非常类似于C语言中的宏定义。或者说，事实上它就是一个宏。
但我们现在依然还是没能解决当我们只修改一个文件时就要全部重新编译的问题。而且如果我们修改的是calc.h文件，make就无法察觉到变化了（所以有必要为头文件专门设置一个常量，并将其加入到依赖关系表中）。下面，我们来想一想如何解决这个问题。考虑到在标准的编译过程中，源文件往往是先被编译成目标文件，然后再由目标文件连接成可执行文件的。我们可以利用这一点来调整一下这些文件之间的依赖关系：

```
    cc = gcc
    prom = calc
    deps = calc.h
    obj = main.o getch.o getop.o stack.o
     
    $(prom): $(obj)
        $(cc) -o $(prom) $(obj)

    main.o: main.c $(deps)
        $(cc) -c main.c

    getch.o: getch.c $(deps)
        $(cc) -c getch.c

    getop.o: getop.c $(deps)
        $(cc) -c getop.c

    stack.o: stack.c $(deps)
        $(cc) -c stack.c
```

这样一来，上面的问题显然是解决了，但同时我们又让代码变得非常啰嗦，啰嗦往往伴随着低效率，是不祥之兆。经过再度观察，我们发现所有.c都会被编译成相同名称的.o文件。我们可以根据该特点再对其做进一步的简化：

```
    cc = gcc
    prom = calc
    deps = calc.h
    obj = main.o getch.o getop.o stack.o
    
    $(prom): $(obj)
        $(cc) -o $(prom) $(obj)

    %.o: %.c $(deps)
        $(cc) -c $< -o $@
```

在这里，我们用到了几个特殊的宏。首先是%.o:%.c，这是一个模式规则，表示所有的.o目标都依赖于与它同名的.c文件（当然还有deps中列出的头文件）。再来就是命令部分的@，其中^），具体到我们这里就是%.c。而$@代表的是当前语句的目标，即%.o。这样一来，make命令就会自动将所有的.c源文件编译成同名的.o文件。不用我们一项一项去指定了。整个代码自然简洁了许多。

到目前为止，我们已经有了一个不错的makefile，至少用来维护这个小型工程是没有什么问题了。当然，如果要进一步增加上面这个项目的可扩展性，我们就会需要用到一些Makefile中的伪目标和函数规则了。例如，如果我们想增加自动清理编译结果的功能就可以为其定义一个带伪目标的规则；

```
    cc = gcc
    prom = calc
    deps = calc.h
    obj = main.o getch.o getop.o stack.o
    
    $(prom): $(obj)
        $(cc) -o $(prom) $(obj)

    %.o: %.c $(deps)
        $(cc) -c $< -o $@

    clean:
        rm -rf $(obj) $(prom)
```

有了上面最后两行代码，当我们在终端中执行make clean命令时，它就会去删除该工程生成的所有编译文件。

另外，如果我们需要往工程中添加一个.c或.h，可能同时就要再手动为obj常量再添加第一个.o文件，如果这列表很长，代码会非常难看，为此，我们需要用到Makefile中的函数，这里我们演示两个：

```
    cc = gcc
    prom = calc
    deps = $(shell find ./ -name "*.h")
    src = $(shell find ./ -name "*.c")
    obj = $(src:%.c=%.o) 
    
    $(prom): $(obj)
        $(cc) -o $(prom) $(obj)

    %.o: %.c $(deps)
        $(cc) -c $< -o $@

    clean:
        rm -rf $(obj) $(prom)
```

其中，shell函数主要用于执行shell命令，具体到这里就是找出当前目录下所有的.c和.h文件。而$(src:%.c=%.o)则是一个字符替换函数，它会将src所有的.c字串替换成.o，实际上就等于列出了所有.c文件要编译的结果。有了这两个设定，无论我们今后在该工程加入多少.c和.h文件，Makefile都能自动将其纳入到工程中来。

到这里，我们就基本上将日常会用到的Makefile写法介绍了一遍。如果你想了解更多关于makefile和make的知识，请参考 [GNU Make Manual](http://www.cs.utexas.edu/~cannata/cs345/GNU%20Make%20Manual.pdf)。

