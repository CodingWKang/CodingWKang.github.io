---
title: Miniob环境搭建
date: 2023-08-22 15:37:24
tags:
- Database
categories: 
- Database
index_img: /imgs/miniob.png
---

# MiniOb环境搭建

> 本文简介MiniOb的环境搭建

前提条件：详见https://oceanbase.github.io/miniob/game/gitee-instructions.html

1. build libevent

   ```shell
   git submodule add https://github.com/libevent/libevent deps/libevent
   cd deps
   cd libevent
   git checkout release-2.1.12-stable
   mkdir build
   cd build
   cmake .. -DEVENT__DISABLE_OPENSSL=ON
   make
   sudo make install
   ```

2. build google_test

   ```shell
   git submodule add https://github.com/google/googletest deps/googletest
   cd deps
   cd googletest
   mkdir build
   cd build
   cmake ..
   make
   sudo make install
   ```

3. build jsoncpp

   ```shell
   git submodule add https://github.com/open-source-parsers/jsoncpp.git deps/jsoncpp
   cd deps
   cd jsoncpp
   mkdir build
   cd build
   cmake -DJSONCPP_WITH_TESTS=OFF -DJSONCPP_WITH_POST_BUILD_UNITTEST=OFF ..
   make
   sudo make install
   ```

4. build miniob

   ```shell
   cd `project home`
   mkdir build
   cd build
   # 建议开启DEBUG模式编译，更方便调试
   cmake .. -DDEBUG=ON
   make
   ```

   上述操作完成后进行git push即可进行提测。

