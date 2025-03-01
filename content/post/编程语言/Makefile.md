# 参考链接

[跟我一起写Makefile — 跟我一起写Makefile 1.0 文档 (seisman.github.io)](https://seisman.github.io/how-to-write-makefile/index.html)

# 概述

```makefile
targets: prerequisites 
    commands
targets:prerequisties;commands
```

*`targets`*

- 可以是一个object file（目标文件），也可以是一个可执行文件，还可以是一个标签（label）。
- targets是文件名，以空格分开，可以使用通配符。一般来说，我们的目标基本上是一个文件，但也有可能是多个文件。

*`prerequisites`*

- 目标所依赖的文件（或依赖目标）。如果其中的某个文件要比目标文件要新，那么，目标就被认为是“过时的”，被认为是需要重生成的。

*`commands`*

- 该target要执行的命令（任意的shell命令）。
- commands是命令行，如果其不与“target:prerequisites”在一行，那么，必须以 `Tab` 键开头。
- 如果和prerequisites在一行，那么可以用分号做为分隔。（见上）
- 如果命令太长，可以使用反斜杠（ `\` ）作为换行符。make对一行上有多少个字符没有限制。

>  [!NOTE]
>
> 这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。说白一点就是说:
>
> ==prerequisites中如果有一个以上的文件比target文件要新的话，commands所定义的命令就会被执行。==

>  [!IMPORTANT]
>
> 注意：makefile有着严格的缩进格式，命令前面需要使用tab键对齐

下面将会按照以下的例子来介绍相关知识：

```makefile
edit : main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o

main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
clean :
    rm edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
```

反斜杠（ `\` ）是换行符的意思。这样比较便于makefile的阅读。我们可以把这个内容保存在名字为“makefile”或“Makefile”的文件中，然后在该目录下直接输入命令 `make` 就可以生成执行文件edit。如果要删除可执行文件和所有的中间目标文件，那么，只要简单地执行一下 `make clean` 就可以了。

在这个makefile中，目标文件（target）包含：可执行文件edit和中间目标文件（ `*.o` ），依赖文件（prerequisites）就是冒号后面的那些 `.c` 文件和 `.h` 文件。每一个 `.o` 文件都有一组依赖文件，而这些 `.o` 文件又是可执行文件 `edit` 的依赖文件。依赖关系的实质就是说明了目标文件是由哪些文件生成的，换言之，目标文件是哪些文件更新的。

在定义好依赖关系后，后续的recipe行定义了如何生成目标文件的操作系统命令，一定要以一个 `Tab` 键作为开头。记住，make并不管命令是怎么工作的，他只管执行所定义的命令。make会比较targets文件和prerequisites文件的修改日期，如果prerequisites文件的日期要比targets文件的日期要新，或者target不存在的话，那么，make就会执行后续定义的命令。

这里要说明一点的是， `clean` 不是一个文件，它只不过是一个动作名字，有点像C语言中的label一样，其冒号后什么也没有，那么，make就不会自动去找它的依赖性，也就不会自动执行其后所定义的命令。要执行其后的命令，就要在make命令后明显得指出这个label的名字。这样的方法非常有用，我们可以在一个makefile中定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份，等等。

## 工作原理

在默认的方式下，也就是我们只输入 `make` 命令。那么文件就需要满足以下的要求：

1. make会在当前目录下找名字叫“Makefile”或“makefile”的文件。
2. 如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到“edit”这个文件，并把这个文件作为最终的目标文件。
3. 如果edit文件不存在，或是edit所依赖的后面的 `.o` 文件的文件修改时间要比 `edit` 这个文件新，那么，他就会执行后面所定义的命令来生成 `edit` 这个文件。
4. 如果 `edit` 所依赖的 `.o` 文件也不存在，那么make会在当前文件中找目标为 `.o` 文件的依赖性，如果找到则再根据那一个规则生成 `.o` 文件。（这有点像一个堆栈的过程）
5. 当然，你的C文件和头文件是存在的啦，于是make会生成 `.o` 文件，然后再用 `.o` 文件生成make的终极任务，也就是可执行文件 `edit` 了。

这就是整个make的依赖性，make会一层又一层地去找文件的依赖关系，直到最终编译出第一个目标文件。在找寻的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make就会直接退出，并报错，而对于所定义的命令的错误，或是编译不成功，make根本不理。==make只管文件的依赖性==，即，如果在我找了依赖关系之后，冒号后面的文件还是不在，那么对不起，我就不工作啦。

通过上述分析，我们知道，像clean这种，没有被第一个目标文件直接或间接关联，那么它后面所定义的命令将不会被自动执行，不过，我们可以显示要make执行。即命令—— `make clean` ，以此来清除所有的目标文件，以便重编译。

于是在我们编程中，如果这个工程已被编译过了，当我们修改了其中一个源文件，比如 `file.c` ，那么根据我们的依赖性，我们的目标 `file.o` 会被重编译（也就是在这个依性关系后面所定义的命令），于是 `file.o` 的文件也是最新的啦，于是 `file.o` 的文件修改时间要比 `edit` 要新，所以 `edit` 也会被重新链接了（详见 `edit` 目标文件后定义的命令）。

而如果我们改变了 `command.h` ，那么， `kdb.o` 、 `command.o` 和 `files.o` 都会被重编译，并且， `edit` 会被重链接。

## 使用变量

现有以下的例子：

```makefile
edit : main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
```

我们可以看到 `.o` 文件的字符串被重复了两次，如果我们的工程需要加入一个新的 `.o` 文件，那么我们需要在两个地方加（应该是三个地方，还有一个地方在clean中）。当然，我们的makefile并不复杂，所以在两个地方加也不累，但如果makefile变得复杂，那么我们就有可能会忘掉一个需要加入的地方，而导致编译失败。所以，为了makefile的易维护，在makefile中我们可以使用变量。makefile的变量也就是一个字符串，理解成C语言中的宏可能会更好。

比如，我们声明一个变量，叫 `objects` ， `OBJECTS` ， `objs` ， `OBJS` ， `obj` 或是 `OBJ` ，反正不管什么啦，只要能够表示obj文件就行了。我们在makefile一开始就这样定义：

```makefile
objects = main.o kbd.o command.o display.o \
     insert.o search.o files.o utils.o
```

于是，我们就可以很方便地在我们的makefile中以 `$(objects)` 的方式来使用这个变量了，于是我们的改良版makefile就变成下面这个样子：

```makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)
main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
clean :
    rm edit $(objects)
```

于是如果有新的 `.o` 文件加入，我们只需简单地修改一下 `objects` 变量就可以了。

## 自动推导

GNU的make很强大，它可以自动推导文件以及文件依赖关系后面的命令，于是我们就没必要去在每一个 `.o` 文件后都写上类似的命令，因为，我们的make会自动识别，并自己推导命令。

只要make看到一个 `.o` 文件，它就会自动的把 `.c` 文件加在依赖关系中，如果make找到一个 `whatever.o` ，那么 `whatever.c` 就会是 `whatever.o` 的依赖文件。并且 `cc -c whatever.c` 也会被推导出来，于是，我们的makefile再也不用写得这么复杂。我们的新makefile又出炉了。

```makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)

main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
    rm edit $(objects)
```

这种方法就是make的“隐式规则”。上面文件内容中， `.PHONY` 表示 `clean` 是个伪目标文件。

## 另一种风格

既然我们的make可以自动推导命令，那么我看到那堆 `.o` 和 `.h` 的依赖就有点不爽，那么多的重复的 `.h` ，能不能把其收拢起来，好吧，没有问题，这个对于make来说很容易，谁叫它提供了自动推导命令和文件的功能呢？来看看最新风格的makefile吧。

```makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)

