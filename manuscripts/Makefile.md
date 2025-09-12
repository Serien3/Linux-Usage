# Makefile



## Makefile介绍

make命令执行时，需要一个makefile文件，以告诉make命令需要怎么样的去编译和链接程序。

**Remember：依赖 ——目标 —— 规则**

假设现在我们的工程有8个c文件，和3个头文件，我们要写一个makefile来告诉make命令如何编译和链接这几个文件。我们的规则是：

1. 如果这个工程没有编译过，那么我们的所有c文件都要编译并被链接。
2. 如果这个工程的某几个c文件被修改，那么我们只编译被修改的c文件，并链接目标程序。
3. 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的c文件，并链接目标程序。

只要我们的makefile写得够好，所有的这一切，我们只用一个make命令就可以完成，make命令会自动智能地根据当前的文件修改的情况来确定哪些文件需要重编译，从而自动编译所需要的文件和链接目标程序。

### Makefile的规则

在讲述这个makefile之前，还是让我们先来粗略地看一看makefile的规则。

```makefile
target ... : prerequisites ...
    recipe
    ...
    ...
```

- target

  可以是一个object file（目标文件），也可以是一个可执行文件，还可以是一个标签（label）。对于标签这种特性，在后续的“伪目标”章节中会有叙述。

- prerequisites

  生成该target所依赖的文件和文件夹或target。

- recipe

  该target要执行的命令（任意的shell命令）。**在Makefile中的命令，必须要以 `Tab` 键开始**

这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。说白一点就是说:

```tex
prerequisites中如果有一个以上的文件比target文件要新的话，recipe所定义的命令就会被执行。
```

这就是makefile的规则，也就是makefile中最核心的内容。

### 示例

正如前面所说，如果一个工程有3个头文件和8个C文件，为了完成前面所述的那三个规则，我们的makefile 应该是下面的这个样子的。

```makefile
edit : main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
    gcc -o edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o

main.o : main.c defs.h
    gcc -c main.c
kbd.o : kbd.c defs.h command.h
    gcc -c kbd.c
command.o : command.c defs.h command.h
    gcc -c command.c
display.o : display.c defs.h buffer.h
    gcc -c display.c
insert.o : insert.c defs.h buffer.h
    gcc -c insert.c
search.o : search.c defs.h buffer.h
    gcc -c search.c
files.o : files.c defs.h buffer.h command.h
    gcc -c files.c
utils.o : utils.c defs.h
    gcc -c utils.c
clean :
    rm edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
```

