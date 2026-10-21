---
title: Make学习手册
date: 2026-10-21 20:09:21
tags:
- Make
category:
- 构建工具
index_img: /imgs/Make.png
excerpt: 摘要：该文档系统讲解了 GNU Make 的使用方法。以 `edit` 文本编辑器项目为例，介绍了 Makefile 的规则语法、变量定义与引用、条件指令、文本函数、隐式规则、配方执行等核心内容。重点阐述了目标与先决条件的依赖关系、自动变量、模式匹配及并行构建等机制，帮助读者掌握自动化编译与构建管理。
---

## 前置说明
为了更好地理解，我们创建一个真实的项目：一个简单的文本编辑器 `edit`，它由多个 C 源文件和头文件组成。

**项目文件结构：**

```shell
project/
├── Makefile
├── main.c
├── kbd.c
├── command.c
├── display.c
├── insert.c
├── search.c
├── files.c
├── utils.c
├── defs.h
├── command.h
└── buffer.h
```

**源文件内容示例：**

`defs.h`（全局定义）：

```c
#ifndef DEFS_H
#define DEFS_H

#define MAX_LINE 1024
extern int global_debug;

#endif
```

`command.h`（命令相关）：

```c
#ifndef COMMAND_H
#define COMMAND_H

void do_command(char *cmd);

#endif
```

`buffer.h`（缓冲区操作）：

```c
#ifndef BUFFER_H
#define BUFFER_H

void insert_char(char c);
void delete_char(void);

#endif
```

`main.c`：

```c
#include <stdio.h>
#include "defs.h"
#include "command.h"

int global_debug = 1;

int main(int argc, char **argv) {
    printf("Editor starting...\n");
    do_command("open");
    return 0;
}
```

`command.c`：

```c
#include <stdio.h>
#include "defs.h"
#include "command.h"

void do_command(char *cmd) {
    printf("Executing command: %s\n", cmd);
}
```

`kbd.c`、`display.c` 等类似，它们各自包含相应的头文件。

## 第一章 Makefile简介

`make`工具能够自动判断需要重新编译大程序的哪些部分，并发出命令进行重新编译。 为了使用`make`，必须编写一个名为`makefile`的文件，该文件描述了程序中文件之间的关系，并提供了更新每个文件的命令。在一个程序中，通常情况下，可执行文件是由目标文件更新而来的，而目标文件又是通过编译源文件得到的。最常见的是编写一个文件告诉`make`如何编译和链接程序。 当`make`重新编译目标时，必须重新编译每个修改后的C源文件。如果头文件发生了变化，每个包含头文件的C源文件都必须重新编译以保证其安全性。每次编译都会产生一个与源文件相对应的`obj`文件。最后，如果任何一个源文件已经被重新编译，所有的目标文件，不管是新生成的还是以前的编译中保存的，都必须被链接在一起，以产生新的可执行程序。

`make` 是一个自动化构建工具。它根据文件的时间戳决定哪些文件需要重新编译，并执行相应的命令。

- 需要写一个 `makefile`，描述文件之间的依赖关系和更新命令。
- 运行 `make` 时，它会读取 `makefile`，检查依赖关系，只重新编译那些过时的文件。
- 可以用命令行参数控制 make 的行为。

**通俗例子**：
假设有一个 C 程序，由 `main.c`、`foo.c` 和 `bar.c` 组成。`main.c` 包含了 `foo.h` 和 `bar.h`。如果修改了 `foo.h`，那么 `main.c` 和 `foo.c` 都需要重新编译。`make` 可以自动帮做这件事，只需要运行 `make`。

### 1.1 规则的样子

一个简单的`makefile`文件编写由"规则"组成，其形状如下：

```makefile
目标 : 先决条件
  配方
```

**目标**通常是由程序生成的文件名称；目标可以是可执行文件或目标文件。目标也可以是要实施的行动的名称，如`clean`。

**先决条件**通常是作为输入来创建目标的文件。一个目标往往依赖于多个文件。

**配方**是一种使之付诸实施的行动。一个配方可能有多个命令，要么在同一条线上，要么在各自的线上。

> 每份配方的开始处需要放置一个`tab`字符。如果想要用除`tab`以外的字符作为配方的前缀，可以用`.RECIPEPREFIX`变量来进行改变。

通常一个配方是有先决条件的规则，是用来创建目标文件的。然而，如果为目标指定配方的规则并不需要先决条件。例如，包含与目标`clean`相关联的删除命令的规则不具备先决条件。

一个规则解释了如何和何时重制某些文件，这些文件是特定规则的目标。制定执行处方的先决条件，以创建或更新目标。一个规则也可以解释如何以及何时实施一个行动。一个`makefile`文件编写可能包含规则之外的其他文本，但一个简单的`makefile`文件编写只需要包含规则即可。

### 1.2 一个简单的 Makefile

这里有一个简单的`makefile`文件编写，描述了一个被称为`edit`的可执行文件依赖于8个对象文件的方式，而对象文件又依赖于8个C源和3个头文件。

在这个例子中，所有的C文件都包含`defs.h`，只有那些定义了`edit`命令的文件包含`command.h`，只有那些改变`edit`缓冲区的低级文件包含`buffer.h`。

```makefile
edit: main.o kbd.o command.o display.o insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o insert.o search.o files.o utils.o
main.o: main.c defs.h
    cc -c main.c
kbd.o: kbd.c defs.h command.h
    cc -c kbd.c
command.o: command.c defs.h command.h
    cc -c command.c
display.o: display.c defs.h buffer.h
    cc -c display.c
insert.o: insert.c defs.h buffer.h
    cc -c insert.c
search.o: search.c defs.h buffer.h
    cc -c search.c
files.o: files.c defs.h buffer.h command.h
    cc -c files.c
utils.o: utils.c defs.h
    cc -c utils.c
clean:
    rm edit main.o kbd.o command.o display.o insert.o search.o files.o utils.o
```

可以使用`\`对于过于长的先决条件或者配方进行划分。要使用此文件编写创建名为`edit`的可执行文件，只需要输入：`make`。要使用此文件编写从目录中删除可执行文件和所有目标文件，只需要输入：`make clean`。

当一个目标是一个文件时，如果它的任何一个先决条件发生变化，它就需要重新编译或重新链接。此外，任何先决条件本身是自动生成的，都应该首先更新。在这个例子中，`edit`取决于8个对象文件中的每一个；对象文件`main.o`依赖于源文件`main.c`和头文件`defs.h`。

目标`clean`不是一个文件，而仅仅是一个动作的名称。由于通常不想执行这个规则中的行动，所以`clean`并不是其他任何规则的先决条件。因此，除非明确地告诉它，否则就不要用它做任何事。需要注意的是，这一规则不仅不是先决条件，它也不具备任何先决条件，因此该规则的唯一目的是运行指定的配方。不指向文件但只是动作的目标被称为`phony`目标。

### 1.3 make 如何处理 Makefile

默认情况下，`make`从第一个目标(不是以`.`开头的目标,除非它们包含一个或多个`/`)开始。这被称为默认目标。(目标就是`make`最终更新的目标。要想更改可以使用`.DEFAULT_GOAL`对默认情况进行覆盖。

在上一节的示例中，默认的目标是更新可执行程序的`edit`；因此，将该规则放在第一位。于是，当给命令`make`时，`make`读取当前目录中的`makefile`，从处理第一个规则开始。在本例中，该规则用于重新链接`edit`,但是`make`在能够完全处理这个规则之前，必须对`edit`所依赖的文件进行规则处理，在这种情况下就是对象文件。这些文件中的每一个都按照自己的规则进行处理。这些规则表示通过编译源文件来更新每个`.o`文件。如果源文件或任何被称为先决条件的头文件比目标文件更新快，或者目标文件不存在，则必须重新编译。 其他规则之所以被处理，是因为它们的目标是作为目标的先决条件出现的。如果目标(或者它所依赖的任何东西)不依赖其他规则，则该规则不被处理，除非用户指定需要这样做(用一个命令如`make clean`)。

在重新编译一个对象文件之前，要考虑更新它的先决条件，源文件和头文件。`makefile`没有为它们指定任何要做的事情,`.c`和`.h`文件不是任何规则的目标,所以`make`对这些文件不做任何事情。但是此时`make`会根据自己的规则更新自动生成的C程序，比如Bison或者Yacc生成的C程序。

在重新编译完对象文件后，做出是否重新链接编辑的决定前。如果文件`edit`不存在，或者其中任何一个对象文件比它新，则必须进行重新编译。如果一个对象文件刚刚被重新编译，那么它现在比`edit`更新了，所以`edit`就会被重新链接。

因此，如果更改文件`insert.c`并运行`make`，`make`就会编译该文件以更新`insert.o`，然后进行链接。如果更改文件`command.h`并运行`make`，`make`将重新编译对象文件`kbd.o`、`command.o`和`files.o`，然后链接文件`edit`。

### 1.4 用变量简化 Makefile

在上面的例子中，为了获得`edit`目标不得不在规则中两次列出所有的对象文件。

```makefile
edit: main.o kbd.o command.o display.o insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o insert.o search.o files.o utils.o
```

这样的复制是容易出错的；如果一个新的对象文件被添加到系统中，可能会将其添加到一个列表中而忘记另一个列表。这个时候就可以通过使用一个变量来消除风险和简化文件编写。变量允许一次定义一个文本字符串，然后在多个位置进行替换。

标准的做法是每个文件编写都有一个变量命名对象，OBJECTS，objs，OBJS，obj，或者OBJ是所有对象文件名的列表。

```makefile
objects = main.o kbd.o command.o display.o  insert.o search.o files.o utils.o
```

然后，如果想要在每个地方放一个对象文件名列表，就可以通过使用`$(object)`来代替变量的值。

下面是对对象文件使用一个变量时，完整的简单文件编写的样子：

```makefile
objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o
edit:$(objects)
    cc -o edit $(objects)
main.o: main.c defs.h
    cc -c main.c
kbd.o: kbd.c defs.h command.h
    cc -c kbd.c
command.o: command.c defs.h command.h
    cc -c command.c
display.o: display.c defs.h buffer.h
    cc -c display.c
insert.o: insert.c defs.h buffer.h
    cc -c insert.c
search.o: search.c defs.h buffer.h
    cc -c search.c
files.o: files.c defs.h buffer.h command.h
    cc -c files.c
utils.o: utils.c defs.h
    cc -c utils.c
clean:
    rm edit $(objects)
```

### 1.5 让make自动推导规则

`make`有一个隐含的规则，使用`cc -c `命令从相应的名为`.c`的文件中更新`.o`文件。因此，可以从对象文件的规则中省略配方。

```makefile
objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o
edit:$(objects)
    cc -o edit $(objects)
main.o: defs.h
kbd.o: defs.h command.h
command.o: defs.h command.h
display.o: defs.h buffer.h
insert.o: defs.h buffer.h
search.o: defs.h buffer.h
files.o: defs.h buffer.h command.h
utils.o: defs.h
.PHONY : clean
clean:
    rm edit $(objects)
```

这才是在实际操作中编写`makefile`的方法。

### 1.6 Makefile的另一种风格

当文件编写的对象仅由隐式规则创建时，可以将条目按其先决条件分组，而不是按其目标分组。

```makefile
objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o
edit:$(objects)
    cc -o edit $(objects)
$(objects) : defs.h
kbd.o command.o files.o: command.h
display.o insert.o search.o files.o: buffer.h
.PHONY : clean
clean:
    rm edit $(objects)
```

### 1.7 目录清理规则

`Makefile`通常还需要做其他一些事情。例如，如何删除所有的目标文件和可执行文件，使目录更干净。

```makefile
clean:
  rm edit $(objects)
```

在实践中，可能希望以一种稍微复杂的方式来编写规则，以处理未预料到的情况。通过将这样做：

```makefile
.PHONY : clean
clean:
  -rm edit $(objects)
```

这就防止了实际文件被称为`clean`的文件混淆，并使其在`rm`错误的情况下继续运行。这样的规则不应该放在文件编写的开头，因为不希望它默认运行。因此，在实例文件编写中，我们希望`edit`的规则，即重新编译的编`edit`，保持默认的目标。

由于`clean`不是编辑的先决条件，因此，如果我们在没有参数的情况下给出`make`命令，那么这个规则根本就不会运行。为了使规则运行，就必须输入`make clean`。

## 第二章 Makefile文件编写

### 2.1 Makefile 包含什么

Makefile包含五种事物：显式规则、隐式规则、变量定义、指令和注释。

*   显式规则表示何时以及如何重制一个或多个文件，它列出了目标所依赖的其他文件，还给出了配方用来创建或更新目标。

*   隐式规则是根据文件的名字来决定何时以及如何重制一个类文件。它描述了一个目标如何依赖于一个与该目标名字相似的文件，并给出了创建或更新该目标的方法。

*   变量定义是为变量指定一个文本字符串值的行，该字符串值可以在以后代入到文本中。

*   指令是在阅读make文件时，为make做一些特殊的事情而发出的指令。其中包括：

    *   读取其他的`makefile`文件
    *   决定是否使用或忽略`makefile`文件的一部分
    *   定义一个包含多行的变量

*   `#`代表注释。如果需要一个`#`，则用`\#`的反斜线将其逃逸。注释可以出现在文件编写中的任何一行。

不能在变量引用或函数调用中使用注释：`#`的任何实例在变量引用或函数调用中按字面值进行处理。

#### 2.1.1 拆分长行

`Makefile`使用"基于行"的语法，其中新行字符是特殊的，并标记着语句的结束。`make`对语句行的长度没有限制，只限于计算机内的内存量。

然而，在不进行包裹或滚动的情况下，很难读取太长的行来显示。因此，可以通过在语句的中间添加新行来格式化`makefile`以便于阅读。可以通过用反斜杠( \ )字符来逃避内部的新行来实现。

在配方行之外，反斜杠/新线被转换为单个空格字符。一旦这样做，所有围绕着反斜线/新行的白色空间就被浓缩成一个单一的空间：这包括反斜线前的所有白色空间，反斜线/新线后的所有白线，以及任何连续的反斜线/新线组合。

如果需要拆分一行但不想添加任何空白，可以使用一种微妙的技巧：用三个字符`$`、`\`和`新行`替换斜线/新行对：

```makefile
var := one$\ 
       word
```

以上等价于：

```makefile
var := one$ word
```

然后将进行变量扩展。变元指称`$`指的是一个不存在的单字名 (空格) 的变元，因此扩展到空字符串，最终的赋值等价于：

```makefile
var := oneword
```

### 2.2 Makefile查找规则

默认情况下，`make`在查找`makefile`时尝试以下名称，顺序依次为：`GNUmakefile`、`makefile`和`Makefile`。一般情况下，应该将自己的文件编写称为`makefile`或者`Makefile`。

如果`make`没有找到这些名称，则不使用任何文件编写。然后就必须用命令去指定一个目标。如果要使用非标准的文件编写名称，可以用`-f`或`--file`选项指定`makefile`名称。如果已经使用命令进行指定了，那么默认的查找规则不会生效。

### 2.3 包含其他的Makefiles

`include`指令告诉`make`暂停读取当前`makefile`并读取一个或多个其他`makefile`。该指令是`makefile`中的一行，如下所示：

```makefile
include 文件名
```

文件名可以包含`shell`文件名模式。如果文件名为空，则不包含任何内容，也不打印任何错误。 行首允许有额外的空格，可以忽略，但第一个字符不能是制表符（或`.RECIPEPREFIX`的值）。如果行以制表符开头，则将被视为配方行。`include`和文件名之间以及文件名之间需要空格；多余的空格在指令的末尾和那里被忽略。允许在行尾添加以`#`开头的注释。如果文件名包含任何变量或函数引用，它们将被展开。 例如，如果您有三个`.mk`文件，`a.mk`、`b.mk`和`c.mk`，并且`$(bar)`扩展为`bish bash`，则以下表达式:

