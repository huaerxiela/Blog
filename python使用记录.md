---
title: python使用记录
date: 2019-12-11 17:22:25
tags: Python使用
category: python
---

本文章记录使用pytho过程中易忘的知识，方便自己查阅。

# Python中join函数和os.path.join用法
> join ： 连接字符串数组。将字符串、元组、列表中的元素以指定的字符(分隔符)连接生成一个新的字符串

> os.path.join() ： 将多个路径组合后返回

```
语法：‘sep’.join（seq）

参数说明：
    sep：分隔符。可以为空
    seq：要连接的元素序列、字符串、元组、字典等
上面的语法即：以sep作为分隔符，将seq所有的元素合并成一个新的字符串.

返回值：返回一个以分隔符sep连接各个元素后生成的字符串
```

例子：
```
import os

seq = ['hello', 'world', '世界','你好']
print(' '.join(seq))
print(os.path.join('home', "me", "mypath"))
```
输出
```
hello world 世界 你好
home/me/mypath
```

# pycharm 设置忽略大小写进行自动补齐
进入下列设置
> settings -> Editor -> General -> Code Completion

将 `Match case` 不勾选就可以忽略大小写进行自动补齐了。

# pycharm 设置 python 代码模板

设置 File > Settings > File and Code Template > Python Script
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# File Name : ${NAME}
# Created by ${USER} on $DATE

__author__ = '${USER}'

if __name__ == '__main__':
    pass
```
一些模板变量含义
```
${PROJECT_NAME} - 当前Project名称;
 ${NAME} - 在创建文件的对话框中指定的文件名;
 ${USER} - 当前用户名;
 ${DATE} - 当前系统日期;
 ${TIME} - 当前系统时间;
 ${YEAR} - 年;
 ${MONTH} - 月;
 ${DAY} - 日;
 ${HOUR} - 小时;
 ${MINUTE} - 分钟；
 ${PRODUCT_NAME} - 创建文件的IDE名称;
 ${MONTH_NAME_SHORT} - 英文月份缩写, 如: Jan, Feb, etc;
 ${MONTH_NAME_FULL} - 英文月份全称, 如: January, February, etc；
```

# pycharm打开虚拟环境时，代码无法补全
打开设置，按如下图所示添加对应的python虚拟环境

![](python使用记录/2020-01-27-13-31-34.png)