反斜杠（ `\` ）是Shell命令行的换行符的意思。这样比较便于makefile的阅读。我们可以把这个内容保存在名字为 “makefile” 或 “Makefile” 的文件中，然后在该目录下直接输入命令 `make` 就可以生成执行文件`edit`。如果要删除可执行文件和所有的中间目标文件，那么，只要简单地执行一下 `make clean` 就可以了。

在这个makefile中，目标文件（target）包含：可执行文件edit和中间目标文件（ `*.o` ），依赖文件（prerequisites）就是冒号后面的那些 `.c` 文件和 `.h` 文件。每一个 `.o` 文件都有一组依赖文件，而这些 `.o` 文件又是可执行文件 `edit` 的依赖文件。依赖关系的实质就是说明了目标文件是由哪些文件生成的，换言之，目标文件是哪些文件更新的。

在定义好依赖关系后，后续的recipe行定义了如何生成目标文件的操作系统命令，一定要以一个 `Tab` 键作为开头。记住，make并不管命令是怎么工作的，他只管执行所定义的命令。make会比较targets文件和prerequisites文件的修改日期，如果prerequisites文件的日期要比targets文件的日期要新，或者target不存在的话，那么，make就会执行后续定义的命令。

这里要说明一点的是， `clean` 不是一个文件，它只不过是一个动作名字，有点像C语言中的label一样，其冒号后什么也没有，那么，make就不会自动去找它的依赖性，也就不会自动执行其后所定义的命令。要执行其后的命令，就要在make命令后明显得指出这个label的名字。这样的方法非常有用，我们可以在一个makefile中定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份，等等。

### make是如何工作的

在默认的方式下，也就是我们只输入 `make` 命令。那么，

1. make会在当前目录下找名字叫“Makefile”或“makefile”的文件。
2. 如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到“edit”这个文件，并把这个文件作为最终的目标文件。
3. 如果edit文件不存在，或是edit所依赖的后面的 `.o` 文件的文件修改时间要比 `edit` 这个文件新，那么，他就会执行后面所定义的命令来生成 `edit` 这个文件。
4. 如果 `edit` 所依赖的 `.o` 文件也不存在，那么make会在当前文件中找目标为 `.o` 文件的依赖性，如果找到则再根据那一个规则生成 `.o` 文件。
5. 当然，我们的C文件和头文件是存在的，于是make会生成 `.o` 文件，然后再用 `.o` 文件生成make的终极任务，也就是可执行文件 `edit` 了。

这就是整个make的依赖性，make会一层又一层地去找文件的依赖关系，直到最终编译出第一个目标文件。可以发现，这是一个自上而下递归的过程，通过分治或继承向下寻找，直到找到终止条件，再进行回溯归纳。在找寻的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make就会直接退出，并报错，而对于所定义的命令的错误，或是编译不成功，make是无法知晓的。make只管文件的依赖性，即，如果在我找了依赖关系之后，冒号后面的文件还是不在，那么它就不会继续工作了。

通过上述分析，我们知道，像clean这种，没有被第一个目标文件直接或间接关联，那么它后面所定义的命令将不会被自动执行，不过，我们可以显示要make执行。即命令—— `make clean` ，以此来清除所有的目标文件，以便重编译。

于是在我们编程中，如果这个工程已被编译过了，当我们修改了其中一个源文件，比如 `file.c` ，那么根据我们的依赖性，我们的目标 `file.o` 会被重编译（也就是在这个依性关系后面所定义的命令），于是 `file.o` 的文件也是最新的，于是 `file.o` 的文件修改时间要比 `edit` 要新，所以 `edit` 也会被重新链接了（详见 `edit` 目标文件后定义的命令）。

而如果我们改变了 `command.h` ，那么， `kdb.o` 、 `command.o` 和 `files.o` 都会被重编译，并且， `edit` 会被重链接。

### Makefile中使用变量

在上面的例子中，先让我们看看edit的规则：

```makefile
edit : main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
    gcc -o edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
```

我们可以看到 `.o` 文件的字符串被重复了两次，如果我们的工程需要加入一个新的 `.o` 文件，那么我们需要在两个地方加（应该是三个地方，还有一个地方在clean中）。如果makefile变得复杂，那么我们就有可能会忘掉一个需要加入的地方，而导致编译失败。所以，为了makefile的易维护，在makefile中我们可以使用变量。makefile的变量也就是一个字符串，理解成C语言中的宏可能会更好。

假设我们声明变量`objects`来表示所有可重定位文件，则我们可以用`$(objects)`来使用变量

```makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    gcc -o edit $(objects)
main.o : main.c defs.h
    gcc -c main.c
kbd.o : kbd.c defs.h command.h
    gcc -c kbd.c
command.o : command.c defs.h command.h
    gcc -c command.c
display.o : display.c defs.h buffer.h
    gcc -c display.c
insert.o : insert.c defs.h buffer.h
    gcc -c insert.c
search.o : search.c defs.h buffer.h
    gcc -c search.c
files.o : files.c defs.h buffer.h command.h
    gcc -c files.c
utils.o : utils.c defs.h
    gcc -c utils.c
clean :
    rm edit $(objects)
```

如果有新的 `.o` 文件加入，我们只需简单地修改一下 `objects` 变量就可以了。

### 让make自动推导

GNU的make很强大，它可以**自动推导**文件以及文件依赖关系后面的命令，于是我们就没必要去在每一个 `.o` 文件后都写上类似的命令，因为，我们的make会自动识别，并自己推导命令。

只要make看到一个 `.o` 文件，它就会自动的把 `.c` 文件加在依赖关系中，如果make找到一个 `whatever.o` ，那么 `whatever.c` 就会是 `whatever.o` 的依赖文件。并且 `gcc -c whatever.c` 也会被推导出来，于是，我们的makefile再也不用写得这么复杂。

```makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    gcc -o edit $(objects)

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

### Makefile的另一种风格

既然我们的make可以自动推导命令，那么多的重复的`.o`和 `.h` ，能不能把其收拢起来？借助自动推导功能，可以写出新风格的Makefile。