```makefile
include foo *.mk $(bar)
```

等价于

```makefile
include foo a.mk b.mk c.mk bish bash
```

当`make`处理一个`include`指令时，它会暂停读取包含的`makefile`，并依次从每个列出的文件中进行读取。完成后，`make`将继续读取`makefile`剩余出现的指令。 使用`include`指令的一种情况是，由不同目录中的单个`makefile`处理的多个程序需要使用一组通用的变量定义。另一种情况是，希望从源文件自动生成先决条件；先决条件可以放在主makefile包含的文件中。

如果指定的名称不是以斜线开头，并且在当前目录中找不到该文件，则会搜索其他几个目录。首先，搜索使用`-I`或`--included-dir`选项指定的任何目录。`.INCLUDE_DIRS`变量将包含用于搜索所包含文件的当前目录列表。 通过在命令行中添加命令行选项`-I`，可以避免在这些默认目录中搜索。这将导致`make`忘记任何已设置的包含目录，包括默认目录。如果在这些目录中找不到包含的`makefile`，则不是立即致命的错误；继续处理包含`include`的`makefile`。一旦完成读取`makefile`，`make`将尝试重新生成任何过期或不存在的文件。只有在它找不到重新生成生成文件的规则，或者找到了规则但配方失败后，才会将丢失的生成文件诊断为致命错误。 如果想让`make`简单地忽略一个不存在或无法重新创建的`makefile`，并且没有错误消息，可以使用`-include`指令而不是`include`，如下所示：

```makefile
-include foo *.mk $(bar)
```

为了与其他`make`实现兼容，`sinclude`是`-include`的另一个名称。

### 2.4 变量MAKEFILES

如果定义了环境变量`MAKEFILES`，`make`会将其值视为要在其他`makefile`之前读取名称列表，这与`include`指令非常相似。在各种目录中搜索这些文件。此外，默认目标永远不会从这些`makefile`中获取，如果找不到`MAKEFILES`中列出的文件，则不会出错。

`MAKEFILES`的主要用途是在`make`的递归调用之间进行通信。通常不希望在顶级调用`make`之前设置环境变量，因为通常最好不要从外部弄乱`makefile`。但是，如果在没有特定生成文件的情况下运行`make`，`MAKEFILES`中的生成文件可以做一些有用的事情来帮助内置的隐式规则更好地工作，例如定义搜索路径。

### 2.5 Makefiles重制过程

如果知道不需要重新制作`Makefile`，可以通过添加以下内容来阻止`make`尝试重新制作它：

```makefile
.PHONY: Makefile
```

等价于

```makefile
Makefile::
```

如果不使用`-f`或`--file`选项指定要读取的任何生成文件，`make`将尝试使用默认的生成文件名。如果默认`makefile`都不存在，`make`将尝试创建每个`makefile`，直到成功创建一个为止，或者尝试的名称用完。如果`make`找不到或创建任何`makefile`，这不是错误；`makefile`并不总是必要的。

当使用`-t`或`--touch`选项时，不真正重新编译，只把目标文件的时间戳"摸"成最新，让 make 以为它已更新。因此，`-t`选项对更新`makefile`没有影响；即使指定了`-t`，它们也会真正更新。同样，`-q`（或`--question`）和`-n`（或`--just-print`）不会阻止更新`makefile`，因为过期的`makefile`会导致其他目标的错误输出。因此，`make -f mfile -n foo`将更新`mfile`，读取它，然后打印配方以更新`foo`及其先决条件，而无需运行它。为`foo`打印的配方将是`mfile`更新内容中指定的配方。

### 2.6 覆盖另一个Makefile的一部分

有时，拥有一个与另一个`makefile`基本相同的`makefile`是有用的。通常可以使用`include`指令将一个包含在另一个中，并添加更多目标或变量定义。但是，两个`makefile`为同一目标提供不同的配方是无效的。但还有另一种方法,在`makefile`中，可以使用模式规则来表示，要重新生成无法从包含`makefile`中的信息生成的任何目标，`make`应该在另一个`make`文件中查找。

例如，如果有一个名为`makefile`的`makefile`，它说明了如何制作目标`foo`（和其他目标），可以编写一个称为`GNUmakefile`的`makefiles`，其中包含：

```makefile
foo:  
  frobnicate > foo
%: force  
  @$(MAKE) -f Makefile $@ 
force: ;
```

如果输入`make foo`，`make`会找到`GNUmakefile`，读取它，并看到要制作`foo`，它需要运行配方`frobnate > foo`。如果`make bar`，`make`将无法在`GNUmakefile`中创建`bar`，因此它将使用模式规则中的配方：`make-f Makefile bar`。如果`Makefile`提供了更新栏的规则，`make`将应用该规则。

### 2.7 make读取Makefile规则

GNU make的工作分为两个不同的阶段。在第一阶段，它读取所有makefile、包含的makefile等，并将所有变量及其值以及隐式和显式规则内部化，并构建所有目标及其先决条件的依赖图。在第二阶段，make使用这些内部数据来确定哪些目标需要更新，并运行更新这些目标所需的配方。

如果扩展发生在第一阶段，则扩展是立即的：make将在解析makefile时扩展构造的该部分。如果不是立即进行，扩展就会推迟。延迟构造部分的扩展会延迟到使用扩展为止：无论是在即时上下文中引用它，还是在第二阶段需要它时。

#### 2.7.1 变量赋值

变量定义解析如下：

```makefile
immediate = deferred 
immediate ?= deferred 
immediate := immediate 
immediate ::= immediate 
immediate :::= immediate-with-escape 
immediate += deferred or immediate 
immediate != immediate

define immediate  
  deferred 
endef  

define immediate =  
  deferred 
endef  

define immediate ?=  
  deferred 
endef

define immediate :=  
  immediate 
endef

define immediate ::=  
  immediate 
endef  

define immediate :::=  
  immediate-with-escape 
endef  

define immediate +=  
  deferred or immediate 
endef  

define immediate !=  
  immediate 
endef
```

对于附加运算符`+=`，如果变量之前被设置为简单变量（`:=`或`::=`），则右侧被视为立即，否则被视为延迟。

#### 2.7.2 条件指令

立即解析条件指令。这意味着，在条件指令中不能使用自动变量，因为在调用该规则的配方之前，不会设置自动变量。如果需要在条件指令中使用自动变量，则必须将条件移动到配方中，并改用shell条件语法。

#### 2.7.3 规则定义

无论形式如何，规则总是以相同的方式扩展：

```makefile
immediate : immediate ; deferred  
  deferred
```

也就是说，目标和先决条件部分会立即扩展，用于构建目标的配方总是会被推迟。对于显式规则、模式规则、后缀规则、静态模式规则和简单的先决条件定义都是如此。

### 2.8 Makefiles是如何解析的

1.  读取完整的逻辑行，包括反斜杠转义行。
2.  删除注释。
3.  如果该行以配方前缀字符开头，并且处于规则上下文中，将该行添加到当前配方中并读取下一行。
4.  展开出现在直接展开上下文中的行元素。
5.  扫描行中的分隔符，如`:`或`=`，以确定该行是宏赋值还是规则。
6.  将结果操作内部化并读取下一行。这样做的结果是，如果宏只有一行长，它可以扩展为整个规则。

```makefile
myrule = target : ; echo built
$(myrule)
```

但是下面这样是不行的：

```makefile
define myrule 
target:  
  echo built 
endef 

$(myrule)
```

上面的`makefile`生成了一个目标`target`的定义，其中包含先决条件`echo`和`build`，就好像`makefile`包含`target:echo build`，而不是一个包含配方的规则一样。展开完成后，行中仍存在的换行符将作为普通空格忽略。为了正确展开多行宏，必须使用**eval**函数：这将导致make解析器在展开的宏的结果上运行。

### 2.9 二次扩展

Make还能够为makefile中定义的部分或所有目标启用先决条件的第二次扩展。为了实现第二次扩展，必须在使用此功能的第一个先决条件列表之前定义`.SECONDEXPANSION`。

```makefile
.SECONDEXPANSION: 
ONEVAR = onefile 
TWOVAR = twofile 
myfile: $(ONEVAR) $$(TWOVAR)
```

对于例子：

```makefile
.SECONDEXPANSION: 
main_OBJS := main.o try.o test.o 
lib_OBJS := lib.o api.o 
main lib: $$($$@_OBJS)
```

初始扩展之后，main和lib目标的先决条件都是`$($@_OBJS)`,而二次扩展后，其将产生`$(main_OBJS)`和`$(lib_OBJS)`这两个引用。

#### 2.9.1 显式规则的二次扩展

在显式规则的二次扩展过程中，`$$@`和`$$%`分别计算目标的文件名，如果目标是存档成员，则计算目标成员名。`$$<`变量的计算结果是此目标的第一个规则中的第一个先决条件。`$$^`和`$$+`计算出同一目标出现的规则的所有先决条件列表(`$$+`有重复，`$$^`没有)。以下示例将有助于说明这些行为：

```makefile
.SECONDEXPANSION: 
foo: foo.1 bar.1 $$< $$^ $$+ # line #1 
foo: foo.2 bar.2 $$< $$^ $$+ # line #2 
foo: foo.3 bar.3 $$< $$^ $$+ # line #3
```

变量`$$？`、`$$*`不可用，展开为空字符串。

#### 2.9.2 静态模式规则的二次扩展

静态模式规则的二次扩展规则与上述显式规则的规则相同，但有一个例外：对于静态模式规则，`$$*`变量设置为模式茎。与明确的规则一样，`$$？`不可用，并扩展为空字符串。

#### 2.9.3 隐式规则的二次扩展

在搜索隐式规则时，它会替换词干，然后用匹配的目标模式对每个规则进行二次扩展。自动变量的值以与静态模式规则相同的方式导出。例如：

```makefile
.SECONDEXPANSION: 
foo: bar 
foo foz: fo%: bo% 
%oo: $$< $$^ $$+ $$*
```

最终结果为`foo: bar bar boo bar boo f`

## 第三章 规则编写

生成文件中会出现一条规则，说明何时以及如何重新生成某些文件，称为规则的目标。它列出了作为目标先决条件的其他文件，以及用于创建或更新目标的配方。

### 3.1 规则语法

一个最常见的规则类似下面这样：

```makefile
targets : prerequisites  
    recipe ...
```

或者

```makefile
targets : prerequisites ; recipe  
    recipe ...
```

一条规则告诉make两件事：目标何时过期，以及如何在必要时更新它们。过期的标准是根据先决条件指定的，先决条件由用空格分隔的文件名组成。如果目标不存在或比任何先决条件都旧（通过比较上次修改时间），则该目标已过期。

### 3.2 先决条件类型

make理解两种不同类型的先决条件：上一节中描述的正常先决条件和仅按顺序先决条件。一个正常的先决条件：首先，它规定了调用配方的顺序：目标的所有先决条件的配方将在目标的配方开始之前完成。其次，它强调了一种依赖关系：如果任何先决条件比目标更新，则认为目标已过时，必须重新构建。

通常，这正是想要的：如果目标的先决条件被更新，那么目标也应该被更新。有时可能希望确保先决条件在目标之前构建，但如果先决条件已更新，则不强制更新目标。仅按顺序先决条件用于创建此类关系。通过在先决条件列表中放置管道符号`(|)`，可以指定仅按顺序排列的先决条件：管道符号左侧的任何先决条件都是正常的；该权利的任何先决条件仅适用于顺序：

```makefile
targets : normal-prerequisites | order-only-prerequisites
```

适用于下面这种情况：

```makefile
main.o: main.c | build 
  gcc -c main.c -o build/main.o 
build: 
  mkdir -p build
```

`build`目录不存在就先创建它, `build`目录存在但时间戳比`main.o`新,使用管道符不会触发`main.o`重建，只有`main.c`变了，才会重新编译。

### 3.3 在文件名中使用通配符

单个文件名可以使用通配符字符指定多个文件,`make`中的通配符为`*`、`?` 和`[...]`。如果一个表达式匹配多个文件，则对结果进行排序。然而，多个表达式不会被全局排序。

通配符扩展在目标和先决条件中是自动进行的。在配方中，`shell`负责通配符的扩展。在其他情况下，只有在使用`wildcard`函数显式请求的情况下，通配符扩展才会发生。通配符可以通过在其前面加上斜线()来关闭。

#### 3.3.1 通配符例子

通配符可以用于规则的配方中，其中它们是由shell扩展的。例如，这里有一个删除所有对象文件的规则：

```makefile
clean:  
  rm -f *.o
```

通配符在规则的先决条件中也是有用的。在文件编写中遵循以下规则，`make print`将打印自上次打印以来发生更改的所有`.c`文件：

```makefile
print: *.c  
  lpr -p $?
  touch print
```

当定义一个变量时，不会发生通配符扩展。因此，如果写这段代码：

```
objects = *.o
```

那么变量对象的值就是实际的字符串`*.o`。然而，如果objects作为目标或先决条件下的值，通配符就会在那里发生扩展。如果要在配方中使用`objet`的值，那么在配方运行时，`shell`可能会执行通配符扩展。若要将对象设置为展开，则改用：

```makefile
objects := $(wildcard *.o)
```

#### 3.3.2 通配符的缺陷

现在这里是一个使用通配符扩展的例子，它想做的是，可执行文件`foo`是由目录中的所有对象文件组成的，可以这样写：

```makefile
objects = *.o 
foo : $(objects)  
  cc -o foo $(CFLAGS) $(objects)
```

对象的值是实际的字符串`*.o`。在`foo`规则中进行通配符扩展，使得每个存在的`.o`文件成为`foo`的先决条件，必要时进行重新编译。但是如果删除了所有的`.o`文件又会是什么呢?当一个通配符与任何文件都不匹配时，就按原样保留，因此`foo`将保留`*.o`。由于不太可能存在这样的文件，`make`会给一个错误，说它不知道如何生成`*.o`。

#### 3.3.3 wildcard函数

通配符扩展在规则中自动发生。但是通配符扩展通常不会在变量被设置时发生，或者在函数的参数内部发生。如果要在这样的地方做通配符扩展，就需要使用通配符功能，如下：

```makefile
$(wildcard pattern...)
```

这个字符串在文件编写中的任何地方使用，被一个与给定文件名模式匹配的现有文件名的空间分离列表所取代。如果没有现有的文件名匹配到一个模式，则从通配符函数的输出中省略该模式。

与规则中的通配符扩展一样，对通配符函数的结果进行排序。但每个单独的表达式是单独排序的，因此`$(wildcard *.c *.h)`将扩展到所有的`.c`进行排序，然后所有的`.h`进行排序。

通配符函数的一个用途是得到一个目录中所有C源文件的列表，如下所示:

