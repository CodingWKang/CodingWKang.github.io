---
title: C语言程序设计
date: 2026-10-21 20:09:14
tags:
- C语言
category:
- 编程语言
index_img: /imgs/C.png
excerpt: 摘要：这份文档介绍了 C 语言学习的前置环境与核心知识。首先讲解使用 MSYS2 安装 mingw64 编译器、配置镜像源并安装 gcc、gdb、make。随后系统讲解 C 语言入门，包括第一个程序、变量与表达式、循环、符号常量、输入输出、数组、函数、字符数组与全局变量，并深入介绍运算符、数据类型、常量、类型转换、控制流、函数、指针与数组、结构体、联合、位字段以及标准输入输出等内容。
---


## 前置环境

在进行C语言代码编写前，首先需要一个C语言编译器的环境。**MSYS2**是一组工具和库的集合，它为我们提供了易于使用的环境，使得构建、安装和运行原生 Windows 软件变得更加简单，找到 **msys2-x86_64.exe**文件进行C语言编译器mingw64的安装。安装步骤如下：

![image](/imgs/C/2106185-20260816154621746-865064160.png)

![image](/imgs/C/2106185-20260816154704890-1041921382.png)

![image](/imgs/C/2106185-20260816154722874-1380702093.png)

![image](/imgs/C/2106185-20260816154732949-348116273.png)

![image](/imgs/C/2106185-20260816154949203-2123950622.png)

![image](/imgs/C/2106185-20260816155002972-472650321.png)

安装完成后可以看到，我们进入了一个msys2的命令行界面：
![image](/imgs/C/2106185-20260816155034995-1579805860.png)

找到**D:\msys64\etc\pacman.d**，在这个路径下有2个配置文件，分别为：**mirrorlist.mingw**和**mirrorlist.msys**。
- 编辑 mirrorlist.mingw，在文件开头添加：
`Server = https://mirrors.ustc.edu.cn/msys2/mingw/$repo/`
`Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/$repo/`
- 编辑 mirrorlist.msys，在文件开头添加：
`Server = https://mirrors.ustc.edu.cn/msys2/msys/$arch/`
`Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/$arch/`

运行 msys2.exe 然后执行 **pacman -Syu** 更新源和msys2。更新完之后然后运行：`pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-gdb mingw-w64-x86_64-make`安装gcc、gdb和make。

## 第一章 C语言入门
> 学习一门新的程序设计语言的最好的办法就是使用它。
### 1.1 第一个C语言程序
编写一个 **hello.c** 文件,输出一个"I Love GDU!!"。

```c
#include <stdio.h>      // 标准输入输出库，提供了输入输出函数的定义
int main(int argc, char *argv[]) {    // 程序的入口
  printf("I Love GDU!!");  // 输出字符串 "I Love GDU!!"
  return 0;  // 返回0表示程序成功结束
} // 这是一个简单的C程序，输出 "I Love GDU!!" 到控制台。

```
然后使用mingw编译器进行编译生成可执行文件：
```shell
gcc hello.c -o hello
.\hello.exe
```
### 1.2 变量及表达式
使用公式℃=(5/9)(℉-32)打印下列华氏温度与摄氏温度对照表。
```c
#include <stdio.h>
/* 当 fahr=0，20，… ，300 时，分别打印华氏温度与摄氏温度对照表 */
int main() {
  int fahr, celsius;
  int lower, upper, step;
  lower = 0;   /* 温度表的下限 */
  upper = 300; /* 温度表的上限 */
  step = 20;   /* 步长 */
  fahr = lower;
  while (fahr <= upper) {
    celsius = 5 * (fahr - 32) / 9;
    printf("%d\t%d\n", fahr, celsius);
    fahr = fahr + step;
  }
  return 0;
}
```
查看输出结果：
```shell
0       -17
20      -6
40      4
60      15
80      26
100     37
120     48
140     60
160     71
180     82
200     93
220     104
240     115
260     126
280     137
300     148
```
由于使用整型运算符，数值计算不太精确，因此需要修改数据类型为浮点型。
```c
#include <stdio.h>
/* print Fahrenheit-Celsius table
for fahr = 0, 20, ..., 300; floating-point version */
int main() {
  float fahr, celsius;
  float lower, upper, step;
  lower = 0;   /* lower limit of temperatuire scale */
  upper = 300; /* upper limit */
  step = 20;   /* step size */
  fahr = lower;
  while (fahr <= upper) {
    celsius = (5.0 / 9.0) * (fahr - 32.0);
    printf("%3.0f %6.1f\n", fahr, celsius);
    fahr = fahr + step;
  }
  return 0;
}
```
观察输出结果：
```shell
  0  -17.8
 20   -6.7
 40    4.4
 60   15.6
 80   26.7
100   37.8
120   48.9
140   60.0
160   71.1
180   82.2
200   93.3
220  104.4
240  115.6
260  126.7
280  137.8
300  148.9
```
**练习题**：编写一个程序打印摄氏温度转换为相应华氏温度的转换表。
### 1.3  for循环
将上面的程序进行改写，改写为使用for循环的方式。
```c
#include <stdio.h>
/* print Fahrenheit-Celsius table
for fahr = 0, 20, ..., 300; floating-point version */
int main() {
  float fahr, celsius;
  float lower, upper, step;
  lower = 0;   /* lower limit of temperatuire scale */
  upper = 300; /* upper limit */
  step = 20;   /* step size */
  fahr = lower;
  // 使用for循环实现
  for (fahr = lower; fahr <= upper; fahr = fahr + step) {
    celsius = (5.0 / 9.0) * (fahr - 32.0);
    printf("%3.0f %6.1f\n", fahr, celsius);
  }
  return 0;
}
```
### 1.4 符号常量
使用符号常量进行改写，`#define` 指令行的末尾没有分号。
```c
#include <stdio.h>
#define LOWER 0   /* lower limit of table */
#define UPPER 300 /* upper limit */
#define STEP 20   /* step size */
int main() {
  int fahr;
  for (fahr = LOWER; fahr <= UPPER; fahr = fahr + STEP)
    printf("%3d %6.1f\n", fahr, (5.0 / 9.0) * (fahr - 32));
  return 0;
}
```
### 1.5 输入/输出
使用getchar()和putchar()实现一个最简单的字符输入/输出操作。
```c
#include <stdio.h>
int main(const int argc, const char *argv[]) {
  int c;
  while ((c = getchar()) != EOF) {
    putchar(c);
  }
  return 0;
}
```
使用getchar()实现一个字符计数操作。
```c
#include <stdio.h>
int main(const int argc, const char *argv[]) {
  long nc = 0;
  while (getchar() != EOF) {
    nc++;
  }
  printf("length: %ld\n", nc);
  return 0;
}
```
**注意**：可以在一个输入行的开头使用`Ctrl+Z`除非EOF结束位。
使用getchar()+判断统计行数。
```c
#include <stdio.h>
int main(const int argc, const char *argv[]) {
  long nl = 0;
  int c;
  while ((c = getchar()) != EOF) {
    if (c == '\n') {
      ++nl;
    }
  }
  printf("lines: %ld\n", nl);
  return 0;
}
```
使用getchar()来实现一个单词计数。
```c
#include <stdio.h>
#define IN 1
#define OUT 0

int main(const int argc, const char *argv[]) {
  int c, nl, nw, nc, state;

  state = OUT;
  nl = nw = nc = 0;

  while ((c = getchar()) != EOF) {
    ++nc;
    if (c == '\n') {
      ++nl;
    }
    if (c == ' ' || c == '\n' || c == '\t') {
      state = OUT;
    } else if (state == OUT) {
      state = IN;
      ++nw;
    }
  }

  printf("%d %d %d\n", nl, nw, nc);
  return 0;
}
```
### 1.6 数组
使用数组记录0~9、空格、换行以及其他字符的个数。
```c
#include <stdio.h>
int main() {
  int c, i, white = 0, other = 0;
  int ndigit[10];
  for (i = 0; i < 10; i++) {
    ndigit[i] = 0;
  }
  while ((c = getchar()) != EOF) {
    if (c >= '0' && c <= '9') {
      ndigit[c - '0']++;
    } else if (c == ' ' || c == '\n' || c == '\t') {
      white++;
    } else {
      other++;
    }
  }
  printf("digits =");
  for (i = 0; i < 10; i++) {
    printf(" %d", ndigit[i]);
  }
  printf(", white space = %d, other = %d\n", white, other);
  return 0;
}
```
### 1.7 函数
```c
#include <stdio.h>
int power(int x, int y);
int main(int argc, char *argv[]) {
  int i;
  for (i = 0; i < 10; i++) {
    printf("%d ", power(2, i));
  }
  return 0;
}
int power(int x, int y) {
  int result = 1;
  for (int i = 0; i < y; i++) {
    result *= x;
  }
  return result;
}
```
### 1.8 字符数组
使用字符数组实现读入一行文本，并把最长行打印出来。
```c
#include <stdio.h>
#define MAXLINE 1000
int getline(char line[], int maxline);
void copy(char to[], char from[]);
int main() {
  int len;               /* current line length */
  int max;               /* maximum length seen so far */
  char line[MAXLINE];    /* current input line */
  char longest[MAXLINE]; /* longest line saved here */
  max = 0;
  while ((len = getline(line, MAXLINE)) > 0)
    if (len > max) {
      max = len;
      copy(longest, line);
    }
  if (max > 0) /* there was a line */
    printf("%s", longest);
  return 0;
}
int getline(char s[], int lim) {
  int c, i;
  for (i = 0; i < lim - 1 && (c = getchar()) != EOF && c != '\n'; ++i)
    s[i] = c;
  if (c == '\n') {
    s[i] = c;
    ++i;
  }
  s[i] = '\0';
  return i;
}
void copy(char to[], char from[]) {
  int i;
  i = 0;
  while ((to[i] = from[i]) != '\0')
    ++i;
}
```
### 1.9 全局变量
> main 函数中的变量是 main 函数的私自变量或局部变量。由于它们是在 main 函数中声明的，因此其它函数不能直接访问它们。函数中的每个局部变量只在函数被调用时存在，在函数执行完毕退出时消失。  

