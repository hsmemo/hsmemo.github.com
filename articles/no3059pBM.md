---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： ネイティブメソッドのダイナミックリンク処理(Registering Native Methods)  
---
[Up](no7882H_v.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： ネイティブメソッドのダイナミックリンク処理(Registering Native Methods)  

--- 
## 該当する JNI 関数
* `RegisterNatives`,
* `UnregisterNatives`,

## 概要(Summary)
ネイティブメソッドのダイナミックリンク処理には 2通りの実行契機(明示的なリンク, 暗黙的なリンク)がある 
(See: [here](no3059oyc.html) for details).
RegisterNatives() は明示的にリンクする場合の処理を行う.

内部的な処理としては, 
RegisterNatives() は指定されたアドレス(関数ポインタ)を methodOop 内にセットするだけ
(なお, 該当の methodOop を探す際に JVMTI の prefix についての処理が必要なので, find_prefixed_native() による処理も行っている).

逆に UnregisterNatives() では, セットしたアドレスを別のアドレスで上書きして飛べなくするだけ
(SharedRuntime::native_method_throw_unsatisfied_link_error_entry() のアドレスで上書きされる).

## 備考(Notes)
jni_RegisterNatives() は, ユーザーから明示的に呼び出された場合以外に, 
HotSpot の内部的な処理からも呼び出されている.

例えば, 標準ライブラリの基本的なクラスのネイティブメソッドは, 
以下に示す関数内で RegisterNatives() を使って明示的に CVMI (JVM_*) 関数にバインドされている.

  * Java_java_lang_Class_registerNatives()
  * Java_java_lang_ClassLoader_registerNatives()
  * Java_java_lang_Compiler_registerNatives()
  * Java_java_lang_Object_registerNatives()
  * Java_java_lang_System_registerNatives()
  * Java_java_lang_Thread_registerNatives()


また, NativeLookup で暗黙的にリンクされる場合(See: [here](no3059oyc.html) for details)でも,
以下に示す一部のメソッドでは
jni_RegisterNatives() を呼び出す初期化用関数にハードコーディングされている.
そしてそれらの関数の中から jni_RegisterNatives() が呼び出される.

(これらのメソッドでは, 
 通常のように `Java_${mangled fully-qualified class name}_${mangled method name}`
 という関数が定義されてその中で jni_RegisterNatives() が呼ばれるのではなく,
 `Java_${mangled fully-qualified class name}_${mangled method name}` が 
 NativeLookup によって直接 JVM_Register*() という CVMI 関数にバインドされ, 
 その中で jni_RegisterNatives() がよばれる) (何故こんな風に2種類のやり方が取られている?? #TODO)

  * Java_sun_misc_Unsafe_registerNatives() => JVM_RegisterUnsafeMethods()
  * Java_java_lang_invoke_MethodHandleNatives_registerNatives() => JVM_RegisterMethodHandleMethods()
  * Java_sun_misc_Perf_registerNatives() => JVM_RegisterPerfMethods()


## 処理の流れ (概要)(Execution Flows : Summary)
### RegisterNatives() の処理
```
jni_RegisterNatives()
-> register_native()
   -> find_prefixed_native()
   -> methodOopDesc::set_native_function() (or methodOopDesc::clear_native_function())
```

### UnregisterNatives() の処理
```
jni_UnregisterNatives()
-> methodOopDesc::clear_native_function()
   -> methodOopDesc::set_native_function()
      (飛び先を SharedRuntime::native_method_throw_unsatisfied_link_error_entry() に設定)
   -> methodOopDesc::clear_code()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_RegisterNatives()
See: [here](no17119D0F.html) for details
### register_native()
See: [here](no17119Q-L.html) for details
### find_prefixed_native()
See: [here](no17119qSY.html) for details
### methodOopDesc::set_native_function()
See: [here](no171193ce.html) for details

### jni_UnregisterNatives()
See: [here](no17119Enk.html) for details
### methodOopDesc::clear_native_function()
See: [here](no17119Rxq.html) for details
### methodOopDesc::clear_code()
See: [here](no17119e7w.html) for details






