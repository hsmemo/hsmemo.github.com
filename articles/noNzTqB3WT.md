---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： ローカル参照/グローバル参照の処理(Global and Local References) ： JNI 参照の生成/破棄
---
[Up](notiXs9FLU.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： ローカル参照/グローバル参照の処理(Global and Local References) ： JNI 参照の生成/破棄

--- 
## 該当する JNI 関数
* `NewGlobalRef`,
* `DeleteGlobalRef`,
* `DeleteLocalRef`,
* `NewLocalRef`,


## 概要(Summary)
NewGlobalRef() や NewLocalRef() の処理は, 指定された oop を
Thread::_active_handles や JNIHandles::_global_handles 内に格納するだけ.

逆に, DeleteGlobalRef() や DeleteLocalRef() の処理は,
JNIHandleBlock 内の対応する箇所に JNIHandles::_deleted_handle の値を書き込むだけ.

(See: JNIHandleBlock)

## 備考(Notes)
* JNI 関数の中には暗黙的に JNI local 参照を生成するものもある.
  そういった関数の内部では JNIHandles::make_local() が呼び出されている.

* JNIHandles::make_local() は, 引数違いのものが (ここで説明するものも含めて) ３種類程ある. 
  中身はどれも同じようなもんだが...

## 処理の流れ (概要)(Execution Flows : Summary)
### NewGlobalRef() の処理
```
jni_NewGlobalRef()
-> JNIHandles::make_global()
   -> JNIHandleBlock::allocate_handle()
      -> JNIHandleBlock::rebuild_free_list()  (<= どうしても空きが見つからない場合に呼び出す)
```

### DeleteGlobalRef() の処理
```
jni_DeleteGlobalRef()
-> JNIHandles::destroy_global()
```

### DeleteLocalRef() の処理
```
jni_DeleteLocalRef()
-> JNIHandles::destroy_local()
```

### NewLocalRef() の処理
```
jni_NewLocalRef()
-> JNIHandles::make_local(JNIEnv* env, oop obj)
   -> JNIHandleBlock::allocate_handle()
      -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### jni_NewGlobalRef()
See: [here](no3718Fuf.html) for details
### JNIHandles::make_global()
See: [here](no3718rgH.html) for details
### JNIHandleBlock::allocate_handle()
See: [here](no3718S_Z.html) for details
### JNIHandleBlock::rebuild_free_list()
See: [here](no37185rU.html) for details

### jni_DeleteGlobalRef()
See: [here](no3718fJg.html) for details
### JNIHandles::destroy_global()
See: [here](no3718sTm.html) for details

### jni_DeleteLocalRef()
See: [here](no37185ds.html) for details
### JNIHandles::destroy_local()
See: [here](no3718Goy.html) for details

### jni_NewLocalRef()
See: [here](no3059COd.html) for details
### JNIHandles::make_local(JNIEnv* env, oop obj)
See: [here](no3059PYj.html) for details