使用全局变量改写上述打印最长行程序。
```c
#include <stdio.h>
#define MAXLINE 1000
int max;
char line[MAXLINE];
char longest[MAXLINE];
int getline();
void copy();
int main() {
  int len;
  max = 0;
  while ((len = getline()) > 0)
    if (len > max) {
      max = len;
      copy();
    }
  if (max > 0)
    printf("%s", longest);
  return 0;
}
int getline() {
  int c, i;
  for (i = 0; i < MAXLINE - 1 && (c = getchar()) != EOF && c != '\n'; ++i)
    line[i] = c;
  if (c == '\n') {
    line[i] = c;
    ++i;
  }
  line[i] = '\0';
  return i;
}
void copy() {
  int i;
  i = 0;
  while ((longest[i] = line[i]) != '\0')
    ++i;
}
```
## 第二章 运算符与表达式
### 2.1 变量名
变量名字是由字母、数字和下划线组成的序列，但其第一个字符必须为字母或下划线开头。选择的变量名要能够尽量从字面上表达变量的用途，这样做不容易引起混淆。
### 2.2 数据类型
常用基本数据类型：char、int、float、double。类型限定符 signed 与 unsigned 可用于限定 char 类型或任何整型。
### 2.3 常量
long型常量以字母`l或L`结尾。如果一个整数太大以至于无法用 int 类型表示时，也将被当作 long 类型处理。无符号常量以字母` u 或 U `结尾。后缀` ul 或 UL `表明是` unsigned long `类型。没有后缀的浮点数常量为 double 类型。后缀` f 或 F `表示 float 类型，而后缀` l 或 L `则表示 long double 类型。带前缀 0 的整型常量表示它为八进制形式；前缀为 0x 或 0X，则表示它为十六进制形式。

常见的转义字符：

![image](/imgs/C/2106185-20260613200125682-1013056672.png)

常量表达式是仅仅只包含常量的表达式。这种表达式在编译时求值，而不在运行时求值。如：
```c
#define MAXLINE 1000 
char line[MAXLINE+1];
```

**注意**：从技术角度看，字符串常量就是字符数组。字符串的内部表示使用一个空字符'\0'作为串的结尾。因此，存储字符串的物理存储单元数比括在双引号中的字符数多一个。

枚举常量是另外一种类型的常量。枚举是一个常量整型值的列表，例如：
```c
enum boolean { NO, YES };
```
**注意**：在没有显式说明的情况下，enum 类型中第一个枚举名的值为 0，第二个为 1，依此类推。如果只指定了部分枚举名的值，那么未指定值的枚举名的值将依着最后一个指定值向后递增
```c
/* FEB 的值为 2，MAR 的值为 3，依此类推 */
enum months { JAN = 1, FEB, MAR, APR, MAY, JUN, JUL, AUG, SEP, OCT, NOV, DEC }; 
```
### 2.4 声明
所有变量都必须先声明后使用，尽管某些变量可以通过上下文隐式地声明。一个声明指定一种变量类型，后面所带的变量表可以包含一个或多个该类型的变量。例如：
```c
int lower, upper, step; 
char c, 1ine[1000];
```
任何变量的声明都可以使用 const 限定符限定。该限定符指定变量的值不能被修改。对数组而言，const 限定符指定数组所有元素的值都不能被修改。
```c
const double e = 2.71828182845905; 
const char msg[] = "warning: ";
```
const 限定符也可配合数组参数使用，它表明函数不能修改数组元素的值：
```c
int strlen(const char[]);
```
### 2.4 运算符
算术运算符：`+`、`-`、`*`、`/`、`%`
关系与逻辑运算符：`>`、`<`、`>=`、`<=`、`==`、`!=`、`&&`、`||`
自增与自减运算符：`a--`、`a++`
位运算符：`&`、`|`、`^`、`<<`、`>>`、`~`
赋值运算符：`=`
条件运算符：`expr1 ? expr2 : expr3`
### 2.5 类型转换
当一个运算符的几个操作数类型不同时，就需要通过一些规则把它们转换为某种共同的类型。一般来说，自动转换是指把“比较窄的”操作数转换为“比较宽的”操作数，并且不丢失信息的转换。

我们可以把不同数据类型想象成不同大小的水杯：有小酒杯（只能装一点点）、中号玻璃杯、大号水桶。当你要把两个杯子里的水混在一起计算容量时，杯子大小不一样没法直接算，必须统一成同一种容器，这个统一数据类型的过程，就叫类型转换。

**通俗比喻**:小酒杯（窄类型，存储范围小）往水桶（宽类型，存储范围大）倒水，水不会洒掉，所有数字都能完整保留，不会出错，这个过程电脑会自动完成，不用我们手动写代码。从小到大宽度排序（越往后容纳数字越大、支持小数）：char（字符 / 极小整数）→ short → int（普通整数）→ long → float（小数）→ double（高精度小数）

## 第三章 控制流
### 3.1 条件判断
二分支条件判断：
```c
if (表达式) 
  语句 1 
else 
  语句 2
```
多分支条件判断：
```c
if (表达式) 
  语句
else if (表达式) 
  语句
else if (表达式) 
  语句
else if (表达式) 
  语句
else 
  语句
```
### 3.2 switch 语句
```c
switch (表达式) { 
  case 常量表达式: 语句序列;
  case 常量表达式: 语句序列;
  default: 语句序列;
}
```
### 3.3 while 循环与 for 循环
while循环语法结构：
```c
while (表达式 1) { 
  语句
  表达式 2; 
}
```
for循环语法结构：
```c
for (表达式 1; 表达式 2; 表达式 3) 
  语句
```
### 3.4 do-while 循环
```c
do {
  语句
} while (表达式);
```
### 3.5 break 语句与 continue 语句
#### break关键字
不通过循环头部或尾部的条件测试而跳出循环，有时是很方便的。break 语句可用于从for、while与do-while等循环中提前退出，就如同从switch语句中提前退出一样。break语句能使程序从 switch 语句或最内层循环中立即跳出。

**生活例子**：我们把循环想象成老师让你依次写数字 1 到 10，break 就代表写到数字 5 时笔彻底坏掉没法继续写了，直接停手，6、7、8、9、10 全都不再书写，整个写数字任务直接结束；break 能用在 for、while、do-while 循环和 switch 分支中，只会跳出离它最近的一层循环。
**程序示例**：
```c
#include <stdio.h>
int main() {
    for(int i = 1; i <= 10; i++){
        if(i == 5){
            printf("笔坏了，停止书写\n");
            break;
        }
        printf("写下数字：%d\n", i);
    }
    printf("书写任务全部结束\n");
    return 0;
}
```
**运行结果**：
```shell
写下数字：1
写下数字：2
写下数字：3
写下数字：4
笔坏了，停止书写
书写任务全部结束
```
#### continue关键字
continue 语句与 break 语句是相关联的，但它没有 break 语句常用。continue 语句用于使 for、while 或 do-while 语句开始下一次循环的执行。在 while 与 do-while语句中，continue 语句的执行意味着立即执行测试部分；在 for 循环中，则意味着使控制转移到递增循环变量部分。continue 语句只用于循环语句，不用于 switch 语句。某个循环包含的 switch 语句中的 continue 语句，将导致进入下一次循环。

**生活例子**：同样是写数字 1 到 10，continue 代表规定不能写数字 5，碰到 5 直接跳过这一次书写步骤，不终止整体任务，写完 5 之后依旧正常写 6、7、8、9、10；continue 只能用在循环里，不能单独放在 switch 中，如果循环内部的 switch 里写 continue，也只是跳过当前一轮循环，while、do-while 执行 continue 会立刻回到循环条件判断，for 循环执行 continue 会直接跳到变量自增步骤。
**程序示例**：
```c
#include <stdio.h>
int main() {
    for(int i = 1; i <= 10; i++){
        if(i == 5){
            printf("跳过数字5，不打印\n");
            continue;
        }
        printf("写下数字：%d\n", i);
    }
    printf("全部数字遍历完毕\n");
    return 0;
}
```
**运行结果**：
```shell
写下数字：1
写下数字：2
写下数字：3
写下数字：4
跳过数字5，不打印
写下数字：6
写下数字：7
写下数字：8
写下数字：9
写下数字：10
全部数字遍历完毕
```
## 第四章 函数
> 函数可以把大的计算任务分解成若干个较小的任务，程序设计人员可以基于函数进一步构造程序，而不需要重新编写一些代码。一个设计得当的函数可以把程序中不需要了解的具体操作细节隐藏起来，从而使整个程序结构更加清晰，并降低修改程序的难度。

### 4.1 函数基本知识
在完整的 C 语言程序开发里，我们不会把所有代码全部堆砌在main函数中，随着业务逻辑变复杂，全部写在主函数里的代码会变得杂乱、难以修改和复用，函数就是用来拆分代码、实现模块化编程的核心工具。函数本质是一段封装了特定功能、拥有独立命名、可以被重复调用的代码片段，我们日常使用的`printf`、`scanf`都属于系统提前封装好的库函数。

