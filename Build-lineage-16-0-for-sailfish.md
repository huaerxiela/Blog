---
title: Build lineage-16.0 for sailfish
date: 2020-06-05 09:16:58
tags: Android系统编译
category: Android源码
---


> 系统环境：Ubuntu 18.04.3
> 
> 编译Android系统版本： lineage-16.0
<!-- TOC -->

- [1. 配置安装平台工具](#1-配置安装平台工具)
  - [1.1. 安装 `adb` 和 `fastboot` ， 可以从谷歌下载它们。提取运行：](#11-安装-adb-和-fastboot--可以从谷歌下载它们提取运行)
- [2. 配置必须的环境](#2-配置必须的环境)
  - [2.1. 安装必须的库](#21-安装必须的库)
  - [2.2. JDK](#22-jdk)
- [3. 下载源码](#3-下载源码)
  - [3.1. 创建目录](#31-创建目录)
  - [3.2. 安装 repo 命令](#32-安装-repo-命令)
  - [3.3. 配置git](#33-配置git)
  - [3.4. 初始化LineageOS源并下载源代码](#34-初始化lineageos源并下载源代码)
  - [3.5. 提取专有Blob](#35-提取专有blob)
    - [3.5.1. 通过 `extract_files.sh` 脚本提取](#351-通过-extract_filessh-脚本提取)
    - [3.5.2. 通过修改 `.repo/local_manifests/*.xml` 文件进行提取](#352-通过修改-repolocal_manifestsxml-文件进行提取)
- [4. 开始编译](#4-开始编译)
  - [4.1. 准备设备特定的代码](#41-准备设备特定的代码)
  - [4.2. 开始构建](#42-开始构建)
  - [4.3. 安装版本](#43-安装版本)

<!-- /TOC -->

# 1. 配置安装平台工具

## 1.1. 安装 `adb` 和 `fastboot` ， 可以从[谷歌下载](https://dl.google.com/android/repository/platform-tools-latest-linux.zip)它们。提取运行：

```
unzip platform-tools-latest-linux.zip -d ~
```

添加 `adb` 和 `fastboot` 到 `PATH` 。打开 `~/.profile` 并添加以下内容：
```
# add Android SDK platform tools to path
if [ -d "$HOME/platform-tools" ] ; then
    PATH="$HOME/platform-tools:$PATH"
fi
```
然后，运行 `source ~/.profile` 以更新环境。


# 2. 配置必须的环境

## 2.1. 安装必须的库
```
bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev
```

对于 20.04 之前的Ubuntu版本，需要安装：
```
libwxgtk3.0-dev
```

对于早于 16.04 的Ubuntu版本，请安装：
```
libwxgtk2.8-dev
```

## 2.2. JDK
* LineageOS 16.0：OpenJDK 1.9（默认包含）
* LineageOS 14.1-15.1：OpenJDK 1.8（安装openjdk-8-jdk）
* LineageOS 11.0-13.0：OpenJDK 1.7（安装openjdk-7-jdk）*


# 3. 下载源码

## 3.1. 创建目录
创建构建环境中设置一些目录：
```
mkdir -p ~/bin
mkdir -p ~/android/lineage
```
## 3.2. 安装 repo 命令
输入以下内容以下载 repo 二进制文件并使其可执行（可运行）：
```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

将~/bin目录放在执行路径中，打开 `~/.profile` 文件，添加下列代码：
```
# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```
然后，运行 `source ~/.profile` 以更新环境。


## 3.3. 配置git
运行以下命令来配置git身份：
```
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

## 3.4. 初始化LineageOS源并下载源代码
```
cd ~/android/lineage
repo init -u https://github.com/LineageOS/android.git -b lineage-16.0

repo sync
```

## 3.5. 提取专有Blob
### 3.5.1. 通过 `extract_files.sh` 脚本提取
确保您的 Pixel 已通过 USB 电缆连接到计算机，并且已启用 ADB 和 root 身份，并且位于 `~/android/lineage/device/google/sailfish` 文件夹中。然后运行 `extract-files.sh`脚本：
```
./extract-files.sh
```
Blob 应被拉入 `~/android/lineage/vendor/google` 文件夹。

### 3.5.2. 通过修改 `.repo/local_manifests/*.xml` 文件进行提取
在 `.repo/local_manifests/roomservice.xml` 文件中添加下列代码
```
<project clone-depth="1" name="TheMuppets/proprietary_vendor_google" path="vendor/google" remote="github" />
```
最终代码：
```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="LineageOS/android_device_google_sailfish" path="device/google/sailfish" remote="github" />
  <project name="LineageOS/android_device_google_marlin" path="device/google/marlin" remote="github" />
  <project name="LineageOS/android_kernel_google_marlin" path="kernel/google/marlin" remote="github" />
  <project clone-depth="1" name="TheMuppets/proprietary_vendor_google" path="vendor/google" remote="github" />
</manifest>
```
然后执行 `repo sync` ， 最终将会 `~/android/lineage/vendor/google` 文件夹如下所示：

![](Build-lineage-16-0-for-sailfish/2020-06-07-11-17-46.png)



# 4. 开始编译
## 4.1. 准备设备特定的代码
进入 `~/android/lineage` 目录中，运行下列命令：
```
$ source build/envsetup.sh  
including device/generic/car/vendorsetup.sh
including device/generic/mini-emulator-arm64/vendorsetup.sh
including device/generic/mini-emulator-armv7-a-neon/vendorsetup.sh
including device/generic/mini-emulator-x86/vendorsetup.sh
including device/generic/mini-emulator-x86_64/vendorsetup.sh
including device/generic/uml/vendorsetup.sh
including device/google/marlin/vendorsetup.sh
including vendor/lineage/vendorsetup.sh
including sdk/bash_completion/adb.bash


$ breakfast sailfish
including vendor/lineage/vendorsetup.sh
Looking for dependencies in device/google/sailfish
Looking for dependencies in device/google/marlin
Looking for dependencies in kernel/google/marlin
kernel/google/marlin has no additional dependencies.

============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=9
LINEAGE_VERSION=16.0-20200607-UNOFFICIAL-sailfish
TARGET_PRODUCT=lineage_sailfish
TARGET_BUILD_VARIANT=userdebug
TARGET_BUILD_TYPE=release
TARGET_ARCH=arm64
TARGET_ARCH_VARIANT=armv8-a
TARGET_CPU_VARIANT=kryo
TARGET_2ND_ARCH=arm
TARGET_2ND_ARCH_VARIANT=armv8-a
TARGET_2ND_CPU_VARIANT=kryo
HOST_ARCH=x86_64
HOST_2ND_ARCH=x86
HOST_OS=linux
HOST_OS_EXTRA=Linux-5.3.0-53-generic-x86_64-Ubuntu-18.04.4-LTS
HOST_CROSS_OS=windows
HOST_CROSS_ARCH=x86
HOST_CROSS_2ND_ARCH=x86_64
HOST_BUILD_TYPE=release
BUILD_ID=PQ3A.190801.002
OUT_DIR=/media/ckcat/5670d030-468f-f443-9854-3a9e65ef901c/lineage/out
============================================
```

## 4.2. 开始构建

```
croot
brunch sailfish
```

## 4.3. 安装版本

假设构建已完成且没有错误（完成后将很明显），请在构建运行的终端窗口中键入以下内容：
```
cd $OUT
```
在这里，您会找到所有已创建的文件。更具兴趣的两个文件是：

* boot.img，这是LineageOS引导映像，并包含recovery-ramdisk。

* lineage-17.1-20200605-UNOFFICIAL-sailfish.zip，这是LineageOS安装程序包。

![](Build-lineage-16-0-for-sailfish/2020-06-07-11-23-55.png)

成功!

参考：
> https://wiki.lineageos.org/devices/sailfish/build
> 
> https://www.reddit.com/r/LineageOS/comments/bjo6v3/error_building_lineageos/
> 