$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h

.PHONY : clean
clean :
    rm edit $(objects)
```

这里 `defs.h` 是所有目标文件的依赖文件， `command.h` 和 `buffer.h` 是对应目标文件的依赖文件。

这种风格能让我们的makefile变得很短，但我们的文件依赖关系就显得有点凌乱了。鱼和熊掌不可兼得。

## 清空目录

每个Makefile中都应该写一个清空目标文件（ `.o` ）和可执行文件的规则，这不仅便于重编译，也很利于保持文件的清洁。这是一个“修养”。一般的风格都是：

```makefile
clean:
    rm edit $(objects)
```

更为稳健的做法是：

```makefile
.PHONY : clean
clean :
    -rm edit $(objects)
```

前面说过， `.PHONY` 表示 `clean` 是一个“伪目标”。而在 `rm` 命令前面加了一个小减号的意思就是，也许某些文件出现问题，但不要管，继续做后面的事。当然， `clean` 的规则不要放在文件的开头，不然，这就会变成make的默认目标，相信谁也不愿意这样。不成文的规矩是——“clean从来都是放在文件的最后”。

## Makefile里有什么？

Makefile里主要包含了五个东西：显式规则、隐式规则、变量定义、指令和注释。

1. 显式规则。显式规则说明了如何生成一个或多个目标文件。这是由Makefile的书写者明显指出要生成的文件、文件的依赖文件和生成的命令。
2. 隐式规则。由于我们的make有自动推导的功能，所以隐式规则可以让我们比较简略地书写Makefile，这是由make所支持的。
3. 变量的定义。在Makefile中我们要定义一系列的变量，变量一般都是字符串，这个有点像你C语言中的宏，当Makefile被执行时，其中的变量都会被扩展到相应的引用位置上。
4. 指令。其包括了三个部分，一个是在一个Makefile中引用另一个Makefile，就像C语言中的include一样；另一个是指根据某些情况指定Makefile中的有效部分，就像C语言中的预编译#if一样；还有就是定义一个多行的命令。有关这一部分的内容，我会在后续的部分中讲述。
5. 注释。Makefile中只有行注释，和UNIX的Shell脚本一样，其注释是用 `#` 字符，这个就像C/C++中的 `//` 一样。如果你要在你的Makefile中使用 `#` 字符，可以用反斜杠进行转义，如： `\#` 。

最后，还值得一提的是，在Makefile中的命令，必须要以 `Tab` 键开始。

## 文件名

默认的情况下，make命令会在当前目录下按顺序寻找文件名为 `GNUmakefile` 、 `makefile` 和 `Makefile` 的文件。在这三个文件名中，最好使用 `Makefile` 这个文件名，因为这个文件名在排序上靠近其它比较重要的文件，比如 `README`。最好不要用 `GNUmakefile`，因为这个文件名只能由GNU `make` ，其它版本的 `make` 无法识别，但是基本上来说，大多数的 `make` 都支持 `makefile` 和 `Makefile` 这两种默认文件名。

当然，你可以使用别的文件名来书写Makefile，比如：“Make.Solaris”，“Make.Linux”等，如果要指定特定的Makefile，你可以使用make的 `-f` 或 `--file` 参数，如： `make -f Make.Solaris` 或 `make --file Make.Linux` 。如果你使用多条 `-f` 或 `--file` 参数，你可以指定多个makefile。

## 包含

在Makefile使用 `include` 指令可以把别的Makefile包含进来，这很像C语言的 `#include` ，被包含的文件会原模原样的放在当前文件的包含位置。 `include` 的语法是：

```
include <filenames>...
```

`<filenames>` 可以是当前操作系统Shell的文件模式（可以包含路径和通配符）。

在 `include` 前面可以有一些空字符，但是绝不能是 `Tab` 键开始。 `include` 和 `<filenames>` 可以用一个或多个空格隔开。举个例子，你有这样几个Makefile： `a.mk` 、 `b.mk` 、 `c.mk` ，还有一个文件叫 `foo.make` ，以及一个变量 `$(bar)` ，其包含了 `bish` 和 `bash` ，那么，下面的语句：

```
include foo.make *.mk $(bar)
```

等价于：

```
include foo.make a.mk b.mk c.mk bish bash
```

make命令开始时，会找寻 `include` 所指出的其它Makefile，并把其内容安置在当前的位置。就好像C/C++的 `#include` 指令一样。如果文件都没有指定绝对路径或是相对路径的话，make会在当前目录下首先寻找，如果当前目录下没有找到，那么，make还会在下面的几个目录下找：

1. 如果make执行时，有 `-I` 或 `--include-dir` 参数，那么make就会在这个参数所指定的目录下去寻找。
2. 接下来按顺序寻找目录 `<prefix>/include` （一般是 `/usr/local/bin` ）、 `/usr/gnu/include` 、 `/usr/local/include` 、 `/usr/include` 。

环境变量 `.INCLUDE_DIRS` 包含当前 make 会寻找的目录列表。你应当避免使用命令行参数 `-I` 来寻找以上这些默认目录，否则会使得 `make` “忘掉”所有已经设定的包含目录，包括默认目录。

如果有文件没有找到的话，make会生成一条警告信息，但不会马上出现致命错误。它会继续载入其它的文件，一旦完成makefile的读取，make会再重试这些没有找到，或是不能读取的文件，如果还是不行，make才会出现一条致命信息。如果你想让make不理那些无法读取的文件，而继续执行，你可以在include前加一个减号“-”。如：

```
-include <filenames>...
```

其表示，无论include过程中出现什么错误，都不要报错继续执行。如果要和其它版本 `make` 兼容，可以使用 `sinclude` 代替 `-include` 。

## 环境变量MAKEFILES

如果你的当前环境中定义了环境变量 `MAKEFILES` ，那么make会把这个变量中的值做一个类似于 `include` 的动作。这个变量中的值是其它的Makefile，用空格分隔。只是，它和 `include` 不同的是，从这个环境变量中引入的Makefile的“默认目标”(the default goal)不会起作用，如果环境变量中定义的文件发现错误，make也会不理。

但是在这里我还是建议不要使用这个环境变量，因为只要这个变量一被定义，那么当你使用make时，所有的Makefile都会受到它的影响，这绝不是你想看到的。在这里提这个事，只是为了告诉大家，也许有时候你的Makefile出现了怪事，那么你可以看看当前环境中有没有定义这个变量。

## make的工作方式

GNU的make工作时的执行步骤如下：（想来其它的make也是类似）

1. 读入所有的Makefile。
2. 读入被include的其它Makefile。
3. 初始化文件中的变量。
4. 推导隐式规则，并分析所有规则。
5. 为所有的目标文件创建依赖关系链。
6. 根据依赖关系，决定哪些目标要重新生成。
7. 执行生成命令。

1-5步为第一个阶段，6-7为第二个阶段。第一个阶段中，如果定义的变量被使用了，那么，make会把其展开在使用的位置。但make并不会完全马上展开，make使用的是拖延战术，如果变量出现在依赖关系的规则中，那么仅当这条依赖被决定要使用了，变量才会在其内部展开。

当然，这个工作方式你不一定要清楚，但是知道这个方式你也会对make更为熟悉。有了这个基础，后续部分也就容易看懂了。

# 书写规则

规则包含两个部分：

1. 依赖关系
2. 生成目标的方法