一段完整的自定义函数由`返回值类型`、`函数名`、`形参列表`、`函数体`、`return` 返回语句五部分构成。如果函数不需要向调用处返回任何数据，返回值类型就填写`void`，此时函数内部可以省略 `return` 语句，或是只写不带参数的`return;`提前结束函数执行。

我们先从最简单的无参无返回值函数入手，直观感受函数定义与调用的完整流程，下面这段代码会封装一段打印问候语的逻辑，在 main 函数中完成调用：
```c
#include <stdio.h>
// 自定义无参无返回值函数
void printGreet() {
    printf("欢迎学习C语言函数模块\n");
    printf("函数可以拆分重复代码，简化程序结构\n");
}
int main(void) {
    // 直接调用自定义函数
    printGreet();
    return 0;
}
```
运行程序后控制台会连续输出两句提示文本，能清晰看到，封装好的printGreet只需要一行调用语句就能执行内部全部代码，如果我们需要多次打印这段文字，只需要多次书写printGreet();，不用重复复制 printf 代码，这就是函数最基础的复用优势。

实际开发中绝大多数函数都需要接收外部传入的数据参与运算，这就需要使用带形参的函数。形参写在函数名后的括号中，每个形参都需要标明数据类型，调用函数时传入的数值称为实参，实参会单向将数据拷贝给形参，函数内部对形参的修改不会影响外部实参变量。我们封装一个计算两个整数求和的函数来演示带参函数：
```c
#include <stdio.h>
// 带两个int类型形参、带int返回值的求和函数
int calcAdd(int num1, int num2) {
    int sum = num1 + num2;
    // return将运算结果传回调用处
    return sum;
}

int main(void) {
    int a = 36, b = 54;
    // 传入实参，接收函数返回结果存入变量res
    int res = calcAdd(a, b);
    printf("%d + %d = %d\n", a, b, res);
    return 0;
}
```
这段代码执行后会输出`36 + 54 = 90`，函数`calcAdd`接收 main 函数传入的两个数字，完成加法运算后通过 return 把总和返回给主函数的 res 变量。

这里需要重点区分函数定义和函数声明两个概念，如果自定义函数写在 main 函数之后，编译器自上而下编译代码时，读取到调用语句时还未识别该函数，会抛出编译警告甚至报错，解决办法就是在文件顶部书写函数声明。函数声明只需要复制函数首行，末尾添加分号即可，不需要书写函数体。下面这段错误示例和修正示例能直观体现声明的必要性：
```c
// 错误代码，函数定义在调用后方，无声明
#include <stdio.h>
int main(void) {
    int maxNum = getMax(12, 78);
    printf("最大值：%d", maxNum);
    return 0;
}
int getMax(int x, int y) {
    if(x > y) return x;
    return y;
}
```
```c
// 修正代码，添加函数声明
#include <stdio.h>
// 函数声明
int getMax(int x, int y);
int main(void) {
    int maxNum = getMax(12, 78);
    printf("最大值：%d", maxNum);
    return 0;
}
int getMax(int x, int y) {
    if(x > y) return x;
    return y;
}
```
### 4.2 返回非整型值的函数
C 语言支持函数返回 char、float、double 等任意基础数据类型，只需要修改函数头部的返回值类型，同时保证 return 后的数据和返回类型匹配即可。日常开发中计算小数运算时，几乎都会用到返回浮点型的函数，我们分别实现返回 float 单精度浮点、double 双精度浮点的函数作为演示。

首先编写计算圆形面积的函数，圆周率使用小数存储，返回 double 类型面积值：
```c
#include <stdio.h>
// 返回double类型的求圆面积函数
double circleArea(double r) {
    const double PI = 3.1415926;
    return PI * r * r;
}
int main(void) {
    double radius = 2.5;
    double area = circleArea(radius);
    // %lf格式化输出double类型数据
    printf("半径%.2lf的圆形面积为：%.4lf\n", radius, area);
    return 0;
}
```
运行后程序输出半径`2.50`的圆形面积为：`19.6349`，可以看到函数返回了高精度浮点数值，主函数使用 double 变量接收，搭配` % lf `完成打印。

再实现一个返回 float 类型的温度转换函数，将摄氏度转换为华氏度，公式为`华氏度 = 摄氏度 × 1.8 + 32`：
```c
#include <stdio.h>
// 返回float单精度浮点
float cToF(float c) {
    return c * 1.8f + 32.0f;
}
int main(void) {
    float c = 28.0f;
    float f = cToF(c);
    printf("摄氏%.1f度 = 华氏%.1f度\n", c, f);
    return 0;
}
```
字符类型 char 本质是 `ASCII` 码整数，同样可以作为函数返回值，我们写一个判断数字奇偶、返回字符标记的函数：
```c
#include <stdio.h>
// 返回char字符类型
char judgeOddEven(int num) {
    if(num % 2 == 0) {
        return 'E'; // E代表偶数Even
    } else {
        return 'O'; // O代表奇数Odd
    }
}
int main(void) {
    int data = 19;
    char tag = judgeOddEven(data);
    if(tag == 'E') printf("%d是偶数\n", data);
    else printf("%d是奇数\n", data);
    return 0;
}
```
书写返回非整型函数时需要注意类型匹配，如果 return 返回的数据和函数声明的返回类型不一致，编译器会自动隐式转换，容易丢失精度引发逻辑错误，例如 double 数值返回给 float 函数会截断小数精度，日常编码要尽量保证返回数据和返回值类型完全对应。

### 4.3 外部变量
变量根据定义位置分为局部变量和外部变量，在所有函数外部、文件最顶部定义的变量称为外部变量，也叫全局变量。外部变量的生命周期贯穿整个程序运行全程，程序启动时分配内存，程序结束才释放，作用域覆盖当前整个源文件，文件内所有函数都可以直接读取、修改外部变量，不需要通过参数传递数据。

我们通过一段代码直观感受外部变量的特性，定义全局计数器，在两个自定义函数中分别修改、读取它：
```c
#include <stdio.h>
// 定义外部全局变量count，所有函数均可访问
int count = 0;
void addCount() {
    // 函数内部直接修改外部变量
    count = count + 10;
}
void printCount() {
    // 函数内部直接读取外部变量
    printf("当前全局计数器数值：%d\n", count);
}
int main(void) {
    printCount();
    addCount();
    printCount();
    addCount();
    printCount();
    return 0;
}
```
程序运行输出结果依次为 0、10、20，能发现addCount函数没有传入参数、没有返回值，却可以直接修改全局 count 变量，修改后的结果能被其他函数读取，这就是外部变量跨函数共享数据的特性。

外部变量虽然能便捷实现多函数数据共享，但滥用会带来严重问题，过多全局变量会让程序逻辑混乱，任意函数都能修改全局变量，出现 bug 时很难定位是哪个函数修改了变量的值。如果我们需要在多个.c 源文件中访问同一个外部变量，需要使用extern关键字声明，告知编译器该变量在其他文件中定义，下面演示 extern 的使用方式：
```c
// 文件A.c
#include <stdio.h>
// 真正定义外部变量
int globalNum = 100;
```
```c
// 文件B.c
#include <stdio.h>
// extern声明，不会创建新变量，只是引用外部已定义的globalNum
extern int globalNum;

void printGlobal() {
    printf("跨文件读取外部变量：%d", globalNum);
}
```
如果给外部变量添加 `static` 修饰符，该外部变量就会变为静态全局变量，作用域被限制在当前单个.c 文件中，其他文件无法通过 `extern` 访问，能有效隔离不同文件的全局变量，避免命名冲突。
### 4.4 作用域规则
作用域规定了变量、函数标识符能够被访问的代码范围，C 语言中分为文件作用域、函数作用域、代码块作用域三类，结合前面的外部变量、局部变量能清晰区分三类作用域的边界。

文件作用域对应定义在所有函数之外的外部变量、函数声明，从定义位置开始，一直延伸到当前源文件末尾，整个文件内所有函数都能访问。函数作用域仅针对 goto 标签，标签只能在定义它的函数内部跳转，不能跨函数使用。最常用的是代码块作用域，使用大括号{}包裹的区域就是独立代码块，在代码块内部定义的变量属于局部变量，仅在当前大括号内有效，代码块执行结束后变量内存立刻释放，外部无法访问。

我们用嵌套代码块的示例演示代码块作用域的屏蔽规则，内层代码块可以定义和外层同名的变量，内层变量会临时屏蔽外层同名变量，内层代码块结束后，外层变量恢复可见：
```c
#include <stdio.h>
int main(void) {
    // 外层代码块变量a，main函数整个大括号内有效
    int a = 10;
    printf("外层a初始值：%d\n", a);

    // 内层独立代码块
    {
        // 内层定义同名变量a，屏蔽外层a
        int a = 50;
        printf("内层代码块a：%d\n", a);
        a = a + 10;
        printf("内层修改后a：%d\n", a);
    }
    // 内层代码块结束，内层局部a销毁，外层a恢复
    printf("离开内层代码块，外层a：%d\n", a);
    return 0;
}
```
输出结果为 10、50、60、10，标识符访问遵循就近原则，程序会优先查找当前最小作用域内的变量，找不到再向外层逐层查找，直到文件作用域，全部查找失败则报未定义标识符错误。
### 4.5 头文件

当项目拆分多个源文件、存在大量通用函数声明、宏定义、结构体定义时，我们不会把所有声明重复写在每个.c 文件顶部，而是将公共内容统一写入.h后缀的头文件中，通过`#include`预处理指令引入，这就是头文件的核心作用。头文件只存放声明，不会存放完整函数实现，函数实现代码写在对应的.c 源文件中，编译链接阶段编译器会将头文件内容复制到引入文件内，统一完成编译。

