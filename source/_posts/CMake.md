---
title: CMake学习
date: 2023-08-28 11:10:33
tags:
- C++
categories: 
- C++
index_img: /imgs/CMake.png
---

# CMake学习

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

Tips: 编译命令`cmake -G "MinGW Makefiles" -DUSE_MYMATH=OFF ..`

## 3.添加库的使用要求

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

