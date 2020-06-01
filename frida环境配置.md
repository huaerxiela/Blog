---
title: frida环境配置
date: 2020-01-19 16:10:09
tags: radare2 frida
category: frida
---

开发环境：ubuntu18.04

> 源码地址：https://github.com/frida/frida
> 官网：https://frida.re/
<!-- TOC -->

- [安装 Frida](#安装-frida)
  - [1. 在安装frida之前最好创建一个 python 虚拟环境，这样可以避免与其他环境产生干扰](#1-在安装frida之前最好创建一个-python-虚拟环境这样可以避免与其他环境产生干扰)
  - [2. 进入虚拟环境，直接运行下来命令即可安装完成](#2-进入虚拟环境直接运行下来命令即可安装完成)
  - [3. 根据f rida 的版本去下载对应的 `frida-server-12.8.7-android-arm64.xz`](#3-根据f-rida-的版本去下载对应的-frida-server-1287-android-arm64xz)
  - [4. 将 `frida-server` push 进 `data/local/tmp` 目录，并给予其运行权限，使用 root 用户启动。](#4-将-frida-server-push-进-datalocaltmp-目录并给予其运行权限使用-root-用户启动)
- [配置开发环境](#配置开发环境)
  - [从官方下载API类型定义](#从官方下载api类型定义)
  - [另一种方式：](#另一种方式)
  - [简单使用](#简单使用)

<!-- /TOC -->

# 安装 Frida
## 1. 在安装frida之前最好创建一个 python 虚拟环境，这样可以避免与其他环境产生干扰
```
mkvirtualenv -p python3 frida_12.8.7
```

## 2. 进入虚拟环境，直接运行下来命令即可安装完成
```
pip install frida-tools # CLI tools
pip install frida       # Python bindings

```
安装完成后，运行 `frida --version` 查看 frida 的版本

![](frida环境配置/2020-01-19-16-20-36.png)

## 3. 根据f rida 的版本去[下载](https://github.com/frida/frida/releases)对应的 `frida-server-12.8.7-android-arm64.xz` 
## 4. 将 `frida-server` push 进 `data/local/tmp` 目录，并给予其运行权限，使用 root 用户启动。
```
# chmod +x frida-server
# ./frida-server
```
执行 `frida-ps -U` ,出现以下信息则表明安装成功。

![](frida环境配置/2020-01-19-16-31-30.png)

# 配置开发环境
## 从官方下载API类型定义
> `https://github.com/frida/frida-gum/blob/6e36eebe1aad51c37329242cf07ac169fc4a62c4/bindings/gumjs/types/frida-gum/frida-gum.d.ts`

在 IDE 中需要使用 FridaAPI 的文件开头加上
`///<reference path='./frida-gum.d.ts' />`
即可。

![](frida环境配置/2020-01-19-16-50-57.png)

## 另一种方式：
frida-gum.d.ts已经被大胡子删掉了，他现在不赞成使用这种方式。
目前最新正确的Frida环境搭建方法：
1. `git clone https://github.com/oleavr/frida-agent-example`
2. `npm install`
3. 使用 PyCharm 等 IDE 打开此工程，在 agent 下编写 typescript ，会有智能提示。
4. `npm run watch` 会监控代码修改自动编译生成 js 文件
5. python 脚本或 cli 加载 `_agent.js` 。


参考：https://bbs.pediy.com/thread-254086.htm

## 简单使用
参考官网文档简单使用一下 Frida .

例子[下载地址](https://github.com/ctfs/write-ups-2015/tree/master/seccon-quals-ctf-2015/binary/reverse-engineering-android-apk-1)