```bash
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    gcc -o edit $(objects)

$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h

.PHONY : clean
clean :
    rm edit $(objects)
```

这里 `defs.h` 是所有目标文件的依赖文件， `command.h` 和 `buffer.h` 是对应部分目标文件的依赖文件。

这种风格能让我们的makefile变得很短，但我们的文件依赖关系就显得有点凌乱了，一是文件的依赖关系看不清楚，二是如果文件一多，要加入几个新的 `.o` 文件，那就理不清楚了。鱼和熊掌不可兼得！

### 清空目录的规则

每个Makefile中都应该写一个清空目标文件（ `.o` ）和可执行文件的规则，这不仅便于重编译，也很利于保持文件的清洁。这是一个“修养”（呵呵，还记得我的《编程修养》吗）。一般的风格都是：

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

前面说过， `.PHONY` 表示 `clean` 是一个“伪目标”。而在 `rm` 命令前面加了一个 `-` 的意思就是，也许某些文件出现问题，但不要管，继续做后面的事。当然， `clean` 的规则不要放在文件的开头，不然，这就会变成make的默认目标，一个不成文的规矩是——“clean从来都是放在文件的最后”。

### Makefile的文件架构

Makefile里主要包含了五个东西：显式规则、隐式规则、变量定义、指令和注释。

1. **显式规则**。显式规则说明了如何生成一个或多个目标文件。这是由Makefile的书写者明显指出要生成的文件、文件的依赖文件和生成的命令。
2. **隐式规则**。由于我们的make有自动推导的功能，所以隐式规则可以让我们比较简略地书写Makefile，这是由make所支持的。
3. **变量的定义**。在Makefile中我们要定义一系列的变量，变量一般都是字符串，这个有点像你C语言中的宏，当Makefile被执行时，其中的变量都会被扩展到相应的引用位置上。
4. **指令**。其包括了三个部分，一个是在一个Makefile中引用另一个Makefile，就像C语言中的include一样；另一个是指根据某些情况指定Makefile中的有效部分，就像C语言中的预编译#if一样；还有就是定义一个多行的命令。
5. **注释**。Makefile中只有行注释，和UNIX的Shell脚本一样，其注释是用 `#` 字符。如果要在Makefile中使用 `#` 字符，可以用反斜杠进行转义，如： `\#` 。

### Makefile的文件名

默认的情况下，make命令会在当前目录下按顺序寻找文件名为 `GNUmakefile` 、 `makefile` 和 `Makefile` 的文件。在这三个文件名中，最好使用 `Makefile` 这个文件名，因为这个文件名在排序上靠近其它比较重要的文件，比如 `README`。最好不要用 `GNUmakefile`，因为这个文件名只能由GNU `make` ，其它版本的 `make` 无法识别，但是基本上来说，大多数的 `make` 都支持 `makefile` 和 `Makefile` 这两种默认文件名。

当然，也可以使用别的文件名来书写Makefile，比如：“Make.Solaris”，“Make.Linux”等，然后在使用make时利用`-f` 或 `--file` 参数指定特定的Makefile，如： `make -f Make.Solaris` 或 `make --file Make.Linux` 。如果你使用多条 `-f` 或 `--file` 参数，你可以指定多个makefile。

### 包含其他Makefile

在Makefile使用 `include` 指令可以把别的Makefile包含进来，这很像C语言的 `#include` ，被包含的文件会原模原样的放在当前文件的包含位置。 `include` 的语法是：

```makefile
include <filenames>...
```

`<filenames>` 可以是当前操作系统Shell的文件模式（可以包含路径和通配符）。

举例，有这样几个Makefile： `a.mk` 、 `b.mk` 、 `c.mk` ，还有一个文件叫 `foo.make` ，以及一个变量 `$(bar)` ，其包含了 `bish` 和 `bash` ，那么，下面的语句：

```makefile
include foo.make *.mk $(bar)
```

等价于：

```makefile
include foo.make a.mk b.mk c.mk bish bash
```

**注意**：在 `include` 前面可以有一些空字符，但是绝不能是 `Tab` 键开始。 `include` 和 `<filenames>` 可以用一个或多个空格隔开。

make命令开始时，会找寻 `include` 所指出的其它Makefile，并把其内容安置在当前的位置。就好像C/C++的 `#include` 指令一样。如果文件都没有指定绝对路径或是相对路径的话，make会在当前目录下首先寻找，如果当前目录下没有找到，那么，make还会在下面的几个目录下找：

