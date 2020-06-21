---
title: xposde源码分析
date: 2020-02-12 01:16:03
tags: Xposed
category: Android逆向
---

<!-- TOC -->

- [Xposed源码剖析——概述](#xposed源码剖析概述)
  - [XposedInstaller的构成](#xposedinstaller的构成)
  - [Xposed原理](#xposed原理)
  - [Xposed的实现方案](#xposed的实现方案)
- [Xposed源码剖析——app_process作用详解](#xposed源码剖析app_process作用详解)
  - [app_main.cpp 源码阅读与对比](#app_maincpp-源码阅读与对比)
  - [XposedBridge.java](#xposedbridgejava)
- [Xposed源码剖析——Xposed初始化](#xposed源码剖析xposed初始化)
  - [xposed::initialize初始化](#xposedinitialize初始化)
  - [onVmCreated 初始化后的准备工作](#onvmcreated-初始化后的准备工作)
  - [libxposed_dalvik.cpp hook环境初始化](#libxposed_dalvikcpp-hook环境初始化)
  - [JNI方法注册逻辑](#jni方法注册逻辑)
- [Xposed源码剖析——hook具体实现](#xposed源码剖析hook具体实现)

<!-- /TOC -->

# Xposed源码剖析——概述

XPosed是与Cydia其名的工具，它能够让Android设备在没有修改源码的情况下修改系统中的API运行结果。我们通常称之为：God Mode（上帝模式）。

之前享大家分享了Xposed的基础，Xposed的作用和最简单的用法。那么，它的原理和它的内部构造是如何构成的？下面，我们从Github上看看，rovo89大神是如何制作的。

rovo89的github地址：https://github.com/rovo89

在主页上我们看到了，xposed其实主要是由三个项目来组成的，如下图所示；

![](xposed源码分析/2020-02-12-01-17-48.png)

三个分别是：

| 项目            | 说明                                                                          |
| --------------- | ----------------------------------------------------------------------------- |
| Xposed          | Xposed框架的native部分（主要是改性app_process二进制文件）                     |
| XposedInstaller | Xposed框架的Android端本地管理，环境架构搭建，以及第三方module资源下载的工具。 |
| XposedBridge    | Xposed向开发者提供的API与相应的工具类库                                       |

## XposedInstaller的构成
Xposed项目使我们最常用的项目，当然，他也是构造Xposed的核心部分。（也许你会说，其实是xposed项目更重要，它主要是替换app_process，ok我们后面再说它）。

如下图所示，是我们在XPosedInstaller apk中见到的，安装xposed框架的界面。

![](xposed源码分析/2020-02-12-01-19-33.png)

InstallerFragment我们能够在其中找到install方法，其中主要就是针对使用不同方式的将自定义的app_process文件替换掉系统的app_process文件。
```
    /**
     * xposed install
     * @return 安装成功返回true，否则false
     */
    private boolean install() {
        // 获取安装的方式，直接写入 or 使用 recovery进行安装
        final int installMode = getInstallMode();

        // 检查获取Root权限
        if (!startShell())
            return false;

        List<String> messages = new LinkedList<String>();
        boolean showAlert = true;
        try {
            messages.add(getString(R.string.sdcard_location, XposedApp.getInstance().getExternalFilesDir(null)));
            messages.add("");

            messages.add(getString(R.string.file_copying, "Xposed-Disabler-Recovery.zip"));

            // 把Xposed-Disabler-Recovery.zip文件 从asset copy到sdcard中
            if (AssetUtil.writeAssetToSdcardFile("Xposed-Disabler-Recovery.zip", 00644) == null) {
                messages.add("");
                messages.add(getString(R.string.file_extract_failed, "Xposed-Disabler-Recovery.zip"));
                return false;
            }

            // 将编译后的app_process二进制文件，从asset文件夹中，copy到/data/data/de.robv.android.xposed.installer/bin/app_process下
            File appProcessFile = AssetUtil.writeAssetToFile(APP_PROCESS_NAME, new File(XposedApp.BASE_DIR + "bin/app_process"), 00700);
            if (appProcessFile == null) {
                showAlert(getString(R.string.file_extract_failed, "app_process"));
                return false;
            }

            if (installMode == INSTALL_MODE_NORMAL) {
                // 普通安装模式
                // 重新挂载/system为rw模式
                messages.add(getString(R.string.file_mounting_writable, "/system"));
                if (mRootUtil.executeWithBusybox("mount -o remount,rw /system", messages) != 0) {
                    messages.add(getString(R.string.file_mount_writable_failed, "/system"));
                    messages.add(getString(R.string.file_trying_to_continue));
                }

                // 查看原有的app_process文件是否已经备份，如果没有备份，现将原有的app_process文件备份一下
                if (new File("/system/bin/app_process.orig").exists()) {
                    messages.add(getString(R.string.file_backup_already_exists, "/system/bin/app_process.orig"));
                } else {
                    if (mRootUtil.executeWithBusybox("cp -a /system/bin/app_process /system/bin/app_process.orig", messages) != 0) {
                        messages.add("");
                        messages.add(getString(R.string.file_backup_failed, "/system/bin/app_process"));
                        return false;
                    } else {
                        messages.add(getString(R.string.file_backup_successful, "/system/bin/app_process.orig"));
                    }

                    mRootUtil.executeWithBusybox("sync", messages);
                }

                // 将项目中的自定义app_process文件copy覆盖系统的app_process,修改权限
                messages.add(getString(R.string.file_copying, "app_process"));
                if (mRootUtil.executeWithBusybox("cp -a " + appProcessFile.getAbsolutePath() + " /system/bin/app_process", messages) != 0) {
                    messages.add("");
                    messages.add(getString(R.string.file_copy_failed, "app_process", "/system/bin"));
                    return false;
                }
                if (mRootUtil.executeWithBusybox("chmod 755 /system/bin/app_process", messages) != 0) {
                    messages.add("");
                    messages.add(getString(R.string.file_set_perms_failed, "/system/bin/app_process"));
                    return false;
                }
                if (mRootUtil.executeWithBusybox("chown root:shell /system/bin/app_process", messages) != 0) {
                    messages.add("");
                    messages.add(getString(R.string.file_set_owner_failed, "/system/bin/app_process"));
                    return false;
                }

            } else if (installMode == INSTALL_MODE_RECOVERY_AUTO) {
                // 自动进入Recovery
                if (!prepareAutoFlash(messages, "Xposed-Installer-Recovery.zip"))
                    return false;

            } else if (installMode == INSTALL_MODE_RECOVERY_MANUAL) {
                // 手动进入Recovery
                if (!prepareManualFlash(messages, "Xposed-Installer-Recovery.zip"))
                    return false;
            }

            File blocker = new File(XposedApp.BASE_DIR + "conf/disabled");
            if (blocker.exists()) {
                messages.add(getString(R.string.file_removing, blocker.getAbsolutePath()));
                if (mRootUtil.executeWithBusybox("rm " + blocker.getAbsolutePath(), messages) != 0) {
                    messages.add("");
                    messages.add(getString(R.string.file_remove_failed, blocker.getAbsolutePath()));
                    return false;
                }
            }

            // copy XposedBridge.jar 到私有目录 XposedBridge.jar是Xposed提供的jar文件，负责在Native层与FrameWork层进行交互
            messages.add(getString(R.string.file_copying, "XposedBridge.jar"));
            File jarFile = AssetUtil.writeAssetToFile("XposedBridge.jar", new File(JAR_PATH_NEWVERSION), 00644);
            if (jarFile == null) {
                messages.add("");
                messages.add(getString(R.string.file_extract_failed, "XposedBridge.jar"));
                return false;
            }

            mRootUtil.executeWithBusybox("sync", messages);

            showAlert = false;
            messages.add("");

            if (installMode == INSTALL_MODE_NORMAL) {
                offerReboot(messages);
            } else {
                offerRebootToRecovery(messages, "Xposed-Installer-Recovery.zip", installMode);
            }
            return true;

        } finally {
            // 删除busybox 工具库
            AssetUtil.removeBusybox();

            if (showAlert)
                showAlert(TextUtils.join("\n", messages).trim());
        }
    }
```
ok，我们看完了代码，发现所有的工作都是为了app_process文件的替换。那么，系统中这个app_process是做什么的？为什么我们需要替换？替换成什么样？替换后对于我们么来说有什么帮助呢？

## Xposed原理
我们在android的源码中的init.rc文件可以看到
```
service zygote /system/bin/app_process -Xzygote /system/bin –zygote –start-system-server
socket zygote stream 666 
onrestart write /sys/android_power/request_state wake
onrestart write /sys/power/state on
onrestart restart media
onrestart restart netd
```
app_process是andriod app的启动程序（具体形式是zygote fork()调用一个 app_process作为Android app的载体）

## Xposed的实现方案

针对Hook的不同进程来说又可以分为全局Hook与单个应用程序进程Hook，我们知道在Android系统中，应用程序进程都是由Zygote进程孵化出来的，而Zygote进程是由Init进程启动的。

Zygote进程在启动时会创建一个Dalvik虚拟机实例，每当它孵化一个新的应用程序进程时，都会将这个Dalvik虚拟机实例复制到新的应用程序进程里面去，从而使得每一个应用程序进程都有一个独立的Dalvik虚拟机实例。所以如果选择对Zygote进程Hook，则能够达到针对系统上所有的应用程序进程Hook，即一个全局Hook。如下图所示：

![](xposed源码分析/2020-02-12-01-21-15.png)

# Xposed源码剖析——app_process作用详解

上面我们分析Xposed项目的源码，从XposedInstaller开始说明了Xposed安装的原理与过程。我们知道，XposedInstaller主要的工作就是：

* 替换系统的app_process（当然，这个操作需要Root权限）
* 将xposed的api文件，XposedBridge.jar文件放置到私有目录中

至于为什么要替换app_process文件？
系统中的app_process文件有什么作用？
替换后的app_process为什么能够帮助我们hook？

下面我们就开始看看，rovo89大神的xposed开源项目。从GitHub上面clone下来xposed项目，我们在目录中看到其目录结构，如下所示：

![](xposed源码分析/2020-02-12-01-22-42.png)

从目录中，我们能够清楚的了解到，其中xposed项目现在已经支持Dalvik虚拟机与art虚拟机的架构了。

## app_main.cpp 源码阅读与对比
ok这里，我们先从app_process的源码开始阅读，打开app_main.cpp文件，估计大家和我一下，一时间也看不出来xposed针对源码修改了一些什么。

那么，我们就直接拿源码与xposed中的app_main.cpp进行对比。

源码地址：/frameworks/base/cmds/app_process/app_main.cpp

发现了，rovo89针对了一下几个地方进行了修改。

**atrace_set_tracing_enabled 进行了替换修改**

![](xposed源码分析/2020-02-12-01-23-35.png)

**onVmCreated 增加了Xposed的回调**

![](xposed源码分析/2020-02-12-01-24-05.png)

**main函数中，增加了 xposed 的 options 操作**

![](xposed源码分析/2020-02-12-01-24-34.png)

我们在xposed.cpp中，能够看到其handleOptions的具体逻辑，其实就是处理一些xposed的特殊命令而已。
如下所示：
```
/** Handle special command line options. */
bool handleOptions(int argc, char* const argv[]) {
    parseXposedProp();

    if (argc == 2 && strcmp(argv[1], "--xposedversion") == 0) {
        printf("Xposed version: %s\n", xposedVersion);
        return true;
    }

    if (argc == 2 && strcmp(argv[1], "--xposedtestsafemode") == 0) {
        printf("Testing Xposed safemode trigger\n");

        if (detectSafemodeTrigger(shouldSkipSafemodeDelay())) {
            printf("Safemode triggered\n");
        } else {
            printf("Safemode not triggered\n");
        }
        return true;
    }

    // From Lollipop coding, used to override the process name
    argBlockStart = argv[0];
    uintptr_t start = reinterpret_cast<uintptr_t>(argv[0]);
    uintptr_t end = reinterpret_cast<uintptr_t>(argv[argc - 1]);
    end += strlen(argv[argc - 1]) + 1;
    argBlockLength = end - start;

    return false;
}
```

main函数中，启动的时候增加了启动一些逻辑.

![](xposed源码分析/2020-02-12-01-25-24.png)

具体的， 我们可以看到。runtime.start那一段。做出了一个启动。
```
    isXposedLoaded = xposed::initialize(zygote, startSystemServer, className, argc, argv);
    if (zygote) {
        // 当xposed成功启动的时候，start XPOSED_CLASS_DOTS_ZYGOTE这个类
        runtime.start(isXposedLoaded ? XPOSED_CLASS_DOTS_ZYGOTE : "com.android.internal.os.ZygoteInit",
                startSystemServer ? "start-system-server" : "");
    } else if (className) {
        // Remainder of args get passed to startup class main()
        runtime.mClassName = className;
        runtime.mArgC = argc - i;
        runtime.mArgV = argv + i;
        // 当xposed成功启动的时候，start XPOSED_CLASS_DOTS_ZYGOTE这个类
        runtime.start(isXposedLoaded ? XPOSED_CLASS_DOTS_TOOLS : "com.android.internal.os.RuntimeInit",
                application ? "application" : "tool");
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied.\n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
        return 10;
    }
```

这里的我们看到，在main函数中启动了逻辑，
```
runtime.start(isXposedLoaded ? XPOSED_CLASS_DOTS_ZYGOTE : "com.android.internal.os.ZygoteInit",
                startSystemServer ? "start-system-server" : "");
```

其中，XPOSED_CLASS_DOTS_ZYGOTE 变量在，xposed.h头文件中有定义，如下所示：
```
#define XPOSED_CLASS_DOTS_ZYGOTE "de.robv.android.xposed.XposedBridge"
```
发现，其实这个类就是我们之前向私有目录防止的XposedBridge项目的包名。

而runtime.start这个包名有什么作用呢？我们在AndroidRuntime中找到start方法的具体逻辑
在源代码中/frameworks/base/core/jni/AndroidRuntime.cpp中看到
```
/*
 * Start the Android runtime.  This involves starting the virtual machine
 * and calling the "static void main(String[] args)" method in the class
 * named by "className".
 *
 * Passes the main function two arguments, the class name and the specified
 * options string.
 */
void AndroidRuntime::start(const char* className, const char* options)
```

系统源码对start方法的定义，就是启动对应类的 start void main入口函数。这里，就将三个项目的逻辑连接起来了。

## XposedBridge.java
我们在XposedBridge.java代码中，看到其main方法，如下所示：
```
    /**
     * Called when native methods and other things are initialized, but before preloading classes etc.
     */
    protected static void main(String[] args) {
        // Initialize the Xposed framework and modules
        try {
            SELinuxHelper.initOnce();
            SELinuxHelper.initForProcess(null);

            runtime = getRuntime();
            if (initNative()) {
                XPOSED_BRIDGE_VERSION = getXposedVersion();
                if (isZygote) {
                    startsSystemServer = startsSystemServer();
                    // 为启动一个新的 zygote做好 hook准备
                    initForZygote();
                }
                // 载入Xposed的一些modules
                loadModules();
            } else {
                log("Errors during native Xposed initialization");
            }
        } catch (Throwable t) {
            log("Errors during Xposed initialization");
            log(t);
            disableHooks = true;
        }

        // 调用系统原来的启动方法
        if (isZygote)
            ZygoteInit.main(args);
        else
            RuntimeInit.main(args);
    }
```
ok，那么，整个app_process的复制hook逻辑，到这里我们已经清楚了。逻辑如下图所示。

![](xposed源码分析/2020-02-12-01-27-01.png)

那么，xposed具体怎么实现系统api逻辑的replace和inject我们下次在做分析。

# Xposed源码剖析——Xposed初始化

之前我们看过了app_main.cpp源码，知道了在其中，启动了XposedBridge.jar方法。那么，其中还做了些什么事情呢？

之前我们也看到了在app_main.cpp还有几处新增的逻辑。xposed::initialize和onVmCreated回调。下面我在仔细的阅读以下源码。

## xposed::initialize初始化

![](xposed源码分析/2020-02-12-01-28-24.png)

对于xposed::initalize的初始化工作，我们能够在xposed.cpp中看到其具体的逻辑实现。
```
/** 
 * 初始化xposed
 */
bool initialize(bool zygote, bool startSystemServer, const char* className, int argc, char* const argv[]) {
#if !defined(XPOSED_ENABLE_FOR_TOOLS)
    if (!zygote)
        return false;
#endif

    xposed->zygote = zygote;
    xposed->startSystemServer = startSystemServer;
    xposed->startClassName = className;
    xposed->xposedVersionInt = xposedVersionInt;

#if XPOSED_WITH_SELINUX
    xposed->isSELinuxEnabled   = is_selinux_enabled() == 1;
    xposed->isSELinuxEnforcing = xposed->isSELinuxEnabled && security_getenforce() == 1;
#else
    xposed->isSELinuxEnabled   = false;
    xposed->isSELinuxEnforcing = false;
#endif  // XPOSED_WITH_SELINUX

    if (startSystemServer) {
        xposed::logcat::start();
    } else if (zygote) {
        // TODO Find a better solution for this
        // Give the primary Zygote process a little time to start first.
        // This also makes the log easier to read, as logs for the two Zygotes are not mixed up.
        sleep(10);
    }

    // 打印rom信息
    printRomInfo();

    if (startSystemServer) {
        if (!xposed::service::startAll())
            return false;
#if XPOSED_WITH_SELINUX
    } else if (xposed->isSELinuxEnabled) {
        if (!xposed::service::startMembased())
            return false;
#endif  // XPOSED_WITH_SELINUX
    }

    // FIXME Zygote has no access to input devices, this would need to be check in system_server context
    if (zygote && !isSafemodeDisabled() && detectSafemodeTrigger(shouldSkipSafemodeDelay()))
        disableXposed();

    if (isDisabled() || (!zygote && shouldIgnoreCommand(argc, argv)))
        return false;

    // 将XposedBridge.jar的路径添加到环境变量classpath中
    return addJarToClasspath();
}
```

## onVmCreated 初始化后的准备工作

![](xposed源码分析/2020-02-12-01-29-28.png)

其具体的逻辑如下所示：
```
/**   
  * 向当前的runtime中载入libxposed_*.so 
  */
void onVmCreated(JNIEnv* env) {
    // Determine the currently active runtime
    const char* xposedLibPath = NULL;
    if (!determineRuntime(&xposedLibPath)) {
        ALOGE("Could not determine runtime, not loading Xposed");
        return;
    }

    // Load the suitable libxposed_*.so for it
    const char *error;
    void* xposedLibHandle = dlopen(xposedLibPath, RTLD_NOW);
    if (!xposedLibHandle) {
        ALOGE("Could not load libxposed: %s", dlerror());
        return;
    }

    // Clear previous errors
    dlerror();

    // Initialize the library
    bool (*xposedInitLib)(XposedShared* shared) = NULL;
    *(void **) (&xposedInitLib) = dlsym(xposedLibHandle, "xposedInitLib");
    if (!xposedInitLib)  {
        ALOGE("Could not find function xposedInitLib");
        return;
    }

#if XPOSED_WITH_SELINUX
    xposed->zygoteservice_accessFile = &service::membased::accessFile;
    xposed->zygoteservice_statFile   = &service::membased::statFile;
    xposed->zygoteservice_readFile   = &service::membased::readFile;
#endif  // XPOSED_WITH_SELINUX

    // 这里的xposed变量，其实是一个全局的XposedShare。
    // 调用XposedShare的onVmCreated则会根据不同的vm架构针对不同的实现。
    if (xposedInitLib(xposed)) {
        xposed->onVmCreated(env);
    }
}
```

## libxposed_dalvik.cpp hook环境初始化
```
/** Called by Xposed's app_process replacement. 
  * 在被替换后的app_process中调用
  */
bool xposedInitLib(xposed::XposedShared* shared) {
    xposed = shared;
    // 将自己的onVmCreated方法，指向onVmCreated方法
    xposed->onVmCreated = &onVmCreated;
    return true;
}

/** Called very early during VM startup. 
  * 在VM启动的时候调用，而且调用时机比较早
  */
void onVmCreated(JNIEnv* env) {
    if (!initMemberOffsets(env))
        return;

    // 找到小米系统的MIUI_RESOURCE做特殊处理
    jclass classMiuiResources = env->FindClass(CLASS_MIUI_RESOURCES);
    if (classMiuiResources != NULL) {
        ClassObject* clazz = (ClassObject*)dvmDecodeIndirectRef(dvmThreadSelf(), classMiuiResources);
        if (dvmIsFinalClass(clazz)) {
            ALOGD("Removing final flag for class '%s'", CLASS_MIUI_RESOURCES);
            clazz->accessFlags &= ~ACC_FINAL;
        }
    }
    env->ExceptionClear();

    jclass classXTypedArray = env->FindClass(CLASS_XTYPED_ARRAY);
    if (classXTypedArray == NULL) {
        ALOGE("Error while loading XTypedArray class '%s':", CLASS_XTYPED_ARRAY);
        dvmLogExceptionStackTrace();
        env->ExceptionClear();
        return;
    }
    prepareSubclassReplacement(classXTypedArray);

    // 获取到全局的XposedBridge
    classXposedBridge = env->FindClass(CLASS_XPOSED_BRIDGE);
    classXposedBridge = reinterpret_cast<jclass>(env->NewGlobalRef(classXposedBridge));

    if (classXposedBridge == NULL) {
        ALOGE("Error while loading Xposed class '%s':", CLASS_XPOSED_BRIDGE);
        dvmLogExceptionStackTrace();
        env->ExceptionClear();
        return;
    }

    // 注册一些 XposedBridge 的 native 方法
    ALOGI("Found Xposed class '%s', now initializing", CLASS_XPOSED_BRIDGE);
    if (register_natives_XposedBridge(env, classXposedBridge) != JNI_OK) {
        ALOGE("Could not register natives for '%s'", CLASS_XPOSED_BRIDGE);
        dvmLogExceptionStackTrace();
        env->ExceptionClear();
        return;
    }

    xposedLoadedSuccessfully = true;
}
```

## JNI方法注册逻辑

这里注册的几个方法都是，Xposed核心的几个方法函数。

```
int register_natives_XposedBridge(JNIEnv* env, jclass clazz) {
    const JNINativeMethod methods[] = {

        NATIVE_METHOD(XposedBridge, getStartClassName, "()Ljava/lang/String;"),
        // 获得Runtime
        NATIVE_METHOD(XposedBridge, getRuntime, "()I"),
        // 启动SystemServer
        NATIVE_METHOD(XposedBridge, startsSystemServer, "()Z"),
        // 获取Xposed的版本信息
        NATIVE_METHOD(XposedBridge, getXposedVersion, "()I"),
        // 初始化navtive
        NATIVE_METHOD(XposedBridge, initNative, "()Z"),
        // hook一个方法的native实现
        NATIVE_METHOD(XposedBridge, hookMethodNative, "(Ljava/lang/reflect/Member;Ljava/lang/Class;ILjava/lang/Object;)V"),
#ifdef ART_TARGET
        NATIVE_METHOD(XposedBridge, invokeOriginalMethodNative,
            "(Ljava/lang/reflect/Member;I[Ljava/lang/Class;Ljava/lang/Class;Ljava/lang/Object;[Ljava/lang/Object;)Ljava/lang/Object;"),
#endif
        NATIVE_METHOD(XposedBridge, setObjectClassNative, "(Ljava/lang/Object;Ljava/lang/Class;)V"),
        NATIVE_METHOD(XposedBridge, dumpObjectNative, "(Ljava/lang/Object;)V"),
        NATIVE_METHOD(XposedBridge, cloneToSubclassNative, "(Ljava/lang/Object;Ljava/lang/Class;)Ljava/lang/Object;"),
    };
    return env->RegisterNatives(clazz, methods, NELEM(methods));
}
```
我们看到RegisterNatives这个方法的时候不是很理解，这里做一个简介。

以前在jni中写本地方法时，都会写成 Java_com_example_hellojni_HelloJni_stringFromJNI的形式，函数名很长，而且当类名变了的时候，函数名必须一个一个的改，麻烦。
现在好了有了RegisterNatives，可以简化我们的书写
和传统方法相比，使用RegisterNatives的好处有三点：
1. C＋＋中函数命名自由，不必像javah自动生成的函数声明那样，拘泥特定的命名方式；
2. 效率高。传统方式下，Java类call本地函数时，通常是依靠VM去动态寻找.so中的本地函数(因此它们才需要特定规则的命名格式)，而使用RegisterNatives将本地函数向VM进行登记，可以让其更有效率的找到函数；
3. 运行时动态调整本地函数与Java函数值之间的映射关系，只需要多次call RegisterNatives()方法，并传入不同的映射表参数即可。

# Xposed源码剖析——hook具体实现

之前我们看到了xposed各种初始化的工作，其实都是完成了针对系统中各种method的hook和替换工作。

那么具体如何替换，其实都是调用了其中的。XposedBridge_hookMethodNative函数。这里，我们详细的看看XposedBridge_hookMethodNative函数中，做了一些什么操作。
```
/**
  *
  * 将输入的Class中的Method方法的nativeFunc替换为xposedCallHandler 
  * 
  * @param env JniEnv
  * @param reflectedMethodIndirect 待反射的函数
  * @param declaredClassIndirect 定义的class
  * @param slot 函数偏移量
  * @param additionalInfoIndirect 添加的函数
  * 
  */
void XposedBridge_hookMethodNative(JNIEnv* env, jclass clazz, jobject reflectedMethodIndirect,
            jobject declaredClassIndirect, jint slot, jobject additionalInfoIndirect) {
    // 容错
    if (declaredClassIndirect == NULL || reflectedMethodIndirect == NULL) {
        dvmThrowIllegalArgumentException("method and declaredClass must not be null");
        return;
    }

    // 根据函数的偏移量，从classloader中找到准备替换的函数。
    ClassObject* declaredClass = (ClassObject*) dvmDecodeIndirectRef(dvmThreadSelf(), declaredClassIndirect);
    Method* method = dvmSlotToMethod(declaredClass, slot);
    if (method == NULL) {
        dvmThrowNoSuchMethodError("Could not get internal representation for method");
        return;
    }

    if (isMethodHooked(method)) {
        // already hooked
        return;
    }

    // 保存替换前的数据信息
    XposedHookInfo* hookInfo = (XposedHookInfo*) calloc(1, sizeof(XposedHookInfo));
    memcpy(hookInfo, method, sizeof(hookInfo->originalMethodStruct));
    hookInfo->reflectedMethod = dvmDecodeIndirectRef(dvmThreadSelf(), env->NewGlobalRef(reflectedMethodIndirect));
    hookInfo->additionalInfo = dvmDecodeIndirectRef(dvmThreadSelf(), env->NewGlobalRef(additionalInfoIndirect));

    // 替换函数方法 , 让nativeFunction指向本地的hookedMethodCallback
    SET_METHOD_FLAG(method, ACC_NATIVE);
    method->nativeFunc = &hookedMethodCallback;
    method->insns = (const u2*) hookInfo;
    method->registersSize = method->insSize;
    method->outsSize = 0;

    if (PTR_gDvmJit != NULL) {
        // reset JIT cache
        char currentValue = *((char*)PTR_gDvmJit + MEMBER_OFFSET_VAR(DvmJitGlobals,codeCacheFull));
        if (currentValue == 0 || currentValue == 1) {
            MEMBER_VAL(PTR_gDvmJit, DvmJitGlobals, codeCacheFull) = true;
        } else {
            ALOGE("Unexpected current value for codeCacheFull: %d", currentValue);
        }
    }
}
```
对vm不熟悉的，解释一下几个不怎么常用的函数。

| 名称                 | 说明                                    |
| -------------------- | --------------------------------------- |
| dvmDecodeIndirectRef | 将间接引用jobject转换为对象引用Object*  |
| dvmSlotToMethod      | 根据偏移量，从ClassLoader中获取函数指针 |

```
/** 
  * hook方法调用时的回调
  */
void hookedMethodCallback(const u4* args, JValue* pResult, const Method* method, ::Thread* self) {
    if (!isMethodHooked(method)) {
        dvmThrowNoSuchMethodError("Could not find Xposed original method - how did you even get here?");
        return;
    }

    XposedHookInfo* hookInfo = (XposedHookInfo*) method->insns;
    Method* original = (Method*) hookInfo;
    Object* originalReflected = hookInfo->reflectedMethod;
    Object* additionalInfo = hookInfo->additionalInfo;

    // convert/box arguments
    const char* desc = &method->shorty[1]; // [0] is the return type.
    Object* thisObject = NULL;
    size_t srcIndex = 0;
    size_t dstIndex = 0;

    // for non-static methods determine the "this" pointer
    if (!dvmIsStaticMethod(original)) {
        thisObject = (Object*) args[0];
        srcIndex++;
    }

    ArrayObject* argsArray = dvmAllocArrayByClass(objectArrayClass, strlen(method->shorty) - 1, ALLOC_DEFAULT);
    if (argsArray == NULL) {
        return;
    }

    while (*desc != '\0') {
        char descChar = *(desc++);
        JValue value;
        Object* obj;

        switch (descChar) {
        case 'Z':
        case 'C':
        case 'F':
        case 'B':
        case 'S':
        case 'I':
            value.i = args[srcIndex++];
            obj = (Object*) dvmBoxPrimitive(value, dvmFindPrimitiveClass(descChar));
            dvmReleaseTrackedAlloc(obj, self);
            break;
        case 'D':
        case 'J':
            value.j = dvmGetArgLong(args, srcIndex);
            srcIndex += 2;
            obj = (Object*) dvmBoxPrimitive(value, dvmFindPrimitiveClass(descChar));
            dvmReleaseTrackedAlloc(obj, self);
            break;
        case '[':
        case 'L':
            obj  = (Object*) args[srcIndex++];
            break;
        default:
            ALOGE("Unknown method signature description character: %c", descChar);
            obj = NULL;
            srcIndex++;
        }
        setObjectArrayElement(argsArray, dstIndex++, obj);
    }

    // 调用Java中的对应方法，即之前我们用到，的handleHookedMethod
    JValue result;
    dvmCallMethod(self, xposedHandleHookedMethod, NULL, &result,
        originalReflected, (int) original, additionalInfo, thisObject, argsArray);

    dvmReleaseTrackedAlloc(argsArray, self);

    // exceptions are thrown to the caller
    if (dvmCheckException(self)) {
        return;
    }

    // return result with proper type
    ClassObject* returnType = dvmGetBoxedReturnType(method);
    if (returnType->primitiveType == PRIM_VOID) {
        // ignored
    } else if (result.l == NULL) {
        if (dvmIsPrimitiveClass(returnType)) {
            dvmThrowNullPointerException("null result when primitive expected");
        }
        pResult->l = NULL;
    } else {
        if (!dvmUnboxPrimitive(result.l, returnType, pResult)) {
            dvmThrowClassCastException(result.l->clazz, returnType);
        }
    }
}
```


> 转载：https://blog.csdn.net/yzzst/article/details/47659987




