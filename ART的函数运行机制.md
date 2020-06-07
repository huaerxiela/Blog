---
title: ART的函数运行机制
date: 2020-06-03 19:25:03
tag: Android源码阅读
category: Android源码
---

# 类的加载

`dalvik` 的类加载的主要过程如下：

1. `app_process` 作为 `zygote server` 通过 `local socket` 处理进程创建请求，`zygote server` 是在 `ZygoteInit.main` 函数里调用 `ZygoteInit.runSelectLoop` 监听。

2. 接收到 `zygote client` 的 `fork` 请求之后，调用 `ZygoteConnection.runOnce` ，调用 `Zygote.forkAndSpecialize` 创建新进程

3. 进程创建之后，由 `ZygoteConnection.handleParentProc` 来初始化进程，最终会调用 `ActivityThread.main` 函数

4. `ActivityThread.main -> ActivityThread.attach ->  ActivityThread.bindApplication -> Activity.handleBindApplication，handleBindApplication` 会初始化 `BaseDexClassLoader` 。

5. 类的加载经过了 `ClassLoader.loadClass->BaseDexClassLoader.findClass->DexPathList.findClass->DexFile.loadClassBinaryName->DexFile.defineClassNative->DexFile_defineClassNative` (runtime/native/dalvik_system_DexFile.cc) 。

这个初始化过程， `art` 和 `dalvik` 都是一样的。 `art` 的 `DexFile_defineClassNative` 由 `ClassLinker` 的 `DefineClass` 来加载类。

```
static jclass DexFile_defineClassNative(JNIEnv* env,
                                        jclass,
                                        jstring javaName,
                                        jobject javaLoader,
                                        jobject cookie,
                                        jobject dexFile) {
  std::vector<const DexFile*> dex_files;
  const OatFile* oat_file;
  if (!ConvertJavaArrayToDexFiles(env, cookie, /*out*/ dex_files, /*out*/ oat_file)) {
    VLOG(class_linker) << "Failed to find dex_file";
    DCHECK(env->ExceptionCheck());
    return nullptr;
  }

  ScopedUtfChars class_name(env, javaName);
  if (class_name.c_str() == nullptr) {
    VLOG(class_linker) << "Failed to find class_name";
    return nullptr;
  }
  const std::string descriptor(DotToDescriptor(class_name.c_str()));
  const size_t hash(ComputeModifiedUtf8Hash(descriptor.c_str()));
  for (auto& dex_file : dex_files) {
    const DexFile::ClassDef* dex_class_def =
        OatDexFile::FindClassDef(*dex_file, descriptor.c_str(), hash);
    if (dex_class_def != nullptr) {
      ScopedObjectAccess soa(env);
      ClassLinker* class_linker = Runtime::Current()->GetClassLinker();
      StackHandleScope<1> hs(soa.Self());
      Handle<mirror::ClassLoader> class_loader(
          hs.NewHandle(soa.Decode<mirror::ClassLoader>(javaLoader)));
      ObjPtr<mirror::DexCache> dex_cache =
          class_linker->RegisterDexFile(*dex_file, class_loader.Get());
      if (dex_cache == nullptr) {
        // OOME or InternalError (dexFile already registered with a different class loader).
        soa.Self()->AssertPendingException();
        return nullptr;
      }
      // 获取类
      ObjPtr<mirror::Class> result = class_linker->DefineClass(soa.Self(),
                                                               descriptor.c_str(),
                                                               hash,
                                                               class_loader,
                                                               *dex_file,
                                                               *dex_class_def);
      // Add the used dex file. This only required for the DexFile.loadClass API since normal
      // class loaders already keep their dex files live.
      class_linker->InsertDexFileInToClassLoader(soa.Decode<mirror::Object>(dexFile),
                                                 class_loader.Get());
      if (result != nullptr) {
        VLOG(class_linker) << "DexFile_defineClassNative returning " << result
                           << " for " << class_name.c_str();
        return soa.AddLocalReference<jclass>(result);
      }
    }
  }
  VLOG(class_linker) << "Failed to find dex_class_def " << class_name.c_str();
  return nullptr;
}
```