头文件分为系统标准头文件和自定义头文件，系统头文件使用尖括号`#include <stdio.h>`引入，编译器会去系统库目录查找文件；自定义头文件使用双引号`#include "myfunc.h"`引入，编译器优先在当前项目目录查找文件。

我们实操拆分一个自定义头文件，第一步新建`myfunc.h`头文件，存放函数声明和宏定义：
```c
// myfunc.h
// 头文件保护宏，防止头文件被多次重复引入引发重定义错误
#ifndef MYFUNC_H
#define MYFUNC_H
// 宏定义
#define PI 3.14159
// 函数声明
double circleArea(double r);
int getMax(int x, int y);
#endif
```
第二步新建myfunc.c源文件，引入头文件，书写函数完整实现：
```c
// myfunc.c
#include "myfunc.h"
double circleArea(double r) {
    return PI * r * r;
}
int getMax(int x, int y) {
    return x > y ? x : y;
}
```
第三步在主程序main.c中引入自定义头文件，直接调用函数，无需重复书写声明：
```c
// main.c
#include <stdio.h>
#include "myfunc.h"
int main(void) {
    double s = circleArea(3.0);
    int m = getMax(66, 88);
    printf("圆面积：%.2lf，最大值：%d\n", s, m);
    return 0;
}
```
编译运行时需要同时编译main.c和myfunc.c两个文件，完整程序就能正常调用头文件中声明的函数。头文件保护宏`#ifndef`、`#define`、`#endif`是编写自定义头文件的规范，同一个文件多次 `#include` 同一份头文件时，保护宏会拦截重复内容，避免变量、函数重定义编译错误，所有自定义.h 文件都必须添加保护宏。
### 4.6 静态变量
使用`static`关键字修饰的局部变量称为`静态局部变量`，普通局部变量存储在栈内存，函数调用结束后内存立即释放，每次调用函数都会重新创建、初始化局部变量；而静态局部变量存储在全局数据区，程序启动时分配内存，仅在第一次调用函数时执行一次初始化，后续重复调用函数不会重新初始化，变量值会保留上一次调用结束后的结果，生命周期和外部全局变量一致，但作用域仍仅限当前函数内部，其他函数无法访问。

我们编写计数器函数，分别使用普通局部变量和静态局部变量做对比，直观看出二者差异：
```c
#include <stdio.h>
// 使用普通局部变量的计数器
void normalCount() {
    int num = 0; // 每次调用都会初始化为0
    num++;
    printf("普通局部变量：%d\n", num);
}
// 使用static静态局部变量的计数器
void staticCount() {
    static int num = 0; // 仅第一次调用初始化，后续保留值
    num++;
    printf("静态局部变量：%d\n", num);
}
int main(void) {
    printf("连续调用普通变量函数三次：\n");
    normalCount();
    normalCount();
    normalCount();
    printf("\n连续调用静态变量函数三次：\n");
    staticCount();
    staticCount();
    staticCount();
    return 0;
}
```
运行后普通局部变量三次输出全部为 1，每次调用都会重置 num 为 0；静态局部变量依次输出 1、2、3，变量数值持续累加，不会重置，这就是 static 修饰局部变量最核心的特性。

静态变量适合用于需要记录函数调用次数、缓存中间运算结果的场景，同时它兼具全局变量持久存储数据的优势，又拥有局部变量仅函数内可见的隔离性，相比全局变量更安全，不会被其他函数随意篡改。静态变量初始化仅执行一次，初始化值默认是 0，即使不手动赋值，静态整型变量初始也为 0，普通局部变量不赋值则是随机垃圾值。
### 4.7 寄存器变量
C 语言提供register关键字修饰局部变量，称为寄存器变量，普通变量存储在内存中，CPU 运算时需要先从内存读取数据到寄存器，运算完成再写回内存；寄存器变量会建议编译器将变量存储在 CPU 通用寄存器中，省去内存读写的开销，提升频繁运算场景的执行效率，适合循环内高频访问的小型变量，比如循环计数器。

寄存器变量有严格的使用限制：寄存器存储空间极小，不能定义大型数组、结构体作为寄存器变量；无法对寄存器变量取地址，地址操作符&不能作用于 register 变量；同时 register 只是给编译器的优化建议，CPU 寄存器资源不足时，编译器会自动把 register 变量降级为普通内存变量，不会强制存入寄存器。

我们写一段循环累加代码，用 register 修饰循环计数器演示使用方式：
```c
#include <stdio.h>

int main(void) {
    // register修饰循环计数器，高频循环场景优化
    register int i;
    int sum = 0;
    for(i = 1; i <= 10000; i++) {
        sum = sum + i;
    }
    printf("1~10000累加和：%d\n", sum);
    return 0;
}
```
现代编译器自带强大的自动优化功能，会自动识别高频访问变量并放入寄存器，手动书写 register 的优化效果已经不明显，日常开发很少主动使用寄存器变量，仅作语法特性了解即可。同时 register 关键字不能修饰全局变量、静态变量，仅能用于函数内部普通局部变量。
### 4.8 程序块结构
C 语言中一对大括号`{}`包裹的任意代码区域都属于独立程序块，也就是代码块，函数整体、if 判断分支、for/while 循环、switch 分支都自带程序块。每一个程序块都会创建独立的作用域，块内定义的变量仅在当前块内有效，块执行完毕后变量内存释放，不同块之间同名变量互不干扰。

嵌套程序块是开发中十分常见的写法，内层程序块可以访问外层块定义的变量，外层无法访问内层块变量，内层定义同名变量会屏蔽外层变量，我们通过多层嵌套代码演示完整块结构规则：
```c
#include <stdio.h>
int main(void) {
    int outer = 10;
    printf("最外层块outer = %d\n", outer);
    // 第一层嵌套块
    {
        int mid = 50;
        printf("中层块读取外层outer = %d，自身mid = %d\n", outer, mid);
        // 第二层深层嵌套块
        {
            int inner = 999;
            // 内层可以读取外层所有块变量
            printf("深层块outer=%d mid=%d inner=%d\n", outer, mid, inner);
            // 定义同名变量屏蔽外层outer
            int outer = 0;
            printf("深层同名屏蔽outer = %d\n", outer);
        }
        // 深层块销毁，inner变量失效，外层outer恢复可见
        printf("离开深层块，无法访问inner，outer恢复：%d\n", outer);
    }
    // 中层块销毁，mid变量失效
    printf("离开中层块，仅保留outer：%d\n", outer);
    return 0;
}
```
合理划分程序块能规范代码逻辑，将功能独立的代码封装在单独块中，缩小变量作用范围，减少同名变量冲突的概率，降低bug出现的可能性。同时 if、for 等语句如果只执行一行代码，可以省略大括号，但规范编码建议无论代码行数多少都添加程序块大括号，避免后续新增代码时出现逻辑错误。
### 4.9 初始化
初始化指变量定义时直接赋予初始值的操作，不同存储类型、作用域的变量初始化规则存在明显区别，我们按照局部普通变量、静态局部变量、外部全局变量三类分别梳理初始化特性。

普通函数局部变量存储在栈内存，定义时不手动初始化，变量内存中保留随机垃圾值，必须手动赋值才能正常使用；静态局部变量和外部全局变量存储在全局数据区，编译器会自动执行零初始化，int 默认 0、char 默认 '\0'、浮点默认 0.0，即使不手动写初始值也能直接读取。

变量初始化的执行时机也有差异：普通局部变量每次进入代码块都会重新初始化；静态局部变量、全局变量仅在程序加载阶段初始化一次，程序运行过程中不会重复执行初始化语句。我们通过代码对比三类变量默认初始化结果：
```c
#include <stdio.h>
// 外部全局变量，自动零初始化
int globalA;
void testInit() {
    // 静态局部变量，仅首次调用初始化，默认0
    static int staticB;
    // 普通局部变量，无初始化，垃圾值
    int localC;
    printf("全局变量globalA=%d，静态变量staticB=%d，普通局部localC=%d\n", globalA, staticB, localC);
    staticB++;
}
int main(void) {
    testInit();
    testInit();
    return 0;
}
```
两次调用函数输出中 globalA 恒为 0，staticB 第一次 0、第二次 1，localC 每次都是随机无意义数字，清晰区分三类变量初始化规则。数组、字符串也支持定义时批量初始化，未填满的数组元素自动补零，字符数组初始化字符串时末尾自动补充结束符\0：
```c
#include <stdio.h>
int main(void) {
    // 数组部分初始化，剩余元素自动0
    int arr[5] = {1, 2};
    char str[10] = "hello";
    printf("数组第3位：%d，字符串：%s\n", arr[2], str);
    return 0;
}
```
常量变量使用 const 修饰，定义时必须完成初始化，程序运行全程无法修改数值；寄存器变量和普通局部变量初始化规则一致，无默认零值，必须手动赋值。
### 4.10 递归
递归是函数直接或间接调用自身的编程技巧，递归算法由两个核心部分构成：基线条件（递归出口，终止函数调用，避免无限循环）和递归条件（缩小问题规模，调用自身处理子问题）。递归适合拆解能够拆分出相同子问题的场景，比如阶乘计算、斐波那契数列、二分查找、树形遍历等。

