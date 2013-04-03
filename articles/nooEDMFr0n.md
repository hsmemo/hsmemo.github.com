---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： JNI によるフィールドアクセス ： 初期化時の処理
---
[Up](no5248c5L.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： JNI によるフィールドアクセス ： 初期化時の処理

--- 
## 概要(Summary)
初期化時に呼び出される quicken_jni_functions() の内で,
jni_NativeInterface (チェック無し版の JNIEnv) 内の
Get<Primitive>Field() 関数を高速版に置き換える.

## 処理の流れ (概要)(Execution Flows : Summary)
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> quicken_jni_functions()
      -> JNI_FastGetField::generate_fast_get_<Primitive>_field()
         -> (See: [here](no305911u.html) for details)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### quicken_jni_functions()
See: [here](no1711960P.html) for details