```makefile
$(wildcard *.c)
```

通过将结果中的`.c`后缀替换为`.o`，我们可以将C源文件的列表转换成对象文件的列表:

```makefile
$(patsubst %.c,%.o,$(wildcard *.c))
```

### 3.4 为先决条件设置搜索目录

#### 3.4.1 VPATH：搜索所有先决条件的路径。

`make`变量`VPATH`的值指定了`make`应该搜索的目录列表。通常情况下，目录被期望包含不在当前目录中的先决条件文件；然而，使用`VPATH`作为规则的先决条件和目标的搜索列表。

因此，如果当前目录中不存在被列为目标或前提的文件，则在`VPATH`列出的目录中搜索该名称的文件。如果在其中一个文件中发现了该文件，则该文件可能成为先决条件。然后，规则可以指定先决条件列表中的文件名称，就好像它们都存在于当前目录中。

在`VPATH`变量中，目录名用冒号或空格分隔。目录被列出的顺序是在其搜索过程中所遵循的顺序。

```makefile
VPATH = src:../headers
```

有了`VPATH`的这个值，就有了下面的规律：

```makefile
foo.o : foo.c
```

被解读为仿佛是这样写的：

```makefile
foo.o : src/foo.c
```

假设文件`foo.c`不存在于当前目录中，而是存在于`src`目录中。

#### 3.4.2 vpath指令

与VPATH变量类似，但更有选择性的是vpath指令，它允许为特定类型的文件名指定搜索路径：匹配特定模式的文件名。这样就可以为一类文件名提供一定的搜索目录，为其他文件名提供其他目录。

vpath指令有三种形式：

1.  `vpath pattern directories`:指定与模式匹配的文件名的搜索路径目录。
2.  `vpath pattern`:清除与模式相关的搜索路径。
3.  `vpath`:清除之前用vpath指令指定的所有搜索路径。

`vpath`模式是一个包含`%`字符的字符串。该字符串必须与正在搜索的先决条件的文件名相匹配，`%`字符匹配任何零或更多字符的序列。

当先决条件在当前目录中不存在时，如果vpath指令中的模式与先决条件文件名匹配，那么该指令中的目录就像`VPATH`变量中的目录一样被搜索。如下：

```makefile
vpath %.h ../headers
```

如果在当前目录中没有找到该文件，则告诉make在目录`../headers`中查找任何名称以`.h`结尾的必备条件。

若多个`vpath`模式与先决文件名匹配，则对每个匹配的`vpath`指令逐一进行处理，搜索每个指令中提到的所有目录，`make`按照多个vpath指令在`make`文件中出现的顺序进行处理；同一模式的多个指令是相互独立的。

```makefile
vpath %.c foo 
vpath % blish 
vpath %.c bar
```

会在foo中寻找一个以`.c`结尾的文件，然后blish，然后bar。

```makefile
vpath %.c foo:bar 
vpath % blish
```

会在foo中寻找一个以`.c`结尾的文件，然后是bar，然后是blish。

#### 3.4.3 如何进行目录搜索

当通过目录搜索找到先决条件时，无论类型如何，所找到的路径名都可能不是先决条件列表中实际提供给的路径名。有时通过目录搜索发现的路径会被丢弃。`make`算法用于决定是否保留或放弃通过目录搜索发现的路径如下:

1.  如果在文件编写中指定的路径上不存在目标文件，则进行目录搜索。
2.  如果目录搜索成功，则保留该路径，并暂时将该文件作为目标存储。
3.  这一目标的所有先决条件都用同样的方法进行检验。
4.  在处理完先决条件后，目标可能需要重新构建：a. 如果目标不需要重建，目录搜索过程中找到的文件路径将用于包含该目标的任何先决条件列表。如果make不需要重新构建目标，那么就使用通过目录搜索找到的路径。b. 如果目标确实需要重建，则丢弃目录搜索时发现的路径名，使用文件编写中指定的文件名重建目标。如果make必须重建，那么目标在本地重建，而不是在通过目录搜索找到的目录中重建。

#### 3.4.4 使用目录搜索编写食谱

当通过目录搜索在另一个目录中找到先决条件时，无法改变规则的配方。因此，必须谨慎地编写配方，以便它会在目录中寻找使它找到的先决条件。

#### 3.4.5 目录搜索和隐式规则

通过`VPATH`中指定的目录或使用vpath的搜索也发生在考虑隐式规则的过程中。例如，当文件`foo.o`没有显式规则时，如果文件`foo.c`存在，则考虑隐式规则，如内置规则来编译`foo.c`。如果当前目录中缺少这样的文件，则为其查找合适的目录。如果`foo.c`存在，则采用C编译的隐式规则。

#### 3.4.6 链接库的目录搜索

当写一个名字是`-lname`形式的先决条件时，这个特殊的特征就发挥了作用。当一个先决条件的名字具有`-lname`形式时，通过搜索当前目录中的文件库名`.so`，并对其进行特殊处理，如果未找到，则搜索当前目录中的文件库名`.a`。

虽然待搜索文件的默认集合为`libname.so`和`libname.a`，但可以通过`.LIBPATTERNS`变量进行修改。这个变量值中的每个词都是一个模式串。当出现类似`-lname`的先决条件时，make将用`name`替换列表中每个模式的%，并使用每个库文件名执行上述目录搜索。`.LIBPATTERNS`的默认值是`lib%.so lib%.a`。

### 3.5 Phony目标

一个虚假的目标是一个不真正的文件名；它只是一个名字，当提出一个明确的请求时，要执行一个菜谱。使用假目标有两个原因：一是为了避免与同名文件发生冲突，二是为了提高性能。 如果写了一个规则，它的配方不会创建目标文件，那么每当目标出现需要重新制作时，该配方就会被执行。例如：

```makefile
clean:  
  rm *.o temp
```

由于`rm`命令没有创建一个命名为`clean`的文件，所以很可能根本就不存在这样的文件。因此，每次输入`make clean`时，都会执行`rm`命令。 在这个例子中，如果在这个目录中创建了一个名为clean的文件，则clean目标将不能正常工作。由于它没有先决条件，所以clean总是被认为是最新的，它的配方不会被执行。为了避免这个问题，可以将目标明确地声明为假目标，将其作为特殊目标的先决条件。

```makefile
.PHONY: clean
clean:  
  rm *.o temp
```

一旦这样做，无论是否有一个名为clean的文件，`make clean`都会运行这个配方。PHONY的先决条件总是被理解为字面上的目标名称，从不被理解为模式。

虚假目标可以有先决条件。当一个目录包含多个程序时，在一个`./Makefile`中描述所有的程序是最方便的。由于默认情况下重做的目标将是文件编写中的第一个目标，因此通常将这个虚假的目标命名为`all`，并将其作为先决条件赋予所有单独的程序。例如:

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

### 3.6 没有配方或先决条件的规则

如果一条规则既没有先决条件（prerequisites）也没有配方（recipe/命令），并且该规则的目标是一个不存在的文件，那么每当执行到这条规则时，Make 就会认为这个目标已经被更新过了。依赖于这个目标的所有目标都将始终运行其食谱。如下：

```makefile
clean: FORCE  
  rm $(objects) 
FORCE:
```

这里的目标`FORCE`满足特定条件，因此依赖它的目标`clean`就被迫运行它的食谱。

### 3.7 用空目标文件记录事件

空目标是假目标的一种变体；它用于为不时明确要求的某个动作保留菜谱。与一个假目标不同，这个目标文件是可以真实存在的；但是文件的内容无所谓，而且通常都是空的。

空目标文件的目的是用它的最后一次修改时间记录规则的配方最后一次执行的时间。空目标文件应该有一些先决条件（否则它就没有意义了）。当要求重新生成这个空目标时，如果任何先决条件比该目标更新，那么配方（recipe/命令）就会被执行；换句话说，如果某个先决条件自上次重新生成该目标以来发生了改变，就会触发执行。下面是一个例子：

```makefile
print: foo.c bar.c  
  lpr -p $? 
  touch print
```

### 3.8 特殊内建目标名称

| 特殊目标                   | 核心作用                       | 关键细节 / 注意事项                                          | 典型场景                                         |
| -------------------------- | ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| `.PHONY`                   | 将先决条件声明为伪目标         | 无条件执行配方，忽略文件是否存在及时间戳                     | `clean`、`all`、`install` 等不生成实际文件的操作 |
| `.SUFFIXES`                | 定义后缀规则使用的后缀列表     | 老式隐式规则写法，现多被模式规则取代                         | 兼容旧式 Makefile，如 `.c.o` 规则                |
| `.DEFAULT`                 | 为找不到规则的目标提供默认配方 | 也作用于“作为先决条件但无目标规则”的文件                     | 兜底构建，防止因缺少规则而报错                   |
| `.PRECIOUS`                | 保护目标不被删除               | Make 被中断时不删除；中间文件不再需要时也不删除；可保护匹配模式（如 `%.o`）的中间文件 | 保护编译耗时长的 `.o` 文件                       |
| `.INTERMEDIATE`            | 将先决条件视为中间文件         | 无先决条件时无效                                             | 显式标记某些文件为临时产物                       |
| `.NOTINTERMEDIATE`         | 先决条件永不视为中间文件       | 无先决条件时，所有目标都视为非中间文件；先决条件为模式时，匹配该模式的目标也非中间 | 防止 Make 自动删除某些文件                       |
| `.SECONDARY`               | 视为中间文件，但永不自动删除   | 可避免冗余重建；无先决条件时所有目标都受保护                 | 避免因中间文件丢失而触发不必要的重新链接         |
| `.SECONDEXPANSION`         | 启用先决条件的二次展开         | 在它之后定义的先决条件列表会在所有 Makefile 读取完后再次展开 | 在先决条件中使用自动变量（如 `$$@`）实现动态依赖 |
| `.DELETE_ON_ERROR`         | 配方失败时删除已修改的目标     | 只要在 Makefile 中出现一次即全局生效                         | 防止失败产物被误认为成功，导致下次构建出错       |
| `.IGNORE`                  | 忽略配方执行中的错误           | 有先决条件时只忽略这些文件的错误；无先决条件时忽略所有错误（不推荐） | 容忍某些非关键命令的失败                         |
| `.LOW_RESOLUTION_TIME`     | 处理低精度时间戳文件           | 适用于 `cp -p` 等丢弃亚秒部分的命令；归档成员自动处理，无需声明 | 避免因时间戳精度问题误判文件过期                 |
| `.SILENT`                  | 不打印配方命令                 | 有先决条件时只静默这些文件；无先决条件时静默所有（可用 `-s` 替代） | 减少构建输出噪音                                 |
| `.EXPORT_ALL_VARIABLES`    | 默认导出所有变量给子进程       | 等价于不带参数的 `export`                                    | 递归 Make 时统一传递变量                         |
| `.NOTPARALLEL`             | 禁止并行执行                   | 无先决条件时本次调用全部串行（递归子 Make 仍可并行）；有先决条件时仅这些目标的先决条件串行（隐式加 `.WAIT`） | 解决并行构建时的资源冲突                         |
| `.ONESHELL`                | 整个配方交给同一个 Shell 执行  | 默认每行独立 Shell，启用后所有行在同一 Shell 中              | 需要跨行保持 `cd`、变量等状态                    |
| `.POSIX`                   | 以 POSIX 兼容模式解析和运行    | 配方像带 `-e` 一样，首个失败命令立即终止；GNU 高级特性仍可用 | 需要严格 POSIX 行为的可移植构建                  |
| `.c.o`**&#x20;等后缀组合** | 老式后缀规则                   | 任何已定义隐式规则后缀作为目标时也算特殊目标；原理上任意目标名拆成两半加入后缀列表即可 | 维护旧式 Makefile                                |


### 3.9 多目标规则

当一个明确的规则有多个目标时，可以以两种可能的方式之一来处理：作为独立的目标或作为分组的目标。它们被处理的方式由目标列表后出现的分隔符决定。

#### 3.9.1 具有独立目标的规则

使用标准目标分隔符`:`的规则定义的是独立目标，这等价于为每个目标分别写一遍相同的规则，先决条件和配方都被复制了一份。通常，配方会使用诸如`$@`这样的自动变量来指定当前正在构建的是哪个目标。带有独立目标的规则在两种情况下很有用：

1.  只想要先决条件，不需要配方。
2.  所有目标使用相似的配方。

#### 3.9.2 带分组目标的规则

如果希望一条配方在一次调用中生成多个文件（而不是像独立目标那样每个目标各执行一次），可以通过声明规则使用分组目标来表达这种关系。分组目标规则使用分隔符 &:（这里的 & 用来表示“全部”）。 当 make 构建分组目标中的任意一个时，它会理解为：由于该配方的执行，组内所有其他目标也一并被更新了。此外，如果组内只有部分目标过期或缺失，make 会意识到运行一次配方就能更新所有目标。最后，如果组内任意一个目标过期，则所有分组目标都被视为过期。例如，以下规则定义了一个分组目标：

```makefile
foo bar biz &: baz boz
	echo $^ > foo
	echo $^ > bar
	echo $^ > biz
```

在执行分组目标的配方时，自动变量 \$@ 被设置为组内触发该规则的那个特定目标的名字。如果在分组目标规则的配方中依赖这个变量，必须格外小心。 与独立目标不同，分组目标规则必须包含配方。不过，作为分组目标成员的目标，也可以出现在没有配方的独立目标规则定义中。 每个目标只能关联一个配方。如果一个分组目标出现在某个独立目标规则中，或出现在另一个带配方的分组目标规则中，会收到一个警告，并且后者的配方会替换前者的配方。此外，该目标会从之前的分组中被移除，只出现在新的分组中。 如果希望一个目标出现在多个分组中，那么在声明包含该目标的所有分组时，必须使用双冒号分组目标分隔符 &::。分组双冒号目标各自独立对待，每个分组双冒号规则的配方最多执行一次——前提是其多个目标中至少有一个需要更新。

### 3.10 一个目标的多条规则

一个文件可以作为多条规则的目标。所有规则中提到的先决条件会合并成一个该目标的先决条件列表。如果目标比任何一条规则中的任何先决条件都旧，则执行配方。一个文件只能有一个要执行的配方。如果多条规则为同一个文件提供了配方，make 使用最后给出的那一个，并打印一条错误信息。

#### 3.10.1 用“只含先决条件的额外规则”批量添加依赖

一条只含先决条件的额外规则，可以用来一次性给多个文件添加额外的先决条件。例如，Makefile 中常有一个变量（如 objects），包含系统中所有编译器输出文件的列表。一种简单的方法来声明“如果 config.h 改变，它们全部必须重新编译”，可以这样写：

```makefile
objects = foo.o bar.o
foo.o : defs.h
bar.o : defs.h test.h
$(objects) : config.h
```

#### 3.10.2 用变量指定额外先决条件

另一个巧妙之处是：额外先决条件可以用一个通过命令行参数设置的变量来指定。

```makefile
extradeps=
$(objects) : $(extradeps)
```

### 3.11 静态模式规则

静态模式规则是指定多个目标、并根据目标名构造每个目标的先决条件名的规则。它们比带多个目标的普通规则更通用，因为目标不必拥有完全相同的先决条件。它们的先决条件必须是类似的，但不一定完全相同。

#### 3.11.1 静态模式规则的语法

静态模式规则的语法如下：

