---
title: CMake学习指南
date: 2026-10-21 20:08:53
tags:
- CMake
category:
- 构建工具
index_img: /imgs/CMake.png
excerpt: 摘要：CMake 是跨平台构建工具，通过 CMakeLists.txt 配置项目。文档涵盖常用命令如 `cmake_minimum_required`、`project`、`add_executable` 等，并逐步讲解构建基本项目、添加库、设置使用要求、生成器表达式、安装测试、系统自省、自定义命令、打包安装程序、选择静态或共享库、导出配置以及打包调试与发布版本，帮助开发者掌握 CMake 的完整构建流程。
---

> [CMake官方文档](https://cmake.org/cmake/help/latest/)

## 常用命令

`cmake_minimum_required`：设置项目所需要的cmake的最低版本

`project`：设置项目名及版本信息

`add_executable`：使用指定的源文件生成可执行文件

`set(CMAKE_CXX_STANDARD 14)`:指定 C++ 标准，这里是使用C++14

`set(CMAKE_CXX_STANDARD_REQUIRED True)`：指定是否`CXX_STANDARD`是必须的

`configure_file`:将文件复制到另一个位置并修改其内容

`target_include_directories`:指定编译给定目标时要使用的包含目录

`target_link_libraries`：指定链接给定目标或其依赖项时要使用的库或标志

## 1.构建一个基本项目

这一节的目标很简单：用 CMake 编译一个 C++ 程序，让它在没有参数时打印版本号和用法，在有参数时计算平方根。

整个项目的结构非常小，一个 CMakeLists.txt、一个 tutorial.cxx、一个 TutorialConfig.h.in。真正值得讲的是 configure_file 这一环。TutorialConfig.h.in 里写的是 #define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@，这里的 @...@ 是 CMake 的变量占位符。配置阶段，CMake 会把 project(Tutorial VERSION 1.0) 生成的 Tutorial_VERSION_MAJOR 和 Tutorial_VERSION_MINOR 替换进去，在构建目录里生成 TutorialConfig.h。也就是说，源码目录里只有模板，真正的头文件是配置时生成的。

这就带来一个关键点：生成的头文件在 `${PROJECT_BINARY_DIR}`，不在源码目录。所以必须用 `target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")`把它加入包含路径，否则 tutorial.cxx 里的 #include "TutorialConfig.h" 找不到文件。这一节其实在强调一个习惯：构建目录和源码目录是分离的，生成物应该放在构建目录，源码里只保留模板和源文件。

tutorial.cxx 本身没有太多 CMake 相关的内容，它用 std::stod 把命令行参数转成 double，再调用 std::sqrt。唯一和 CMake 有关的是它包含了 TutorialConfig.h，并使用里面定义的版本宏。这一节结束后，你已经能用一个最小的 CMake 项目完成配置、构建、运行三步。
### 具体代码：

#### CMakeLists.txt

```cmake
# 设置CMake的最小版本需要
cmake_minimum_required(VERSION 3.10)
# 设置项目名为Tutorial以及版本号为1.0
project(Tutorial VERSION 1.0)
# 设置C++标准为C++11
set(CMAKE_CXX_STANDARD 11)
# 设置C++标准为必需
set(CMAKE_CXX_STANDARD_REQUIRED True)
# 使用configure_file生成TutorialConfig.h头件，以此可以动态定义版本号等
configure_file(TutorialConfig.h.in TutorialConfig.h)
# 生成一个可执行程序Tutorial
add_executable(Tutorial tutorial.cxx)
# 指定编译给定目标时要使用的包含目录为build目录
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
```

#### tutorial.cxx

```c++
#include <cmath>
#include <iostream>
#include <string>
#include "TutorialConfig.h"
int main(int argc, char* argv[])
{
    if (argc < 2) {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."<< Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

## 2.添加库
上一节的平方根直接写在主程序里，这一节开始把它抽成一个独立的库 MathFunctions，主程序通过链接这个库来使用它。

项目结构因此变成两层：顶层 CMakeLists.txt 负责主程序，`MathFunctions/CMakeLists.txt` 负责库。顶层用 `add_subdirectory(MathFunctions)` 把子目录纳入构建，子目录里用 `add_library(MathFunctions MathFunctions.cxx)` 生成库目标。然后在顶层用 `target_link_libraries(Tutorial PUBLIC MathFunctions)` 把库链接给主程序，再用 `target_include_directories 把 MathFunctions` 目录加入包含路径，这样 tutorial.cxx 才能 #include "MathFunctions.h"。

这一节还引入了一个可配置开关 `option(USE_MYMATH "..." ON)`。它的作用是让用户在配置时决定用自己实现的 mysqrt，还是用标准库的 std::sqrt。如果 USE_MYMATH 为 ON，CMake 会通过 `target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")` 给库加一个编译宏，MathFunctions.cxx 里的 #ifdef USE_MYMATH 才会生效。同时再建一个静态库 SqrtLibrary 放 mysqrt.cxx，并把它链接进 MathFunctions。用户可以在配置时用 `cmake -DUSE_MYMATH=OFF ..` 关掉这个开关。

这一节真正想传达的是"库的目标化"。不要用全局的 `include_directories` 和 `link_libraries`，而是用 target_* 命令把属性绑定到具体目标上。这样每个目标的依赖关系是清晰的，不会互相污染。
### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.10)
project(Tutorial VERSION 1.0)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)
configure_file(TutorialConfig.h.in TutorialConfig.h)
# 添加MtahFucntions子目录到Tutorial中
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
# 添加MathFucntions的库到Tutorial中
target_link_libraries(Tutorial PUBLIC MathFunctions)
# 将MathFunctions目录包含到Tutorial中
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           "${PROJECT_SOURCE_DIR}/MathFunctions"
                           )
```

#### tutorial.cxx

```c++
#include <cmath>
#include <iostream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/CMakeLists.txt

```cmake
# 添加一个库，使用MathFunctions.cxx生成MathFucntions库
add_library(MathFunctions MathFunctions.cxx)
# 创建一个USE_MYMATH变量并设置默认值为ON。可以在cmake中使用-D命令进行指定
option(USE_MYMATH "Use tutorial provided math implementation" ON)
# 如果USE_MYMATH属性为ON, 使用target_compile_definitions来进行传递
if (USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  add_library(SqrtLibrary STATIC mysqrt.cxx)
  target_link_libraries(MathFunctions PUBLIC SqrtLibrary)
endif()
```

#### MathFunctions/mysqrt.h

```c++
#pragma once
namespace mathfunctions
{
    namespace detail
    {
        double mysqrt(double x);
    }
} // namespace mathfunctions
```