我们先实现经典的阶乘递归函数，n 的阶乘定义为` n! = n × (n-1)!` ，0! 和 1! 固定等于 1，这就是基线出口：
```c
#include <stdio.h>
// 递归计算阶乘
long long factorial(int n) {
    // 基线条件：递归终止出口
    if(n == 0 || n == 1) {
        return 1;
    }
    // 递归条件：调用自身，缩小问题规模
    return n * factorial(n - 1);
}
int main(void) {
    int num = 7;
    long long res = factorial(num);
    printf("%d的阶乘 = %lld\n", num, res);
    return 0;
}
```
程序计算 7!，函数逐层调用 factorial (6)、factorial (5)…… 直到 factorial (1) 触达基线条件停止递归，再逐层向上返回乘法结果完成计算。如果缺少基线条件，函数会无限调用自身，栈内存持续累积函数调用帧，最终引发栈溢出程序崩溃，书写递归时必须优先确定递归出口。

再实现斐波那契数列递归，数列规则 `F (1)=1、F (2)=1，F (n)=F (n-1)+F (n-2)`：
```c
#include <stdio.h>
int fib(int n) {
    if(n == 1 || n == 2) return 1;
    return fib(n - 1) + fib(n - 2);
}
int main(void) {
    // 输出前10项斐波那契数列
    for(int i = 1; i <= 10; i++) {
        printf("%d ", fib(i));
    }
    return 0;
}
```
递归代码逻辑简洁直观，但存在性能缺陷，大量重复子问题会重复计算，同时每一次递归调用都会在栈中保存当前函数现场，数值过大时容易栈溢出，复杂场景可以搭配循环迭代优化，或是使用缓存记录已计算结果降低重复运算。间接递归是函数 A 调用函数 B，函数 B 再调用函数 A，同样需要设置基线条件终止调用链。
### 4.11 C预处理器

C 预处理器是编译阶段最先执行的步骤，在正式编译代码前，根据#开头的预处理指令处理源文件，完成文本替换、文件引入、条件编译等操作，不会识别 C 语言语法，仅做纯文本处理，本章前面用到的#include、#define都属于预处理指令。

#### 宏定义`#define`

`#define` 用于定义文本替换宏，分为无参宏和带参宏，无参宏常用来定义常量，预处理阶段会把文件中所有宏名替换为对应文本：
```c
#include <stdio.h>
// 无参宏定义
#define MAX_NUM 100
// 带参宏，注意参数括号防止运算优先级错误
#define SQUARE(x) ((x) * (x))
int main(void) {
    int a = MAX_NUM;
    int b = SQUARE(5 + 2);
    printf("a=%d，5+2平方=%d\n", a, b);
    return 0;
}
```
#### 条件编译指令

`#ifdef、#ifndef、#else、#endif`实现条件编译，满足条件时才编译对应代码块，前面头文件保护宏就是依靠 `#ifndef` 实现，除此之外可以用于区分调试版本和发布版本：
```c
#include <stdio.h>
// 定义调试宏
#define DEBUG
#ifdef DEBUG
    // 调试模式编译这段打印日志代码
    #define LOG printf("调试日志输出\n")
#else
    // 发布模式不输出日志
    #define LOG
#endif
int main(void) {
    LOG;
    return 0;
}
```
注释指令`#include`文件引入、`#undef`取消宏定义、`#line`修改行号、`#error`抛出编译错误都属于常用预处理指令。预处理器独立于 C 语法，所有替换操作只发生在编译前期，不会占用程序运行时性能，合理使用预处理能提升代码复用性、方便多环境适配。

## 第五章 指针和数组
> 指针是一种保存变量地址的变量，使用指针可以生成更高效、更紧凑的代码。

### 5.1 指针与地址
通常机器有一系列连续编号或编址的存储单元，这些单元可以单个进行操纵，也可以以连续成组的方式操纵。一般来说一个字节可以存放一个char类型的数据，两个相邻的字节存储单元可以存储一个short类型的数据，4个相邻的字节存储单元可以存储一个long类型的数据。指针是能够存放一个地址的一组存储单元。

通过使用运算符&可以用于取一个对象的地址，因此
```c
p = &c
```
将把c的地址赋值给变量p，称p为指向c的指针。地址运算符&只能应用于内存中的变量，即变量与数组元素。
一元运算符*是间接寻址或间接引用运算符，当其用作于指针时，将访问指针所指向的对象。下面的代码说明了如何在程序中声明指针以及使用运算符`&`和`*`。
```c
int x =1,y = 2,z[10];
int *ip;
ip = &x;
y = *ip;
*ip = 0;
ip = &z[0];
```
### 5.2 指针与函数参数
由于C语言是以传值的方式将参数值传递给被调用函数的，因此，被调用函数不能直接修改主调函数中变量的值。如，一个简单的swap函数是用来交换两个次序颠倒的元素，但是，如果将swap函数定义为：
```c
void swap(int x,int y){
  int temp;
  temp = x;
  x = y;
  y = temp;
}
swap(a,b);
```
无法实现两数交换的目的，因为由于参数传递采用传值的方式，因此swap函数不会影响到调用它的程序中的参数a和b的值，仅仅只是交换了a和b的副本的值。
所以，为了实现上述目的，我们需要修改函数传值的方式为传递指针。如下：
```c
void swap(int *px,int *py){
  int temp;
  temp = *px;
  *px = *py;
  *py = temp;
}
swap(&a,&b);
```
### 5.3 指针与数组

在C语言中，指针与数组之间关系密切。通过数组下标所能完成的任何操作都可以通过指针进行实现，一般来说使用指针编写的程序比数组下标编写的程序执行速度快。

声明
```c
int a[10];
```
定义了一个长度为10的数组a。换句话说，定义了一个由10个对象组成的集合，10个对象存储在相邻的内存区域中，名字分别为`a[0],a[1],.....,a[9]`。

![image](/imgs/C/2106185-20260621225946314-56158726.png)

a[i]表示该数组的第i个元素，如果pa声明为
```c
int *pa;
```
则说明它是一个指向整型对象的指针，那么赋值语句
```c
pa = &a[0];
```
则可以将指针pa指向数组a的第0个元素，即pa的值为数组元素a[0]的地址。

![image](/imgs/C/2106185-20260621230325394-375666041.png)

这样的话，赋值语句
```c
x = *pa;
```
将把数组元素a[0]中的内容复制到变量x中

如果pa指向数组中的某个特定元素，那么根据指针运算的定义，pa+1将指向下一个元素，pa+i将指向pa所指想数组元素之后的第i个元素，而pa-i将指向pa所指向数组元素之前的第i个元素。隐藏如果指针pa指向a[0]，那么*(pa+1)引用的是数组元素a[1]的内容，pa+i是数组元素a[i]的地址，*(pa+i)引用的是数组元素a[i]的内容。

下标和指针运算之间是具有密切的对应关系。执行赋值语句
```c
pa = &a[0];
```
后，pa和a具有相同的值。由于数组名代表的就是该数组最开始的一个元素的地址，所以上述赋值语句也可以写成下列形式：
```c
pa = a;
```
对数组元素a[i]的引用也可以写成`*(a+i)`这种形式，在计算数组元素a[i]的值是，C语言实际上先将其转换为`*(a+i)`的形式，然后再进行求值。但是数组名和指针之间有一个不同之处，指针是一个变量，**数组名不是变量**。

### 5.4 地址运算符

如果p是一个指向数组中某个元素的指针，那么p++将对p进行自增运算并指向下一个元素，而`p+=i`将对p进行加i运算，使其指向指针p当前所指向元素之后的第i个元素。

指针的算术运算符具有一致性，如果处理的数据类型是比字符型占据更多存储空间的浮点类型，并且p是一个指向浮点类型的指针，那么在执行p++后，p将指向下一个浮点数的地址。

### 5.5 字符指针与函数
字符串常量是一个字符数组，例如：
```txt
"I am a string"
```
在字符串的内部表示中，字符数组以空字符'\0'结尾，所以，程序可以通过检查空字符找到字符数组的结尾。字符串常量占据的存储单元数也因此比双引号内的字符数大 1。

下面两个定义之间有很大的差别：
```c
char amessage[] = "nw is the time"; /* 定义一个数组 */ 
char *pmessage = "now is the time"; /* 定义一个指针 */
```
上述声明中，amessage 是一个仅仅足以存放初始化字符串以及空字符'\0'的一维数组。数组中的单个字符可以进行修改，但 amessage 始终指向同一个存储位置。另一方面，pmessage
是一个指针，其初值指向一个字符串常量，之后它可以被修改以指向其它地址，但如果试图修改字符串的内容，结果是没有定义的。

### 5.6 多维数组
C 语言提供了类似于矩阵的多维数组，但实际上它们并不像指针数组使用得那样广泛。我们考虑一个日期转换的问题，把某月某日这种日期表示形式转换为某年中第几天的表示形式，反之亦然。例如，3 月 1 日是非闰年的第 60 天，是闰年的第 61 天。在这里，我们定义下列两个函数以进行日期转换：函数 day_of_year 将某月某日的日期表示形式转换为某一年中第几天的表示形式，函数 month_day 则执行相反的转换。因为后一个函数要返回两个值，所以在函数 month_day 中，月和日这两个参数使用指针的形式。例如，下列语句将把 m 的值设置为 2，把 d 的值设置为 29（2 月 29 日）。
```c
month_day(1988, 60, &m, &d);
```
这些函数都要用到一张记录每月天数的表（如“9 月有 30 天”等）。对闰年和非闰年来说，每个月的天数不同，所以，将这些天数分别存放在一个二维数组的两行中比在计算过程中判断 2 月有多少天更容易。该数组以及执行日期转换的函数如下所示：
```c
static char daytab[2][13] = {
    {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31},
    {0, 31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}};
/* day_of_year: set day of year from month & day */
int day_of_year(int year, int month, int day) {
  int i, leap;
  leap = year % 4 == 0 && year % 100 != 0 || year % 400 == 0;
  for (i = 1; i < month; i++)
    day += daytab[leap][i];
  return day;
}
/* month_day: set month, day from day of year */
void month_day(int year, int yearday, int *pmonth, int *pday) {
  int i, leap;
  leap = year % 4 == 0 && year % 100 != 0 || year % 400 == 0;
  for (i = 1; yearday > daytab[leap][i]; i++)
    yearday -= daytab[leap][i];
  *pmonth = i;
  *pday = yearday;
}
```
**注意**：如果将二维数组作为参数传递给函数，那么在函数的参数声明中必须指明数组的列数,数组的行数没有太大关系。因为前面已经讲过，函数调用时传递的是一个指针，它指向由行向量构成的一维数组，其中每个行向量是具有 13 个整型元素的一维数组。