1. 如果make执行时，使用了额 `-I` 或 `--include-dir` 参数，那么make就会在这个参数所指定的目录下去寻找。
2. 接下来按顺序寻找目录 `<prefix>/include` （一般是 `/usr/local/bin` ）、 `/usr/gnu/include` 、 `/usr/local/include` 、 `/usr/include` 。

环境变量 `.INCLUDE_DIRS` 包含当前 make 会寻找的目录列表。你应当避免使用命令行参数 `-I` 来寻找以上这些默认目录，否则会使得 `make`忽略掉所有已经设定的包含目录，包括默认目录。

如果有文件没有找到的话，make会生成一条警告信息，但不会马上出现致命错误。它会继续载入其它的文件，一旦完成makefile的读取，make会再重试这些没有找到，或是不能读取的文件，如果还是不行，make才会出现一条致命信息。如果你想让make不理那些无法读取的文件，而继续执行，你可以在include前加一个减号“-”。如：

```makefile
-include <filenames>...
```

其表示，无论include过程中出现什么错误，都不要报错继续执行。如果要和其它版本 `make` 兼容，可以使用 `sinclude` 代替 `-include` 。

### 环境变量MAKEFILES

如果你的当前环境中定义了环境变量 `MAKEFILES` ，那么make会把这个变量中的值做一个类似于 `include` 的动作。这个变量中的值是其它的Makefile，用空格分隔。只是，它和 `include` 不同的是，从这个环境变量中引入的Makefile的“默认目标”不会起作用，如果环境变量中定义的文件发现错误，make也会不理。

### make的工作方式

GNU的make工作时的执行步骤如下：

1. 读取 `Makefile`里的规则与变量（读入所有的Makefile，读入被include的其它Makefile，初始化文件中的变量，推导隐式规则并分析所有规则）。

2. 以用户指定的目标（默认是第一个 target）为根，递归构建依赖图（DAG）。

3. 对每个目标，`make` 比较目标文件与其依赖文件的时间戳：若目标缺失或比任一依赖旧，就运行该目标对应的 recipe（命令）来生成它。

4. 按依赖顺序执行命令；如果没有必要（都已最新），则什么也不做。



## 书写规则

规则会告诉make两件事，一个是生成目标的依赖关系，一个是生成目标的方法。

在Makefile中，规则的顺序是很重要的，因为，Makefile中只应该有一个最终目标，其它的目标都是被这个目标所连带出来的，所以一定要让make知道你的最终目标是什么。一般来说，定义在Makefile中的目标可能会有很多，但是第一条规则中的目标将被确立为最终的目标。如果第一条规则中的目标有很多个，那么，第一个目标会成为最终的目标。make所完成的也就是这个目标。

### 规则的语法

```makefile
targets : prerequisites
    command
    ...
```

或是这样：

```makefile
targets : prerequisites ; command
    command
    ...
```

targets是文件名，以空格分开，可以使用通配符。一般来说，我们的目标基本上是一个文件，但也有可能是多个文件。

command是命令行，如果其不与“target:prerequisites”在一行，那么，必须以 `Tab` 键开头，如果和prerequisites在一行，那么可以用分号做为分隔。（见上）

prerequisites也就是目标所依赖的文件（或依赖目标）。如果其中的某个文件要比目标文件要新，那么，目标就被认为是“过时的”，被认为是需要重生成的。这个在前面已经讲过了。

如果命令太长，你可以使用反斜杠（ `\` ）作为换行符。make对一行上有多少个字符没有限制。规则告诉make两件事，文件的依赖关系和如何生成目标文件。

一般来说，make会以UNIX的标准Shell，也就是 `/bin/sh` 来执行命令。

### 在规则中允许使用通配符

如果想定义一系列比较类似的文件，我们很自然地就想起使用通配符。make支持三个通配符： `*` ， `?` 和 `~` 。与UNIX的Shell一样。

通配符代替了一系列的文件，如 `*.c` 表示所有后缀为c的文件。一个需要我们注意的是，如果我们的文件名中含有通配符字符，如： `*` ，那么可以用转义字符 `\` ，如 `\*` 来表示真实的 `*` 字符，而不是任意长度的字符串。

例如：

```makefile
clean:
    rm -f *.o
    
