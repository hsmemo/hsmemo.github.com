---
layout: default
title: HotSpot の起動/終了処理 ： HotSpot の終了処理 ： 正常終了の場合の処理 (= DestroyJavaVM() の処理)  
---
[Up](no28916GoL.html) [Top](../index.html)

#### HotSpot の起動/終了処理 ： HotSpot の終了処理 ： 正常終了の場合の処理 (= DestroyJavaVM() の処理)  

--- 
## 概要(Summary)
(以下の内容はほとんど [HotSpot Runtime Overview](http://openjdk.java.net/groups/hotspot/docs/RuntimeOverview.html#VM%20Lifecycle|outline) の受け売り. こちらも参照のこと)

DestroyJavaVM() による HotSpot の終了処理は以下のようになる.

(なおこの関数は HotSpot を終了させるために launcher によって呼び出される.
 HotSpot 内部で深刻なエラーが起こった場合は, HotSpot 自身によって呼び出されることもある)

  1. 現在のスレッドが唯一のユーザスレッド(non-daemon thread to execute)になるまで待つ

  1. java.lang.Shutdown.shutdown() を呼び出し, 
     shutdown 用のフック関数やfinalizer処理を行う.

  1. before_exit() を呼び出し, 以下の処理を行う.

     1. JVM_OnExit() で登録されたフック関数への準備をする (ただし, JVM_OnExit() は使われていないが...)
     2. Profiler, StatSampler, Watcher, GC threads を停止させる.
     3. JVMTI/PI に static event を投げて, JVMPI を停止させ, Signal スレッドを停止させる.

  1. JavaThread::exit() を呼び出し, 以下の処理を行う
     (この段階で HotSpot は Java のコードが実行不可能な状態になる).

     1. JNI handle block を解放
     2. stack guard pages を解放
     3. このスレッドを Threads list から削除する.

  1. VMThread を停止させる.

     (これにより VM は safepoint 状態になり, Compiler スレッドも停止される)

  1. まだネイティブコードを実行中のスレッドのために, 
     _vm_exited フラグをセットして終了したと通達する

  1. このスレッドを削除する

  1. exit_globals() で IO や PerfMonitor 用のリソースを解放する

  1. リターンする

## 処理の流れ (概要)(Execution Flows : Summary)
(なお, JNI の DestroyJavaVM() 関数は HotSpot 内では jni_DestroyJavaVM() という名前で定義されている)

(実際の処理の大半は Threads::destroy_vm() に実装されている)

```
jni_DestroyJavaVM()
-> jni_AttachCurrentThread()
-> Threads::destroy_vm()         (← この中で実際の終了処理の全てが行われる)
   -> Universe::run_finalizers_on_exit()  or  JavaThread::invoke_shutdown_hooks()
   -> before_exit()
      -> 
   -> JavaThread::exit()
   -> VMThread::wait_for_vm_thread_exit()
   -> VMThread::destroy()
   -> VM_Exit::set_vm_exited()
   -> notify_vm_shutdown()
   -> exit_globals()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### jni_DestroyJavaVM()
See: [here](no4230Kih.html) for details
### Threads::destroy_vm()
See: [here](no4230wNV.html) for details
### Universe::run_finalizers_on_exit()
See: [here](no31977B2C.html) for details
### JavaThread::invoke_shutdown_hooks()
See: [here](no31977OAJ.html) for details
### before_exit()
See: [here](no31977bKP.html) for details
### VMThread::wait_for_vm_thread_exit()
See: [here](no3059_wh.html) for details
### VMThread::is_terminated()
See: [here](no3059mP0.html) for details
### VMThread::destroy()
See: [here](no3059ZFu.html) for details
### VM_Exit::set_vm_exited()
See: [here](no31977oUV.html) for details
### notify_vm_shutdown()
See: [here](no319771eb.html) for details
### exit_globals()
See: [here](no31977Cph.html) for details






