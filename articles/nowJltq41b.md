---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： 弱グローバル参照の処理(Weak Global References)
---
[Up](no7882H_v.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： 弱グローバル参照の処理(Weak Global References)

--- 
## 該当する JNI 関数
* `NewWeakGlobalRef`,
* `DeleteWeakGlobalRef`,

## 概要(Summary)
NewWeakGlobalRef() の処理は, 指定された oop を JNIHandles::_weak_global_handles に格納するだけ.

逆に, DeleteWeakGlobalRef() の処理は, JNIHandleBlock 内の対応する箇所に JNIHandles::_deleted_handle の値を書き込むだけ.

(このため, 処理としては NewGlobalRef() や DeleteGlobalRef(), NewLocalRef() や DeleteLocalRef() とほぼ同じ (See: [here](noNzTqB3WT.html) for details))

## 備考(Notes)
WeakGlobalRef が GC 時に NULL になる処理は
ReferenceProcessor::process_phaseJNI() 内で行われている
(See: [here](no289169tf.html) for details).


## 処理の流れ (概要)(Execution Flows : Summary)
### NewWeakGlobalRef() の処理
```
jni_NewWeakGlobalRef()
-> JNIHandles::make_weak_global()
   -> JNIHandleBlock::allocate_handle()
      -> JNIHandleBlock::rebuild_free_list()  (<= どうしても空きが見つからない場合に呼び出す)
```

### DeleteWeakGlobalRef() の処理
```
jni_DeleteWeakGlobalRef()
-> JNIHandles::destroy_weak_global()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_NewWeakGlobalRef()
See: [here](no3718SGO.html) for details
### JNIHandles::make_weak_global()
See: [here](no3718fQU.html) for details
### JNIHandleBlock::allocate_handle()
See: [here](no3718S_Z.html) for details
### JNIHandleBlock::rebuild_free_list()
See: [here](no37185rU.html) for details

### jni_DeleteWeakGlobalRef()
See: [here](no37184xB.html) for details
### JNIHandles::destroy_weak_global()
See: [here](no3718F8H.html) for details






