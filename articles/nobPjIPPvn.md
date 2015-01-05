---
layout: default
title: JNI の処理 ： native method の処理 ： native method の dynamic loading 処理(java.lang.System.loadLibrary() の処理) (JNI_OnLoad() の呼び出し処理も含む)
---
[Up](noNisy_uNv.html) [Top](../index.html)

#### JNI の処理 ： native method の処理 ： native method の dynamic loading 処理(java.lang.System.loadLibrary() の処理) (JNI_OnLoad() の呼び出し処理も含む)

--- 
## 概要(Summary)
基本的には, 各 OS が提供しているダイナミックロード用のシステムコールを呼び出すだけ.

なお, JNI_OnLoad() シンボルが含まれている場合には, それを呼び出す処理も行われる.

## 備考(Notes)
JNI_OnLoad のシンボル名は, プラットフォームによっては (より具体的に言うと Windows の場合には) 複数の候補が存在することがある.


```cpp
    ((cite: hotspot/src/os/windows/vm/jvm_windows.h))
    #define JNI_ONLOAD_SYMBOLS      {"_JNI_OnLoad@8", "JNI_OnLoad"}
```


## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
java.lang.System.loadLibrary()
-&gt; java.lang.Runtime.loadLibrary0()
   -&gt; java.lang.ClassLoader.loadLibrary()
      -&gt; java.lang.ClassLoader.findLibrary()    (&lt;= 必要に応じて呼び出される)
      -&gt; java.lang.ClassLoader.loadLibrary0()
         -&gt; java.lang.ClassLoader$NativeLibrary.load()
            -&gt; Java_java_lang_ClassLoader_00024NativeLibrary_load()
               -&gt; (1) ライブラリをロードする.
                      -&gt; JVM_LoadLibrary()
                         -&gt; os::dll_load()
                            * Linux の場合
                              -&gt; dlopen()
                            * Solaris の場合
                              -&gt; dlopen()
                            * Windows の場合
                              -&gt; LoadLibrary()
                  (1) もし JNI_OnLoad シンボルが含まれていればそれを呼び出す.
                      -&gt; JVM_FindLibraryEntry()
                         -&gt; os::dll_lookup()
                            * Linux の場合
                              -&gt; dlsym()
                            * Solaris の場合
                              -&gt; dlsym()
                            * Windows の場合
                              -&gt; GetProcAddress()
                  (1) ロードしたライブラリの JNI version をチェックする.
                      -&gt; JVM_IsSupportedJNIVersion()
                         -&gt; Threads::is_supported_jni_version_including_1_1()
                            -&gt; Threads::is_supported_jni_version()
                  (1) もし何かエラーが発生していたり, jni version が合わなかった場合はアンロード.
                      -&gt; JVM_UnloadLibrary()
                         -&gt; os::dll_unload()
                            * Linux の場合
                              -&gt; dlclose()
                            * Solaris の場合
                              -&gt; dlclose()
                            * Windows の場合
                              -&gt; FreeLibrary()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### java.lang.System.loadLibrary()
See: [here](no1711993k.html) for details
### java.lang.Runtime.loadLibrary0()
See: [here](no17119KCr.html) for details
### java.lang.ClassLoader.loadLibrary()
See: [here](no17119XMx.html) for details
### java.lang.ClassLoader.findLibrary()
See: [here](no17119WgG.html) for details
### java.lang.ClassLoader.loadLibrary0()
See: [here](no17119JWA.html) for details
### java.lang.ClassLoader$nativeLibraryContext.push()
(#Under Construction)

### java.lang.ClassLoader$nativeLibraryContext.pop()
(#Under Construction)

### Java_java_lang_ClassLoader_00024NativeLibrary_load()
See: [here](no17119jqM.html) for details
### initIDs()
See: [here](no3059QZq.html) for details
### JNU_GetStringPlatformChars()
(#Under Construction)

### JVM_LoadLibrary()
See: [here](no17119w0S.html) for details
### os::dll_load()  (Linux の場合)
See: [here](no17119KQT.html) for details
### os::dll_load()  (Solaris の場合)
See: [here](no17119XaZ.html) for details
### os::dll_load()  (Windows の場合)
See: [here](no3059djw.html) for details
### JVM_FindLibraryEntry()
See: [here](no17119JPM.html) for details
### os::dll_lookup()  (Linux の場合)
See: [here](no17119WZS.html) for details
### os::dll_lookup()  (Solaris の場合)
See: [here](no17119jjY.html) for details
### os::dll_lookup()  (Windows の場合)
See: [here](no3059qt2.html) for details
### JVM_IsSupportedJNIVersion()
See: [here](no17119xnx.html) for details
### Threads::is_supported_jni_version_including_1_1()
See: [here](no17119TBh.html) for details
### Threads::is_supported_jni_version()
See: [here](no17119Gwm.html) for details
### JVM_UnloadLibrary()
See: [here](no171199-Y.html) for details
### os::dll_unload()  (Linux の場合)
See: [here](no17119w7G.html) for details
### os::dll_unload()  (Solaris の場合)
See: [here](no171199FN.html) for details
### os::dll_unload()  (Windows の場合)
See: [here](no3059c3F.html) for details