在Makefile中，规则的顺序是很重要的，因为，Makefile中只应该有一个最终目标，其它的目标都是被这个目标所连带出来的，所以一定要让make知道你的最终目标是什么。一般来说，定义在Makefile中的目标可能会有很多，但是第一条规则中的目标将被确立为最终的目标。如果第一条规则中的目标有很多个，那么，第一个目标会成为最终的目标。make所完成的也就是这个目标，Makefile的语法为：

```makefile
targets: prerequisites 
    commands
targets:prerequisties;commands
```

*`targets`*

- 可以是一个object file（目标文件），也可以是一个可执行文件，还可以是一个标签（label）。
- targets是文件名，以空格分开，可以使用通配符。一般来说，我们的目标基本上是一个文件，但也有可能是多个文件。

*`prerequisites`*

- 目标所依赖的文件（或依赖目标）。如果其中的某个文件要比目标文件要新，那么，目标就被认为是“过时的”，被认为是需要重生成的。

*`commands`*

- 该target要执行的命令（任意的shell命令）。
- commands是命令行，如果其不与“target:prerequisites”在一行，那么，必须以 `Tab` 键开头。
- 如果和prerequisites在一行，那么可以用分号做为分隔。（见上）
- 如果命令太长，可以使用反斜杠（ `\` ）作为换行符。make对一行上有多少个字符没有限制。

>  [!NOTE] 
>
>  这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。说白一点就是说:
>
>  ==prerequisites中如果有一个以上的文件比target文件要新的话，commands所定义的命令就会被执行。==

>  [!IMPORTANT] 
>
>  注意：makefile有着严格的缩进格式，命令前面需要使用tab键对齐

==例子==



```makefile
foo.o: foo.c defs.h       # foo模块
    cc -c -g foo.c
```

上述例子中， `foo.o` 是我们的目标， `foo.c` 和 `defs.h` 是目标所依赖的源文件，而只有一个命令 `cc -c -g foo.c` （以Tab键开头）。这个规则告诉我们两件事：

1. 文件的依赖关系， `foo.o` 依赖于 `foo.c` 和 `defs.h` 的文件，如果 `foo.c` 和 `defs.h` 的文件日期要比 `foo.o` 文件日期要新，或是 `foo.o` 不存在，那么依赖关系发生。
2. 生成或更新 `foo.o` 文件，就是那个cc命令。它说明了如何生成 `foo.o` 这个文件。（当然，foo.c文件include了defs.h文件）

## 通配符

如果想定义一系列比较类似的文件，则可使用通配符。make支持三个通配符：

1. `*`: 匹配任意数量的字符（包括零个字符），但不匹配路径分隔符 `/`。
2. `?`: 匹配单个任意字符，但不匹配路径分隔符 `/`。
3. `[...]`: 匹配括号内的任意单个字符。例如，`[abc]` 匹配 `a`、`b` 或 `c`。

> [!important]
>
> 通配符可以直接用在命令行中，不可以直接用在依赖中，如果想要在依赖中使用，需要稍加处理。
>
> 在依赖中可以使用`%`来代替`*`，如：
>
> ```makefile
> target:%.c
> 	cc target -o $<
> ```
>
> 

下面举一个例子：

```makefile
clean:
    rm -f *.o
```

其实在这个clean:后面可以加上你想做的一些事情，如果你想看到在编译完后看看main.c的源代码，你可以在加上cat这个命令，例子如下：

```makefile
clean:
    cat main.c
    rm -f *.o
```

其结果你试一下就知道的。 上面这个例子我不不多说了，这是操作系统Shell所支持的通配符。这是在命令中的通配符。

```makefile
print: *.c
    lpr -p $?
    touch print
```

上面这个例子说明了通配符也可以在我们的规则中，目标print依赖于所有的 `.c` 文件。其中的 `$?` 是一个自动化变量，后面会讲述。

```makefile
objects = *.o
```

上面这个例子，表示了通配符同样可以用在变量中。并不是说 `*.o` 会展开，不！objects的值就是 `*.o` 。Makefile中的变量其实就是C/C++中的宏。如果你要让通配符在变量中展开，也就是让objects的值是所有 `.o` 的文件名的集合，那么，==你可以这样==：

```makefile
objects := $(wildcard *.o)
```

另给一个变量使用通配符的例子：

1. 列出一确定文件夹中的所有 `.c` 文件。

   ```makefile
   objects := $(wildcard *.c)
   ```

2. 列出(1)中所有文件对应的 `.o` 文件，在（3）中我们可以看到它是由make自动编译出的:

   ```makefile
   $(patsubst %.c,%.o,$(wildcard *.c))
   ```

3. 由(1)(2)两步，可写出编译并链接所有 `.c` 和 `.o` 文件

   ```makefile
   objects := $(patsubst %.c,%.o,$(wildcard *.c))
   foo : $(objects)
       cc -o foo $(objects)
   ```

这种用法由关键字“wildcard”，“patsubst”指出，关于Makefile的关键字，我们将在后面讨论。



## 文件搜索

在一些大的工程中，有大量的源文件，我们通常的做法是把这许多的源文件分类，并存放在不同的目录中。所以，当make需要去找寻文件的依赖关系时，你可以在文件前加上路径，但最好的方法是把一个路径告诉make，让make在自动去找。

Makefile文件中的特殊变量 `VPATH` 就是完成这个功能的，如果没有指明这个变量，make只会在当前的目录中去找寻依赖文件和目标文件。如果定义了这个变量，那么，make就会在当前目录找不到的情况下，到所指定的目录中去找寻文件了。

```makefile
VPATH = src:../headers
```

上面的定义指定两个目录，`src`和`../headers`，make会按照这个顺序进行搜索。目录由`:`分隔。（==当然，当前目录永远是最高优先搜索的地方==）。

另一个设置文件搜索路径的方法是使用make的`vpath`关键字（注意，它是全小写的），这不是变量，这是一个make的关键字，这和上面提到的那个VPATH变量很类似，但是它更为灵活。它可以指定不同的文件在不同的搜索目录中。这是一个很灵活的功能。它的使用方法有三种：

- `vpath <pattern> <directories>`：为符合模式 `<pattern>`的文件指定搜索目录`<directories>`。
- `vpath <pattern>`：清除符合模式`<pattern>`的文件的搜索目录。
- `vpath`：清除所有已被设置好了的文件搜索目录。

vpath使用方法中的`<pattern>`需要包含 `%` 字符。 `%` 的意思是匹配零或若干字符，（需引用 `%` ，使用 `\` ）例如， `%.h` 表示所有以 `.h` 结尾的文件。`<pattern>`指定了要搜索的文件集，而`<directories>`则指定了`<pattern>`的文件集的搜索的目录。例如：

```makefile
vpath %.h ../headers
```

该语句表示，要求make在“../headers”目录下搜索所有以 `.h` 结尾的文件。（如果某文件在当前目录没有找到的话）

我们可以连续地使用vpath语句，以指定不同搜索策略。如果连续的vpath语句中出现了相同的`<pattern>` ，或是被重复了的`<pattern>`，那么，make会按照vpath语句的先后顺序来执行搜索。如：

```makefile
vpath %.c foo
vpath %   blish
vpath %.c bar
```

其表示 `.c` 结尾的文件，先在“foo”目录，然后是“blish”，最后是“bar”目录。

```makefile
vpath %.c foo:bar
vpath %   blish
```

而上面的语句则表示 `.c` 结尾的文件，先在“foo”目录，然后是“bar”目录，最后才是“blish”目录。

## 伪目标

最早先的一个例子中，我们提到过一个“clean”的目标，这是一个“伪目标”，

```makefile
clean:
    rm *.o temp