```makefile
targets ...: target-pattern: prereq-patterns ...
	recipe
	...
```

*   targets 列表指定该规则适用的目标。目标可以包含通配符，就像普通规则的目标一样。
*   target-pattern 和 prereq-patterns 说明如何计算每个目标的先决条件。每个目标与 target-pattern 匹配，提取出目标名的一部分，称为词干（stem）。这个词干被替换到每个 prereq-pattern 中，从而生成先决条件名。

每个模式通常只包含一次字符 %。当 target-pattern 匹配一个目标时，% 可以匹配目标名的任意部分；这部分称为词干。模式的其余部分必须精确匹配。例如，目标 foo.o 匹配模式 %.o，词干为 foo。目标 foo.c 和 foo.out 不匹配该模式。 每个目标的先决条件名，是通过把词干替换到每个先决条件模式的 % 中生成的。例如，如果一个先决条件模式是 %.c，那么替换词干 foo 后得到先决条件名 foo.c。写一个不含 % 的先决条件模式也是合法的；此时该先决条件对所有目标都相同。

模式规则中的 % 字符可以用前置反斜杠（\）来转义。原本用于转义 % 的反斜杠可以用更多反斜杠来转义。转义 % 或其他反斜杠的反斜杠，在模式与文件名比较或替换词干之前会被移除。不会影响 % 字符的反斜杠则原样保留。 下面是一个例子，它从对应的 .c 文件编译出 foo.o 和 bar.o：

```makefile
objects = foo.o bar.o

all: $(objects)

$(objects): %.o: %.c
	$(CC) -c $(CFLAGS) $< -o $@
```

#### 3.11.2 静态模式规则 vs 隐式规则

*   静态模式规则与定义为模式规则的隐式规则有很多共同点。两者都有目标模式和用于构造先决条件名的模式。区别在于 make 如何决定规则何时适用。
*   隐式规则可以适用于任何匹配其模式的目标，但仅当该目标没有以其他方式指定配方，且先决条件可以被找到时才适用。如果有多条隐式规则看起来都适用，只有一条会生效；选择取决于规则顺序。静态模式规则适用于在规则中精确指定的目标列表。它不能适用于任何其他目标，并且总是适用于所指定的每个目标。如果两条冲突的规则都适用，且都有配方，那是错误。 静态模式规则优于隐式规则的原因：
*   可能希望覆盖少数几个文件的常规隐式规则——这些文件的名称无法从语法上归类，但可以在显式列表中给出。
*   如果无法确定所用目录的精确内容，可能不确定哪些无关文件会导致 make 使用错误的隐式规则。选择可能取决于隐式规则搜索的顺序。使用静态模式规则就没有不确定性：每条规则精确适用于指定的目标。

### 3.12 双冒号规则

双冒号规则是用`::`代替,写在目标名之后的显式规则。当同一个目标出现在多条规则中时，它们的处理方式与普通规则不同。带双冒号的模式规则有完全不同的含义。当一个目标出现在多条规则中时，所有规则必须是同一类型：全是普通规则，或全是双冒号规则。如果是双冒号规则，则每条规则彼此独立。每条双冒号规则的配方在该目标比该规则的任何先决条件都旧时执行。如果该规则没有先决条件，则其配方总是执行（即使目标已存在）。这可能导致双冒号规则一条都不执行、执行任意几条、或全部执行。 同一个目标的多个双冒号规则实际上是完全独立的。每条双冒号规则被单独处理，就像不同目标的规则被处理一样。 一个目标的多条双冒号规则按它们在 Makefile 中出现的顺序执行。然而，双冒号规则真正有意义的场合，是那些配方执行顺序无关紧要的情况。双冒号规则有些晦涩，且不常很有用；它们提供了一种机制，用于“更新目标所使用的方法因哪个先决文件导致更新而不同”的情况，而这种情况很罕见。 每条双冒号规则都应指定一个配方；如果没有，且存在适用的隐式规则，则会使用隐式规则。

### 3.13 自动生成先决条件

在程序的 Makefile 中，需要写的许多规则往往只是说明某个目标文件依赖于某个头文件。例如，如果 main.c 通过 #include 使用了 defs.h，会写：

```makefile
main.o: defs.h
```

需要这条规则，以便 make 知道每当 defs.h 改变时必须重新生成 main.o。可以看到，对于大型程序，必须在 Makefile 中写几十条这样的规则。而且，每次添加或删除 #include 时，都必须非常小心地更新 Makefile。

## 第四章 配方规则

### 4.1 配方语法

> 本章节就是一句话：Makefile 里的命令，写的时候有什么规则？

#### 4.1.1 规则 1：每条命令必须以 Tab 开头

```makefile
target: prerequisites
<Tab>命令1
<Tab>命令2
```

注意： 必须是 Tab 字符，不能是空格！这是 Makefile 最容易踩的坑。

#### 4.1.2 规则 2：空行和注释

空行会被忽略,以#开头会被忽略。 配方里的注释会被传给shell，由shell决定怎么处理。

```makefile
target:
	# 这是注释，会被传给 shell
	echo "hello"
```

#### 4.1.3 规则 3：续行

如果一条命令太长，可以用 \ 续行：

```makefile
target:
	echo "这是一条很长的命令" \
	     "可以分成多行写"
```

续行后，make 会把它们合并成一条命令，交给同一个 shell 执行。

#### 4.1.4 规则 4：配方中的变量和函数

配方里的 \$ 会被 make 展开：

```makefile
target:
	echo $(FOO)
```

$(FOO) 会被 make 替换成变量的值。 如果想传给 shell 一个$，需要写 \$\$：

```makefile
target:
	echo $$HOME    # 传给 shell 的是 $HOME
```

### 4.2 配方回显

> 一句话：make 执行命令时，要不要把命令打印出来？

#### 4.2.1 默认行为：回显

```makefile
target:
	echo "hello"
```

运行 make target，会看到：

```makefile
echo "hello"
hello
```

第一行是make回显的命令本身,第二行是命令的实际输出。

#### 4.2.2 场景 1：用 @ 禁止回显

```makefile
target:
	@echo "hello"
```

运行结果：

```makefile
hello
```

只输出结果，不显示命令本身。

#### 4.2.3 场景 2：用 -s 全局禁止回显

```makefile
make -s
```

或者

```makefile
make --silent
```

所有命令都不回显。

#### 4.2.4 场景 3：用 .SILENT 禁止特定目标的回显

```makefile
.SILENT: target1 target2
```

target1、target2目标的命令不回显。如果 .SILENT 没有先决条件：

```makefile
.SILENT:
```

则所有命令都不回显（等价于 -s）。

### 4.3 配方执行

> 一句话：make 怎么执行配方里的命令？

#### 4.3.1 规则 1：每行命令一个独立的 shell

```makefile
target:
	cd /tmp
	pwd
```

可能以为先 cd /tmp，然后 pwd 输出 /tmp。 实际结果是 pwd 输出的是当前目录，不是 /tmp！ 原因：make 为每一行命令启动一个独立的 shell。第一行的 cd /tmp 只影响第一个 shell，第二个 shell 还是在原目录。

#### 4.3.2 规则 2：怎么让多行命令在同一个 shell 里执行？

##### 方法 1：用 ; 连接

```makefile
target:
	cd /tmp; pwd
```

##### 方法 2：用 \ 续行

```makefile
target:
	cd /tmp; \
	pwd
```

##### 方法 3：用 .ONESHELL

```makefile
.ONESHELL:

target:
	cd /tmp
	pwd
```

这样整个配方的所有行都在同一个 shell 里执行。

#### 4.3.3 规则 3：并行执行与 -j

默认情况下，make 是串行执行的：一个目标一个目标地做。 用 -j 可以并行执行：

```makefile
make -j4
```

同时运行 4 个任务。注意：并行执行时，配方的输出会混在一起，很难看懂。

#### 4.3.4 规则 4：-n (只打印不执行)

```makefile
make -n
```

make 会打印所有要执行的命令，但不真正执行。常用于调试。

#### 4.3.5 规则 5：-t (只标记不执行)

```makefile
make -t
```

make 会把目标文件的时间戳更新，但不执行配方。用途：假装目标已经是最新的，跳过实际编译。

#### 4.3.6 规则 6：-q (只检查不执行)

```makefile
make -q
```

make 不执行任何命令，只检查目标是否是最新的,如果是最新的，返回 0。 如果过期了，返回 1。用途：在脚本里判断"是否需要重新构建"。

### 4.4 并行执行

> 一句话：怎么让 make 同时做多件事？

#### 4.4.1 场景1：用 -j 开启并行

```makefile
make -j4
```

同时运行 4 个任务。

```
make -j
```

不限制任务数（尽可能多）。

#### 4.4.2 场景2：并行执行的依赖关系

make 只会并行执行没有依赖关系的目标。

```makefile
all: foo bar baz
```

如果 foo、bar、baz 互不依赖，make 可以同时编译它们。 如果 bar 依赖 foo，那么 bar 必须等 foo 完成。

#### 4.4.3 场景3：并行执行的输出混乱问题

```makefile
make -j4
```

多个任务的输出会交错在一起，很难看懂,可以用`--output-sync`来解决:

```makefile
make -j4 --output-sync
```

这样每个任务的输出会整块输出，不会交错。

#### 4.4.4 场景4：.NOTPARALLEL(禁止并行)

```makefile
.NOTPARALLEL:
```

整个 Makefile 串行执行，即使加了 -j。带先决条件的 .NOTPARALLEL：

```makefile
.NOTPARALLEL: foo bar
```

只让 foo 和 bar 的先决条件串行执行，其他目标仍然可以并行。

#### 4.4.5 场景 5：.WAIT(在并行中插入等待点)

```makefile
all: foo .WAIT bar
```

foo 先执行,.WAIT 表示`等foo 完成`,然后 bar 才执行。可以用于在并行构建中精确控制顺序。

### 4.5 Recipe 中的错误

> 一句话：默认出错停止，`-` 开头忽略错误，`-k` 继续执行其他目标。

**默认行为**：recipe 行失败（返回非零），make **立即停止**，返回非零状态。

**`-` 前缀**：

```makefile
clean:
        -rm *.o
```

`-rm` 开头的 `-` 表示**忽略这一行的错误**。

**`-i` 或 `--ignore-errors`**：

```makefile
make -i
```

忽略**所有** recipe 的错误。等价于所有 recipe 行都以 `-` 开头。

**`-k` 或 `--keep-going`**：

```makefile
make -k
```

出错后**继续执行其他不依赖失败目标的目标**。

**例子**：

```makefile
all: foo bar baz

foo:
        false    # 失败

bar:
        echo "bar"

baz:
        echo "baz"
```

- `make`：`foo` 失败后立即停止。
- `make -k`：`foo` 失败后，继续构建 `bar` 和 `baz`。

**用途**：测试程序编译时，尽可能多地发现错误。

**错误被忽略时的消息**：

```makefile
make: [target] Error 1 (ignored)
```

**`.DELETE_ON_ERROR`**：

```makefile
.DELETE_ON_ERROR:
```

recipe 失败时**删除目标文件**。

**为什么需要**：如果 recipe 已经开始修改目标文件然后失败，目标文件是损坏的，但时间戳是新的。下次 make 会认为它是最新的，不重新生成。删除它可以避免这个问题。

**默认行为**：make 不删除（为了历史兼容）。需要用 `.DELETE_ON_ERROR` 显式请求。

### 4.6 中断或杀死 make

> 一句话：make 被杀死时删除正在更新的目标文件，`.PRECIOUS` 防止删除。

**行为**：make 收到致命信号（如 Ctrl-C）时，如果目标文件的最后修改时间已经改变（说明 recipe 已经开始修改它），make 会**删除**该文件。

**为什么**：假设你 Ctrl-C 杀死编译器，它已经写了一部分 `foo.o`。`foo.o` 的时间戳比 `foo.c` 新。如果不删除，下次 make 会认为 `foo.o` 是最新的，链接时出错。

**`.PRECIOUS` 防止删除**：

```makefile
.PRECIOUS: foo.o
```

或：

```makefile
.PRECIOUS: %.o
```

**使用 `.PRECIOUS` 的理由**：

- 目标是原子更新的。
- 目标只记录修改时间（内容不重要）。
- 目标必须始终存在。

**无法清理的情况**：

- make 被不可捕获的信号杀死。
- 被调用的程序崩溃，留下损坏的目标文件。
- make 本身有 Bug 崩溃。

**防御性 recipe**：

创建临时文件，成功后重命名到目标文件名。许多编译器已经这样做。

### 4.7 递归使用 make

> 一句话：用 `$(MAKE)` 在 makefile 中调用 make，用 `-C` 切换目录。

**含义**：在 makefile 中把 make 作为命令使用。

**用途**：为组成大系统的不同子系统使用单独的 makefile。

**例子**：

假设你有一个子目录 `subdir` 有自己的 makefile，你希望包含它的目录的 makefile 运行子目录的 make：

```makefile
subsystem:
        cd subdir && $(MAKE)
```

或等价地：

```makefile
subsystem:
        $(MAKE) -C subdir
```

**关键点**：

- 用 `$(MAKE)` 而不是 `make`。
- 可以声明为 `.PHONY`。

**`CURDIR` 变量**：

- make 启动时（处理完 `-C` 选项后）设置为当前工作目录的绝对路径。
- 之后不再改变。
- 包含其他目录的文件时 `CURDIR` 不变。
- 优先级和 makefile 中设置的变量一样。

#### 4.7.1 MAKE 变量如何工作

> 一句话：`MAKE` 变量保存 make 的路径，在 recipe 中使用会影响 `-t`、`-n`、`-q` 的行为。

**`MAKE` 的值**：调用 make 时的文件名。如果是 `/bin/make`，recipe 执行 `cd subdir && /bin/make`。

**特殊特性**：在 recipe 中使用 `MAKE` 变量会改变 `-t`、`-n`、`-q` 的行为。

**`-t` 选项的例子**：

```makefile
subsystem:
        cd subdir && $(MAKE)
```

`make -t` 通常会**创建**一个名为 `subsystem` 的文件，不做其他事。但我们真正想要的是运行 `cd subdir && make -t`。

**特殊特性**：每当 recipe 行包含 `MAKE` 变量时，`-t`、`-n`、`-q` 不适用于该行。包含 `MAKE` 的 recipe 行**正常执行**，即使有这些标志。

**注意**：这个特殊特性只在 `MAKE` 变量**直接**出现在 recipe 中时启用。如果 `MAKE` 通过其他变量间接引用，不生效，必须用 `+` 前缀。

#### 4.7.2 向子 make 传递变量

> 一句话：用 `export` 导出变量，`unexport` 阻止导出，`MAKELEVEL` 记录递归深度。

**默认情况**：

- make 只导出：从环境继承的变量 + 命令行定义且名字只含字母、数字、下划线的变量。
- `SHELL` 的值不导出。
- `MAKEFLAGS` 总是导出。
- `MAKEFILES` 如果设置了就导出。
- 命令行定义的变量自动通过 `MAKEFLAGS` 传递。

**显式导出**：

```makefile
export variable ...
```

**阻止导出**：

```makefile
unexport variable ...
```

**同时定义并导出**：

```makefile
export variable = value
```

等价于：

```makefile
variable = value
export variable
```

**追加并导出**：

```makefile
export variable += value
```

