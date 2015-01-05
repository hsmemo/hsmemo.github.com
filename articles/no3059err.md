---
layout: default
title: JNI の処理 ： native method の処理 ： native method の dynamic linking 処理 ： 暗黙的なダイナミックリンク  
---
[Up](no3059oyc.html) [Top](../index.html)

#### JNI の処理 ： native method の処理 ： native method の dynamic linking 処理 ： 暗黙的なダイナミックリンク  

--- 
## 概要(Summary)
暗黙的なダイナミック処理は以下の2つの契機で行われる.

  * 該当のネイティブメソッドが呼び出された時
  * 該当のネイティブメソッドが JIT コンパイル対象になった時

どちらの場合も, 実際の処理は NativeLookup クラスの lookup() メソッドで行われる.

NativeLookup 内では, JNI 仕様で定められた名前のネイティブ関数を JNI 仕様で定められた順番で探索する.
見つからなければ, JVMTI で指定された prefix 分を無視するなど何通りかの可能性を考慮して, 複数回探索を行う.

この探索処理は, (システムクラスか否か等に応じて少し変わるが)
一般的には Java の ClassLoader オブジェクトの findNative() メソッドで行われ,
どの場合も最後は os::dll_lookup() によって関数ポインタが探索される.

なお, 最後まで見つからなかった場合は UnsatisfiedLinkError になる.

