duke c programming - Fancier make options





```makefile
CFLAGS=-std=gnu99 -pedantic -Wall
myProgram: oneFile.o anotherFile.o
    gcc -o myProgram oneFile.o anotherFile.o
oneFile.o: oneFile.c oneHeader.h someHeader.h
    gcc $(CFLAGS) -c oneFile.c
anotherFile.o: anotherFile.c anotherHeader.h someHeader.h
    gcc $(CFLAGS) -c anotherFile.c
```

### Generic rules

当我们使用一个变量来保存编译标志时，我们的示例Makefile略有改进。然而，我们的Makefile仍然有很多重复的地方，如果我们有几个以上的源文件，维护起来就会很麻烦。如果你看一下我们写的东西，我们正在做几乎相同的事情，将我们的每个.c源文件编译成一个对象文件。每当我们发现自己在重复的时候，应该有一个更好的方法。

在make中，我们可以编写通用规则。一个通用规则可以让我们指定，我们希望能够从`(something).c`构建`(something).o`，其中我们用百分号(`%`)表示something。作为第一步，我们可以这样写（注意，#表示行末的注释）。



```makefile
# A good start, but we lost the dependencies on the header files
CFLAGS=-std=gnu99 -pedantic -Wall
myProgram: oneFile.o anotherFile.o
    gcc -o myProgram oneFile.o anotherFile.o
%.o: %.c
    gcc $(CFLAGS) -c $<
.PHONY: clean
clean:
    rm -f myProgram *.o *.c~ *.h~
```

在这里，我们用一条通用规则取代了我们为每个对象文件制定的两条规则。它规定了如何从一个同名的文件中制作一个以`.o`结尾的文件，只是用`.c`而不是`.o`。在这个规则中，我们不能写源文件的字面名称，因为它在规则的每个实例中都会改变。相反，我们必须使用特殊的内置变量`$<`，make会将其设置为规则的第一个前提条件的名称（在本例中，是`.c`文件的名称）。

然而，我们现在引入了一个重要的问题。我们已经使我们的对象文件不再依赖于相关的头文件。如果我们要改变一个头文件，那么make可能不会重建所有相关的对象文件。这样的错误会导致奇怪和混乱的bug，因为一个对象文件可能期望在一个旧的布局中的数据，但现在代码将被传递在一个不同的布局中的数据。我们可以让每个对象文件都**依赖**于每个头文件（通过写`%.o : %.c *.h`），然而，这种方法是过犹不及的--当我们改变一个头文件时，我们肯定会重建所有需要的东西，因为我们要重建每个对象文件，即使我们只需要重建几个。

我们可以通过在我们的Makefile中添加额外的依赖信息，以更好的方式解决这个问题。

```makefile
# This fixes the problem
CFLAGS=-std=gnu99 -pedantic -Wall
myProgram: oneFile.o anotherFile.o
    gcc -o myProgram oneFile.o anotherFile.o
%.o: %.c
    gcc $(CFLAGS) -c $<

.PHONY: clean
clean:
    rm -f myProgram *.o *.c~ *.h~

oneFile.o: oneHeader.h someHeader.h
anotherFile.o: anotherHeader.h someHeader.h
```



在这里，我们仍然有通用规则，但我们也单独指定了额外的依赖关系。尽管看起来我们有两条规则，但要明白，我们只是提供额外的依赖信息，因为我们没有指定任何命令。如果我们指定了命令，它们将取代这些目标的通用规则。

当然，手工管理所有这些依赖信息将是乏味和容易出错的。程序员必须弄清楚每个源文件所包含的每一个文件，并随着代码的变化保持这些信息的更新。相反，有一个叫makedepend的工具，它可以编辑Makefile，把所有这些信息放在最后。在最简单的用法中，makedepend把所有的源文件（即所有的.c和/或.cpp文件）作为参数，并编辑Makefile。它也可以被赋予各种选项，比如-I path，告诉它在指定的路径中寻找包含文件。参见` man makedepend `获取更多细节。

### Built-in generic rules

有些通用规则非常普遍，以至于它们被内置于make中，而我们甚至不需要写它们。正如你可能怀疑的那样，从类似的.c文件中建立一个.o文件是很常见的，因为这是C程序员最常做的事情。因此，如果我们对这种模式的内置通用规则感到满意的话，我们甚至不需要明确地编写%.o: %.c规则。

我们可以通过使用`make -p`查看所有的规则（包括那些内置的和由Makefile指定的）。如果我们想避免构建任何东西，我们可以使用make -p -f /dev/null来使用特殊文件/dev/null作为我们的Makefile（从/dev/null读取的结果是立即结束文件，所以结果将是一个没有规则的Makefile，因此它不会做任何事情）。