**导出所有变量**：

```makefile
export
```

告诉 make 所有没有显式 `unexport` 的变量都导出。

**等价的老式写法**：

```makefile
.EXPORT_ALL_VARIABLES:
```

**注意**：默认导出时，只有名字只含字母、数字、下划线的变量被导出。要导出其他变量必须显式 `export`。

**副作用**：导出变量时需要展开它的值。如果展开有副作用（如 `info`、`eval`），每次命令调用都会触发。

**`MAKELEVEL` 变量**：

- 值：递归深度（十进制字符串）。
- 顶层 make：`0`。
- 子 make：`1`。
- 子子 make：`2`。
- 用途：在条件指令中测试，写行为不同的 makefile。

**例子**：

```makefile
ifeq ($(MAKELEVEL),0)
# 顶层 make 做的事
else
# 子 make 做的事
endif
```

**`MAKEFILES` 变量**：让所有子 make 命令使用额外的 makefile。

#### 4.7.3 向子 make 传递选项

> 一句话：`MAKEFLAGS` 自动传递选项，`MAKEOVERRIDES` 保存命令行变量定义。

**`MAKEFLAGS`**：

- make 自动设置，包含收到的标志字母。
- 例如：`make -ks` → `MAKEFLAGS` 的值为 `ks`。
- 每个子 make 在环境中得到 `MAKEFLAGS`，像命令行参数一样处理。
- 环境中的 `MAKEFLAGS` 优先于 makefile 中的。

**格式**：

- 无参数的选项：单个字母的组。
- 然后一个空格。
- 然后是带参数或有长名字的选项。
- 如果命令行没有单字母选项，值以空格开头。

**命令行变量定义**：

- 包含 `=` 的词被 make 当作变量定义，像在命令行一样处理。

**不放入 `MAKEFLAGS` 的选项**：`-C`、`-f`、`-o`、`-w`。

**`-j` 特殊处理**：

- 如果设置为数字 `N` 且操作系统支持，父 make 和所有子 make 通信，确保总共只有 `N` 个任务。
- 标记为递归的任务不占用总任务数。
- 如果操作系统不支持，`-j` 不加入 `MAKEFLAGS`，子 make 串行运行。

**`MAKEOVERRIDES`**：

- 命令行变量定义实际出现在 `MAKEOVERRIDES` 中。
- `MAKEFLAGS` 包含对 `MAKEOVERRIDES` 的引用。
- 重置 `MAKEOVERRIDES =` 可以不传递命令行变量定义。

**`MFLAGS`**：

- 历史兼容变量。
- 值和 `MAKEFLAGS` 相同，但不含命令行变量定义，且总以 `-` 开头。

**`GNUMAKEFLAGS`**：

- 只被 GNU make 解析，其他 make 忽略。
- 用于设置 GNU make 特有的标志。

#### 4.7.4 `--print-directory` 选项

> 一句话：`-w` 打印进入和离开目录的信息。

**例子**：在 `/u/gnu/make` 运行 `make -w`：

```makefile
make: Entering directory '/u/gnu/make'
...
make: Leaving directory '/u/gnu/make'
```

**自动开启**：

- 使用 `-C` 选项时。
- 在子 make 中。

**自动关闭**：

- 使用 `-s`（静默）。
- 使用 `--no-print-directory`。

### 4.8 定义 Canned Recipe

> 一句话：用 `define` 定义 canned recipe，在 recipe 中引用。

**Canned recipe**：一组可重用的命令序列。

**定义**：

```makefile
define run-yacc =
yacc $(firstword $^)
mv y.tab.c $@
endef
```

**使用**：

```makefile
foo.c : foo.y
        $(run-yacc)
```

**分析**：

- `run-yacc` 是变量名。
- `endef` 标记结束。
- 命令中使用 `$^`（所有依赖）和 `$@`（目标）。
- `foo.y` 被替换为 `$^` 的值，`foo.c` 被替换为 `$@`。

**关键点**：

- `define` 不展开 recipe 中的变量引用和函数调用。
- `define` 定义的变量是**递归展开**变量。
- recipe 中的变量在**使用时**展开。

**每行独立 shell**：

- canned recipe 的每行被当作独立 recipe 行，每行一个子 shell。
- 可以用特殊前缀（`@`、`-`、`+`）。

**例子**：

```makefile
define frobnicate =
@echo "frobnicating target $@"
frob-step-1 $< -o $@.step-1
frob-step-2 $@.step-1 -o $@
endef

frob.out: frob.in
        @$(frobnicate)
```

- `@` 在引用行上，作用于 canned recipe 的**每一行**，所有行都不回显。
- `@` 在 canned recipe 内部的行上，只作用于**那一行**。

### 4.9 使用空 Recipe

> 一句话：空 recipe 用 `;` 或空行定义，防止使用隐式规则。

**定义**：

```makefile
target: ;
```

或

```makefile
target:
        <Tab>makefile
```

**用途**：

1. **防止目标获得隐式 recipe**：如果你不想 make 使用隐式规则，给目标一个空 recipe。
2. **避免错误**：对于会作为其他 recipe 副作用创建的目标，空 recipe 确保 make 不会报错说不知道如何构建。

## 第五章 如何使用变量

> 一句话：变量是文本字符串的容器，可以代表文件名列表、编译选项、命令等，在不同上下文中替换使用。

### 5.1 变量引用的基础

> 一句话：用 `$(VAR)` 或 `${VAR}` 引用变量，`$$` 表示字面 `$`。

**基本语法**：

```makefile
objects = program.o foo.o utils.o
program : $(objects)
        cc -o program $(objects)
```

- `$(objects)` 和 `${objects}` 都有效，效果一样。
- 变量引用可以出现在任何地方：目标、依赖、recipe、大多数指令、新变量值。

**严格文本替换**：

```makefile
foo = c
prog.o : prog.$(foo)
        $(foo)$(foo) -$(foo) prog.$(foo)
```

等价于：

```makefile
prog.o : prog.c
        cc -c prog.c
```

`$(foo)` 严格替换为 `c`。

**`$` 的特殊意义**：这是为什么你必须在文件名或 recipe 中写 `$$` 来表示一个 `$`。

**变量名规则**：

- 可以是任何不包含 `:`、`=`、空白的字符序列。
- 区分大小写：`foo`、`FOO`、`Foo` 是不同变量。
- 传统用大写，内部目的推荐用小写。

**自动变量**：一些变量的名字是单个标点字符或几个字符，它们是自动变量，有特殊用途。

### 5.2 变量的两种类型

> 一句话：递归展开变量（`=`）每次使用时展开，简单展开变量（`:=`）定义时展开一次。

#### 5.2.1 递归展开变量

**定义**：用 `=` 或 `define` 不带赋值操作符定义。

**行为**：变量的值在使用时**递归展开**。

```makefile
foo = $(bar)
bar = $(ugh)
ugh = Huh?

all: ; @echo $(foo)
```

输出：`Huh?`。

**展开过程**：

1. `$(foo)` → `$(bar)`
2. `$(bar)` → `$(ugh)`
3. `$(ugh)` → `Huh?`

**优点**：可以引用后面才定义的变量。

**缺点**：

- 可能死循环。make 会检测并报错：

```makefile
CFLAGS = $(CFLAGS) -O
```

报错：`Recursive variable 'CFLAGS' references itself (eventually). Stop.`

- 函数每次展开时执行。如果引用 `wildcard` 或 `shell` 函数，可能产生不可预测的结果，且 make 变慢。

#### 5.2.2 简单展开变量

**定义**：用 `:=` 或 `::=`。

**行为**：变量的值在**定义时**扫描一次，展开所有变量引用和函数。之后使用时**逐字复制**。

```makefile
x := foo
y := $(x) bar
x := later
```

等价于：

```makefile
y := foo bar
x := later
```

**优点**：

- 更可预测，像大多数编程语言中的变量。
- 可以重定义一个变量使用它自己的值。
- 函数只调用一次，更高效。

**复杂例子**：

```makefile
ifeq (0,${MAKELEVEL})
whoami := $(shell whoami)
host-type := $(shell arch)
MAKE := ${MAKE} host-type=${host-type} whoami=${whoami}
endif
```

- `$(shell whoami)` 只执行一次（定义时）。
- 之后 `whoami` 是固定字符串。

**引入受控前导空白**：

```makefile
nullstring :=
space := $(nullstring) # end of the line
```

- `space` 的值精确地是一个空格。
- 因为前导空白在替换前被丢弃，用 `$(nullstring)` 保护空格。

**警告**：

```makefile
dir := /foo/bar    # directory to put the frobs in
```

- `dir` 的值是 `/foo/bar`（带 4 个尾随空格），可能不是你的意图。
- 变量值的尾随空白**不被**去除。

#### 5.2.3 立即展开变量赋值

**定义**：用 `:::=`。

**行为**：右侧立即展开，然后**转义**（所有 `$` 变成 `$$`），结果是**递归变量**。

```makefile
var = first
OUT :::= $(var)
var = second
```

- `OUT` 的值是 `first`。

```makefile
var = one$$two
OUT :::= $(var)
var = three$$four
```

- `OUT` 的值是 `one$$two`。
- 展开时得到 `one$two three$four`。

**与 `:=` 的区别**：

- `:::=` 结果是递归变量。
- 之后 `+=` 时右侧不立即展开。
- 效率略低（使用时需要重新展开，但只是解除转义）。

**BSD make 的 `:=`**：

- BSD make 的 `:=` 等价于 GNU make 的 `:::=`。
- POSIX Issue 8 加入了 `:::=` 以提供可移植性。

#### 5.2.4 条件变量赋值

**定义**：用 `?=`。

**行为**：如果变量**未定义**，则赋值；否则**不赋值**。

```makefile
FOO ?= bar
```

等价于：

```makefile
ifeq ($(origin FOO), undefined)
FOO = bar
endif
```

**关键点**：变量设为**空值**也算已定义，`?=` 不会覆盖。

### 5.3 变量引用的高级特性

#### 5.3.1 替换引用

**形式**：`$(var:a=b)` 或 `${var:a=b}`。

**行为**：取 `var` 的值，把每个词末尾的 `a` 替换成 `b`。

**"末尾"的含义**：`a` 必须出现在词尾（后面是空白或值结尾）。

**例子**：

```makefile
foo := a.o b.o l.a c.o
bar := $(foo:.o=.c)
```

`bar` 的值是 `a.c b.c l.a c.c`。

**等价于**：

```makefile
bar := $(patsubst %.o,%.c,$(foo))
```

**带 `%` 的形式**：

```makefile
foo := a.o b.o l.a c.o
bar := $(foo:%.o=%.c)
```

`bar` 的值是 `a.c b.c l.a c.c`。

#### 5.3.2 计算变量名

**含义**：变量名中可以包含变量引用。

**两层嵌套**：

```makefile
x = y
y = z
a := $($(x))
```

`a` 的值是 `z`。

**展开过程**：

1. `$(x)` → `y`
2. `$($(x))` → `$(y)` → `z`

**三层嵌套**：

```makefile
x = y
y = z
z = u
a := $($($(x)))
```

`a` 的值是 `u`。

**与递归展开变量的交互**：

```makefile
x = $(y)
y = z
z = Hello
a := $($(x))
```

`a` 的值是 `Hello`。

**嵌套引用可以包含函数调用**：

```makefile
x = variable1
variable2 := Hello
y = $(subst 1,2,$(x))
z = y
a := $($($(z)))
```

`a` 的值是 `Hello`。

**计算变量名不必只包含一个变量引用**：

```makefile
a_dirs := dira dirb
1_dirs := dir1 dir2
a_files := filea fileb
1_files := file1 file2

ifeq "$(use_a)" "yes"
a1 := a
else
a1 := 1
endif

ifeq "$(use_dirs)" "yes"
df := dirs
else
df := files
endif

dirs := $($(a1)_$(df))
```

`dirs` 的值取决于 `use_a` 和 `use_dirs`，可能是 `a_dirs`、`1_dirs`、`a_files` 或 `1_files`。

**在替换引用中使用**：

```makefile
a_objects := a.o b.o c.o
1_objects := 1.o 2.o 3.o

sources := $($(a1)_objects:.o=.c)
```

`sources` 是 `a.c b.c c.c` 或 `1.c 2.c 3.c`。

**限制**：不能指定函数名的一部分。

```makefile
ifdef do_sort
func := sort
else
func := strip
endif

bar := a d b g q c
foo := $($(func) $(bar))
```

`foo` 的值是变量 `sort a d b g q c` 或 `strip a d b g q c` 的值，而不是把 `bar` 排序或 strip。

**在赋值左侧使用**：

```makefile
dir = foo
$(dir)_sources := $(wildcard $(dir)/*.c)

define $(dir)_print =
lpr $($(dir)_sources)
endef
```

这定义了 `foo_sources` 和 `foo_print`。

### 5.4 变量如何获得值

变量可以有多种方式获得值：

1. **命令行覆盖**：`make CFLAGS='-g -O'`。
2. **makefile 中赋值**：`CFLAGS = -g`。
3. **`let` 函数**：短生命周期的局部变量。
4. **`foreach` 函数**：循环中的变量。
5. **环境变量**：make 启动时环境中的变量。
6. **自动变量**：每个规则执行时设置。
7. **内置变量**：make 有初始值的变量。

### 5.5 设置变量

> 一句话：`=` 递归，`:=` 或 `::=` 简单，`:::=` 立即，`?=` 条件，`!=` shell。

```makefile
objects = main.o foo.o bar.o utils.o
```

- 变量名和 `=` 周围的空白被忽略。
- `objects` 的值是 `main.o foo.o bar.o utils.o`。

**`:=` 和 `::=`**：

```makefile
foo := $(bar)
```

- 定义时展开 `bar`。
- 两个操作符在 GNU make 中等价。
- 只有 `:=` 是 POSIX 标准，`::=` 是 POSIX Issue 8 加入。

**`:::=`**：

```makefile
foo :::= $(bar)
```

- 右侧立即展开，然后转义。
- 结果是递归变量。

**`?=`**：

```makefile
FOO ?= bar
```

- 如果 `FOO` 未定义，赋值 `bar`。

**`!=` shell 赋值**：

```makefile
hash != printf '\043'
file_list != find . -name '*.c'
```

- 右侧立即求值，传给 shell。
- 执行结果存储在变量中，变量是**递归展开变量**。
- 如果结果以换行结尾，一个换行被移除，其他换行变成空格。
- 结果中的 `$` 需要写成 `$$` 以避免被当作 make 变量引用。

**等价写法**：

```makefile
hash := $(shell printf '\043')
var := $(shell find . -name "*.c")
```

`.SHELLSTATUS` 变量存储 shell 脚本的退出状态。

### 5.6 向变量追加文本

> 一句话：`+=` 追加文本，行为取决于变量之前的类型。

```makefile
objects = main.o foo.o bar.o utils.o
objects += another.o
```

`objects` 的值是 `main.o foo.o bar.o utils.o another.o`。

**行为细节**：

- 如果变量**未定义**：`+=` 像 `=` 一样定义递归变量。
- 如果变量是**简单展开**（`:=` 或 `::=`）：`+=` 立即展开右侧，追加。
- 如果变量是**递归展开**（`=`）：`+=` 追加**未展开**的文本。

**例子**：