```

正像我们前面例子中的`clean`一样，既然我们生成了许多文件编译文件，我们也应该提供一个清除它们的“目标”以备完整地重编译而用。 （以`make clean`来使用该目标）

因为，我们并不生成`clean`这个文件。“伪目标”并不是一个文件，只是一个标签，由于“伪目标”不是文件，所以make无法生成它的依赖关系和决定它是否要执行。我们只有通过显式地指明这个“目标”才能让其生效。当然，“伪目标”的取名不能和文件名重名，不然其就失去了“伪目标”的意义了。

当然，为了避免和文件重名的这种情况，我们可以使用一个特殊的标记`.PHONY`来显式地指明一个目标是“伪目标”，向make说明，不管是否有这个文件，这个目标就是“伪目标”。

```makefile
.PHONY : clean
```

只要有这个声明，不管是否有“clean”文件，要运行“clean”这个目标，只有“make clean”这样。于是整个过程可以这样写：

```makefile
.PHONY : clean
clean :
    rm *.o temp
```

伪目标一般没有依赖的文件。但是，我们也可以为伪目标指定所依赖的文件。伪目标同样可以作为“默认目标”，只要将其放在第一个。一个示例就是，如果你的Makefile需要一口气生成若干个可执行文件，但你只想简单地敲一个make完事，并且，所有的目标文件都写在一个Makefile中，那么你可以使用“伪目标”这个特性：

```makefile
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
    cc -o prog1 prog1.o utils.o

prog2 : prog2.o
    cc -o prog2 prog2.o

prog3 : prog3.o sort.o utils.o
    cc -o prog3 prog3.o sort.o utils.o
```

我们知道，Makefile中的第一个目标会被作为其默认目标。我们声明了一个“all”的伪目标，其依赖于其它三个目标。由于默认目标的特性是:总是被执行的，但由于“all”又是一个伪目标，伪目标只是一个标签不会生成文件，所以不会有“all”文件产生。于是，其它三个目标的规则总是会被决议。也就达到了我们一口气生成多个目标的目的。 `.PHONY : all` 声明了“all”这个目标为“伪目标”。（注：这里的显式“.PHONY : all” 不写的话一般情况也可以正确的执行，这样make可通过隐式规则推导出:“all” 是一个伪目标，执行make不会生成“all”文件，而执行后面的多个目标。建议：显式写出是一个好习惯。）

随便提一句，从上面的例子我们可以看出，目标也可以成为依赖。所以，伪目标同样也可成为依赖。看下面的例子：

```makefile
.PHONY : cleanall cleanobj cleandiff

cleanall : cleanobj cleandiff
    rm program

cleanobj :
    rm *.o

cleandiff :
    rm *.diff
```

“make cleanall”将清除所有要被清除的文件。“cleanobj”和“cleandiff”这两个伪目标有点像“子程序”的意思。我们可以输入“make cleanall”和“make cleanobj”和“make cleandiff”命令来达到清除不同种类文件的目的。

## 多目标

Makefile的规则中的目标可以不止一个，其支持多目标，有可能我们的多个目标同时依赖于一个文件，并且其生成的命令大体类似。于是我们就能把其合并起来。当然，多个目标的生成规则的执行命令不是同一个，这可能会给我们带来麻烦，不过好在我们可以使用一个自动化变量 `$@` （关于自动化变量，将在后面讲述），这个变量表示着目前规则中所有的目标的集合，这样说可能很抽象，还是看一个例子吧。

```makefile
bigoutput littleoutput : text.g
    generate text.g -$(subst output,,$@) > $@
```

上述规则等价于：

```makefile
bigoutput : text.g
    generate text.g -big > bigoutput
littleoutput : text.g
    generate text.g -little > littleoutput
```

其中， `-$(subst output,,$@)` 中的 `$` 表示执行一个Makefile的函数，函数名为subst，后面的为参数。关于函数，将在后面讲述。这里的这个函数是替换字符串的意思， `$@` 表示目标的集合，就像一个数组， `$@` 依次取出目标，并执于命令。

## 静态模式

静态模式可以更加容易地定义多目标的规则，可以让我们的规则变得更加的有弹性和灵活。我们还是先来看一下语法：

```makefile
<targets ...> : <target-pattern> : <prereq-patterns ...>
    <commands>
    ...
```

相关参数如下：

- targets定义了一系列的目标文件，可以有通配符。是目标的一个集合。
- target-pattern是指明了targets的模式，也就是的目标集模式。
- prereq-patterns是目标的依赖模式，它对target-pattern形成的模式再进行一次依赖目标的定义。

这样描述这三个东西，可能还是没有说清楚，还是举个例子来说明一下吧。

- 如果我们的`<target-pattern>`定义成 `%.o` ，意思是我们的`<target>`集合中都是以 `.o` 结尾的。
- 如果我们的`<prereq-patterns>`定义成 `%.c` ，意思是对`<target-pattern>`所形成的目标集进行二次定义，其计算方法是，取`<target-pattern>`模式中的 `%` （也就是去掉了 `.o` 这个结尾），并为其加上 `.c` 这个结尾，形成的新集合。

所以，我们的“目标模式”或是“依赖模式”中都应该有 `%` 这个字符，如果你的文件名中有 `%` 那么你可以使用反斜杠 `\` 进行转义，来标明真实的 `%` 字符。

> [!note]
>
> 简单来说就是一个逐级依赖的过程，`target`依赖`target-pattern` ，而`target-pattern`依赖于`prereq-patterns`

看一个例子：

```makefile
objects = foo.o bar.o

all: $(objects)

$(objects): %.o: %.c
    $(CC) -c $(CFLAGS) $< -o $@
```

上面的例子中，指明了我们的目标从$object中获取， `%.o` 表明要所有以 `.o` 结尾的目标，也就是 `foo.o bar.o` ，也就是变量 `$object` 集合的模式，而依赖模式 `%.c` 则取模式 `%.o` 的 `%` ，也就是 `foo bar` ，并为其加下 `.c` 的后缀，于是，我们的依赖目标就是 `foo.c bar.c` 。而命令中的 `$<` 和 `$@` 则是自动化变量， `$<` 表示第一个依赖文件， `$@` 表示目标集（也就是“foo.o bar.o”）。于是，上面的规则展开后等价于下面的规则：

```makefile
foo.o : foo.c
    $(CC) -c $(CFLAGS) foo.c -o foo.o
bar.o : bar.c
    $(CC) -c $(CFLAGS) bar.c -o bar.o
```

试想，如果我们的 `%.o` 有几百个，那么我们只要用这种很简单的“静态模式规则”就可以写完一堆规则，实在是太有效率了。“静态模式规则”的用法很灵活，如果用得好，那会是一个很强大的功能。再看一个例子：

```makefile
files = foo.elc bar.o lose.o

$(filter %.o,$(files)): %.o: %.c
    $(CC) -c $(CFLAGS) $< -o $@
$(filter %.elc,$(files)): %.elc: %.el
    emacs -f batch-byte-compile $<
```

$(filter %.o,$(files))表示调用Makefile的filter函数，过滤“$files”集，只要其中模式为“%.o”的内容。

## 自动生成依赖性

在Makefile中，我们的依赖关系可能会需要包含一系列的头文件，比如，如果我们的main.c中有一句 `#include "defs.h"` ，那么我们的依赖关系应该是：

```makefile
main.o : main.c defs.h
```