#### MathFunctions/mysqrt.cxx

```c++
#include "mysqrt.h"
#include <iostream>
namespace mathfunctions {
namespace detail {
double mysqrt(double x) {
  if (x <= 0) {
    return 0;
  }
  double result = x;
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }
  return result;
}
} // namespace detail
} // namespace mathfunctions
```

#### MathFunctions/MathFunctions.h

```c++
#pragma once
namespace mathfunctions {
double sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#  include "mysqrt.h"
#endif
namespace mathfunctions {
double sqrt(double x)
{
#ifdef USE_MYMATH
  return detail::mysqrt(x);
#else
  return std::sqrt(x);
#endif
}
}
```

Tips: 编译命令`cmake -G "MinGW Makefiles" -DUSE_MYMATH=OFF ..`

## 3.添加库的使用要求
这一节讲的是 CMake 里一个很重要的概念——使用要求，也就是 usage requirements。核心是 `INTERFACE`、`PUBLIC`、`PRIVATE` 这三个关键字。

先要把它们的意思弄清楚。**PRIVATE 表示这个属性只影响自己，不传播给依赖方**；*INTERFACE 表示自己不用，只传播给依赖方*；***PUBLIC 则是自己用，也传播给依赖方***。这三个关键字决定了包含目录、链接库、编译选项这些东西会不会沿着依赖链传递下去。

本节先创建了一个接口库 `tutorial_compiler_flags`，用 `add_library(... INTERFACE)` 生成，再用 `target_compile_features(... INTERFACE cxx_std_11) `给它加上 C++ 标准要求。接口库不产生实际的二进制文件，它的作用就是携带"要求"，任何链接它的目标都会自动获得这些要求。

然后在` MathFunctions/CMakeLists.txt` 里，`target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})` 用的是 INTERFACE 而不是 PUBLIC。原因是 MathFunctions.cxx 自己不需要额外包含自己的目录，但链接 MathFunctions 的人需要找到 MathFunctions.h。这正是使用要求的典型场景：我不用，但用我的人需要。

这一节的关键收获是，不要到处写全局的 `include_directories`，而是用 `target_include_directories` 配合 INTERFACE、PUBLIC、PRIVATE 精确控制传播范围。这样项目的依赖关系才是可控的。
### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.10)
project(Tutorial VERSION 1.0)
# 创建一个interface库
add_library(tutorial_compiler_flags INTERFACE)
# 指定cxx_std_11为目标编译器功能
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
# 将Tutorial和MathFunctions链接到tutorial_compiler_flags
target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```

#### tutorial.cxx

```c++
#include <cmath>
#include <iostream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}

```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions MathFunctions.cxx)
# 设置只有链接到MathFucntions库的需要包含MathFunctions源目录，而其自身则不需要
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if (USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  add_library(SqrtLibrary STATIC mysqrt.cxx)
  target_link_libraries(MathFunctions PUBLIC SqrtLibrary tutorial_compiler_flags)
endif()
```

#### MathFunctions/mysqrt.h

```c++
#pragma once
namespace mathfunctions {
namespace detail {
double mysqrt(double x);
}
}
```

#### MathFunctions/mysqrt.cxx

```c++
#include "mysqrt.h"
#include <iostream>
namespace mathfunctions {
namespace detail {
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }
  double result = x;
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }
  return result;
}
}
}
```

#### MathFunctions/MathFunctions.h

```c++
#pragma once
namespace mathfunctions {
double sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#  include "mysqrt.h"
#endif
namespace mathfunctions {
double sqrt(double x)
{
#ifdef USE_MYMATH
  return detail::mysqrt(x);
#else
  return std::sqrt(x);
#endif
}
}
```

## 4.添加生成器表达式
这一节引入生成器表达式，用来根据编译器、语言、构建类型等条件动态生成编译选项。

代码里先定义了两个变量，`gcc_like_cxx` 和`msvc_cxx`，它们分别对应"编译器是 GCC 系"和"编译器是 MSVC"这两个条件。`$<COMPILE_LANG_AND_ID:CXX,MSVC> `这种写法会在生成构建系统时求值，判断当前编译语言是不是 CXX、编译器 ID 是不是 MSVC。然后 `target_compile_options` 里用 $<条件:选项> 的形式，让不同编译器拿到不同的警告选项：GCC 系用` -Wall -Wextra -Wshadow -Wformat=2 -Wunused`，MSVC 用` /W3`。外面再套一层 `$<BUILD_INTERFACE:...>`，表示这些选项只在构建树里生效，不导出给安装后的使用者。

这样写的好处是不需要写一堆 `if(MSVC)`、`if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")` 这样的分支，生成器表达式本身就能在生成阶段拿到足够的上下文信息，条件判断更精确，也更适合被复用。

这一节还把 `USE_MYMATH`的默认值从 ON 改成了 OFF，说明这个开关是可以随时调整的，用户可以通过 `-DUSE_MYMATH=ON`打开。生成器表达式和前面的接口库配合起来，就能做到"不同编译器、不同构建类型、不同使用场景下，自动应用不同的编译选项"。
### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(Tutorial VERSION 1.0)
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```

#### tutorial.cxx

```c++
#include <cmath>
#include <iostream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}

```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions MathFunctions.cxx)
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
option(USE_MYMATH "Use tutorial provided math implementation" OFF)
if (USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  add_library(SqrtLibrary STATIC mysqrt.cxx)
  target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
  target_link_libraries(MathFunctions PUBLIC SqrtLibrary)
endif()
target_link_libraries(MathFunctions PUBLIC tutorial_compiler_flags)
```

#### MathFunctions/mysqrt.h

```c++
#pragma once
namespace mathfunctions {
namespace detail {
double mysqrt(double x);
}
}
```

#### MathFunctions/mysqrt.cxx

```c++
#include "mysqrt.h"
#include <iostream>
namespace mathfunctions {
namespace detail {
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }
  double result = x;
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }
  return result;
}
}
}
```

#### MathFunctions/MathFunctions.h

```c++
#pragma once
namespace mathfunctions {
double sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#  include "mysqrt.h"
#endif
namespace mathfunctions {
double sqrt(double x)
{
#ifdef USE_MYMATH
  return detail::mysqrt(x);
#else
  return std::sqrt(x);
#endif
}
}
```

## 5.安装和测试
这一节分成两块：安装规则和测试。

