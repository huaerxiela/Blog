---
title: Windows 编译GDB
date: 2020-09-14 13:11:38
tags: Debug
category: Android
---


# 准备环境

1. 安装 MSYS2 

直接去 https://www.msys2.org/ 网站[下载](https://repo.msys2.org/distrib/x86_64/msys2-x86_64-20200903.exe)并安装 MSYS2 。


2. 安装 mingw64

下载 [mingw64](https://sourceforge.net/projects/mingw-w64/files/) 并解压到 `mingw64/mingw64` 目录。

3. 安装其他工具

```
pacman -S pactoys
# pacboy -S gcc:x python3:x  # mingw-w64-x86_64-gcc mingw-w64-x86_64-python3
pacman -S make texinfo bison git dejagnu 
```
如果后续编译报错，则根据报错信息安装相应的工具即可。

这里我使用的是[此网站](https://sourceforge.net/projects/mingw-w64/files/)下载的gdb。

# 开始编译
1. 下载源码
直接去官网下载对应的源码 https://sourceware.org/gdb/ 

2. 启动 `msys64/mingw64.exe`, 执行下列命令
```
cd gdb-9.2
mkdir build
cd build
../configure 
make
```
等待编译完成。


参考：
```
https://sourceware.org/gdb/wiki/BuildingOnWindows
https://blog.csdn.net/pfysw/article/details/105451883
https://github.com/ikonst/gdb-7.7-android
https://www.msys2.org/
https://www.cntofu.com/book/46/gdb/188.md
https://mudongliang.github.io/2017/08/12/compile-gdb-with-python-script-support.html
```