但是，如果是一个比较大型的工程，你必需清楚哪些C文件包含了哪些头文件，并且，你在加入或删除头文件时，也需要小心地修改Makefile，这是一个很没有维护性的工作。为了避免这种繁重而又容易出错的事情，我们可以使用C/C++编译的一个功能。大多数的C/C++编译器都支持一个`-M`的选项，即自动找寻源文件中包含的头文件，并生成一个依赖关系。例如，如果我们执行下面的命令:

```makefile
cc -M main.c
```

其输出是：

```makefile
main.o : main.c defs.h
```

于是由编译器自动生成的依赖关系，这样一来，你就不必再手动书写若干文件的依赖关系，而由编译器自动生成了。需要提醒一句的是，如果你使用GNU的C/C++编译器，你得用 `-MM` 参数，不然， `-M` 参数会把一些标准库的头文件也包含进来。

gcc -M main.c的输出是:

```makefile
main.o: main.c defs.h /usr/include/stdio.h /usr/include/features.h \
    /usr/include/sys/cdefs.h /usr/include/gnu/stubs.h \
    /usr/lib/gcc-lib/i486-suse-linux/2.95.3/include/stddef.h \
    /usr/include/bits/types.h /usr/include/bits/pthreadtypes.h \
    /usr/include/bits/sched.h /usr/include/libio.h \
    /usr/include/_G_config.h /usr/include/wchar.h \
    /usr/include/bits/wchar.h /usr/include/gconv.h \
    /usr/lib/gcc-lib/i486-suse-linux/2.95.3/include/stdarg.h \
    /usr/include/bits/stdio_lim.h
```

gcc -MM main.c的输出则是:

```makefile
main.o: main.c defs.h
```

那么，编译器的这个功能如何与我们的Makefile联系在一起呢。因为这样一来，我们的Makefile也要根据这些源文件重新生成，让 Makefile 自己依赖于源文件？这个功能并不现实，不过我们可以有其它手段来迂回地实现这一功能。GNU组织建议把编译器为每一个源文件的自动生成的依赖关系放到一个文件中，为每一个 `name.c` 的文件都生成一个 `name.d` 的Makefile文件， `.d` 文件中就存放对应 `.c` 文件的依赖关系。

于是，我们可以写出 `.c` 文件和 `.d` 文件的依赖关系，并让make自动更新或生成 `.d` 文件，并把其包含在我们的主Makefile中，这样，我们就可以自动化地生成每个文件的依赖关系了。

这里，我们给出了一个模式规则来产生 `.d` 文件：

```makefile
%.d: %.c
    @set -e; rm -f $@; \
    $(CC) -M $(CPPFLAGS) $< > $@.$$$$; \
    sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
    rm -f $@.$$$$
```

这个规则的意思是:

- 所有的 `.d` 文件依赖于 `.c` 文件
- `rm -f $@` 的意思是删除所有的目标，也就是 `.d` 文件
- 第二行的意思是，为每个依赖文件 `$<` ，也就是 `.c` 文件生成依赖文件， `$@` 表示模式 `%.d` 文件，如果有一个C文件是name.c，那么 `%` 就是 `name` ， `$$$$` 意为一个随机编号，第二行生成的文件有可能是“name.d.12345”
- 第三行使用sed命令做了一个替换，关于sed命令的用法请参看相关的使用文档。
- 第四行就是删除临时文件。

总而言之，这个模式要做的事就是在编译器生成的依赖关系中加入 `.d` 文件的依赖，即把依赖关系：

```makefile
main.o : main.c defs.h
```

转成：

```makefile
main.o main.d : main.c defs.h
```

于是，我们的 `.d` 文件也会自动更新了，并会自动生成了，当然，你还可以在这个 `.d` 文件中加入的不只是依赖关系，包括生成的命令也可一并加入，让每个 `.d` 文件都包含一个完整的规则。一旦我们完成这个工作，接下来，我们就要把这些自动生成的规则放进我们的主Makefile中。我们可以使用Makefile的“include”命令，来引入别的Makefile文件，例如：

```
sources = foo.c bar.c

include $(sources:.c=.d)
```

上述语句中的 `$(sources:.c=.d)` 中的 `.c=.d` 的意思是做一个替换，把变量 `$(sources)` 所有 `.c` 的字串都替换成 `.d` ，关于这个“替换”的内容，在后面我会有更为详细的讲述。当然，你得注意次序，因为include是按次序来载入文件，最先载入的 `.d` 文件中的目标会成为默认目标。



# 书写命令

每条规则中的命令和操作系统Shell的命令行是一致的。make会按顺序一条一条的执行命令，每条命令的开头必须以 `Tab` 键开头，除非，命令是紧跟在依赖规则后面的分号后的。在命令行之间中的空格或是空行会被忽略，但是如果该空格或空行是以Tab键开头的，那么make会认为其是一个空命令。

我们在UNIX下可能会使用不同的Shell，但是make的命令默认是被 `/bin/sh` ——UNIX的标准Shell 解释执行的。除非你特别指定一个其它的Shell。Makefile中， `#` 是注释符，很像C/C++中的 `//` ，其后的本行字符都被注释。

## 显示命令

通常，make会把其要执行的命令行在命令执行前输出到屏幕上。当我们用 `@` 字符在命令行前，那么，这个命令将不被make显示出来，最具代表性的例子是，我们用这个功能来向屏幕显示一些信息。如:

```
@echo 正在编译XXX模块......
```

当make执行时，会输出“正在编译XXX模块……”字串，但不会输出命令，如果没有“@”，那么，make将输出:

```
echo 正在编译XXX模块......
正在编译XXX模块......
```

如果make执行时，带入make参数 `-n` 或 `--just-print` ，那么其只是显示命令，但不会执行命令，这个功能很有利于我们调试我们的Makefile，看看我们书写的命令是执行起来是什么样子的或是什么顺序的。

而make参数 `-s` 或 `--silent` 或 `--quiet` 则是全面禁止命令的显示。

## 命令执行

当依赖目标新于目标时，也就是当规则的目标需要被更新时，make会一条一条的执行其后的命令。需要注意的是，如果你要让上一条命令的结果应用在下一条命令时，你应该使用分号分隔这两条命令。比如你的第一条命令是cd命令，你希望第二条命令得在cd之后的基础上运行，那么你就不能把这两条命令写在两行上，而应该把这两条命令写在一行上，用分号分隔。如：

- 示例一：

```makefile
exec:
    cd /home/hchen
    pwd
```

- 示例二：

```makefile
exec:
    cd /home/hchen; pwd
```

当我们执行 `make exec` 时，第一个例子中的cd没有作用，pwd会打印出当前的Makefile目录，而第二个例子中，cd就起作用了，pwd会打印出“/home/hchen”。

make一般是使用环境变量SHELL中所定义的系统Shell来执行命令，默认情况下使用UNIX的标准Shell——/bin/sh来执行命令。但在MS-DOS下有点特殊，因为MS-DOS下没有SHELL环境变量，当然你也可以指定。如果你指定了UNIX风格的目录形式，首先，make会在SHELL所指定的路径中找寻命令解释器，如果找不到，其会在当前盘符中的当前目录中寻找，如果再找不到，其会在PATH环境变量中所定义的所有路径中寻找。MS-DOS中，如果你定义的命令解释器没有找到，其会给你的命令解释器加上诸如 `.exe` 、 `.com` 、 `.bat` 、 `.sh` 等后缀。

## 命令出错