```makefile
CFLAGS = $(includes) -O
...
CFLAGS += -pg
```

- `CFLAGS` 是递归变量。
- `+=` 追加 `-pg`（未展开）。
- 结果是 `$(includes) -O -pg`。
- 如果 `includes` 后来定义，`$(CFLAGS)` 仍然使用它的值。

**对比**：

```makefile
CFLAGS := $(CFLAGS) -pg
```

- `CFLAGS` 变成简单变量。
- `$(CFLAGS)` 立即展开。
- 如果 `includes` 未定义，结果是 `-O -pg`。
- 后来定义 `includes` 无影响。

**总结**：用 `+=` 保留对 `includes` 的引用。

### 5.7 override 指令

> 一句话：`override` 覆盖命令行定义的变量。

**问题**：命令行定义的变量（如 `make CFLAGS='-g'`）会覆盖 makefile 中的赋值。

**解决**：用 `override` 指令。

```makefile
override variable = value
```

或：

```makefile
override variable := value
```

**追加**：

```makefile
override variable += more text
```

**优先级**：

- 用 `override` 标记的变量赋值优先级最高，除了另一个 `override`。
- 后续没有 `override` 的赋值或追加会被忽略。

**典型用途**：

```makefile
override CFLAGS += -g
```

用户可以用命令行指定其他 CFLAGS，但 `-g` 总是被追加。

**和 `define` 一起用**：

```makefile
override define foo =
bar
endef
```

### 5.8 定义多行变量

> 一句话：用 `define` 定义多行变量，`endef` 结束。

**语法**：

```makefile
define variable
value
...
endef
```

或带赋值操作符：

```makefile
define variable =
value
...
endef
```

**分析**：

- `define` 后跟变量名和可选的赋值操作符。
- 值在后续行中。
- `endef` 标记结束。
- 变量名可以包含函数和变量引用。
- 最后的换行不包括在值中。要包含尾随换行，用两个空行。

**例子：定义包含换行的变量**：

```makefile
define newline


endef
```

`newline` 的值是一个换行符。

**省略赋值操作符**：

- 默认 `=`，创建递归展开变量。

**用 `+=`**：

- 追加到之前的值，用空格分隔。

**嵌套 `define`**：

```makefile
define two-lines
echo foo
echo $(bar)
endef
```

- make 跟踪嵌套，未正确关闭会报错。
- 以 recipe 前缀开头的行是 recipe 的一部分，其中的 `define`/`endef` 不被当作 make 指令。

**在 recipe 中的等价**：

```makefile
two-lines = echo foo; echo $(bar)
```

- 但用两行意味着 make 调用 shell 两次，运行独立的子 shell。

**用 `override` + `define`**：

```makefile
override define two-lines =
foo
$(bar)
endef
```

### 5.9 取消变量定义

> 一句话：用 `undefine` 让变量看起来从未定义过。

```makefile
foo := foo
bar = bar

undefine foo
undefine bar

$(info $(origin foo))
$(info $(flavor bar))
```

输出：`undefined` 和 `undefined`。

**取消命令行变量**：

```makefile
override undefine CFLAGS
```

### 5.10 来自环境的变量

> 一句话：环境变量成为 make 变量，makefile 中的赋值覆盖环境。

**默认行为**：

- make 启动时，环境中的每个变量变成 make 变量，名字和值相同。
- makefile 中的显式赋值或命令行参数覆盖环境。

**`-e` 标志**：让环境变量**覆盖** makefile 中的赋值。不推荐。

**典型用途**：

```makefile
CFLAGS='-g -O' make
```

这样 `CFLAGS` 影响所有 C 编译。但有些 makefile 显式设置 CFLAGS，不受影响。

**recipe 中的环境**：

- make 执行 recipe 时，一些变量被放入命令的环境。
- 默认只有从 make 环境继承的变量或命令行设置的变量。
- 用 `export` 指令传递其他变量。

**不推荐依赖环境变量**：因为不同用户会得到不同结果。

**`SHELL` 特殊处理**：环境中的 `SHELL` 是用户交互 shell 的选择，不用于 makefile 的 shell。

### 5.11 目标特定变量值

> 一句话：目标特定变量只对该目标及其依赖有效。

**语法**：

```makefile
target ... : variable-assignment
```

**例子**：

```makefile
prog : CFLAGS = -g
prog : prog.o foo.o bar.o
```

- `prog` 的 recipe 中 `CFLAGS` 是 `-g`。
- `prog.o`、`foo.o`、`bar.o` 的 recipe 中也是 `-g`。
- 其他目标不受影响。

**多个目标**：

```makefile
prog1 prog2 : CFLAGS = -g
```

为每个目标单独创建目标特定变量。

**支持的赋值形式**：所有有效形式：`=`、`:=`、`::=`、`:::=`、`+=`、`?=`。

**变量在目标上下文中展开**：`variable-assignment` 中的变量在目标上下文中展开。

**与全局变量的区别**：目标特定变量和全局变量是**不同**的变量（可以有不同 flavor）。

**优先级**：

- 和任何 makefile 变量一样。
- 命令行变量（和环境变量，如果 `-e`）优先。
- 用 `override` 可以让目标特定变量优先。

**继承**：

- 目标特定变量对目标的所有依赖有效。
- 也对其依赖的依赖有效，除非被覆盖。

**例子**：

```makefile
prog : CFLAGS = -g
prog : prog.o foo.o bar.o
```

- `prog.o`、`foo.o`、`bar.o` 的 recipe 中 `CFLAGS` 是 `-g`。
- 它们依赖的文件的 recipe 中也是 `-g`。

**警告**：一个依赖在一次 make 调用中只构建一次。如果同一文件是多个目标的依赖，每个目标有不同的目标特定变量值，**第一个**构建的目标的值被使用。

**前缀关键字**：

- `export`、`unexport`、`override`、`private` 可以加在目标特定赋值前。
- 作用只限于这个变量实例。

### 5.12 模式特定变量值

> 一句话：模式特定变量对所有匹配模式的目标有效。

**语法**：

```makefile
pattern ... : variable-assignment
```

**例子**：

```makefile
%.o : CFLAGS = -O
```

为所有匹配 `%.o` 的目标设置 `CFLAGS = -O`。makefile

**多个模式**：为每个模式单独创建。

**命令行程优先级**：命令行变量设置优先，除非用 `override`。

**更具体的模式优先**：

```makefile
%.o : %.c
        $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@

lib/%.o: CFLAGS := -fPIC -g
%.o: CFLAGS := -g

all: foo.o lib/bar.o
```

- `lib/bar.o` 匹配 `lib/%.o`（词干 `bar`）和 `%.o`（词干 `lib/bar`）。
- 词干更短的（`lib/%.o`）优先。
- 所以 `CFLAGS` 是 `-fPIC -g`。

**相同词干长度**：按 makefile 中定义顺序。

**搜索顺序**：

1. 目标特定变量。
2. 模式特定变量。
3. 父目标的模式特定变量。

### 5.13 抑制继承

> 一句话：`private` 修饰符让变量不被依赖继承。

**语法**：

```makefile
private variable-assignment
```

**例子**：

```makefile
debug : private CFLAGS = -g
debug : prog
```

- `debug` 的 recipe 中 `CFLAGS` 是 `-g`。
- `prog` 的 recipe 中 `CFLAGS` **不是** `-g`（不被继承）。

**用途**：目标特定和模式特定变量，不希望对依赖产生影响。

### 5.14 其他特殊变量

#### `.DEFAULT_GOAL`

- 当前默认目标。
- 可以查询、重置、设置。

**例子**：

```makefile
ifeq ($(.DEFAULT_GOAL),)
$(warning no default goal is set)
endif

.PHONY: foo
foo: ; @echo $@

$(warning default goal is $(.DEFAULT_GOAL))

# 重置默认目标
.DEFAULT_GOAL :=

.PHONY: bar
bar: ; @echo $@

$(warning default goal is $(.DEFAULT_GOAL))

# 设置自己的默认目标
.DEFAULT_GOAL := foo
```

输出：

```makefile
no default goal is set
default goal is foo
default goal is bar
foo
```

- 给 `.DEFAULT_GOAL` 赋多个目标名是无效的，会报错。

#### `MAKE_RESTARTS`

- make 重启的次数。
- 只在 make 重启时设置。
- 和递归不同（递归由 `MAKELEVEL` 计数）。
- 不应该设置、修改或导出。

#### `MAKE_TERMOUT` 和 `MAKE_TERMERR`

- make 启动时检查 stdout 和 stderr 是否在终端上显示输出。
- 如果是，设置 `MAKE_TERMOUT` 和 `MAKE_TERMERR` 为终端设备名（或 `true`）。
- 被标记为导出。
- 用于判断 make 是否写入终端。
- 如果调用子 make 重定向了 stdout/stderr，需要重置或取消导出。

#### `.RECIPEPREFIX`

- 值的第一个字符是 recipe 行前缀字符。
- 默认为空 → 标准 Tab。
- 可以多次改变。

**例子**：

```makefile
.RECIPEPREFIX = >
all:
> echo Hello, world
```

#### `.VARIABLES`

- 展开为到目前为止定义的所有全局变量名列表。
- 包括空值变量和内置变量。
- 不包括目标特定上下文中的变量。
- 给它赋值无效。

#### `.FEATURES`

- 展开为这个版本的 make 支持的特殊特性列表。
- 可能的值包括：`archives`、`check-symlink`、`else-if`、`extra-prereqs`、`grouped-target`、`guile`、`jobserver`、`jobserver-fifo`、`load`、`notintermediate`、`oneshell`、`order-only`、`output-sync`、`second-expansion`、`shell-export`、`shortest-stem`、`target-specific`、`undefine`。

#### `.INCLUDE_DIRS`

- 展开为 make 搜索包含 makefile 的目录列表。
- 修改它不改变搜索目录。

#### `.EXTRA_PREREQS`

- 每个词是一个新依赖，添加到设置了它的目标。
- 不出现在任何自动变量中。
- 允许定义不影响 recipe 的依赖。

**例子**：

```makefile
myprog: myprog.o file1.o file2.o
        $(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^ $(LDLIBS)

myprog: .EXTRA_PREREQS = $(CC)
```

- `$(CC)` 是 `myprog` 的额外依赖。
- 不出现在 `$^` 中，不被传给链接命令。
- 更新编译器会触发 `myprog` 重新链接。


## 第六章 Makefile 条件

> 一句话：条件指令根据变量值决定 makefile 的哪部分被 make "看到"。

### 6.1 条件示例

```makefile
libs_for_gcc = -lgnu
normal_libs =

foo: $(objects)
ifeq ($(CC),gcc)
        $(CC) -o foo $(objects) $(libs_for_gcc)
else
        $(CC) -o foo $(objects) $(normal_libs)
endif
```

**分析**：

- `ifeq ($(CC),gcc)`：如果 `CC` 是 `gcc`。
- `else`：否则。
- `endif`：结束。

**`CC=gcc` 时**：

```makefile
foo: $(objects)
        $(CC) -o foo $(objects) $(libs_for_gcc)
```

**`CC` 是其他值时**：

```makefile
foo: $(objects)
        $(CC) -o foo $(objects) $(normal_libs)
```

**等价写法（条件化变量赋值）**：

```makefile
libs_for_gcc = -lgnu
normal_libs =

ifeq ($(CC),gcc)
libs = $(libs_for_gcc)
else
libs = $(normal_libs)
endif

foo: $(objects)
        $(CC) -o foo $(objects) $(libs)
```

### 6.2 条件语法

> 一句话：`ifeq`、`ifneq`、`ifdef`、`ifndef` 是四种条件指令。

**简单条件**：

```makefile
conditional-directive
text-if-true
endif
```

**复杂条件**：

```makefile
conditional-directive
text-if-true
else
text-if-false
endif
```

或：

```makefile
conditional-directive-one
text-if-one-is-true
else conditional-directive-two
text-if-two-is-true
else
text-if-one-and-two-are-false
endif
```

**四种条件指令**：

**`ifeq (arg1, arg2)`**：

- 展开 `arg1` 和 `arg2` 中的所有变量引用，比较。
- 相同 → `text-if-true`。
- 不同 → `text-if-false`。

**形式**：

```makefile
ifeq (arg1, arg2)
ifeq 'arg1' 'arg2'
ifeq "arg1" "arg2"
ifeq "arg1" 'arg2'
ifeq 'arg1' "arg2"
```

**空值测试**：

```makefile
ifeq ($(strip $(foo)),)
text-if-empty
endif
```

即使 `$(foo)` 包含空白，`text-if-empty` 也会生效。

**`ifneq (arg1, arg2)`**：

- 和 `ifeq` 相反：不同 → `text-if-true`。

**`ifdef variable-name`**：

- 取变量**名**作为参数，不是引用。
- 如果变量有非空值 → `text-if-true`。

**例子**：

```makefile
bar = true
foo = bar
ifdef $(foo)
frobozz = yes
endif
```

- `$(foo)` 展开为 `bar`。
- `bar` 有非空值 → `frobozz = yes`。

**注意**：`ifdef` 只测试变量是否有值，不展开变量看值是否非空。

```makefile
bar =
foo = $(bar)
ifdef foo
frobozz = yes
else
frobozz = no
endif
```

- `foo` 已定义，但值是 `$(bar)`（未展开）。
- `ifdef foo` 为真 → `frobozz = yes`。

**`ifndef variable-name`**：

- 如果变量有空值 → `text-if-true`。
- 规则和 `ifdef` 相同。

**语法细节**：

- 行首允许多余空格，但**不允许 Tab**（否则是 recipe 的一部分）。
- 指令名或参数内不能有多余空格/Tab。
- 行尾可以有 `#` 注释。
- `else` 和 `endif` 是无参数的单词。
- 行首尾允许空格/Tab。
- 行尾可以有 `#` 注释。

**条件的性质**：

- 条件影响 make 使用 makefile 的哪些行。
- 条件为真时，make 读取 `text-if-true` 作为 makefile 的一部分；为假时完全忽略。
- makefile 的语法单元（如规则）可以跨条件的开始或结束。
- make 在读取 makefile 时求值条件，所以**不能**在条件测试中使用自动变量。
- 不能在一个 makefile 中开始条件，在另一个中结束。但可以在条件中 `include`，只要不在包含文件中终止条件。

### 6.3 测试标志的条件

**用途**：测试 make 命令行标志，如 `-t`。

**方法**：用 `MAKEFLAGS` 变量和 `findstring` 函数。

**`MAKEFLAGS` 的特性**：

- 所有单字母选项（如 `-t`）在第一个词中。
- 没有单字母选项时，第一个词为空。
- 为了处理这种情况，在 `MAKEFLAGS` 前加一个值，确保有一个词。

**例子**：

```makefile
archive.a: ...
ifneq (,$(findstring t,$(firstword -$(MAKEFLAGS))))
        +touch archive.a
        +ranlib -t archive.a
else
        ranlib archive.a
endif
```

**分析**：

- `-$(MAKEFLAGS)` 确保第一个词存在。
- `$(firstword ...)` 取第一个词。
- `$(findstring t,...)` 查找 `t`。
- `ifneq (,...)` 如果非空 → 有 `-t`。
- `+` 前缀标记 recipe 行为递归，即使有 `-t` 也执行。


## 第七章 文本转换函数

> 一句话：函数在 makefile 中做文本处理，计算结果替换到调用点。

