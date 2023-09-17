---
title: CMU15-445
date: 2023-08-24 18:29:38
tags:
- Database
categories: 
- Database
index_img: /imgs/CMU.png
---

# CMU15-445环境搭建及实操

> 官网：https://15445.courses.cs.cmu.edu/spring2023/

## 前提

提前安装好WSL2，这里我使用的系统是Ubuntu 22.04 LTS版

CMU15-445必需的环境：

1. git
2. clang
3. gdb
4. cmake

```shell
sudo apt install git cmake clang gdb
```

### 配置CMU15-445

1. 创建一个Github私有库

2. 克隆公开的BusTub仓库

   ```shell
   $ git clone --bare https://github.com/cmu-db/bustub.git bustub-public
   ```

3. 将公共的BusTub存储库push到自己的私有BusTub存储库

   ```shell
   cd bustub-public
   git push 你的私有库的git仓库地址 master
   rm -rf bustub-public
   ```

4. 将私有存储库克隆到使用的开发计算机中

   ```shell
   git clone 你的私有库的git仓库地址
   ```

5. **从私有存储库的项目设置中禁用 GitHub Actions**，否则您可能会用完 GitHub Actions 配额

   ```shell
   Settings > Actions > General > Actions permissions > Disable actions.
   ```

### 编译CMU15-445

1. 为了确保您的计算机上有正确的软件包，请运行以下脚本来自动安装它们

   ```shell
   sudo build_support/packages.sh
   ```

2. 然后运行以下命令来构建系统

   ```shell
    mkdir build
    cd build
    cmake ..
    make
   ```

3. 如果要在调试模式下编译系统，请将以下标志传递给 cmake,默认情况下，这会启用[AddressSanitizer](https://github.com/google/sanitizers)。

   ```shell
   cmake -DCMAKE_BUILD_TYPE=Debug ..
   make -j`nproc`
   ```

4. 测试

   ```shell
   cd build
   make check-tests
   ```

## C++ Primer

> 详见[C++ Primer](https://15445.courses.cs.cmu.edu/spring2023/project0/)

## Buffer Pool Manager

> 详见[Buffer Pool Manager](https://15445.courses.cs.cmu.edu/spring2023/project1/)

## B+Tree Index

> 详见[B+Tree Index](https://15445.courses.cs.cmu.edu/spring2023/project2/)

## Query Execution

> 详见[Query Execution](https://15445.courses.cs.cmu.edu/spring2023/project3/)

## Concurrency Control

> 详见[Concurrency Control](https://15445.courses.cs.cmu.edu/spring2023/project4/)







