---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： クラスローダー検索 (Class Loader Search) ： AddToBootstrapClassLoaderSearch() 及び AddToSystemClassLoaderSearch() の処理  
---
[Up](no2gPnYMEo.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： クラスローダー検索 (Class Loader Search) ： AddToBootstrapClassLoaderSearch() 及び AddToSystemClassLoaderSearch() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
### AddToBootstrapClassLoaderSearch() の処理
```
JvmtiEnv::AddToBootstrapClassLoaderSearch()
-> ClassLoader::create_class_path_zip_entry()
   -> ClassPathZipEntry::ClassPathZipEntry()
   -> ClassLoader::add_to_list()
```

### AddToSystemClassLoaderSearch() の処理
```
JvmtiEnv::AddToSystemClassLoaderSearch()
-> ClassLoader::create_class_path_zip_entry()
-> JavaCalls::call_special()
   -> sun.misc.Launcher.AppClassLoader.appendToClassPathForInstrumentation()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::AddToBootstrapClassLoaderSearch()
(#Under Construction)

### ClassLoader::create_class_path_zip_entry()
(#Under Construction)

### ClassPathZipEntry::ClassPathZipEntry()
(#Under Construction)

### ClassLoader::add_to_list()
See: [here](no75174il.html) for details
### JvmtiEnv::AddToSystemClassLoaderSearch()
(#Under Construction)

### sun.misc.Launcher.AppClassLoader.appendToClassPathForInstrumentation()
(#Under Construction)