每当命令运行完后，make会检测每个命令的返回码，如果命令返回成功，那么make会执行下一条命令，当规则中所有的命令成功返回后，这个规则就算是成功完成了。如果一个规则中的某个命令出错了（命令退出码非零），那么make就会终止执行当前规则，这将有可能终止所有规则的执行。

有些时候，命令的出错并不表示就是错误的。例如mkdir命令，我们一定需要建立一个目录，如果目录不存在，那么mkdir就成功执行，万事大吉，如果目录存在，那么就出错了。我们之所以使用mkdir的意思就是一定要有这样的一个目录，于是我们就不希望mkdir出错而终止规则的运行。

为了做到这一点，忽略命令的出错，我们可以在Makefile的==命令行前加一个减号 `-` （在Tab键之后）==，标记为不管命令出不出错都认为是成功的。如：

```makefile
clean:
    -rm -f *.o
```

还有一个全局的办法是，给make加上 `-i` 或是 `--ignore-errors` 参数，那么，Makefile中所有命令都会忽略错误。而如果一个规则是以 `.IGNORE` 作为目标的，那么这个规则中的所有命令将会忽略错误。这些是不同级别的防止命令出错的方法，你可以根据你的不同喜欢设置。

还有一个要提一下的make的参数的是 `-k` 或是 `--keep-going` ，这个参数的意思是，如果某规则中的命令出错了，那么就终止该规则的执行，但继续执行其它规则。

## 嵌套执行make

在一些大的工程中，我们会把我们不同模块或是不同功能的源文件放在不同的目录中，我们可以在每个目录中都书写一个该目录的Makefile，这有利于让我们的Makefile变得更加地简洁，而不至于把所有的东西全部写在一个Makefile中，这样会很难维护我们的Makefile，这个技术对于我们模块编译和分段编译有着非常大的好处。

例如，我们有一个子目录叫subdir，这个目录下有个Makefile文件，来指明了这个目录下文件的编译规则。那么我们总控的Makefile可以这样书写：

```makefile
subsystem:
    cd subdir && $(MAKE)
```

其等价于：

```makefile
subsystem:
    $(MAKE) -C subdir
```

定义$(MAKE)宏变量的意思是，也许我们的make需要一些参数，所以定义成一个变量比较利于维护。这两个例子的意思都是先进入“subdir”目录，然后执行make命令。

我们把这个Makefile叫做“总控Makefile”，总控Makefile的变量可以传递到下级的Makefile中（如果你显示的声明），但是不会覆盖下层的Makefile中所定义的变量，除非指定了 `-e` 参数。

如果你要传递变量到下级Makefile中，那么你可以使用这样的声明:

```makefile
export <variable ...>;
```

如果你不想让某些变量传递到下级Makefile中，那么你可以这样声明:

```makefile
unexport <variable ...>;
```

> [!tip]
>
> 实际上`export`和`unexpect`指令类似于c语言中的`extern`，指定其他文件也可以使用这个变量。

==示例一==

```makefile
export variable = value
```

其等价于：

```makefile
variable = value
export variable
```

其等价于：

```makefile
export variable := value
```

其等价于：

```makefile
variable := value
export variable
```

==示例二==

```makefile
export variable += value
```

其等价于：

```makefile
variable += value
export variable
```

如果你要传递所有的变量，那么，只要一个export就行了。后面什么也不用跟，表示传递所有的变量。

> [!note]
>
> 需要注意的是，有两个变量，一个是 `SHELL` ，一个是 `MAKEFLAGS` ，这两个变量不管你是否export，其总是要传递到下层 Makefile中，特别是 `MAKEFLAGS` 变量，其中包含了make的参数信息，如果我们执行“总控Makefile”时有make参数或是在上层 Makefile中定义了这个变量，那么 `MAKEFLAGS` 变量将会是这些参数，并会传递到下层Makefile中，这是一个系统级的环境变量。
>
> 但是make命令中的有几个参数并不往下传递，它们是 `-C` , `-f` , `-h`, `-o` 和 `-W` ，如果你不想往下层传递参数，那么，你可以：
>
> ```makefile
> subsystem:
>     cd subdir && $(MAKE) MAKEFLAGS=
> ```
>
> 如果你定义了环境变量 `MAKEFLAGS` ，那么你得确信其中的选项是大家都会用到的，如果其中有 `-t` , `-n` 和 `-q` 参数，那么将会有让你意想不到的结果，或许会让你异常地恐慌。
>
> 还有一个在“嵌套执行”中比较有用的参数， `-w` 或是 `--print-directory` 会在make的过程中输出一些信息，让你看到目前的工作目录。比如，如果我们的下级make目录是“/home/hchen/gnu/make”，如果我们使用 `make -w` 来执行，那么当进入该目录时，我们会看到:
>
> ```
> make: Entering directory `/home/hchen/gnu/make'.
> ```
>
> 而在完成下层make后离开目录时，我们会看到:
>
> ```
> make: Leaving directory `/home/hchen/gnu/make'
> ```
>
> 当你使用 `-C` 参数来指定make下层Makefile时， `-w` 会被自动打开的。如果参数中有 `-s` （ `--slient` ）或是 `--no-print-directory` ，那么， `-w` 总是失效的。



## 定义命令包

如果Makefile中出现一些相同命令序列，那么我们可以为这些相同的命令序列定义一个变量。定义这种命令序列的语法以 `define` 开始，以 `endef` 结束，如:

```makefile
define run-yacc
yacc $(firstword $^)
mv y.tab.c $@
endef
```

这里，“run-yacc”是这个命令包的名字，其不要和Makefile中的变量重名。在 `define` 和 `endef` 中的两行就是命令序列。这个命令包中的第一个命令是运行Yacc程序，因为Yacc程序总是生成“y.tab.c”的文件，所以第二行的命令就是把这个文件改改名字。还是把这个命令包放到一个示例中来看看吧。

```makefile
foo.c : foo.y
    $(run-yacc)
```

我们可以看见，要使用这个命令包，我们就好像使用变量一样。在这个命令包的使用中，命令包“run-yacc”中的 `$^` 就是 `foo.y` ， `$@` 就是 `foo.c` （有关这种以 `$` 开头的特殊变量，我们会在后面介绍），make在执行命令包时，命令包中的每个命令会被依次独立执行。



# 使用变量

在Makefile中的定义的变量，就像是C/C++语言中的宏一样，他代表了一个文本字串，在Makefile中执行的时候其会自动原模原样地展开在所使用的地方。其与C/C++所不同的是，你可以在Makefile中改变其值。在Makefile中，变量可以使用在“目标”，“依赖目标”， “命令”或是Makefile的其它部分中。

变量的命名字可以包含字符、数字，下划线（可以是数字开头），但不应该含有 `:` 、 `#` 、 `=` 或是空字符（空格、回车等）。变量是大小写敏感的，“foo”、“Foo”和“FOO”是三个不同的变量名。传统的Makefile的变量名是全大写的命名方式，但我推荐使用大小写搭配的变量名，如：MakeFlags。这样可以避免和系统的变量冲突，而发生意外的事情。

有一些变量是很奇怪字串，如 `$<` 、 `$@` 等，这些是自动化变量。

## 变量基础

变量在声明时需要给予初值，而在使用时，需要给在变量名前加上 `$` 符号，但最好用小括号 `()` 或是大括号 `{}` 把变量给包括起来。如果你要使用真实的 `$` 字符，那么你需要用 `$$` 来表示。

变量可以使用在许多地方，如规则中的“目标”、“依赖”、“命令”以及新的变量中。先看一个例子：

```makefile
objects = program.o foo.o utils.o
program : $(objects)
    cc -o program $(objects)

$(objects) : defs.h
```