### 5.7 指针与多维数组
对于 C 语言的初学者来说，很容易混淆二维数组与指针数组之间的区别，比如上面例子中的 name。假如有下面两个定义：
```c
int a[10][20]; 
int *b[10];
```
从语法角度讲，a[3][4]和 b[3][4]都是对一个 int 对象的合法引用。但 a 是一个真正的二维数组，它分配了 200 个 int 类型长度的存储空间，并且通过常规的矩阵下标计算公式 20×row+col（其中，row 表示行，col 表示列）计算得到元素 a[row][col]的位置。但是，对 b 来说，该定义仅仅分配了 10 个指针，并且没有对它们初始化，它们的初始化必须以显式的方式进行，比如静态初始化或通过代码初始化。假定 b 的每个元素都指向一个具有 20 个元素的数组，那么编译器就要为它分配 200 个 int 类型长度的存储空间以及 10 个指针的存储空间。指针数组的一个重要优点在于，数组的每一行长度可以不同，也就是说，b 的每个元素不必都指向一个具有 20 个元素的向量，某些元素可以指向具有 2 个元素的向量，某些元素可以指向具有 50 个元素的向量，而某些元素可以不指向任何向量。
指针数组最频繁的用处是存放具有不同长度的字符串，比如函数 month_name 中的情况。结合下面的声明和图形化描述，我们可以做一个比较。下面是指针数组的声明和图形化描述：
```c
char *name[]={"Illegal manth", "Jan", "Feb", "Mar"};
```
![image](/imgs/C/2106185-20260622021833314-824071475.png)

下面是二维数组的声明和图形化描述：
```c
char aname[][15] = { "Illegal month", "Jan", "Feb", "Mar" };
```

![image](/imgs/C/2106185-20260622022214408-786305443.png)

### 5.8 命令行参数
在支持 C 语言的环境中，可以在程序开始执行时将命令行参数传递给程序。调用主函数main 时，它带有两个参数。第一个参数（习惯上称为 argc，用于参数计数）的值表示运行程序时命令行中参数的数目；第二个参数（称为 argv，用于参数向量）是一个指向字符串数组的指针，其中每个字符串对应一个参数。按照 C 语言的约定，argv[0]的值是启动该程序的程序名，因此 argc 的值至少为 1。如果 argc 的值为 1，则说明程序名后面没有命令行参数。

### 5.9 指向函数的指针
在 C 语言中，函数本身不是变量，但可以定义指向函数的指针。这种类型的指针可以被赋值、存放在数组中、传递给函数以及作为函数的返回值等等。指向函数的指针的声明如下:
```c
int (*comp)(void *, void *);
```
它表明 comp 是一个指向函数的指针，该函数具有两个 void *类型的参数，其返回值类型为int。
comp 的使用和其声明是一致的，comp 是一个指向函数的指针，*comp 代表一个函数。下列语句是对该函数进行调用：
```c
(*comp)(v[i], v[left])
```
## 第六章 结构体
> 结构是一个或多个变量的集合，这些变量可能为不同的类型，为了处理的方便而将这些变量组织在一个名字之下。

### 6.1 结构的基本知识

我们首先来建立一些适用于图形领域的结构。点是最基本的对象，假定用 x 与 y 坐标表示它，且 x、y 的坐标值都为整数。

![image](/imgs/C/2106185-20260622023559847-624827336.png)

我们可以采用结构存放这两个坐标，其声明如下：
```c
struct point { 
  int x; 
  int y; 
};
```
关键字 struct 引入结构声明。结构声明由包含在花括号内的一系列声明组成。关键字struct 后面的名字是可选的，称为结构标记（这里是 point）。结构标记用于为结构命名，在定义之后，结构标记就代表花括号内的声明，可以用它作为该声明的简写形式。

结构中定义的变量称为成员。结构成员、结构标记和普通变量（即非成员）可以采用相同的名字，它们之间不会冲突，因为通过上下文分析总可以对它们进行区分。另外，不同结构中的成员可以使用相同的名字。

struct 声明定义了一种数据类型。在标志结构成员表结束的右花括号之后可以跟一个变量表，这与其它基本类型的变量声明是相同的。例如：
```c
struct { ... } x, y, z; 
```
从语法角度来说，这种方式的声明与声明
```c
int x, y, z; 
```
具有类似的意义。这两个声明都将 x、y 与 z 声明为指定类型的变量，并且为它们分配存储空间。如果结构声明的后面不带变量表，则不需要为它分配存储空间，它仅仅描述了一个结构的模板或轮廓。但是，如果结构声明中带有标记，那么在以后定义结构实例时便可以使用该标记定义。

在表达式中，可以通过下列形式引用某个特定结构中的成员：***结构名.成员***。例如：
```c
struct rect { 
  struct point pt1; 
  struct point pt2; 
}; 
```
结构 rect 包含两个 point 类型的成员。如果按照下列方式声明 screen 变量：
```c
struct rect screen; 
```
则可以用语句`screen.pt1.x` 引用 screen 的成员 pt1 的 x 坐标。
### 6.2 结构与函数
结构的合法操作只有几种：作为一个整体复制和赋值，通过&运算符取地址，访问其成员。其中，复制和赋值包括向函数传递参数以及从函数返回值。结构之间不可以进行比较，可以用一个常量成员值列表初始化结构，自动结构也可以通过赋值进行初始化。

传递结构成员的方法有三种：一是分别传递各个结构成员，二是传递整个结构，三是传递指向结构的指针。

观察下面函数makepoint，其带有两个整型参数，并返回一个point类型结构：
```c
struct point makepoint(int x,int y){
  struct point temp;
  temp.x = x;
  temp.y = y;
  return temp;
}
```
现在可以使用makepoint函数动态地初始化任意结构，也可以向函数提供结构类型的参数。
```c
struct point screen;
struct point middle;
struct point makepoint(int,int);
screen.pt1 = makepoint(0,0);
screen.pt2 = makepoint(XMAX,YMAX);
middle = makepoint((screen.pt1.x+screen.pt2.x)/2,(screen.pt1.y+screen.pt2.y)/2);
struct point addpoint(struct point p1,struct point p2){
  p1.x += p2.x;
  p1.y += p2.y;
  return p1;
}
```
其中函数的参数和返回值都是结构类型。如果传递给函数的结构很大，使用指针方式的效率通常比复制整个结构的效率更高，结构体指针类似于普通变量指针。声明如下：
```c
struct point *pp;
```
将pp定义为一个指向struct point类型对象的指针，如果pp指向一个point结构，那么*pp即为该结构，而`(*pp).x`和`(*pp).y`则是结构成员。
```c
struct point origin,*pp;
pp = &origin;
printf("origin is (%d,%d)\n",(*pp).x,(*pp).y);
```
其中(*pp).x中的圆括号是必需的，因为结构成员运算符`.`的优先级比`*`的优先级高。由于结构指针的使用频度非常高，为了方便使用，C语言提供了另一种简写方式。假定p是一个指向结构体的指针，可以用`p->结构成员`对结构成员进行引用。
```c
printf("origin is (%d,%d)\n",pp->x,pp->y);
```
运算符`.`和`->`都是从左至右结合的，所以对于下面的声明：
```c
struct rect r,*rp = &r;
```
下面的四个表达式是等价的：
```txt
r.pt1.x
rp->pt1.x
(r.pt1).x
(rp->pt1).x
```
### 6.3 结构数组
考虑这样一个场景，统计输入中各个C语言关键字出现的次数，一种实现方法是：
```c
char *keyword[NKEYS];
int keycount[NKEYS];
```
可以发现这两个数组的大小相同，可以采用另一种不同的组织方式：
```c
struct key{
  char *word;
  int count;
} keytab[NKEYS];
```
它声明了一个结构类型key，并定义了该类型的结构数组keytab，同时为其分配存储空间。
对于结构体数组，有时候需要计算结构体数组内的项数，可以借用`sizeof`进行计算，因为数组的长度在编译时已经完全确定，所以项数可以使用：
```txt
sizeof(结构体)/sizeof(类型名)
```
### 6.4 指向结构的指针
为了进一步说明指向结构体的指针和结构数组，可以采用指针编写关键字统计程序。
```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>
#define NKEYS 100
#define MAXWORD 100
struct key{
    char *word;
    int count;
}keytab[NKEYS];

int getword(char *, int);
struct key *binsearch(char *, struct key *, int);
int main()
{
    char word[MAXWORD];
    struct key *p;
    while (getword(word, MAXWORD) != EOF)
        if (isalpha(word[0]))
            if ((p = binsearch(word, keytab, NKEYS)) != NULL)
                p->count++;
    for (p = keytab; p < keytab + NKEYS; p++)
        if (p->count > 0)
            printf("%4d %s\n", p->count, p->word);
    return 0;
}
struct key *binsearch(char *word, struct key *tab, int n)
{
    int cond;
    struct key *low = &tab[0];
    struct key *high = &tab[n];
    struct key *mid;
    while (low < high) {
        mid = low + (high - low) / 2;
        if ((cond = strcmp(word, mid->word)) < 0)
            high = mid;
        else if (cond > 0)
            low = mid + 1;
        else
            return mid;
    }
    return NULL;
}
```
### 6.5 类型定义
C语言提供了一个称为typedef的功能，用来建立新的数据类型名。
```c
typedef int Length;
```
将Length定义为与int具有相同意义的名字，类型Length可以用于类型声明，类型转换等，它的类型和int完全相同。但是需要强调的是typedef声明并未创建一个新的类型，其只是为某个已经存在的类型增加了一个新名称而已。
### 6.6 联合(union)与位字段
#### 6.6.1 联合
联合是可以保存不同类型和长度的对象的变量，编译器负责跟踪对象的长度和对齐要求。联合提供了一种方式，以在单块存储区中管理不同类型的数据，而不需要在程序中嵌入任何和机器有关的信息。
```c
union u_tag{
  int ival;
  float fval;
  char *sval;
} u;
```
可以通过下列语法访问联合的成员：
```txt
联合名.成员
```
或
```txt
联合指针->成员
```
#### 6.6.2 位字段
在存储空间很宝贵的情况下，有可能需要将多个对象保存在一个机器字中。C语言中提供了直接定义和访问一个字中的位字段的能力，而不是通过按位逻辑运算符。位字段是字中相邻位的集合。字是单个的存储单元。
```c
struct {
  unsigned int is_keyword : 1;
  unsigned int is_extern : 1;
  unsigned int is_static : 1;
}flags;
```
这里定义了一个变量flags，其包含3个一位的字段，冒号后面的数字表示字段的宽度。
## 第七章 输入/输出
### 7.1 标准输入输出
标准库实现了简单的文本输入/输出模式。文本流由一些列行组成，每一行的结尾是一个换行符。如果系统没遵循这种模式，则标准库将通过一些措施使得该系统适应这种模式。
最简单的输入机制是使用getchar()函数从标准输入中一次读入一个字符，getchar()在每次调用时返回下一个输入字符，若遇见文件结尾则返回EOF。而putchar()常被用来输出数据，在默认情况下，标准输出为屏幕显示，若未发生错误，则函数putchar()将返回输出的字符，如果发生了错误，则返回EOF。
### 7.2 格式化输出——printf函数
输出函数printf将内部数值转换为字符的形式，函数printf在输出格式format的控制下，将其参数进行转换与格式化，并在标准输出设备上打印出来，返回值为打印的字符数。
格式字符串包含两种类型的对象：普通字符和转换说明。在输出时，普通字符将原样不动地复制到输出流中，而转换说明并不直接输出到输出流中，而是用于控制printf中参数的转换和打印。每个转换说明都由一个百分号字符（%）开始，并以一个转换字符结束。在字符%和转换字符中间可能依次包含下列组成部分：