print: *.c
    lpr -p $?
    touch print
```

### 文件搜寻

在一些大的工程中，有大量的源文件，我们通常的做法是把这许多的源文件分类，并存放在不同的目录中。所以，当make需要去找寻文件的依赖关系时，当然可以在文件前加上路径，但最好的方法是把一个路径告诉make，让make在自动去找。

Makefile文件中的特殊变量 `VPATH` 就是完成这个功能的，如果没有指明这个变量，make只会在当前的目录中去找寻依赖文件和目标文件。如果定义了这个变量，那么，make就会在当前目录找不到的情况下，到所指定的目录中去找寻文件了。

```makefile
VPATH = src:../headers
```

上面的定义指定两个目录，“src”和“../headers”，make会按照这个顺序进行搜索。目录由“冒号”分隔（同Linux中的环境变量分隔方式）。（当然，当前目录永远是最高优先搜索的地方）

另一个设置文件搜索路径的方法是使用make的“vpath”关键字（注意，它是全小写的），这不是变量，这是一个make的关键字，这和上面提到的那个VPATH变量很类似，但是它更为灵活。它可以指定不同的文件在不同的搜索目录中。这是一个很灵活的功能。它的使用方法有三种：

- `vpath <pattern> <directories>`

  为符合模式`<pattern>`的文件指定搜索目录`<directories>`。

- `vpath <pattern>`

  清除符合模式`<pattern>`的文件的搜索目录。

- `vpath`

  清除所有已被设置好了的文件搜索目录。



### 伪目标

```makefile
clean:
    rm *.o temp
```

正像前面例子中的“clean”一样，既然生成了许多文件编译文件，也应该提供一个清除它们的“目标”以备完整地重编译而用。 （以“make clean”来使用该目标）

“伪目标”并不是一个文件，只是一个标签，由于“伪目标”不是文件，所以make无法生成它的依赖关系和决定它是否要执行。我们只有通过显式地指明这个“目标”才能让其生效。当然，“伪目标”的取名不能和文件名重名，不然其就失去了“伪目标”的意义了。

当然，为了避免和文件重名的这种情况，我们可以使用一个特殊的标记“.PHONY”来显式地指明一个目标是“伪目标”，向make说明，不管是否有这个文件，这个目标就是“伪目标”。

```makefile
.PHONY : clean
clean :
    rm *.o temp
```

伪目标一般没有依赖的文件。但是，我们也可以为伪目标指定所依赖的文件。伪目标同样可以作为“默认目标”，只要将其放在第一个。一个示例就是，如果Makefile需要一口气生成若干个可执行文件，但我们只想简单地敲一个make完事，并且，所有的目标文件都写在一个Makefile中，那么你可以使用“伪目标”这个特性：

```makefile
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
    gcc -o prog1 prog1.o utils.o

prog2 : prog2.o
    gcc -o prog2 prog2.o

prog3 : prog3.o sort.o utils.o
    gcc -o prog3 prog3.o sort.o utils.o
```

Makefile中的第一个目标会被作为其默认目标，这里声明了一个“all”的伪目标，其依赖于其它三个目标。由于默认目标的特性是，总是被执行的，但由于“all”又是一个伪目标，伪目标只是一个标签不会生成文件，所以不会有“all”文件产生。于是，其它三个目标的规则总是会被决议。也就达到了我们一口气生成多个目标的目的。 `.PHONY : all` 声明了“all”这个目标为“伪目标”。（注：这里的显式“.PHONY : all” 不写的话一般情况也可以正确的执行，这样make可通过隐式规则推导出， “all” 是一个伪目标，执行make不会生成“all”文件，而执行后面的多个目标。建议：显式写出是一个好习惯。）

BTW，从上面的例子我们可以看出，目标也可以成为依赖。所以，伪目标同样也可成为依赖。看下面的例子：

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

### 多目标

Makefile的规则中的目标可以不止一个，其支持多目标，有可能我们的多个目标同时依赖于一个文件，并且其生成的命令大体类似。于是我们就能把其合并起来。当然，多个目标的生成规则的执行命令不是同一个，这可能会给我们带来麻烦，不过好在我们可以使用一个自动化变量 `$@` ，这个变量表示着目前规则中所有的目标的集合。

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

### 静态模式