变量会在使用它的地方精确地展开，就像C/C++中的宏一样，例如：

```makefile
foo = c
prog.o : prog.$(foo)
    $(foo)$(foo) -$(foo) prog.$(foo)
```

展开后得到：

```makefile
prog.o : prog.c
    cc -c prog.c
```

当然，千万不要在你的Makefile中这样干，这里只是举个例子来表明Makefile中的变量在使用处展开的真实样子。可见其就是一个“替代”的原理。

另外，给变量加上括号完全是为了更加安全地使用这个变量，在上面的例子中，如果你不想给变量加上括号，那也可以，但我还是强烈建议你给变量加上括号。

## 变量中的变量

在定义变量的值时，我们可以使用其它变量来构造变量的值，在Makefile中有两种方式来在用变量定义变量的值。

先看第一种方式，也就是简单的使用 `=` 号，在 `=` 左侧是变量，右侧是变量的值，右侧变量的值可以定义在文件的任何一处，也就是说，右侧中的变量不一定非要是已定义好的值，其也可以使用后面定义的值。如：

```makefile
foo = $(bar)
bar = $(ugh)
ugh = Huh?

all:
    echo $(foo)
```

我们执行`make all`将会打出变量 `$(foo)` 的值是 `Huh?` （ `$(foo)` 的值是 `$(bar)` ， `$(bar)` 的值是 `$(ugh)` ， `$(ugh)` 的值是 `Huh?` ）可见，变量是可以使用后面的变量来定义的。

这个功能有好的地方，也有不好的地方，好的地方是，我们可以把变量的真实值推到后面来定义，如：

```makefile
CFLAGS = $(include_dirs) -O
include_dirs = -I foo -I bar
```

当 `CFLAGS` 在命令中被展开时，会是 `-I foo -I bar -O` 。但这种形式也有不好的地方，那就是递归定义，如：

```makefile
CFLAGS = $(CFLAGS) -O
```

或：

```makefile
A = $(B)
B = $(A)
```

这会让make陷入无限的变量展开过程中去，当然，我们的make是有能力检测这样的定义，并会报错。还有就是如果在变量中使用函数，那么，这种方式会让我们的make运行时非常慢，更糟糕的是，他会使用得两个make的函数“wildcard”和“shell”发生不可预知的错误。因为你不会知道这两个函数会被调用多少次。

为了避免上面的这种方法，我们可以使用make中的另一种用变量来定义变量的方法。这种方法使用的是 `:=` 操作符，如：

```
x := foo
y := $(x) bar
x := later
```

其等价于：

```
y := foo bar
x := later
```

值得一提的是，这种方法，前面的变量不能使用后面的变量，只能使用前面已定义好了的变量。如果是这样：

```
y := $(x) bar
x := foo
```

那么，y的值是“bar”，而不是“foo bar”。

上面都是一些比较简单的变量使用了，让我们来看一个复杂的例子，其中包括了make的函数、条件表达式和一个系统变量“MAKELEVEL”的使用：

```
ifeq (0,${MAKELEVEL})
cur-dir   := $(shell pwd)
whoami    := $(shell whoami)
host-type := $(shell arch)
MAKE := ${MAKE} host-type=${host-type} whoami=${whoami}
endif
```

关于条件表达式和函数，我们在后面再说，对于系统变量“MAKELEVEL”，其意思是，如果我们的make有一个嵌套执行的动作（参见前面的“嵌套使用make”），那么，这个变量会记录了我们的当前Makefile的调用层数。

下面再介绍两个定义变量时我们需要知道的，请先看一个例子，如果我们要定义一个变量，其值是一个空格，那么我们可以这样来：

```
nullstring :=
space := $(nullstring) # end of the line
```

nullstring是一个Empty变量，其中什么也没有，而我们的space的值是一个空格。因为在操作符的右边是很难描述一个空格的，这里采用的技术很管用，先用一个Empty变量来标明变量的值开始了，而后面采用“#”注释符来表示变量定义的终止，这样，我们可以定义出其值是一个空格的变量。请注意这里关于“#”的使用，注释符“#”的这种特性值得我们注意，如果我们这样定义一个变量：

```
dir := /foo/bar    # directory to put the frobs in
```

dir这个变量的值是“/foo/bar”，后面还跟了4个空格，如果我们这样使用这个变量来指定别的目录——“$(dir)/file”那么就完蛋了。

还有一个比较有用的操作符是 `?=` ，先看示例：

```
FOO ?= bar
```

其含义是，如果FOO没有被定义过，那么变量FOO的值就是“bar”，如果FOO先前被定义过，那么这条语将什么也不做，其等价于：

```
ifeq ($(origin FOO), undefined)
    FOO = bar
endif
```

## 变量高级用法[¶](https://seisman.github.io/how-to-write-makefile/variables.html#id4)

这里介绍两种变量的高级使用方法，第一种是变量值的替换。

我们可以替换变量中的共有的部分，其格式是 `$(var:a=b)` 或是 `${var:a=b}` ，其意思是，把变量“var”中所有以“a”字串“结尾”的“a”替换成“b”字串。这里的“结尾”意思是“空格”或是“结束符”。

还是看一个示例吧：

```
foo := a.o b.o c.o
bar := $(foo:.o=.c)
```

这个示例中，我们先定义了一个 `$(foo)` 变量，而第二行的意思是把 `$(foo)` 中所有以 `.o` 字串“结尾”全部替换成 `.c` ，所以我们的 `$(bar)` 的值就是“a.c b.c c.c”。

另外一种变量替换的技术是以“静态模式”（参见前面章节）定义的，如：

```
foo := a.o b.o c.o
bar := $(foo:%.o=%.c)
```

这依赖于被替换字串中的有相同的模式，模式中必须包含一个 `%` 字符，这个例子同样让 `$(bar)` 变量的值为“a.c b.c c.c”。

第二种高级用法是——“把变量的值再当成变量”。先看一个例子：

```
x = y
y = z
a := $($(x))
```

在这个例子中，$(x)的值是“y”，所以$($(x))就是$(y)，于是$(a)的值就是“z”。（注意，是“x=y”，而不是“x=$(y)”）

我们还可以使用更多的层次：

```
x = y
y = z
z = u
a := $($($(x)))
```

这里的 `$(a)` 的值是“u”，相关的推导留给读者自己去做吧。

让我们再复杂一点，使用上“在变量定义中使用变量”的第一个方式，来看一个例子：

```
x = $(y)
y = z
z = Hello
a := $($(x))
```

这里的 `$($(x))` 被替换成了 `$($(y))` ，因为 `$(y)` 值是“z”，所以，最终结果是： `a:=$(z)` ，也就是“Hello”。

再复杂一点，我们再加上函数：

```
x = variable1
variable2 := Hello
y = $(subst 1,2,$(x))
z = y
a := $($($(z)))
```

这个例子中， `$($($(z)))` 扩展为 `$($(y))` ，而其再次被扩展为 `$($(subst 1,2,$(x)))` 。 `$(x)` 的值是“variable1”，subst函数把“variable1”中的所有“1”字串替换成“2”字串，于是，“variable1”变成 “variable2”，再取其值，所以，最终， `$(a)` 的值就是 `$(variable2)` 的值——“Hello”。（喔，好不容易）

在这种方式中，或要可以使用多个变量来组成一个变量的名字，然后再取其值：

```
first_second = Hello
a = first
b = second
all = $($a_$b)
```

这里的 `$a_$b` 组成了“first_second”，于是， `$(all)` 的值就是“Hello”。