### 7.1 函数调用语法

**形式**：

```makefile
$(function arguments)
${function arguments}
```

- `function` 是函数名。
- `arguments` 是参数，用空格或 Tab 与函数名分隔。
- 多个参数用逗号分隔。
- 空白和逗号不属于参数值。
- 参数中的变量引用和函数调用会先展开。
- 用括号或大括号，可以嵌套但要注意匹配。

**特殊字符处理**：

- make 不支持用反斜杠转义。
- 因为参数在展开前被分割，可以把特殊字符放在变量中。

**需要隐藏的字符**：

- 逗号
- 第一个参数的前导空白
- 不匹配的开括号或大括号

**例子**：

```makefile
comma:= ,
empty:=
space:= $(empty) $(empty)
foo:= a b c
bar:= $(subst $(space),$(comma),$(foo))
```

`bar` 的值是 `a,b,c`。

### 7.2 字符串替换和分析函数

#### `$(subst from,to,text)`

- 把 `text` 中的 `from` 替换为 `to`。

**例子**：

```makefile
$(subst ee,EE,feet on the street)
```

结果：`fEEt on the strEEt`。

#### `$(patsubst pattern,replacement,text)`

- 把 `text` 中匹配 `pattern` 的词替换为 `replacement`。
- `pattern` 和 `replacement` 中可以用 `%`。
- `%` 匹配任意字符序列。

**例子**：

```makefile
$(patsubst %.c,%.o,x.c.c bar.c)
```

结果：`x.c.o bar.o`。

**简写**：

```makefile
$(var:pattern=replacement)
```

等价于：

```makefile
$(patsubst pattern,replacement,$(var))
```

**更简写**：

```makefile
$(var:suffix=replacement)
```

等价于：

```makefile
$(patsubst %suffix,%replacement,$(var))
```

#### `$(strip string)`

- 移除 `string` 的前导和尾随空白，内部连续空白变成一个空格。

**例子**：

```makefile
$(strip a b c )
```

结果：`a b c`。

**用途**：和条件一起用，避免空白被当作非空值。

#### `$(findstring find,in)`

- 在 `in` 中查找 `find`。找到 → 返回 `find`；否则 → 空。

**例子**：

```makefile
$(findstring a,a b c)
$(findstring a,b c)
```

结果：`a` 和空。

#### `$(filter pattern...,text)`

- 返回 `text` 中匹配任何 `pattern` 的词。

**例子**：

```makefile
sources := foo.c bar.c baz.s ugh.h
foo: $(sources)
        cc $(filter %.c %.s,$(sources)) -o foo
```

结果：`foo.c bar.c baz.s`。

#### `$(filter-out pattern...,text)`

- 返回 `text` 中不匹配任何 `pattern` 的词。

**例子**：

```makefile
$(filter-out %.c,$(sources))
```

结果：`baz.s ugh.h`。

#### `$(sort list)`

- 按字典序排序 `list` 中的词，移除重复。

**例子**：

```makefile
$(sort foo bar lose)
```

结果：`bar foo lose`。

#### `$(word n,text)`

- 返回 `text` 的第 `n` 个词（1 开始）。

**例子**：

```makefile
$(word 2, foo bar baz)
```

结果：`bar`。

#### `$(wordlist s,e,text)`

- 返回 `text` 中从第 `s` 到第 `e` 的词。

**例子**：

```makefile
$(wordlist 2, 3, foo bar baz)
```

结果：`bar baz`。

#### `$(words text)`

- 返回 `text` 中的词数。

**例子**：

```makefile
$(words foo bar baz)
```

结果：`3`。

#### `$(firstword names...)`

- 返回第一个词。

**例子**：

```makefile
$(firstword foo bar)
```

结果：`foo`。

#### `$(lastword names...)`

- 返回最后一个词。

**例子**：

```makefile
$(lastword foo bar)
```

结果：`bar`。

#### 实际例子：VPATH 转 CFLAGS

```makefile
override CFLAGS += $(patsubst %,-I%,$(subst :, ,$(VPATH)))
```

- `$(subst :, ,$(VPATH))`：把冒号变成空格。
- `$(patsubst %,-I%,...)`：每个目录前加 `-I`。
- 结果：`-Isrc -I../headers`。

### 7.3 文件名函数

- **`$(dir names...)`**：目录部分（包括尾斜杠）。

```makefile
$(dir src/foo.c hacks)
```

结果：`src/ ./`。

- **`$(notdir names...)`**：非目录部分。

```makefile
$(notdir src/foo.c hacks)
```

结果：`foo.c hacks`。

- **`$(suffix names...)`**：后缀。

```makefile
$(suffix src/foo.c src-1.0/bar.c hacks)
```

结果：`.c .c`。

- **`$(basename names...)`**：去后缀。

```makefile
$(basename src/foo.c src-1.0/bar.c hacks)
```

结果：`src/foo src-1.0/bar hacks`。

- **`$(addsuffix suffix,names...)`**：加后缀。

```makefile
$(addsuffix .c,foo bar)
```

结果：`foo.c bar.c`

- **`$(addprefix prefix,names...)`**：加前缀。

```makefile
$(addprefix src/,foo bar)
```

结果：`src/foo src/bar`。

- **`$(join list1,list2)`**：把两个列表按位置拼接。

```makefile
$(join a b,.c .o)
```

结果：`a.c b.o`。

1. 如果两个列表长度不同，较长列表的多余部分原样保留。
2. 词之间的空白被替换成单个空格。
3. 可以合并 `dir` 和 `notdir` 的结果，恢复原始列表。

- **`$(wildcard pattern)`**：

```makefile
$(wildcard *.c)
```

1. `pattern` 是文件名模式（通常含通配符）。
2. 返回匹配的现有文件列表。

- **`$(realpath names...)`**：

1. 对每个文件名返回**规范绝对名**。
2. 规范名不含 `.`、`..`、重复斜杠、符号链接。
3. 失败返回空字符串。

- **`$(abspath names...)`**：

1. 对每个文件名返回**绝对名**。
2. 不含 `.`、`..`、重复斜杠。
3. 与 `realpath` 不同：不解析符号链接，不要求文件存在。

### 7.4 条件函数

**关键点**：不是所有参数都会立即展开，只有需要的才展开。

**`$(if condition,then-part[,else-part])`**：

- `condition` 去掉前后空白后展开。
- 非空 → 展开 `then-part`。
- 空 → 展开 `else-part`（如果有）。
- 只展开其中一个分支。

**例子**：

```makefile
$(if $(CC),$(CC),gcc)
```

如果 `CC` 非空用它，否则用 `gcc`。

**`$(or condition1[,condition2[,condition3...]])`**：

- 逐个展开参数。
- 第一个非空的作为结果。
- 全空则返回空。

**例子**：

```makefile
$(or $(A),$(B),default)
```

**`$(and condition1[,condition2[,condition3...]])`**：

- 逐个展开参数。
- 任何一个空 → 返回空。
- 全部非空 → 返回最后一个的值。

**例子**：

```makefile
$(and $(A),$(B))
```

**`$(intcmp lhs,rhs[,lt-part[,eq-part[,gt-part]]])`**：

- 数值比较 `lhs` 和 `rhs`。
- 没有更多参数时：相等返回数值，不等返回空。
- `lhs < rhs` → `lt-part`。
- `lhs == rhs` → `eq-part`。
- `lhs > rhs` → `gt-part`。
- `gt-part` 缺失时默认 `eq-part`。
- `eq-part` 缺失时默认空。

**例子**：

```makefile
$(intcmp 9,7,hello,world,)
```

结果：空（因为 `gt-part` 是空）。

```makefile
$(intcmp 9,7,hello,world)
```

结果：`world`（没有逗号，所以 `world` 是 `gt-part`）。

### 7.5 let 函数

**语法**：

```makefile
$(let var [var ...],[list],text)
```

- `var` 和 `list` 先展开。
- `text` 不在同一时间展开。
- `list` 的每个词绑定到 `var` 中的变量。
- 最后一个变量绑定剩余的列表。
- 变量在 `let` 内是简单展开变量。
- 绑定后展开 `text`。

**例子：反转列表**：

```makefile
reverse = $(let first rest,$1,\
            $(if $(rest),$(call reverse,$(rest)) $(first)))

all: ; @echo $(call reverse,d c b a)
```

输出：`a b c d`。

- `first` 绑定 `d`，`rest` 绑定 `c b a`。
- 递归调用反转 `c b a`。
- 最后拼接。

**作用域**：`let` 结束后，`first` 和 `rest` 恢复原值（或未定义）。

### 7.6 foreach 函数

**语法**：

```makefile
$(foreach var,list,text)
```

- `var` 和 `list` 先展开。
- `text` 不立即展开。
- 对 `list` 的每个词，设置 `var` 为该词，展开 `text`。
- 结果用空格连接。

**例子：获取所有目录的文件**：

```makefile
dirs := a b c d
files := $(foreach dir,$(dirs),$(wildcard $(dir)/*))
```

等价于：

```makefile
files := $(wildcard a/* b/* c/* d/*)
```

**改进可读性**：

```makefile
find_files = $(wildcard $(dir)/*)
dirs := a b c d
files := $(foreach dir,$(dirs),$(find_files))
```

- 用 `=` 定义 `find_files`（递归展开），否则 `wildcard` 只调用一次。

**关键点**：

- `var` 是简单展开变量。
- `foreach` 结束后 `var` 恢复原值或未定义。
- 变量名可能意外匹配，注意奇怪的名字。

### 7.7 file 函数

**语法**：

```makefile
$(file op filename[,text])
```

**操作符**：

- `>`：覆盖写入。
- `>>`：追加写入。
- `<`：读取。

**写文件**：

```makefile
program: $(OBJECTS)
        $(file >$@.in,$^)
        $(CMD) $@.in
        @rm $@.in
```

**读文件**：

- 返回文件内容，去掉最后一个换行。
- 读取不存在的文件返回空。

**每行一个参数写文件**：

```makefile
program: $(OBJECTS)
        $(file >$@.in) $(foreach O,$^,$(file >>$@.in,$O))
        $(CMD) $@.in
```

### 7.8 call 函数

**语法**：

```makefile
$(call variable,param,param,...)
```

- 每个 `param` 赋给临时变量 `$(1)`、`$(2)` 等。
- `$(0)` 是变量名。
- 在临时赋值上下文中展开 `variable`。

**例子 1：反转参数**：

```makefile
reverse = $(2) $(1)
foo = $(call reverse,a,b)
```

`foo` 是 `b a`。

**例子 2：在 PATH 中搜索程序**：

```makefile
pathsearch = $(firstword $(wildcard $(addsuffix /$(1),$(subst :, ,$(PATH)))))
LS := $(call pathsearch,ls)
```

`LS` 是 `/bin/ls` 或类似。

**嵌套**：

```makefile
map = $(foreach a,$(2),$(call $(1),$(a)))
o = $(call map,origin,o map MAKE)
```

`o` 是 `file file default`。

**注意**：`call` 展开 `param`。有特殊展开规则的函数（`foreach`、`if`）可能行为不同。多余空白会被保留。

### 7.9 value 函数

**语法**：

```makefile
$(value variable)
```

- 返回 `variable` 的值，**不展开**。
- `variable` 是变量名，不是引用。

**例子**：

```makefile
FOO = $PATH

all:
        @echo $(FOO)
        @echo $(value FOO)
```

- 第一个输出 `ATH`（`$P` 被当作 make 变量）。
- 第二个输出 `$PATH`（环境变量的值）。

**用途**：和 `eval` 一起用。

### 7.10 eval 函数

**语法**：

```makefile
$(eval text)
```

- `text` 展开后作为 makefile 语法解析。
- 结果总是空字符串。
- **双重展开**：`eval` 展开一次，解析时再展开一次。
- 可能需要额外转义 `$`。

**例子**：

```makefile
PROGRAM_template = \
  $(1)_OBJS := $$($(1)_SRCS:.c=.o) \
  $(1): $$($(1)_OBJS) \
        $$(CC) -o $$@ $$($(1)_OBJS)

$(foreach prog,foo bar,$(eval $(call PROGRAM_template,$(prog))))
```

**用途**：动态生成规则。

### 7.11 origin 函数

**语法**：

```makefile
$(origin variable)
```

**返回值**：

- `undefined`：未定义。
- `default`：内置默认。
- `environment`：环境变量。
- `environment override`：环境变量且 `-e` 覆盖 makefile。
- `file`：makefile 中定义。
- `command line`：命令行定义。
- `override`：`override` 指令定义。
- `automatic`：自动变量。

**例子**：

```makefile
ifdef bletcher
ifeq ($(origin bletcher), environment)
bletcher = barf, gag, etc.
endif
endif
```

### 7.12 flavor 函数

**语法**：

```makefile
$(flavor variable)
```

**返回值**：

- `undefined`：未定义。
- `recursive`：递归展开变量。
- `simple`：简单展开变量。

### 7.13 控制 make 的函数

**`$(error text...)`**：

- 生成致命错误，make 退出。

```makefile
ifdef ERROR1
$(error error is $(ERROR1))
endif
```

**`$(warning text...)`**：

- 生成警告，make 继续。

**`$(info text...)`**：

- 打印信息到标准输出，继续。

### 7.14 shell 函数

**语法**：

```makefile
$(shell command)
```

- 执行 shell 命令，返回输出。
- 换行变成空格，尾随换行被移除。

**例子**：

```makefile
contents := $(shell cat foo)
files := $(shell echo *.c)
```

**性能注意**：在递归展开变量中会多次执行，在简单展开变量中只执行一次。

**`!=` 赋值**：

```makefile
contents != cat foo
```

**导出变量的循环**：

```makefile
export HI = $(shell echo hi)
```

递归展开会形成循环，make 用环境值避免。

### 7.15 guile 函数

**语法**：

```makefile
$(guile expression)
```

- 如果支持 Guile，执行表达式。
- 结果转为字符串。
- 用 `.FEATURES` 检查 `guile`。


## 第八章 如何运行 make

> 一句话：命令行参数控制 make 的行为：指定 makefile、目标、选项。

### 8.1 指定 Makefile 的参数

**`-f file` 或 `--file=file`**：

```makefile
make -f altmake
```

- 多个 `-f` 会联合使用。
- 不用 `-f` 时默认查找 `GNUmakefile`、`makefile`、`Makefile`。

### 8.2 指定目标的参数

**目标作为参数**：

```makefile
make clean
make foo bar
```

- 默认目标是 makefile 中第一个目标。
- 多个目标按顺序处理。
- 目标可以不在 makefile 中（用隐式规则）。

**`MAKECMDGOALS` 变量**：命令行目标列表。

**典型用途**：

```makefile
sources = foo.c bar.c

ifeq (, $(filter clean, $(MAKECMDGOALS)))
include $(sources:.c=.d)
endif
```

### 8.3 不执行 Recipe

**`-n`、`--just-print`、`--dry-run`、`--recon`**：

- 打印 recipe，但不执行。
- 例外：`+` 开头或含 `$(MAKE)` 的行仍执行。

**`-t`、`--touch`**：

- 标记目标为最新，不实际改变内容。
- 用 `touch` 效果。
- 不适用于伪目标（除非有 `+` 或 `$(MAKE)` 的行）。

**`-q`、`--question`**：