安装规则用 **install** 命令。`install(TARGETS Tutorial DESTINATION bin)` 把可执行文件装到 bin 目录，`install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h" DESTINATION include) `把生成的头文件装到 include 目录。在 `MathFunctions/CMakeLists.txt` 里，先把要安装的库收集到一个列表 `installable_libs` 里，再用 `if(TARGET SqrtLibrary)` 判断 SqrtLibrary 是否存在，存在就追加进去，最后统一 `install(TARGETS ${installable_libs} DESTINATION lib)`，把头文件 MathFunctions.h 和 mysqrt.h 装到 include。这里的 DESTINATION 都是相对于 CMAKE_INSTALL_PREFIX 的路径，安装时可以用 `cmake --install . --prefix /some/prefix` 指定前缀。

测试部分用 **enable_testing()** 开启测试功能，再用 add_test 定义测试。`add_test(NAME Runs COMMAND Tutorial 25)` 表示运行 Tutorial 25 这个命令，`set_tests_properties(Usage PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")` 则用正则表达式判断输出是否匹配，匹配就算通过。后面还定义了一个 do_test 函数，把重复的测试定义封装起来，避免写一大堆样板代码。运行测试用 ctest，加 `--output-on-failure` 可以在失败时看到输出。

这一节想说明的是，CMake 不只是构建工具，它还能管理安装和测试。ctest 是 CMake 自带的测试驱动，和 add_test 配合使用，能在本地或 CI 里统一跑测试。

### 安装

#### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(Tutorial VERSION 1.0)
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
# 将Tutorial可执行程序安装到bin目录下
install(TARGETS Tutorial DESTINATION bin)
# 将TutorialConfig.h安装到include文件夹下
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h" DESTINATION include)
```

#### tutorial.cxx

```c++
#include <cmath>
#include <iostream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}

```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions MathFunctions.cxx)
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if (USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  add_library(SqrtLibrary STATIC mysqrt.cxx)
  target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()
target_link_libraries(MathFunctions PUBLIC tutorial_compiler_flags)
# 指定所要安装的lib
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
# 将库文件安装到lib文件夹下
install(TARGETS ${installable_libs} DESTINATION lib)
# 将头文件安装到include文件夹下
install(FILES MathFunctions.h mysqrt.h  DESTINATION include)
```

#### MathFunctions/mysqrt.h

```c++
#pragma once
namespace mathfunctions {
namespace detail {
double mysqrt(double x);
}
}
```

#### MathFunctions/mysqrt.cxx

```c++
#include "mysqrt.h"
#include <iostream>
namespace mathfunctions {
namespace detail {
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }
  double result = x;
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }
  return result;
}
}
}
```

#### MathFunctions/MathFunctions.h

```c++
#pragma once
namespace mathfunctions {
double sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#  include "mysqrt.h"
#endif
namespace mathfunctions {
double sqrt(double x)
{
#ifdef USE_MYMATH
  return detail::mysqrt(x);
#else
  return std::sqrt(x);
#endif
}
}
```

### 测试

#### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(Tutorial VERSION 1.0)
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
# 将Tutorial可执行程序安装到bin目录下
install(TARGETS Tutorial DESTINATION bin)
# 将TutorialConfig.h安装到include文件夹下
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h" DESTINATION include)
# ========================================================================
# 开启测试功能
enable_testing()
# 添加一个叫做Runs的测试:
# $ Tutorial 25
add_test(NAME Runs COMMAND Tutorial 25)
# 添加一个叫做Usage的测试并设置测试属性:
# $ Tutorial
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )
# 添加一个叫做StandardUse的测试并设置测试属性:
# $ Tutorial 4
add_test(NAME StandardUse COMMAND Tutorial 4)
set_tests_properties(StandardUse
  PROPERTIES PASS_REGULAR_EXPRESSION "4 is 2"
  )