## 備考(Notes)
* JNI で指定された関数名は `Java_${mangled fully-qualified class name}_${mangled method name}`.

  (オーバーロードされたメソッドでは, それぞれを区別できるように, この後に __${mangled argument signature} が続く)

  探索の順番としては, まず "__${mangled argument signature}" 部分のない短い名前を探索し,
  見つからなければ "__${mangled argument signature}" まで付けた長い名前で探索することになっている.

  (参考: <http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/design.html#wp615> (See: Resolving Native Method Names))

* ネイティブ関数には os 固有の prefix/suffix がついていることがある.
  そのため, まず os 固有の prefix/suffix を付けた状態で探索し, 駄目なら prefix/suffix を外した状態で探索している.

  (なお, HotSpot で対応しているのは, 32bit Windows での _stdcall 用に prefix/suffix のみ)

  参考: x86 Windows 環境では JNICALL マクロが __stdcall に #define されている

```cpp
    ((cite: hotspot/src/cpu/x86/vm/jni_x86.h))
    #if defined(SOLARIS) || defined(LINUX)
    ...
    #else
    ...
      #define JNICALL __stdcall
```


* 一部のシステムクラスについては, 対応付けがハードコーディングされているものがある.
  例えば, Java_sun_misc_Unsafe_registerNatives なら JVM_RegisterUnsafeMethods(), 等.
  (See: lookup_special_native())

  この対応づけ情報は, lookup_special_native_methods という配列に納められている.

```cpp
    ((cite: hotspot/src/share/vm/prims/nativeLookup.cpp))
    static JNINativeMethod lookup_special_native_methods[] = {
      // Next two functions only exist for compatibility with 1.3.1 and earlier.
      { CC"Java_java_io_ObjectOutputStream_getPrimitiveFieldValues",   NULL, FN_PTR(JVM_GetPrimitiveFieldValues)     },  // intercept ObjectOutputStream getPrimitiveFieldValues for faster serialization
      { CC"Java_java_io_ObjectInputStream_setPrimitiveFieldValues",    NULL, FN_PTR(JVM_SetPrimitiveFieldValues)     },  // intercept ObjectInputStream setPrimitiveFieldValues for faster serialization
    
      { CC"Java_sun_misc_Unsafe_registerNatives",                      NULL, FN_PTR(JVM_RegisterUnsafeMethods)       },
      { CC"Java_java_lang_invoke_MethodHandleNatives_registerNatives", NULL, FN_PTR(JVM_RegisterMethodHandleMethods) },
      { CC"Java_sun_misc_Perf_registerNatives",                        NULL, FN_PTR(JVM_RegisterPerfMethods)         }
    };
```

## 処理の流れ (概要)(Execution Flows : Summary)
### 該当のネイティブメソッドが呼び出された時の処理
<div class="flow-abst"><pre>
(See: <a href="no3059asZ.html">here</a> for details)
-&gt; InterpreterRuntime::prepare_native_call()
   -&gt; NativeLookup::lookup()
      -&gt; NativeLookup::lookup_base()
         -&gt; (1) まず, JVMTI で指定された prefix を除去しない状態で探索してみる.
                -&gt; NativeLookup::lookup_entry()
       
                   この中で, NativeLookup::lookup_style() を最大4回呼び出す.
                   (1) まず &quot;__${mangled argument signature}&quot; 部分のない短い名前を探索.
                   (1) 次に &quot;__${mangled argument signature}&quot; まで付けた長い名前で探索.
                   (1) 次に, os 固有の prefix/suffix を付けずに, &quot;__${mangled argument signature}&quot; 部分のない短い名前を探索.
                   (1) 次に, os 固有の prefix/suffix を付けずに, &quot;__${mangled argument signature}&quot; まで付けた長い名前で探索.
       
                   -&gt; NativeLookup::lookup_style()
                      -&gt; (1) システムクラスの場合には, 最初に以下の関数で探索しておく
                             (lookup_special_native() で調べた後, なければ libjava 内から探索する)
                             -&gt; lookup_special_native()
                             -&gt; os::dll_lookup()
                         (1) Java の ClassLoader オブジェクトの findNative() メソッドで探索する.
                             -&gt; JavaCalls::call_static()
                                -&gt; (See: <a href="no3059iJu.html">here</a> for details)
                                   -&gt; java.lang.ClassLoader.findNative()
                                      -&gt; java.lang.ClassLoader$NativeLibrary.find()
                                          -&gt; Java_java_lang_ClassLoader_00024NativeLibrary_find()
                                             -&gt; JVM_FindLibraryEntry()
                                                 -&gt; os::dll_lookup()
                         (1) 見つからなければ, JVMTI agent としてロードしたライブラリ内を探索する.
                             -&gt; os::dll_lookup()
       
            (1) 見つからなければ, JVMTI で指定された prefix を除去した名前で再度探索する.
                -&gt; NativeLookup::lookup_entry_prefixed()
                   -&gt; NativeLookup::lookup_entry()
                      -&gt; (同上)
    
      -&gt; methodOopDesc::set_native_function()
</pre></div>

### 該当のネイティブメソッドが JIT コンパイルされた時の処理
<div class="flow-abst"><pre>
(See: <a href="no293548G.html">here</a> for details)
-&gt; CompileBroker::compile_method()
   -&gt; NativeLookup::lookup()
      -&gt; (同上)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### NativeLookup::lookup()
See: [here](no17119sNy.html) for details
### NativeLookup::lookup_base()
See: [here](no17119rhH.html) for details
### NativeLookup::lookup_entry()
See: [here](no171194rN.html) for details
### NativeLookup::pure_jni_name()
(#Under Construction)

### NativeLookup::long_jni_name()
(#Under Construction)

### NativeLookup::lookup_style()
See: [here](no17119F2T.html) for details
### os::print_jni_name_prefix_on()  (Linux の場合)
See: [here](no30592LS.html) for details
### os::print_jni_name_suffix_on()  (Linux の場合)
See: [here](no3059DWY.html) for details
### os::print_jni_name_prefix_on()  (Solaris の場合)
See: [here](no3059Qge.html) for details
### os::print_jni_name_suffix_on()  (Solaris の場合)
See: [here](no3059dqk.html) for details
### os::print_jni_name_prefix_on()  (Windows の場合)
See: [here](no3059q0q.html) for details
### os::print_jni_name_suffix_on()  (Windows の場合)
See: [here](no30593-w.html) for details
### lookup_special_native()
See: [here](no17119jck.html) for details
### os::native_java_library()
See: [here](no17119kkf.html) for details
### os::dll_lookup()  (Linux の場合)
See: [here](no17119WZS.html) for details
### os::dll_lookup()  (Solaris の場合)
See: [here](no17119jjY.html) for details
### os::dll_lookup()  (Windows の場合)
See: [here](no3059qt2.html) for details
### java.lang.ClassLoader.findNative()
See: [here](no171199ww.html) for details
### Java_java_lang_ClassLoader_00024NativeLibrary_find()
See: [here](no17119K72.html) for details
### JVM_FindLibraryEntry()
See: [here](no17119JPM.html) for details
### os::dll_lookup()  (Linux の場合)
See: [here](no17119WZS.html) for details
### os::dll_lookup()  (Solaris の場合)
See: [here](no17119jjY.html) for details
### os::dll_lookup()  (Windows の場合)
See: [here](no3059qt2.html) for details
### NativeLookup::lookup_entry_prefixed()
See: [here](no171194yB.html) for details