- 静默检查目标是否最新。
- 最新返回 0，需要更新返回 1，错误返回 2。

**`-W file`、`--what-if=file`、`--assume-new=file`、`--new-file=file`**：

- 假装 `file` 的修改时间是现在。
- 和 `-n` 一起用可以看到"如果修改某文件会怎样"。

**三个标志不能同时用**。

### 8.4 避免重新编译某些文件

**场景**：改了头文件但不想重新编译所有依赖。

**`-t` 方法**：

1. `make` 确保目标最新。
2. 改头文件。
3. `make -t` 标记所有目标为最新。

**`-o file` 方法**：

1. `make -o headerfile` 编译需要的目标。
2. `make -t` 标记所有目标。

**`-o` 的含义**：文件被视为"旧"，不重新生成，也不导致其他目标重新生成。

### 8.5 覆盖变量

**语法**：

```makefile
make CFLAGS='-g -O'
```

- 命令行变量覆盖 makefile 中的赋值。
- 除非 makefile 用 `override`。
- 可以是递归或简单变量。

### 8.6 测试程序编译

**`-k` 或 `--keep-going`**：

```makefile
make -k
```

- 出错后继续其他不依赖失败目标的目标。
- 用于尽可能多地发现错误。
- Emacs 的 `M-x compile` 默认用 `-k`。

### 8.7 临时文件

- `MAKE_TMPDIR` 环境变量指定临时文件目录。
- 未设置时用系统默认（POSIX：`TMPDIR` 或 `/tmp`；Windows：`TMP`、`TEMP`、`TMPDIR`）。
- 目录必须存在，make 不会创建。
- 不能在 makefile 中设置。

### 8.8 选项总结

**主要选项**：

| 短选项                 | 长选项                    | 含义               |
| :--------------------- | :------------------------ | :----------------- |
| `-b`                   |                           | 忽略（兼容）       |
| `-B`                   | `--always-make`           | 所有目标都重新生成 |
| `-C dir`               | `--directory=dir`         | 切换目录           |
| `-d`                   |                           | 打印调试信息       |
| `--debug=options`      |                           | 调试输出级别       |
| `-e`                   | `--environment-overrides` | 环境覆盖 makefile  |
| `-E string`            | `--eval=string`           | 求值 makefile 语法 |
| `-f file`              | `--file=file`             | 读取 makefile      |
| `-h`                   | `--help`                  | 帮助               |
| `-i`                   | `--ignore-errors`         | 忽略错误           |
| `-I dir`               | `--include-dir=dir`       | 包含目录           |
| `-j [jobs]`            | `--jobs[=jobs]`           | 并行任务数         |
| `--jobserver-style`    |                           | jobserver 风格     |
| `-k`                   | `--keep-going`            | 继续执行           |
| `-l [load]`            | `--load-average[=load]`   | 负载限制           |
| `-L`                   | `--check-symlink-times`   | 检查符号链接时间   |
| `-n`                   | `--just-print`            | 只打印             |
| `-o file`              | `--old-file=file`         | 视为旧文件         |
| `-p`                   | `--print-data-base`       | 打印数据库         |
| `-q`                   | `--question`              | 只检查             |
| `-r`                   | `--no-builtin-rules`      | 无内置规则         |
| `-R`                   | `--no-builtin-variables`  | 无内置变量         |
| `-s`                   | `--silent`                | 静默               |
| `-S`                   | `--no-keep-going`         | 取消 `-k`          |
| `--shuffle[=mode]`     |                           | 打乱依赖顺序       |
| `-t`                   | `--touch`                 | 标记为最新         |
| `-v`                   | `--version`               | 版本               |
| `-w`                   | `--print-directory`       | 打印目录           |
| `--no-print-directory` |                           | 不打印目录         |
| `-W file`              | `--what-if=file`          | 假设文件被修改     |


## 第九章 使用隐式规则

> 一句话：隐式规则让 make 自动推导如何从源文件生成目标文件，不用写 recipe。

### 9.1 使用隐式规则

**方法**：不写 recipe，或写没有 recipe 的规则。

```makefile
foo: foo.o bar.o
        cc -o foo foo.o bar.o $(CFLAGS) $(LDFLAGS)
```

`foo.o` 没有规则，make 自动查找隐式规则。

**隐式规则的选择**：

- 有多个隐式规则可用时，选依赖存在的那个。
- 如果 `foo.c` 存在，用 C 编译规则。
- 否则如果 `foo.p` 存在，用 Pascal 编译规则。

**链式规则**：

- 隐式规则可以递归应用。
- 例如从 `.y` 到 `.c` 到 `.o`。

**空 recipe 阻止隐式规则**：

```makefile
foo.o: ;
```

### 9.2 内置规则目录

**常见规则**：

- **C 编译**：`n.o` 从 `n.c`，recipe：`$(CC) $(CPPFLAGS) $(CFLAGS) -c`
- **C++ 编译**：`n.o` 从 `n.cc`/`n.cpp`/`n.C`，recipe：`$(CXX) $(CPPFLAGS) $(CXXFLAGS) -c`
- **Pascal**：`n.o` 从 `n.p`，recipe：`$(PC) $(PFLAGS) -c`
- **Fortran**：`n.o` 从 `n.f`/`n.F`/`n.r`
- **汇编**：`n.o` 从 `n.s`，recipe：`$(AS) $(ASFLAGS)`
- **链接**：`n` 从 `n.o`，recipe：`$(CC) $(LDFLAGS) n.o $(LOADLIBES) $(LDLIBS)`
- **Yacc**：`n.c` 从 `n.y`
- **Lex**：`n.c` 从 `n.l`
- **Texinfo**：`n.info` 从 `n.texinfo`/`n.texi`/`n.txinfo`
- **RCS**：从 `n,v` 或 `RCS/n,v` 提取
- **SCCS**：从 `s.n` 或 `SCCS/s.n` 提取

**变量如 `COMPILE.c`、`LINK.o`、`PREPROCESS.S`** 保存 recipe。

**`OUTPUT_OPTION`**：`-o $@` 或空，取决于编译选项。

### 9.3 隐式规则使用的变量

**程序名变量**：

| 变量       | 默认值     | 用途           |
| :--------- | :--------- | :------------- |
| `AR`       | `ar`       | 归档维护       |
| `AS`       | `as`       | 汇编           |
| `CC`       | `cc`       | C 编译         |
| `CXX`      | `g++`      | C++ 编译       |
| `CPP`      | `$(CC) -E` | C 预处理       |
| `FC`       | `f77`      | Fortran        |
| `M2C`      | `m2c`      | Modula-2       |
| `PC`       | `pc`       | Pascal         |
| `CO`       | `co`       | RCS 提取       |
| `GET`      | `get`      | SCCS 提取      |
| `LEX`      | `lex`      | Lex            |
| `YACC`     | `yacc`     | Yacc           |
| `LINT`     | `lint`     | Lint           |
| `MAKEINFO` | `makeinfo` | Texinfo        |
| `TEX`      | `tex`      | TeX            |
| `TEXI2DVI` | `texi2dvi` | Texinfo 到 DVI |
| `WEAVE`    | `weave`    | Web 到 TeX     |
| `CWEAVE`   | `cweave`   | C Web 到 TeX   |
| `TANGLE`   | `tangle`   | Web 到 Pascal  |
| `CTANGLE`  | `ctangle`  | C Web 到 C     |
| `RM`       | `rm -f`    | 删除           |

**参数变量**：`ARFLAGS`、`ASFLAGS`、`CFLAGS`、`CXXFLAGS`、`CPPFLAGS`、`FFLAGS`、`LDFLAGS`、`LDLIBS`、`LFLAGS`、`YFLAGS`、`PFLAGS`、`RFLAGS`、`COFLAGS`、`GFLAGS`、`LINTFLAGS`。

### 9.4 隐式规则链

**含义**：多个隐式规则依次应用。

**例子**：从 `foo.y` 到 `foo.o`：

1. `foo.y` → `foo.c`（Yacc）
2. `foo.c` → `foo.o`（C 编译）

`foo.c` 是**中间文件**。

**中间文件的特点**：

1. 如果不存在，make 不创建它，除非它的依赖过时。
2. 如果创建了，用完后删除。

**`.INTERMEDIATE`**：标记为中间文件。

**`.NOTINTERMEDIATE`**：不标记为中间文件。

**`.SECONDARY`**：中间文件但永不自动删除。

**规则不能重复**：同一个隐式规则在链中不能出现两次。

**优化规则**：有些规则直接从 `.y` 到 `.o`，不用中间 `.c`。

**匹配任何事物规则**：非终端的匹配任何事物规则不用于隐式规则的依赖。

### 9.5 定义和重新定义模式规则

**模式规则**：目标包含 `%` 的规则。

makefile

```makefile
%.o : %.c
        $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

**`%` 匹配任何非空子串**。匹配的部分叫**词干**。

**内置规则和自定义规则**：自定义优先。

#### 9.5.1 模式规则介绍

- 目标中有 `%`。
- 依赖中的 `%` 代表相同的词干。
- 可以有不变的依赖。

#### 9.5.2 模式规则示例

**内置 C 编译规则**：

```makefile
%.o : %.c
        $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

**RCS 规则**：

```makefile
% :: RCS/%,v
        $(CO) $(COFLAGS) $<
```

**双目标规则**：

```makefile
%.tab.c %.tab.h: %.y
        bison -d $<
```

#### 9.5.3 自动变量

| 变量 | 含义               |
| :--- | :----------------- |
| `$@` | 目标文件名         |
| `$%` | 归档成员名         |
| `$<` | 第一个依赖         |
| `$?` | 比目标新的依赖     |
| `$^` | 所有依赖           |
| `$+` | 所有依赖（含重复） |
| `$|` | order-only 依赖    |
| `$*` | 词干               |

**变体**：`$(@D)`、`$(@F)`、`$(*D)`、`$(*F)`、`$(%D)`、`$(%F)`、`$(<D)`、`$(<F)`、`$(^D)`、`$(^F)`、`$(+D)`、`$(+F)`、`$(?D)`、`$(?F)`。

**`$?` 例子**：

```makefile
lib: foo.o bar.o lose.o win.o
        ar r lib $?
```

只复制比 `lib` 新的目标文件。

#### 9.5.4 模式如何匹配

- 目标模式 = 前缀 + `%` + 后缀。
- `%` 匹配的词干。
- 依赖模式中的 `%` 被替换为词干。
- 目录名在比较前被移除，依赖名生成后再加回。

**选择规则**：

1. 所有依赖存在或可构建的规则。
2. 最短词干优先。
3. 相同词干长度时，makefile 中先出现的优先。

**例子**：

```makefile
%.o : %.c
        $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
%.o : %.f
        $(COMPILE.F) $(OUTPUT_OPTION) $<
lib/%.o: lib/%.c
        $(CC) -fPIC -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

- `bar.o` 同时有 `bar.c` 和 `bar.f` → 用第一个。
- `lib/bar.o` 同时有 `lib/bar.c` 和 `lib/bar.f` → 用第三个（词干更短）。

#### 9.5.5 匹配任何事物的模式规则

**目标只是 `%`**。

**约束**：

- **终端规则**（`::`）：依赖必须实际存在，不继续链式搜索。
- **非终端规则**（`:`）：不能用于隐式规则的依赖，也不能用于有特定后缀的文件名。

**例子**：RCS 和 SCCS 规则是终端的。

**dummy 规则**：用于识别文件名，防止非终端匹配任何事物规则。

#### 9.5.6 取消隐式规则

**覆盖**：定义同目标同依赖但不同 recipe 的模式规则。

**取消**：定义同目标同依赖但无 recipe 的模式规则。

```makefile
%.o : %.s
```

取消汇编规则。

### 9.6 定义最后手段的默认规则

**最后手段规则**：终端的匹配任何事物模式规则，无依赖。

```makefile
%:
   touch $@
```

**`.DEFAULT`**：

```makefile
.DEFAULT:
        touch $@
```

用于没有规则的目标。

**清除 `.DEFAULT`**：

```makefile
.DEFAULT:
```

### 9.7 老式后缀规则

**双后缀规则**：`.c.o` 等价于 `%.o: %.c`。

```makefile
.c.o:
        $(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $<
```

**单后缀规则**：`.c` 等价于 `%: %.c`。

**后缀列表**：`.SUFFIXES` 的依赖。

**`.SUFFIXES` 无依赖**：清除默认后缀。

**`-r`**：默认后缀列表为空。

### 9.8 隐式规则搜索算法

1. 拆分 `t` 为目录部分 `d` 和其余 `n`。
2. 列出匹配 `t` 或 `n` 的模式规则。
3. 移除非终端匹配任何事物规则（除非 `t` 是隐式规则的依赖）。
4. 移除没有 recipe 的规则。
5. 对每个规则：
   - 找到词干 `s`。
   - 计算依赖名。
   - 测试依赖是否存在或应该存在。
6. 如果没有找到，再试一次，允许递归查找依赖。
7. 放宽"应该存在"的定义，再试一次。
8. 如果没有隐式规则适用，用 `.DEFAULT`。

**匹配后的操作**：为规则的其他目标模式存储文件名，recipe 执行后标记为已更新。


## 第十章 更新归档文件

> 一句话：归档文件（如 `.a`）的成员可以作为目标，make 有专门的规则处理。

### 10.1 归档成员作为目标

**语法**：

```makefile
archive(member)
```

**例子**：

```makefile
foolib(hack.o): hack.o
        ar cr foolib hack.o
```

**多个成员**：

```makefile
foolib(hack.o kludge.o)
```

等价于：

```makefile
foolib(hack.o) foolib(kludge.o)
```

**通配符**：

```makefile
foolib(*.o)
```

### 10.2 归档成员目标的隐式规则

**特殊规则**：目标模式 `(%)` 匹配任何归档成员。

**例子**：

```makefile
foo.a(bar.o)
```

make 会：

1. 编译 `bar.c` → `bar.o`（隐式规则）。
2. 把 `bar.o` 复制到 `foo.a` 作为成员 `bar.o`。
3. 删除中间文件 `bar.o`。

**`$%`**：归档成员名。

**带目录名**：

```makefile
foo.a(dir/file.o)
```

recipe：`ar r foo.a dir/file.o`。

#### 10.2.1 更新归档符号目录

**`__.SYMDEF`**：归档的特殊成员，包含外部符号目录。

**更新**：

```makefile
libfoo.a: libfoo.a(x.o y.o ...)
        ranlib libfoo.a
```

**注意**：GNU `ar` 会自动更新，不需要 `ranlib`。

### 10.3 使用归档的危险

**并行构建问题**：内置归档规则与并行构建不兼容。

**解决方法**：

```makefile
(%) : % ;
%.a : ; $(AR) $(ARFLAGS) $@ $?
```

**显式规则**：

```makefile
libfoo.a: libfoo.a(x.o y.o ...)
        $(AR) $(ARFLAGS) $@ $?
```

### 10.4 归档文件的后缀规则

**`.c.a`**：

```makefile
.c.a:
        $(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $*.o
        $(AR) r $@ $*.o
        $(RM) $*.o
```

等价于：

```makefile
(%.o): %.c
        $(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $*.o
        $(AR) r $@ $*.o
        $(RM) $*.o
```

**同时生成两个规则**：

- `(%.o): %.x`
- `%.a: %.x`