如果我们使用`make -p`来探索从`.c`文件构建`.o`文件的内置规则，我们会发:

```makefile
%.o: %.c
# commands to execute (built-in):
    $(COMPILE.c) $(OUTPUT_OPTION) $<
```

理解这个规则需要我们看一下`COMPILE.c`和`OUTPUT_OPTION`的定义，它们也包含在`make -p`的输出中。

```makefile
COMPILE.c = $(CC) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
OUTPUT_OPTION = -o $@
```

默认情况下，`CFLAGS`（C编译器的标志）和`CPPFLAGS`（C预处理器的标志143），以及`TARGET_ARCH`（指定目标架构的标志）是空的。默认情况下，CC（C-编译器命令）是cc（可能是也可能不是gcc，取决于我们系统的配置）。这些变量（或任何其他变量）的默认值都可以通过在Makefile中指定它们的值而被覆盖。注意，OUTPUT_OPTION中的`$@`是一个特殊的变量，它是当前目标的名称（就像`$<`是第一个先决条件的名称）。

所有这些听起来可能有点复杂，但基本上可以归结为这样一个事实：默认规则是：`cc -c -o something.o something.c`，我们可以在使用默认规则的同时，覆盖具体内容以获得我们想要的行为。也就是说，我们可以使用下面的Makefile:

```makefile
CC = gcc
CFLAGS = -std=gnu99 -pedantic -Wall
myProgram: oneFile.o anotherFile.o
    gcc -o myProgram oneFile.o anotherFile.o
   
.PHONY: clean depend
clean:
    rm -f myProgram *.o *.c~ *.h~
depend:
    makedepend anotherFile.c oneFile.c
# DO NOT DELETE
anotherFile.o: anotherHeader.h someHeader.h
oneFile.o: oneHeader.h someHeader.h
```

在这里，我们已经指定要使用gcc作为C-compiler（CC），并指定了我们想要的CFLAGS。现在，当我们试图从C文件编译一个对象文件时，默认的规则将导致

```makefile
 gcc -std=gnu99 -pedantic -Wall -c -o something.o something.c
```

你还应该注意到，我们添加了另一个假目标，`depend`，它用我们正在处理的两个C源文件运行makedepend。`# DO NOT DELETE`一行和它下面的所有内容是我运行`make depend`时makedepend添加到我们的`Makefile`中的。注意，如果我们重新运行`makedepend`（最好是通过`make depend`），它将寻找这一行来告诉我们在哪里删除旧的依赖信息并添加新的信息。



### Built-in functions

我们的`Makefile`看起来更像是我们可以在一个大项目中使用的东西，但我们仍然在几个地方手动列出了我们的源文件和对象文件。如果我们添加了一个新的源文件，但忘记更新`makedepend`命令行，那么当我们运行`make depend`时，我们将无法获得该文件的正确依赖关系。同样，我们可能忘记在正确的地方添加对象文件（例如，如果我们把它添加到编译命令行中，但没有添加整个程序的依赖关系，我们可能在需要时没有重建该对象文件）。



我们可以通过使用make的一些内置函数来解决这些问题，自动计算出当前目录中的.c文件集，然后从该列表中生成目标对象文件的列表。make中函数调用的语法是`$(functionName arg1, arg2, arg3)`。我们可以使用`$(wildcard pattern)`函数来生成当前目录中的.c文件列表。`SRCS = $(wildcard *.c)`。然后我们可以使用`$(patsubst pattern, replacement, text)`函数将`.c`的结尾替换为`.o`的结尾。`OBJS = $(patsubst %.c, %.o, $(SRCS))`。一旦我们完成了这些，我们就可以在我们的Makefile中使用`$(SRCS)`和`$(OBJS)`。

```makefile
CC = gcc
CFLAGS = -std=gnu99 -pedantic -Wall
SRCS=$(wildcard *.c)
OBJS=$(patsubst %.c,%.o,$(SRCS))
myProgram: $(OBJS)
    gcc -o $@ $(OBJS)
.PHONY: clean depend
clean:
    rm -f myProgram *.o *.c~ *.h~
depend:
    makedepend $(SRCS)
# DO NOT DELETE
anotherFile.o: anotherHeader.h someHeader.h
oneFile.o: oneHeader.h someHeader.h
```