# 添加一个do_test测试函数
function(do_test target arg result)
  add_test(NAME test_${arg} COMMAND ${target} ${arg})
  set_tests_properties(test_${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction()
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is (-nan|nan|0)")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

#### tutorial.cxx

```c++
#include <cmath>
#include <iostream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char* argv[])
{
  if (argc < 2) {
    // report version
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions MathFunctions.cxx)
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if (USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  add_library(SqrtLibrary STATIC mysqrt.cxx)
  target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()
target_link_libraries(MathFunctions PUBLIC tutorial_compiler_flags)
# 指定所要安装的lib
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
# 将库文件安装到lib文件夹下
install(TARGETS ${installable_libs} DESTINATION lib)
# 将头文件安装到include文件夹下
install(FILES MathFunctions.h mysqrt.h  DESTINATION include)
```

#### MathFunctions/mysqrt.h

```c++
#pragma once
namespace mathfunctions {
namespace detail {
double mysqrt(double x);
}
}
```

#### MathFunctions/mysqrt.cxx

```c++
#include "mysqrt.h"
#include <iostream>
namespace mathfunctions {
namespace detail {
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }
  double result = x;
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }
  return result;
}
}
}
```

#### MathFunctions/MathFunctions.h

```c++
#pragma once
namespace mathfunctions {
double sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#  include "mysqrt.h"
#endif
namespace mathfunctions {
double sqrt(double x)
{
#ifdef USE_MYMATH
  return detail::mysqrt(x);
#else
  return std::sqrt(x);
#endif
}
}
```

## 6.添加对测试仪表板的支持
这一节把测试结果提交到 CDash，一个开源的测试仪表板。做法是把 `enable_testing()` 换成 `include(CTest)`，后者会自动调用 enable_testing()，同时引入 CDash 相关支持。然后添加一个 `CTestConfig.cmake`，里面设置项目名、夜间构建开始时间、CDash 的地址和提交路径。这些变量告诉 CTest：项目叫什么、往哪个 CDash 实例提交、提交到哪个路径。配置好之后，运行 `ctest -D Experimental` 就会执行测试并把结果上传到 CDash。

这一节的实际意义在于，团队或 CI 环境下，测试结果需要集中展示，CDash 就是 CMake 生态里做这件事的工具。个人学习时如果不想提交，可以跳过这一步，但要知道 include(CTest) 是比 enable_testing() 更现代的做法，因为它把测试和仪表板支持一起引入了。
### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(Tutorial VERSION 1.0)
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
# 更改为include(CTest)
include(CTest)
add_test(NAME Runs COMMAND Tutorial 25)
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction()
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is (-nan|nan|0)")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

#### CTestConfig.cmake

```c++
# 项目名称
set(CTEST_PROJECT_NAME "CMakeTutorial")
# 每晚项目开始时间
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")
# 将发送提交的生成文档的 CDash 实例的 URL
set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```

#### tutorial.cxx

```c++
#include <cmath>
#include <iostream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions MathFunctions.cxx)
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if (USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  add_library(SqrtLibrary STATIC mysqrt.cxx)
  target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()
target_link_libraries(MathFunctions PUBLIC tutorial_compiler_flags)
# 指定所要安装的lib
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
# 将库文件安装到lib文件夹下
install(TARGETS ${installable_libs} DESTINATION lib)
# 将头文件安装到include文件夹下
install(FILES MathFunctions.h mysqrt.h  DESTINATION include)
```

#### MathFunctions/mysqrt.h

```c++
#pragma once
namespace mathfunctions {
namespace detail {
double mysqrt(double x);
}
}
```

#### MathFunctions/mysqrt.cxx

```c++
#include "mysqrt.h"
#include <iostream>
namespace mathfunctions {
namespace detail {
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }
  double result = x;
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }
  return result;
}
}
}
```

#### MathFunctions/MathFunctions.h

```c++
#pragma once
namespace mathfunctions {
double sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#  include "mysqrt.h"
#endif
namespace mathfunctions {
double sqrt(double x)
{
#ifdef USE_MYMATH
  return detail::mysqrt(x);
#else
  return std::sqrt(x);
#endif
}
}
```

> Tips:运行`ctest -D Experimental`进行提交，提交成功后可访问[Test Dashboard网站](https://my.cdash.org/submit.php?project=CMakeTutorial)

## 7.添加系统自省
这一节讲系统自省，也就是在配置阶段检查系统是否支持某些函数或头文件，而不是等到编译时才报错。

关键在 `MathFunctions/CMakeLists.txt` 里引入 `CheckCXXSourceCompiles` 模块，然后用 `check_cxx_source_compiles` 尝试编译一小段代码。比如检查 std::log 是否可用，就写一段包含 <cmath> 并调用 std::log(1.0) 的代码；检查 std::exp 同理。如果编译成功，对应的变量 HAVE_LOG、HAVE_EXP 就会被设为真。接着用 `target_compile_definitions(SqrtLibrary PRIVATE "HAVE_LOG" "HAVE_EXP")` 把这些宏传给 SqrtLibrary。

在 mysqrt.cxx 里，就可以用 `#if defined(HAVE_LOG) && defined(HAVE_EXP)` 来判断：如果两个函数都可用，就用 std::exp(std::log(x) * 0.5) 计算平方根；否则退回牛顿迭代。这样代码就能根据系统能力选择不同实现，而不是硬编码平台判断。

这一节的重点是，CMake 可以在配置阶段做能力探测。类似模块还有 `CheckIncludeFile、CheckFunctionExists、CheckCXXCompilerFlag` 等，思路都是一样的：写一小段测试代码，尝试编译，根据结果设置变量，再用变量控制宏或选项。

### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(Tutorial VERSION 1.0)
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
# 更改为include(CTest)
include(CTest)
add_test(NAME Runs COMMAND Tutorial 25)
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction()
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is (-nan|nan|0)")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

#### CTestConfig.cmake

```c++
# 项目名称
set(CTEST_PROJECT_NAME "CMakeTutorial")
# 每晚项目开始时间
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")
# 将发送提交的生成文档的 CDash 实例的 URL
set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```

#### tutorial.cxx

```c++
#include <cmath>
#include <iostream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions MathFunctions.cxx)
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if (USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  add_library(SqrtLibrary STATIC mysqrt.cxx)
  target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
  # 包含CheckCXXSourceCompiles模块,这个函数让我们在真正的源代码编译之前尝试编译具有所需依赖项的简单代码
  include(CheckCXXSourceCompiles)
  # 使用CheckCXXSourceCompiles模块检查log和exp函数是否可用
  check_cxx_source_compiles("
    #include <cmath>
    int main() {
      std::log(1.0);
      return 0;
    }
  " HAVE_LOG)
  check_cxx_source_compiles("
    #include <cmath>
    int main() {
      std::exp(1.0);
      return 0;
    }
  " HAVE_EXP)
  # 如果log和exp都可用，使用target_compile_definitions()指定 HAVE_LOG和HAVE_EXP作为PRIVATE编译定义。
  if(HAVE_LOG AND HAVE_EXP)
    target_compile_definitions(SqrtLibrary PRIVATE "HAVE_LOG" "HAVE_EXP")
  endif()
  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()
target_link_libraries(MathFunctions PUBLIC tutorial_compiler_flags)
# 指定所要安装的lib
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
# 将库文件安装到lib文件夹下
install(TARGETS ${installable_libs} DESTINATION lib)
# 将头文件安装到include文件夹下
install(FILES MathFunctions.h mysqrt.h  DESTINATION include)
```

#### MathFunctions/mysqrt.h

```c++
#pragma once
namespace mathfunctions {
namespace detail {
double mysqrt(double x);
}
}
```

#### MathFunctions/mysqrt.cxx

```c++
#include "mysqrt.h"
#include <cmath>
#include <iostream>
namespace mathfunctions
{
  namespace detail
  {
    double mysqrt(double x)
    {
      if (x <= 0)
      {
        return 0;
      }
// 如果HAVE_LOG和HAVE_EXP已经被定义了
#if defined(HAVE_LOG) && defined(HAVE_EXP)
      double result = std::exp(std::log(x) * 0.5);
      std::cout << "Computing sqrt of " << x << " to be " << result
                << " using log and exp" << std::endl;
#else
      double result = x;
      for (int i = 0; i < 10; ++i)
      {
        if (result <= 0)
        {
          result = 0.1;
        }
        double delta = x - (result * result);
        result = result + 0.5 * delta / result;
        std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
      }
#endif
      return result;
    }
  }
}

```

#### MathFunctions/MathFunctions.h

```c++
#pragma once
namespace mathfunctions {
double sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#  include "mysqrt.h"
#endif
namespace mathfunctions {
double sqrt(double x)
{
#ifdef USE_MYMATH
  return detail::mysqrt(x);
#else
  return std::sqrt(x);
#endif
}
}
```

## 8.添加自定义命令和生成的文件
这一节讲如何用 CMake 在构建过程中生成文件。核心是 `MathFunctions/MakeTable.cmake`，里面先 `add_executable(MakeTable MakeTable.cxx)` 生成一个小工具，然后用 `add_custom_command` 定义一条自定义命令：输出是 ${CMAKE_CURRENT_BINARY_DIR}/Table.h，命令是运行 MakeTable 生成它，依赖是 MakeTable 目标。这样 CMake 就知道，要生成 Table.h，得先编译 MakeTable，再运行它。

然后在 `MathFunctions/CMakeLists.txt` 里 `include(MakeTable.cmake)`，把 Table.h 列为 SqrtLibrary 的源文件之一。虽然它是头文件，但这样写能让 CMake 知道 SqrtLibrary 依赖 Table.h，而 Table.h 是由自定义命令生成的。再用 `target_include_directories(SqrtLibrary PRIVATE ${CMAKE_CURRENT_BINARY_DIR})` 让 mysqrt.cxx 能找到生成的 Table.h。mysqrt.cxx 里就可以 #include "Table.h"，并在 x 落在 [1, 10) 区间时用 sqrtTable 查表得到初始值，再继续牛顿迭代。

这一节的重点是，CMake 可以在构建时生成源代码或头文件。add_custom_command 负责生成文件，add_custom_target 可以定义更顶层的自定义目标。生成的文件通常放在构建目录，不要污染源码目录。
### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(Tutorial VERSION 1.0)
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
include(CTest)
add_test(NAME Runs COMMAND Tutorial 25)
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction()
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is (-nan|nan|0)")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

#### CTestConfig.cmake

```c++
# 项目名称
set(CTEST_PROJECT_NAME "CMakeTutorial")
# 每晚项目开始时间
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")
# 将发送提交的生成文档的 CDash 实例的 URL
set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```

#### tutorial.cxx

```c++
#include <cmath>
#include <iostream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/MakeTable.cmake

```cmake
add_executable(MakeTable MakeTable.cxx)
target_link_libraries(MakeTable PRIVATE tutorial_compiler_flags)
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
)
```

#### MathFunctions/MakeTable.cxx

```c++
#include <cmath>
#include <fstream>
#include <iostream>
int main(int argc, char *argv[])
{
  if (argc < 2)
  {
    return 1;
  }
  std::ofstream fout(argv[1], std::ios_base::out);
  const bool fileOpen = fout.is_open();
  if (fileOpen)
  {
    fout << "double sqrtTable[] = {" << std::endl;
    for (int i = 0; i < 10; ++i)
    {
      fout << sqrt(static_cast<double>(i)) << "," << std::endl;
    }
    fout << "0};" << std::endl;
    fout.close();
  }
  return fileOpen ? 0 : 1;
}
```

#### MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions MathFunctions.cxx)
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if (USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  # 导入MakeTable.cmake模块
  include(MakeTable.cmake)
  # 生成SqrtLibrary静态库
  add_library(SqrtLibrary STATIC mysqrt.cxx ${CMAKE_CURRENT_BINARY_DIR}/Table.h)
  target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
# 添加生成的Table.h头文件到包含目录
target_include_directories(SqrtLibrary PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(MathFunctions PUBLIC tutorial_compiler_flags)
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs} DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

#### MathFunctions/mysqrt.h

```c++
#pragma once
namespace mathfunctions {
namespace detail {
double mysqrt(double x);
}
}
```

#### MathFunctions/mysqrt.cxx

```c++
#include "mysqrt.h"
#include <iostream>
// 导入Table.h头文件
#include "Table.h"
namespace mathfunctions
{
  namespace detail
  {
    double mysqrt(double x)
    {
      if (x <= 0)
      {
        return 0;
      }
      double result = x;
      if (x >= 1 && x < 10)
      {
        std::cout << "Use the table to help find an initial value " << std::endl;
        result = sqrtTable[static_cast<int>(x)];
      }
      for (int i = 0; i < 10; ++i)
      {
        if (result <= 0)
        {
          result = 0.1;
        }
        double delta = x - (result * result);
        result = result + 0.5 * delta / result;
        std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
      }
      return result;
    }
  }
}
```

#### MathFunctions/MathFunctions.h

```c++
#pragma once
namespace mathfunctions {
double sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#  include "mysqrt.h"
#endif
namespace mathfunctions {
double sqrt(double x)
{
#ifdef USE_MYMATH
  return detail::mysqrt(x);
#else
  return std::sqrt(x);
#endif
}
}
```

## 9.打包安装程序

> Windows需要安装[NSIS](https://prdownloads.sourceforge.net/nsis/nsis-3.09-setup.exe?download)

这一节讲如何用 CPack 生成安装包。在顶层 CMakeLists.txt 里引入 `InstallRequiredSystemLibraries`，它会自动把运行所需的系统库加入安装包，Windows 上尤其常见。然后设置 `CPACK_RESOURCE_FILE_LICENSE` 指定许可证文件，`CPACK_PACKAGE_VERSION_MAJOR` 和 `CPACK_PACKAGE_VERSION_MINOR` 指定包版本，`CPACK_SOURCE_GENERATOR` 设为 TGZ 表示源码包格式，最后 include(CPack) 启用 CPack。启用之后，运行 cpack 就会按照安装规则生成安装包，也可以用 -G 指定生成器，比如 cpack -G NSIS 在 Windows 上生成 NSIS 安装程序，`cpack -G TGZ` 生成压缩包。Windows 上要生成 NSIS 安装程序，需要先安装 NSIS。

这一节的重点是，CMake 不只构建，还能打包。install 规则定义"装什么"，CPack 定义"打成什么格式"。两者配合，就能从同一份 CMake 配置里生成可分发的安装包。

### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(Tutorial VERSION 1.0)
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
include(CTest)
add_test(NAME Runs COMMAND Tutorial 25)
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction()
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is (-nan|nan|0)")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
# 以下为安装程序
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
set(CPACK_SOURCE_GENERATOR "TGZ")
include(CPack)
```

#### CTestConfig.cmake

```cmake
# 项目名称
set(CTEST_PROJECT_NAME "CMakeTutorial")
# 每晚项目开始时间
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")
# 将发送提交的生成文档的 CDash 实例的 URL
set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```

#### License.txt

```text
This is the open source License.txt file introduced in
CMake/Tutorial/Step9...
```

#### tutorial.cxx

```cmake
#include <cmath>
#include <iostream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```

#### TutorialConfig.h.in

```cmake
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/MakeTable.cmake

```cmake
add_executable(MakeTable MakeTable.cxx)
target_link_libraries(MakeTable PRIVATE tutorial_compiler_flags)
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
)
```

#### MathFunctions/MakeTable.cxx

```c++
#include <cmath>
#include <fstream>
#include <iostream>
int main(int argc, char *argv[])
{
  if (argc < 2)
  {
    return 1;
  }
  std::ofstream fout(argv[1], std::ios_base::out);
  const bool fileOpen = fout.is_open();
  if (fileOpen)
  {
    fout << "double sqrtTable[] = {" << std::endl;
    for (int i = 0; i < 10; ++i)
    {
      fout << sqrt(static_cast<double>(i)) << "," << std::endl;
    }
    fout << "0};" << std::endl;
    fout.close();
  }
  return fileOpen ? 0 : 1;
}
```

#### MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions MathFunctions.cxx)
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if (USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  # 导入MakeTable.cmake模块
  include(MakeTable.cmake)
  # 生成SqrtLibrary静态库
  add_library(SqrtLibrary STATIC mysqrt.cxx ${CMAKE_CURRENT_BINARY_DIR}/Table.h)
  target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
# 添加生成的Table.h头文件到包含目录
target_include_directories(SqrtLibrary PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(MathFunctions PUBLIC tutorial_compiler_flags)
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs} DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

#### MathFunctions/mysqrt.h

```c++
#pragma once
namespace mathfunctions {
namespace detail {
double mysqrt(double x);
}
}
```

#### MathFunctions/mysqrt.cxx

```c++
#include "mysqrt.h"
#include <iostream>
// 导入Table.h头文件
#include "Table.h"
namespace mathfunctions
{
  namespace detail
  {
    double mysqrt(double x)
    {
      if (x <= 0)
      {
        return 0;
      }
      double result = x;
      if (x >= 1 && x < 10)
      {
        std::cout << "Use the table to help find an initial value " << std::endl;
        result = sqrtTable[static_cast<int>(x)];
      }
      for (int i = 0; i < 10; ++i)
      {
        if (result <= 0)
        {
          result = 0.1;
        }
        double delta = x - (result * result);
        result = result + 0.5 * delta / result;
        std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
      }
      return result;
    }
  }
}
```

#### MathFunctions/MathFunctions.h

```c++
#pragma once
namespace mathfunctions {
double sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#  include "mysqrt.h"
#endif
namespace mathfunctions {
double sqrt(double x)
{
#ifdef USE_MYMATH
  return detail::mysqrt(x);
#else
  return std::sqrt(x);
#endif
}
}
```

## 10.选择静态或共享库
这一节讲如何控制生成静态库还是共享库。代码里先设置 `CMAKE_ARCHIVE_OUTPUT_DIRECTORY、CMAKE_LIBRARY_OUTPUT_DIRECTORY、CMAKE_RUNTIME_OUTPUT_DIRECTORY`，分别控制静态库、共享库、可执行文件和 Windows DLL 的输出目录。然后 option(BUILD_SHARED_LIBS "Build using shared libraries" ON) 决定 add_library 默认生成共享库还是静态库：ON 就是共享库，OFF 就是静态库。

在 Windows 上导出共享库需要处理符号导出，所以 MathFunctions.h 里用 #if defined(_WIN32) 加 `__declspec(dllexport)` 和 `__declspec(dllimport)`，并通过 `EXPORTING_MYMATH` 宏区分"编译库本身"和"使用库"两种情况。然后在 MathFunctions/CMakeLists.txt 里用 `target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")` 在编译库时定义这个宏。另外，共享库要求位置无关代码，所以 SqrtLibrary 用 `set_target_properties(... POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS})` 把 `POSITION_INDEPENDENT_CODE` 和 `BUILD_SHARED_LIBS` 绑定起来，生成共享库时开启，生成静态库时关闭。

这一节的重点是，静态库和共享库的差异不只是文件后缀，还涉及符号导出、位置无关代码、运行时路径等问题。CMake 提供了 BUILD_SHARED_LIBS 这个开关，让同一份 CMakeLists.txt 可以灵活切换两种模式。
### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(Tutorial VERSION 1.0)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
option(BUILD_SHARED_LIBS "Build using shared libraries" ON)
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
include(CTest)
add_test(NAME Runs COMMAND Tutorial 25)
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction()
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is (-nan|nan|0)")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
set(CPACK_SOURCE_GENERATOR "TGZ")
include(CPack)
```

#### CTestConfig.cmake

```cmake
set(CTEST_PROJECT_NAME "CMakeTutorial")
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")
set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```

#### License.txt

```text
This is the open source License.txt file introduced in
CMake/Tutorial/Step9...
```

#### tutorial.cxx

```c++
#include <iostream>
#include <sstream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char *argv[])
{
  if (argc < 2)
  {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/MakeTable.cmake

```cmake
add_executable(MakeTable MakeTable.cxx)
target_link_libraries(MakeTable PRIVATE tutorial_compiler_flags)
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  )
```

#### MathFunctions/MakeTable.cxx

```c++
#include <cmath>
#include <fstream>
#include <iostream>
int main(int argc, char *argv[])
{
  if (argc < 2)
  {
    return 1;
  }
  std::ofstream fout(argv[1], std::ios_base::out);
  const bool fileOpen = fout.is_open();
  if (fileOpen)
  {
    fout << "double sqrtTable[] = {" << std::endl;
    for (int i = 0; i < 10; ++i)
    {
      fout << sqrt(static_cast<double>(i)) << "," << std::endl;
    }
    fout << "0};" << std::endl;
    fout.close();
  }
  return fileOpen ? 0 : 1;
}
```

#### MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions MathFunctions.cxx)
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if(USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  include(MakeTable.cmake)
  add_library(SqrtLibrary STATIC mysqrt.cxx ${CMAKE_CURRENT_BINARY_DIR}/Table.h)
  target_include_directories(SqrtLibrary PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
  target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()
target_link_libraries(MathFunctions PUBLIC tutorial_compiler_flags)
target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs} DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

#### MathFunctions/mysqrt.h

```c++
namespace mathfunctions
{
    namespace detail
    {
        double mysqrt(double x);
    }
}

```

#### MathFunctions/mysqrt.cxx

```c++
#include <iostream>
#include "MathFunctions.h"
#include "Table.h"
namespace mathfunctions
{
  namespace detail
  {
    double mysqrt(double x)
    {
      if (x <= 0)
      {
        return 0;
      }
      double result = x;
      if (x >= 1 && x < 10)
      {
        std::cout << "Use the table to help find an initial value " << std::endl;
        result = sqrtTable[static_cast<int>(x)];
      }
      for (int i = 0; i < 10; ++i)
      {
        if (result <= 0)
        {
          result = 0.1;
        }
        double delta = x - (result * result);
        result = result + 0.5 * delta / result;
        std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
      }
      return result;
    }
  }
}
```

#### MathFunctions/MathFunctions.h

```c++
#if defined(_WIN32)
#if defined(EXPORTING_MYMATH)
#define DECLSPEC __declspec(dllexport)
#else
#define DECLSPEC __declspec(dllimport)
#endif
#else // non windows
#define DECLSPEC
#endif
namespace mathfunctions
{
    double DECLSPEC sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#include "mysqrt.h"
#endif
namespace mathfunctions
{
  double sqrt(double x)
  {
#ifdef USE_MYMATH
    return detail::mysqrt(x);
#else
    return std::sqrt(x);
#endif
  }
}
```

## 11.添加导出配置
这一节讲如何让别的 CMake 项目能通过 `find_package(MathFunctions) `使用你的库。核心是 `install(EXPORT ...)` 把目标导出成 `MathFunctionsTargets.cmake`，安装到 lib/cmake/MathFunctions。然后用 `configure_package_config_file` 根据 `Config.cmake.in`生成 `MathFunctionsConfig.cmake`，用 `write_basic_package_version_file` 生成版本文件，供 find_package 检查版本兼容性。最后用 install(FILES ...) 把这两个文件装到 lib/cmake/MathFunctions，再用 export(EXPORT ...) 在构建树里也导出一份，方便不安装直接使用。

Config.cmake.in 的内容很简单，就是 @PACKAGE_INIT@ 加上一行 include("${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake")。这样别的项目写 find_package(MathFunctions REQUIRED) 之后，就能直接 target_link_libraries(myapp PRIVATE MathFunctions)，CMake 会自动带上包含目录、链接库、编译选项等所有使用要求。

同时，MathFunctions/CMakeLists.txt 里的包含目录要改成构建接口和安装接口分开：`$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}> `和 `$<INSTALL_INTERFACE:include>`。这样在构建树里使用时包含源码目录，安装后使用时包含 include 目录。这一节的重点是现代 CMake 的包导出机制，它让库可以被其他项目以目标形式使用，而不是靠全局变量。
### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(Tutorial VERSION 1.0)
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
# 设置是否生成动态库
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
option(BUILD_SHARED_LIBS "Build using shared libraries" ON)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)
include(CMakePackageConfigHelpers)
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/example"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
    VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
    COMPATIBILITY AnyNewerVersion
)
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake
  DESTINATION lib/cmake/MathFunctions
)
export(EXPORT MathFunctionsTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake"
)
# 以下为Test
include(CTest)
add_test(NAME Runs COMMAND Tutorial 25)
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction()
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is (-nan|nan|0)")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
# 以下为安装包
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
set(CPACK_SOURCE_GENERATOR "TGZ")
include(CPack)
```

#### CTestConfig.cmake

```cmake
set(CTEST_PROJECT_NAME "CMakeTutorial")
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")
set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```

#### Config.cmake.in

```text
@PACKAGE_INIT@
include ( "${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake" )
```

#### License.txt

```text
This is the open source License.txt file introduced in
CMake/Tutorial/Step9...
```

#### tutorial.cxx

```c++
#include <iostream>
#include <sstream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char *argv[])
{
  if (argc < 2)
  {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/MakeTable.cmake

```cmake
add_executable(MakeTable MakeTable.cxx)
target_link_libraries(MakeTable PRIVATE tutorial_compiler_flags)
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  )
```

#### MathFunctions/MakeTable.cxx

```c++
#include <cmath>
#include <fstream>
#include <iostream>
int main(int argc, char *argv[])
{
  if (argc < 2)
  {
    return 1;
  }
  std::ofstream fout(argv[1], std::ios_base::out);
  const bool fileOpen = fout.is_open();
  if (fileOpen)
  {
    fout << "double sqrtTable[] = {" << std::endl;
    for (int i = 0; i < 10; ++i)
    {
      fout << sqrt(static_cast<double>(i)) << "," << std::endl;
    }
    fout << "0};" << std::endl;
    fout.close();
  }
  return fileOpen ? 0 : 1;
}
```

#### MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions MathFunctions.cxx)
target_include_directories(MathFunctions
                           INTERFACE
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                           $<INSTALL_INTERFACE:include>
                           )
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if(USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  include(MakeTable.cmake)
  add_library(SqrtLibrary STATIC mysqrt.cxx ${CMAKE_CURRENT_BINARY_DIR}/Table.h)
  target_include_directories(SqrtLibrary PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
  set_target_properties(SqrtLibrary PROPERTIES POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS})
  target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()
target_link_libraries(MathFunctions PUBLIC tutorial_compiler_flags)
target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs}
        EXPORT MathFunctionsTargets
        DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

#### MathFunctions/mysqrt.h

```c++
namespace mathfunctions
{
    namespace detail
    {
        double mysqrt(double x);
    }
}
```

#### MathFunctions/mysqrt.cxx

```c++
#include <iostream>
#include "MathFunctions.h"
#include "Table.h"
namespace mathfunctions
{
  namespace detail
  {
    double mysqrt(double x)
    {
      if (x <= 0)
      {
        return 0;
      }
      double result = x;
      if (x >= 1 && x < 10)
      {
        std::cout << "Use the table to help find an initial value " << std::endl;
        result = sqrtTable[static_cast<int>(x)];
      }
      for (int i = 0; i < 10; ++i)
      {
        if (result <= 0)
        {
          result = 0.1;
        }
        double delta = x - (result * result);
        result = result + 0.5 * delta / result;
        std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
      }
      return result;
    }
  }
}
```

#### MathFunctions/MathFunctions.h

```c++
#if defined(_WIN32)
#if defined(EXPORTING_MYMATH)
#define DECLSPEC __declspec(dllexport)
#else
#define DECLSPEC __declspec(dllimport)
#endif
#else // non windows
#define DECLSPEC
#endif
namespace mathfunctions
{
    double DECLSPEC sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#include "mysqrt.h"
#endif
namespace mathfunctions
{
  double sqrt(double x)
  {
#ifdef USE_MYMATH
    return detail::mysqrt(x);
#else
    return std::sqrt(x);
#endif
  }
}
```

## 12.打包调试和发布

这一节讲如何同时打包 `Debug` 和 `Release` 两个版本。首先用 `set(CMAKE_DEBUG_POSTFIX d)` 设置 Debug 版本的目标名后缀，再用 `set_target_properties(Tutorial PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})` 把它应用到 Tutorial 目标上。这样 Debug 版本的可执行文件叫 Tutoriald，Release 版本叫 Tutorial，两者的库文件名也不会冲突，可以放在同一个目录里。

然后是运行时路径的设置。if(APPLE) 下用 `@executable_path/../lib`，elseif(UNIX) 下用 $ORIGIN/../lib，分别对应 macOS 和 Linux 的运行时库搜索路径。这样安装后的可执行文件能找到同目录下的库。

打包两个版本靠的是 `MultiCPackConfig.cmake`。它先 include("release/CPackConfig.cmake")，然后用 `CPACK_INSTALL_CMAKE_PROJECTS` 指定同时打包 Debug 和 Release 两个构建目录。使用方式是先分别用 `-DCMAKE_BUILD_TYPE=Debug` 和 `-DCMAKE_BUILD_TYPE=Release` 配置并构建两个目录，分别运行 cpack，最后用 `cpack --config MultiCPackConfig.cmake` 合并打包。另外，`MathFunctions/CMakeLists.txt` 里用 `set_property(TARGET MathFunctions PROPERTY VERSION "1.0.0")` 和 `set_property(TARGET MathFunctions PROPERTY SOVERSION "1")` 设置库的完整版本号和 ABI 版本号，影响共享库的文件名，比如 libMathFunctions.so.1。

这一节的重点是，Debug 和 Release 可以共存，并且可以用 CPack 合并打包。同时共享库需要关注版本号和 RPATH，否则安装后可能找不到库。
### 具体代码：

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(Tutorial VERSION 1.0)
set(CMAKE_DEBUG_POSTFIX d)
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
option(BUILD_SHARED_LIBS "Build using shared libraries" ON)
if(APPLE)
  set(CMAKE_INSTALL_RPATH "@executable_path/../lib")
elseif(UNIX)
  set(CMAKE_INSTALL_RPATH "$ORIGIN/../lib")
endif()
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_subdirectory(MathFunctions)
add_executable(Tutorial tutorial.cxx)
set_target_properties(Tutorial PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})
target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
enable_testing()
add_test(NAME Runs COMMAND Tutorial 25)
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction()
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is (-nan|nan|0)")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include(CPack)
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)
include(CMakePackageConfigHelpers)
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/example"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion
)
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake
  DESTINATION lib/cmake/MathFunctions
  )
export(EXPORT MathFunctionsTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake"
)
```

#### CTestConfig.cmake

```cmake
set(CTEST_PROJECT_NAME "CMakeTutorial")
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")
set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```

#### Config.cmake.in

```text
@PACKAGE_INIT@
include ( "${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake" )
```

#### MultiCPackConfig.cmake

```cmake
include("release/CPackConfig.cmake")
set(CPACK_INSTALL_CMAKE_PROJECTS
    "debug;Tutorial;ALL;/"
    "release;Tutorial;ALL;/"
)
```

#### License.txt

```text
This is the open source License.txt file introduced in
CMake/Tutorial/Step9...
```

#### tutorial.cxx

```c++
#include <iostream>
#include <sstream>
#include <string>
#include "MathFunctions.h"
#include "TutorialConfig.h"
int main(int argc, char *argv[])
{
  if (argc < 2)
  {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  const double inputValue = std::stod(argv[1]);
  const double outputValue = mathfunctions::sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```

#### TutorialConfig.h.in

```text
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

#### MathFunctions/MakeTable.cmake

```cmake
add_executable(MakeTable MakeTable.cxx)
target_link_libraries(MakeTable PRIVATE tutorial_compiler_flags)
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  )
```

#### MathFunctions/MakeTable.cxx

```c++
#include <cmath>
#include <fstream>
#include <iostream>
int main(int argc, char *argv[])
{
  if (argc < 2)
  {
    return 1;
  }
  std::ofstream fout(argv[1], std::ios_base::out);
  const bool fileOpen = fout.is_open();
  if (fileOpen)
  {
    fout << "double sqrtTable[] = {" << std::endl;
    for (int i = 0; i < 10; ++i)
    {
      fout << sqrt(static_cast<double>(i)) << "," << std::endl;
    }
    fout << "0};" << std::endl;
    fout.close();
  }
  return fileOpen ? 0 : 1;
}
```

#### MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions MathFunctions.cxx)
set_property(TARGET MathFunctions PROPERTY VERSION "1.0.0")
set_property(TARGET MathFunctions PROPERTY SOVERSION "1")
target_include_directories(MathFunctions
                           INTERFACE
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                            $<INSTALL_INTERFACE:include>
                           )
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if(USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  include(MakeTable.cmake)
  add_library(SqrtLibrary STATIC
              mysqrt.cxx
              ${CMAKE_CURRENT_BINARY_DIR}/Table.h
              )
  target_include_directories(SqrtLibrary PRIVATE
                             ${CMAKE_CURRENT_BINARY_DIR}
                             )
  set_target_properties(SqrtLibrary PROPERTIES
                        POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS}
                        )
  target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()
target_link_libraries(MathFunctions PUBLIC tutorial_compiler_flags)
target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs}
        EXPORT MathFunctionsTargets
        DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

#### MathFunctions/mysqrt.h

```c++
namespace mathfunctions
{
    namespace detail
    {
        double mysqrt(double x);
    }
}
```

#### MathFunctions/mysqrt.cxx

```c++
#include <iostream>
#include "MathFunctions.h"
#include "Table.h"
namespace mathfunctions
{
  namespace detail
  {
    double mysqrt(double x)
    {
      if (x <= 0)
      {
        return 0;
      }
      double result = x;
      if (x >= 1 && x < 10)
      {
        std::cout << "Use the table to help find an initial value " << std::endl;
        result = sqrtTable[static_cast<int>(x)];
      }
      for (int i = 0; i < 10; ++i)
      {
        if (result <= 0)
        {
          result = 0.1;
        }
        double delta = x - (result * result);
        result = result + 0.5 * delta / result;
        std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
      }
      return result;
    }
  }
}
```

#### MathFunctions/MathFunctions.h

```c++
#if defined(_WIN32)
#if defined(EXPORTING_MYMATH)
#define DECLSPEC __declspec(dllexport)
#else
#define DECLSPEC __declspec(dllimport)
#endif
#else // non windows
#define DECLSPEC
#endif
namespace mathfunctions
{
    double DECLSPEC sqrt(double x);
}
```

#### MathFunctions/MathFunctions.cxx

```c++
#include "MathFunctions.h"
#include <cmath>
#ifdef USE_MYMATH
#include "mysqrt.h"
#endif
namespace mathfunctions
{
  double sqrt(double x)
  {
#ifdef USE_MYMATH
    return detail::mysqrt(x);
#else
    return std::sqrt(x);
#endif
  }
}
```

> Tips:
>
> Debug版本：`cmake -G "MinGW Makefiles" -DCMAKE_BUILD_TYPE=Debug ..`
>
> Release版本：`cmake -G "MinGW Makefiles" -DCMAKE_BUILD_TYPE=Release ..`
>
> 使用自定义配置文件将两个版本打包到一个版本中:`cpack --config MultiCPackConfig.cmake`

完整版代码可见Gitee仓库：[CMakeStudy](https://gitee.com/CodingWK/CMakeStudy)