- 负数，用于指定被转换的参数按照左对齐的形式输出。
- 数，用于指定最小字符宽度。转换后的参数将打印不小于最小字段宽度的字段。如果有必要字段左边多余的字符位置用空格填充以保证最小字段宽。
- 小数点，用于将字段宽度和精度分开。
- 数，用于指定精度，即指定字符串中要打印的最大字符数，浮点数小数点后的位数，整型最少输出的数字数目。
- 字母h或l,字母h表示将整数作为short类型引用，字母l表示将整数作为long类型打印。

![image](/imgs/C/2106185-20260622180311184-1379460331.png)

### 7.3 变长参数表
函数printf的正确声明为：
```c
int printf(char *fmt,...)
```
其中，省略号表示参数表中参数的数量和类型是可变的。省略号只能出现在参数表的尾部。标准头文件`<stdarg.h>`中包含一组宏定义，其对如何遍历参数表进行了定义。
va_list类型用于声明一个变量，该变量将依次引用各参数。宏va_start将ap初始化为指向第一个无名参数的指针，在使用ap之前，该宏必须被调用一次，参数表中必须至少包含一个有名参数，va_start将最后一个有名参数作为起点。每次调用va_arg，该函数都将返回一个参数，并将ap指向下一个参数，va_arg使用一个类型名来决定返回的对象类型、指针移动的步长，最后必须在函数返回之前调用va_end来完成一些必备的清理工作。