在这一点上，我们有一个可以用于大规模项目的Makefile。当我们添加源文件或在现有源文件中包含新的头文件时，我们唯一需要做的是运行`make depend`来更新依赖信息。除此以外，我们可以用make来构建我们的项目，它只会重新编译所需的文件。

然而，我们可以做得更细致一些。在一个真实的项目中，我们可能想建立一个调试版本的代码（没有优化，`-ggdb3`打开调试信息--关于调试的更多信息见下一个模块），和一个优化版本的代码，使其运行得更快（编译器努力工作以改进其生成的指令，但这些转换通常使调试相当困难）。我们可以在用于调试的标志和用于优化的标志之间来回改变我们的`CFLAGS`，并且记得每次切换时都要做清洁。然而，我们也可以将我们的Makefile设置为同时建立调试和优化的对象文件和不同名称的二进制文件。

```makefile
CC = gcc
CFLAGS = -std=gnu99 -pedantic -Wall -O3
DBGFLAGS = -std=gnu99 -pedantic -Wall -ggdb3 -DDEBUG
SRCS=$(wildcard *.c)
OBJS=$(patsubst %.c,%.o,$(SRCS))
DBGOBJS=$(patsubst %.c,%.dbg.o,$(SRCS))

.PHONY: clean depend all
all: myProgram myProgram-debug
myProgram: $(OBJS)
	gcc -o $@ -O3 $(OBJS)
myProgram-debug: $(DBGOBJS)
	gcc -o $@ -ggdb3 $(DBGOBJS)
%.dbg.o: %.c
	gcc $(DBGFLAGS) -c -o $@ $<
clean:
	rm -f myProgram myProgram-debug *.o *.c~ *.h~
depend:
	makedepend $(SRCS)
	makedepend -a -o .dbg.o $(SRCS)
# DO NOT DELETE
anotherFile.o: anotherHeader.h someHeader.h
oneFile.o: oneHeader.h someHeader.h
```

### Parallelizing computation

`make`的一个有用的功能，特别是在现代多核系统上，是能够让它并行地运行独立的任务。如果你给make加上`-j`选项，它就会要求它尽可能多地并行运行任务。你可能希望要求它在任何时候将并行任务的数量限制在一个特定的数字上，你可以通过指定这个数字作为-j选项的参数来实现（，`make -j8`最多并行运行8个任务）。在大型项目中，这可能会对构建的时间产生重大影响。



你可以用`make`来做更多的事情，而不仅仅是编译C程序。事实上，你可以用make来完成几乎所有你能描述的任务，即从它们所依赖的先决条件中创建目标。对于大多数这样的任务，你可以很好地利用make的并行化能力来加快任务的进度。

我们已经给了你足够的介绍，让你写一个基本的Makefile。然而，就像许多主题一样，我们几乎没有触及表面。你可以通过阅读在线手册找到更多关于make的信息。

[https://www.gnu.org/software/make/manual/](https://www.gnu.org/software/make/manual/) 



### compiler options

`-Werror`选项告诉编译器将所有的警告视为错误，使它拒绝编译程序，直到程序员修复所有的警告。

新手程序员常常想，为什么一开始就要启用警告，特别是要求编译器把警告转换成错误。错误使你的程序无法编译，所以似乎任何能避免错误的方法都是好的，都能得到已编译的二进制文件。然而，程序员的目标不仅仅是产生任何可执行的程序，而是产生一个能在手头正确工作的程序。只要编译器能够提醒你注意潜在的问题，你就能比导致你不得不调试的问题更容易地解决它们。一个好的比喻是购买房子。把编译器想象成你的房屋检查员。虽然你急于购买并搬进新房子，但你更希望检查员在你购买房子并搬进去之前发现一个潜在的问题。



我们强烈建议在编译时至少要有以下警告选项。

```sh
-Wall -Wsign-compare -Wwrite-strings -Wtype-limits -Werror
```

这些选项将有助于捕捉大量的错误，而且不应该对正确编写的代码构成不适当的负担。

最近版本的gcc还支持一个选项`-fsanitize=address`，它将生成包括额外检查的代码，以帮助在运行时检测各种问题。在你的开发周期中，也强烈建议使用这个选项。然而，我们将注意到（至少在gcc 4.8.2和valgrind 3.10.0中）你不能在valgrind中运行用这个选项构建的程序。这两个工具检测的问题不同，但又相互重叠，所以使用这两个工具是个好主意--它们必须分开使用。




Question 5

If you valgrind your program and get the error "Conditional jump or move depends on uninitialized value(s)," what should you do to get more information?

Run valgrind with `--track-origins=yes` to determine the source of the error.
