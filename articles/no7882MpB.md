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
<div class="flow-abst"><pre>
JvmtiEnv::AddToBootstrapClassLoaderSearch()
-&gt; ClassLoader::create_class_path_zip_entry()
   -&gt; ClassPathZipEntry::ClassPathZipEntry()
   -&gt; ClassLoader::add_to_list()
</pre></div>

### AddToSystemClassLoaderSearch() の処理
<div class="flow-abst"><pre>
JvmtiEnv::AddToSystemClassLoaderSearch()
-&gt; ClassLoader::create_class_path_zip_entry()
-&gt; JavaCalls::call_special()
   -&gt; sun.misc.Launcher.AppClassLoader.appendToClassPathForInstrumentation()
</pre></div>


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