### 7.4 格式化输入——scanf函数
输入函数scanf对应于输出函数printf，与后者相反的方向上提供同样的转换功能。scanf函数按照参数中规定的格式扫描字符串string，并把结果分别保存到arg1,arg2,...这些参数中，这些参数必须是指针。
## 小游戏实战（推箱子）
```c
#include <stdio.h>
#include <graphics.h>
#include <conio.h>
#include <windows.h>
#include <mmsystem.h>
#pragma comment(lib, "winmm.lib")
#define ROW 8
#define COL 8
#define CELL 32
#define W_WIN 300
#define H_WIN 300
static int offsetX = 0;
static int offsetY = 0;
#define MAX_LAYER 20
static IMAGE wall, box, target, player, box_on_target, player_on_target;
static int loading = 0;
static int level = 0;
int box_num[MAX_LAYER] = {1, 1, 2, 2, 2, 3, 3, 3, 4, 4, 4, 4, 4, 5, 4, 5, 5, 5, 6, 6};
void loadResource();
void drawMap(int Map[][ROW][COL]);
void playSfx(const TCHAR* file);
void playBgm(const TCHAR* file);
void stopSound();
ExMessage msg;

int main() {
	SetProcessDPIAware();
	int screenW = GetSystemMetrics(SM_CXSCREEN);
	int screenH = GetSystemMetrics(SM_CYSCREEN);
	initgraph(W_WIN, H_WIN);
	HWND hwnd = GetHWnd();
	Sleep(50);
	RECT rect = { 0, 0, W_WIN, H_WIN };
	AdjustWindowRect(&rect, GetWindowLong(hwnd, GWL_STYLE), FALSE);
	int winW = rect.right - rect.left;
	int winH = rect.bottom - rect.top;
	int winX = (screenW - winW) / 2;
	int winY = (screenH - winH) / 2;
	SetWindowPos(hwnd, NULL, winX, winY, 0, 0, SWP_NOSIZE | SWP_NOZORDER);
	offsetX = (getwidth()  - COL * CELL) / 2;
	offsetY = (getheight() - ROW * CELL) / 2;
	SetWindowText(hwnd, "祭奠406");
	BeginBatchDraw();
	//  空白:0 墙壁:1 箱子:2 目的地:4 人物:5 箱子在目的地:6 人在目的地:9
	int Map[][ROW][COL] = {
		// ===== 第 1 关：1 个箱子，入门 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
			0, 1, 4, 0, 0, 1, 0, 0,
			0, 1, 0, 2, 0, 1, 0, 0,
			0, 1, 0, 5, 0, 1, 0, 0,
			0, 1, 0, 0, 0, 1, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
			0, 0, 0, 0, 0, 0, 0, 0,
		},
		// ===== 第 2 关：1 个箱子，转弯 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
			0, 1, 4, 0, 1, 1, 0, 0,
			0, 1, 0, 0, 1, 1, 0, 0,
			0, 1, 0, 2, 0, 1, 0, 0,
			0, 1, 0, 5, 0, 1, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
			0, 0, 0, 0, 0, 0, 0, 0,
		},
		// ===== 第 3 关：2 个箱子，直线 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
			0, 1, 4, 0, 4, 1, 0, 0,
			0, 1, 0, 0, 0, 1, 0, 0,
			0, 1, 2, 0, 2, 1, 0, 0,
			0, 1, 0, 5, 0, 1, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
			0, 0, 0, 0, 0, 0, 0, 0,
		},
		// ===== 第 4 关：2 个箱子，上下推 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
			0, 1, 4, 0, 0, 1, 0, 0,
			0, 1, 0, 2, 0, 1, 0, 0,
			0, 1, 0, 5, 0, 1, 0, 0,
			0, 1, 0, 2, 0, 1, 0, 0,
			0, 1, 0, 4, 0, 1, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
		},
		// ===== 第 5 关：2 个箱子，横向 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 2, 0, 1, 0,
			0, 1, 0, 0, 0, 0, 1, 0,
			0, 1, 2, 0, 5, 0, 1, 0,
			0, 1, 0, 0, 0, 0, 1, 0,
			0, 1, 4, 0, 0, 0, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 6 关：3 个箱子 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
			0, 1, 4, 4, 4, 1, 0, 0,
			0, 1, 0, 0, 0, 1, 0, 0,
			0, 1, 2, 2, 2, 1, 0, 0,
			0, 1, 0, 0, 0, 1, 0, 0,
			0, 1, 0, 5, 0, 1, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
		},
		// ===== 第 7 关：3 个箱子，走廊 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
			0, 1, 4, 0, 4, 1, 0, 0,
			0, 1, 0, 2, 0, 1, 0, 0,
			0, 1, 0, 5, 0, 1, 0, 0,
			0, 1, 0, 2, 0, 1, 0, 0,
			0, 1, 4, 0, 0, 1, 0, 0,
			0, 1, 1, 1, 1, 1, 0, 0,
		},
		// ===== 第 8 关：3 个箱子，L 形 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 0, 0, 1, 0,
			0, 1, 0, 2, 0, 4, 1, 0,
			0, 1, 0, 5, 2, 0, 1, 0,
			0, 1, 0, 2, 0, 4, 1, 0,
			0, 1, 0, 0, 0, 0, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 9 关：4 个箱子，田字 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 4, 0, 1, 0,
			0, 1, 0, 2, 0, 2, 1, 0,
			0, 1, 0, 5, 0, 0, 1, 0,
			0, 1, 0, 2, 0, 2, 1, 0,
			0, 1, 4, 0, 4, 0, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 10 关：4 个箱子，四角 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 0, 4, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 0, 5, 0, 0, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 4, 0, 0, 4, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 11 关：4 个箱子，中间障碍 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 0, 4, 1, 0,
			0, 1, 0, 1, 1, 0, 1, 0,
			0, 1, 2, 0, 5, 2, 1, 0,
			0, 1, 0, 1, 1, 0, 1, 0,
			0, 1, 4, 0, 0, 4, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 12 关：4 个箱子，十字 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 0, 4, 0, 0, 1, 0,
			0, 1, 4, 2, 2, 4, 1, 0,
			0, 1, 0, 5, 0, 0, 1, 0,
			0, 1, 4, 2, 2, 4, 1, 0,
			0, 1, 0, 4, 0, 0, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 13 关：4 个箱子，窄通道 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 0, 4, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 0, 5, 0, 0, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 4, 0, 0, 4, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 14 关：5 个箱子 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 0, 4, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 4, 5, 0, 4, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 4, 0, 0, 0, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 15 关：4 个箱子，回字 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 4, 4, 4, 1, 0,
			0, 1, 0, 1, 1, 0, 1, 0,
			0, 1, 2, 0, 5, 2, 1, 0,
			0, 1, 0, 1, 1, 0, 1, 0,
			0, 1, 2, 0, 0, 2, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 16 关：5 个箱子 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 4, 0, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 4, 5, 0, 4, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 4, 0, 0, 0, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 17 关：5 个箱子，复杂 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 4, 0, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 4, 5, 2, 4, 1, 0,
			0, 1, 0, 2, 0, 0, 1, 0,
			0, 1, 4, 0, 0, 0, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 18 关：5 个箱子，迷宫 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 4, 4, 1, 0,
			0, 1, 0, 1, 0, 0, 1, 0,
			0, 1, 2, 5, 2, 2, 1, 0,
			0, 1, 0, 1, 0, 0, 1, 0,
			0, 1, 4, 0, 4, 0, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 19 关：6 个箱子 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 4, 0, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 4, 5, 2, 4, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 4, 0, 4, 0, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
		// ===== 第 20 关：6 个箱子，终极 =====
		{
			0, 0, 0, 0, 0, 0, 0, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
			0, 1, 4, 0, 4, 0, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 4, 5, 2, 4, 1, 0,
			0, 1, 0, 2, 2, 0, 1, 0,
			0, 1, 4, 0, 4, 0, 1, 0,
			0, 1, 1, 1, 1, 1, 1, 0,
		},
	};
	int levelNum = sizeof(Map) / sizeof(Map[0]);
	loadResource();
	playBgm(_T("bgm.wav"));
	while (1) {
		int count = 0;
		drawMap(Map);
		FlushBatchDraw();
		// 定位角色
		int i, j;
		for ( i = 0; i <= ROW - 1; i++) {
			for (j = 0; j <= COL - 1; j++) {
				if (Map[level][i][j] == 5 || Map[level][i][j] == 9)break;
			}
			if (Map[level][i][j] == 5 || Map[level][i][j] == 9)break;
		}
		if (peekmessage(&msg, EX_KEY, true)) {
			if (msg.message == WM_KEYDOWN) {
				switch (tolower(msg.vkcode)) {
					case 'w':
					case 'W':
					case VK_UP:
						if (Map[level][i - 1][j] == 0 || Map[level][i - 1][j] == 4) {
							Map[level][i - 1][j] += 5;
							Map[level][i][j] -= 5;
							playSfx(_T("move.wav"));
						} else if (Map[level][i - 1][j] == 2 || Map[level][i - 1][j] == 6) {
							if (Map[level][i - 2][j] == 0 || Map[level][i - 2][j] == 4) {
								Map[level][i - 2][j] += 2;
								Map[level][i - 1][j] += 3;
								Map[level][i][j] -= 5;
								playSfx(_T("push.wav"));
							}
						}
						break;
					case 's':
					case 'S':
					case VK_DOWN:
						if (Map[level][i + 1][j] == 0 || Map[level][i + 1][j] == 4) {
							Map[level][i + 1][j] += 5;
							Map[level][i][j] -= 5;
							playSfx(_T("move.wav"));
						} else if (Map[level][i + 1][j] == 2 || Map[level][i + 1][j] == 6) {
							if (Map[level][i + 2][j] == 0 || Map[level][i +  2][j] == 4) {
								Map[level][i + 2][j] += 2;
								Map[level][i + 1][j] += 3;
								Map[level][i][j] -= 5;
								playSfx(_T("push.wav"));
							}
						}
						break;
					case 'a':
					case 'A':
					case VK_LEFT:
						if (Map[level][i][j - 1] == 0 || Map[level][i][j - 1] == 4) {
							Map[level][i][j - 1] += 5;
							Map[level][i][j] -= 5;
							playSfx(_T("move.wav"));
						} else if (Map[level][i][j - 1] == 2 || Map[level][i][j - 1] == 6) {
							if (Map[level][i][j - 2] == 0 || Map[level][i][j - 2] == 4) {
								Map[level][i][j - 2] += 2;
								Map[level][i][j - 1] += 3;
								Map[level][i][j] -= 5;
								playSfx(_T("push.wav"));
							}
						}
						break;
					case 'd':
					case 'D':
					case VK_RIGHT:
						if (Map[level][i][j + 1] == 0 || Map[level][i][j + 1] == 4) {
							Map[level][i][j + 1] += 5;
							Map[level][i][j] -= 5;
							playSfx(_T("move.wav"));
						} else if (Map[level][i][j + 1] == 2 || Map[level][i][j + 1] == 6) {
							if (Map[level][i][j + 2] == 0 || Map[level][i][j + 2] == 4) {
								Map[level][i][j + 2] += 2;
								Map[level][i][j + 1] += 3;
								Map[level][i][j] -= 5;
								playSfx(_T("push.wav"));
							}
						}
						break;
				}
			}
		}
		for ( int m = 0; m <= ROW - 1; m++) {
			for (int n = 0; n <= COL - 1; n++) {
				if (Map[level][m][n] == 6)count++;
			}
		}
		if (count == box_num[level]) {
			stopSound();
			playSfx(_T("win.wav"));
			drawMap(Map);
			FlushBatchDraw();
			if (level < levelNum - 1) {
				TCHAR buf[64];
				snprintf(buf, 64, "恭喜通过第 %d 关！\n点击确定进入第 %d 关。", level + 1, level + 2);
				MessageBox(hwnd, buf, _T("过关"), MB_OK);
				level++;
				playBgm(_T("bgm.wav"));
			} else {
				MessageBox(hwnd, _T("恭喜通关全部关卡！"), _T("胜利"), MB_OK);
				break;
			}
		}
	}
	EndBatchDraw();
	closegraph();
	return 0;
}
void loadResource() {
	if (!loading) {
		loadimage(&wall, _T("wall.png"), CELL, CELL);
		loadimage(&box, _T("box.png"), CELL, CELL);
		loadimage(&target, _T("target.png"), CELL, CELL);
		loadimage(&player, _T("player.png"), CELL, CELL);
		loadimage(&box_on_target, _T("box_on_target.png"), CELL, CELL);
		loadimage(&player_on_target, _T("player_on_target.png"), CELL, CELL);
		loading = 1;
	}
}
void drawMap(int Map[][ROW][COL]) {
	cleardevice();
	// 找有效边界
	int minR = ROW, maxR = -1, minC = COL, maxC = -1;
	for (int i = 0; i < ROW; i++) {
		for (int j = 0; j < COL; j++) {
			if (Map[level][i][j] != 0) {
				if (i < minR) minR = i;
				if (i > maxR) maxR = i;
				if (j < minC) minC = j;
				if (j > maxC) maxC = j;
			}
		}
	}
	int validW = (maxC - minC + 1) * CELL;
	int validH = (maxR - minR + 1) * CELL;
	int ox = (getwidth()  - validW) / 2 - minC * CELL;
	int oy = (getheight() - validH) / 2 - minR * CELL;

	for (int i = 0; i < ROW; i++) {
		for (int j = 0; j < COL; j++) {
			int x = j * CELL + ox;
			int y = i * CELL + oy;
			switch (Map[level][i][j]) {
				case 1:
					putimage(x, y, &wall);
					break;
				case 2:
					putimage(x, y, &box);
					break;
				case 4:
					putimage(x, y, &target);
					break;
				case 5:
					putimage(x, y, &player);
					break;
				case 6:
					putimage(x, y, &box_on_target);
					break;
				case 9:
					putimage(x, y, &player_on_target);
					break;
			}
		}
	}
}
void playSfx(const TCHAR* file) {
	PlaySound(file, NULL, SND_FILENAME | SND_ASYNC);
}
void playBgm(const TCHAR* file) {
	PlaySound(file, NULL, SND_FILENAME | SND_ASYNC | SND_LOOP);
}
void stopSound() {
	PlaySound(NULL, NULL, 0);
}
```
运行效果：

![screenshots](/imgs/C/2106185-20261007230108168-530404537.gif)

## 附录
### ASCII码表
![image](/imgs/C/2106185-20260816160941112-20882329.png)

![image](/imgs/C/2106185-20260816161007955-1104461223.png)