再来看看结合第一种技术的例子：

```
a_objects := a.o b.o c.o
1_objects := 1.o 2.o 3.o

sources := $($(a1)_objects:.o=.c)
```

这个例子中，如果 `$(a1)` 的值是“a”的话，那么， `$(sources)` 的值就是“a.c b.c c.c”；如果 `$(a1)` 的值是“1”，那么 `$(sources)` 的值是“1.c 2.c 3.c”。

再来看一个这种技术和“函数”与“条件语句”一同使用的例子：

```
ifdef do_sort
    func := sort
else
    func := strip
endif

bar := a d b g q c

foo := $($(func) $(bar))
```

这个示例中，如果定义了“do_sort”，那么： `foo := $(sort a d b g q c)` ，于是 `$(foo)` 的值就是 “a b c d g q”，而如果没有定义“do_sort”，那么： `foo := $(strip a d b g q c)` ，调用的就是strip函数。

当然，“把变量的值再当成变量”这种技术，同样可以用在操作符的左边:

```
dir = foo
$(dir)_sources := $(wildcard $(dir)/*.c)
define $(dir)_print
lpr $($(dir)_sources)
endef
```

这个例子中定义了三个变量：“dir”，“foo_sources”和“foo_print”。

## 追加变量值[¶](https://seisman.github.io/how-to-write-makefile/variables.html#id5)

我们可以使用 `+=` 操作符给变量追加值，如：

```
objects = main.o foo.o bar.o utils.o
objects += another.o
```

于是，我们的 `$(objects)` 值变成：“main.o foo.o bar.o utils.o another.o”（another.o被追加进去了）

使用 `+=` 操作符，可以模拟为下面的这种例子：

```
objects = main.o foo.o bar.o utils.o
objects := $(objects) another.o
```

所不同的是，用 `+=` 更为简洁。

如果变量之前没有定义过，那么， `+=` 会自动变成 `=` ，如果前面有变量定义，那么 `+=` 会继承于前次操作的赋值符。如果前一次的是 `:=` ，那么 `+=` 会以 `:=` 作为其赋值符，如：

```
variable := value
variable += more
```

等价于：

```
variable := value
variable := $(variable) more
```

但如果是这种情况：

```
variable = value
variable += more
```

由于前次的赋值符是 `=` ，所以 `+=` 也会以 `=` 来做为赋值，那么岂不会发生变量的递补归定义，这是很不好的，所以make会自动为我们解决这个问题，我们不必担心这个问题。

## override 指令[¶](https://seisman.github.io/how-to-write-makefile/variables.html#override)

如果有变量是通过make的命令行参数设置的，那么Makefile文件中对这个变量的赋值会被忽略。如果你想在Makefile文件中设置这类参数的值，那么，你可以使用“override”指令。其语法是:

```
override <variable>; = <value>;

override <variable>; := <value>;
```

当然，你还可以追加:

```
override <variable>; += <more text>;
```

对于多行的变量定义，我们用define指令，在define指令前，也同样可以使用override指令，如:

```
override define foo
bar
endef
```

## 多行变量[¶](https://seisman.github.io/how-to-write-makefile/variables.html#id6)

还有一种设置变量值的方法是使用define关键字。使用define关键字设置变量的值可以有换行，这有利于定义一系列的命令（前面我们讲过“命令包”的技术就是利用这个关键字）。

define指令后面跟的是变量的名字，而重起一行定义变量的值，定义是以endef 关键字结束。其工作方式和“=”操作符一样。变量的值可以包含函数、命令、文字，或是其它变量。因为命令需要以[Tab]键开头，所以如果你用define定义的命令变量中没有以 `Tab` 键开头，那么make 就不会把其认为是命令。

下面的这个示例展示了define的用法:

```
define two-lines
echo foo
echo $(bar)
endef
```

## 环境变量[¶](https://seisman.github.io/how-to-write-makefile/variables.html#id7)

make运行时的系统环境变量可以在make开始运行时被载入到Makefile文件中，但是如果Makefile中已定义了这个变量，或是这个变量由make命令行带入，那么系统的环境变量的值将被覆盖。（如果make指定了“-e”参数，那么，系统环境变量将覆盖Makefile中定义的变量）

因此，如果我们在环境变量中设置了 `CFLAGS` 环境变量，那么我们就可以在所有的Makefile中使用这个变量了。这对于我们使用统一的编译参数有比较大的好处。如果Makefile中定义了CFLAGS，那么则会使用Makefile中的这个变量，如果没有定义则使用系统环境变量的值，一个共性和个性的统一，很像“全局变量”和“局部变量”的特性。

当make嵌套调用时（参见前面的“嵌套调用”章节），上层Makefile中定义的变量会以系统环境变量的方式传递到下层的Makefile 中。当然，默认情况下，只有通过命令行设置的变量会被传递。而定义在文件中的变量，如果要向下层Makefile传递，则需要使用export关键字来声明。（参见前面章节）

当然，我并不推荐把许多的变量都定义在系统环境中，这样，在我们执行不用的Makefile时，拥有的是同一套系统变量，这可能会带来更多的麻烦。

## 目标变量[¶](https://seisman.github.io/how-to-write-makefile/variables.html#id8)

前面我们所讲的在Makefile中定义的变量都是“全局变量”，在整个文件，我们都可以访问这些变量。当然，“自动化变量”除外，如 `$<` 等这种类量的自动化变量就属于“规则型变量”，这种变量的值依赖于规则的目标和依赖目标的定义。

当然，我也同样可以为某个目标设置局部变量，这种变量被称为“Target-specific Variable”，它可以和“全局变量”同名，因为它的作用范围只在这条规则以及连带规则中，所以其值也只在作用范围内有效。而不会影响规则链以外的全局变量的值。

其语法是：

```
<target ...> : <variable-assignment>;

<target ...> : overide <variable-assignment>
```

<variable-assignment>;可以是前面讲过的各种赋值表达式，如 `=` 、 `:=` 、 `+=` 或是 `?=` 。第二个语法是针对于make命令行带入的变量，或是系统环境变量。

这个特性非常的有用，当我们设置了这样一个变量，这个变量会作用到由这个目标所引发的所有的规则中去。如：

```
prog : CFLAGS = -g
prog : prog.o foo.o bar.o
    $(CC) $(CFLAGS) prog.o foo.o bar.o

prog.o : prog.c
    $(CC) $(CFLAGS) prog.c

foo.o : foo.c
    $(CC) $(CFLAGS) foo.c

bar.o : bar.c
    $(CC) $(CFLAGS) bar.c
```

在这个示例中，不管全局的 `$(CFLAGS)` 的值是什么，在prog目标，以及其所引发的所有规则中（prog.o foo.o bar.o的规则）， `$(CFLAGS)` 的值都是 `-g`

## 模式变量[¶](https://seisman.github.io/how-to-write-makefile/variables.html#id9)

在GNU的make中，还支持模式变量（Pattern-specific Variable），通过上面的目标变量中，我们知道，变量可以定义在某个目标上。模式变量的好处就是，我们可以给定一种“模式”，可以把变量定义在符合这种模式的所有目标上。

我们知道，make的“模式”一般是至少含有一个 `%` 的，所以，我们可以以如下方式给所有以 `.o` 结尾的目标定义目标变量：

```
%.o : CFLAGS = -O
```

同样，模式变量的语法和“目标变量”一样：

```
<pattern ...>; : <variable-assignment>;

<pattern ...>; : override <variable-assignment>;
```

override同样是针对于系统环境传入的变量，或是make命令行指定的变量。

















































