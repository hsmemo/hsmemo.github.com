---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： スレッドの状態遷移処理
---
[Up](noadKcOM5n.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： スレッドの状態遷移処理

--- 
## 概要(Summary)
safepoint 停止処理は, 多くの場合, スレッドの状態が遷移する際に行われる
(つまり, 何らかの状態遷移が起こった際に Safepoint 処理が開始されていれば, そのスレッドは停止する).

スレッドの状態は JavaThreadState で表現されており,
以下のクラスがその遷移処理を担当している
(これらのクラスのコンストラクタとデストラクタで遷移処理が行われる).

  * ThreadInVMfromJava
  * ThreadInVMfromUnknown
  * ThreadInVMfromNative
  * ThreadToNativeFromVM
  * ThreadBlockInVM
  * ThreadInVMfromJavaNoAsyncException

## 備考(Notes)
これらのクラスは, 内部的には ThreadStateTransition クラスの以下のメソッドを使用している.

  * ThreadStateTransition::transition()
  * ThreadStateTransition::transition_from_java()
  * ThreadStateTransition::transition_from_native()
  * ThreadStateTransition::transition_and_fence()
  * ThreadStateTransition::trans()
  * ThreadStateTransition::trans_from_java()
  * ThreadStateTransition::trans_from_native()
  * ThreadStateTransition::trans_and_fence()

## 処理の流れ (概要)(Execution Flows : Summary)
### ThreadInVMfromJava の処理
#### コンストラクタ
```
ThreadInVMfromJava::ThreadInVMfromJava()
-> (1) JavaThreadState を _thread_in_vm に変更 (この場合は, 中間的な状態を経ることなく直接遷移)
       -> ThreadStateTransition::trans_from_java()
          -> ThreadStateTransition::transition_from_java()
```

#### デストラクタ
```
ThreadInVMfromJava::~ThreadInVMfromJava()
-> (1) JavaThreadState を _thread_in_Java に戻す. (Safepoint が始まっていればここでブロックする)
       -> ThreadStateTransition::trans()
          -> ThreadStateTransition::transition()
             -> (1) JavaThreadState を引数で指定された値(この場合は _thread_in_vm_trans) に変更
                (2) メモリアクセスの順序づけを行う
                    -> OrderAccess::fence() or os::write_memory_serialize_page() が生成したコード
                (3) もし Safepoint が開始されていたらブロックする
                    -> SafepointSynchronize::block()
                (4) JavaThreadState を引数で指定された値(この場合は _thread_in_Java) に変更
   (1) 処理対象のスレッドが例外を出していたりサスペンドされたりしていないかチェックし, もし何か起きていたら対処する.
       -> JavaThread::has_special_runtime_exit_condition()
       -> JavaThread::handle_special_runtime_exit_condition()
```


### ThreadInVMfromUnknown の処理
#### コンストラクタ
```
ThreadInVMfromUnknown::ThreadInVMfromUnknown()
-> ThreadStateTransition::transition_from_native()    
   (<= ただし, カレントスレッドが JavaThread であり, かつ _thread_in_native の場合にのみこの呼び出しを行う. そうでなければ何もしない)
   -> (1) JavaThreadState を _thread_in_native_trans に変更
      (1) メモリアクセスの順序づけを行う
          -> OrderAccess::fence() or InterfaceSupport::serialize_memory() が生成したコード
      (1) SafepointSynchronize::_state の値を確認し, Safepoint が始まっていればブロックする
          -> SafepointSynchronize::do_call_back()
          -> JavaThread::check_safepoint_and_suspend_for_native_trans()
             -> SafepointSynchronize::block()
      (1) JavaThreadState を引数で指定された値(この場合は _thread_in_vm) に変更
```

#### デストラクタ
```
ThreadInVMfromUnknown::~ThreadInVMfromUnknown()
-> ThreadStateTransition::transition_and_fence()   
   (<= ただし, コンストラクタで状態変更を行った場合のみ, この呼び出しを行う. そうでなければ何もしない)
   -> (1) JavaThreadState を引数で指定された値(この場合は _thread_in_vm_trans) に変更
      (2) メモリアクセスの順序づけを行う
          -> OrderAccess::fence() or InterfaceSupport::serialize_memory() が生成したコード
      (3) もし Safepoint が開始されていたらブロックする
          -> SafepointSynchronize::block()
      (4) JavaThreadState を引数で指定された値(この場合は _thread_in_native) に変更
```


### ThreadInVMfromNative の処理
#### コンストラクタ
```
ThreadInVMfromNative::ThreadInVMfromNative()
-> ThreadStateTransition::trans_from_native()
   -> (1) JavaThreadState を _thread_in_vm に変更
          -> ThreadStateTransition::transition_from_native()
             -> (上述)
                (この場合は, まず _thread_in_native_trans に変更した後,
                Safepoint チェックを経て, 最終的に _thread_in_vm に変更される)
```

#### デストラクタ
```
ThreadInVMfromNative::~ThreadInVMfromNative()
-> ThreadStateTransition::trans_and_fence()
   -> (1) JavaThreadState を _thread_in_native に変更
          -> ThreadStateTransition::transition_and_fence()
             -> (上述)
                (この場合は, まず _thread_in_vm_trans に変更した後,
                Safepoint チェックを経て, 最終的に _thread_in_native に変更される)
```


### ThreadToNativeFromVM の処理
#### コンストラクタ
```
ThreadToNativeFromVM::ThreadToNativeFromVM()
-> (1) スタックフレームを辿れるようにしておく
       -> JavaFrameAnchor::make_walkable()
   (1) JavaThreadState を _thread_in_native に変更.
       -> ThreadStateTransition::trans_and_fence()
          -> (上述)
             (この場合は, まず _thread_in_vm_trans に変更した後,
             Safepoint チェックを経て, 最終的に _thread_in_native に変更される)
   (1) 処理対象のスレッドが例外を出していたりサスペンドされたりしていないかチェックし, もし何か起きていたら対処する.
       -> JavaThread::has_special_runtime_exit_condition()
       -> JavaThread::handle_special_runtime_exit_condition()  (<= 引数を false で呼び出す)
```

#### デストラクタ
```
ThreadToNativeFromVM::~ThreadToNativeFromVM()
-> (1) JavaThreadState を _thread_in_vm に変更
       -> ThreadStateTransition::transition_from_native()
          -> (上述)
             (この場合は, まず _thread_in_native_trans に変更した後,
              Safepoint チェックを経て, 最終的に _thread_in_vm に変更される)
```


### ThreadBlockInVM の処理
#### コンストラクタ
```
ThreadBlockInVM::ThreadBlockInVM()
-> (1) スタックフレームを辿れるようにしておく
       -> JavaFrameAnchor::make_walkable()
   (1) JavaThreadState を _thread_blocked に変更
       -> ThreadStateTransition::trans_and_fence()
          -> (上述)
             (この場合は, まず _thread_in_vm_trans に変更した後,
             Safepoint チェックを経て, 最終的に _thread_blocked に変更される)
```

#### デストラクタ
```
ThreadBlockInVM::~ThreadBlockInVM()
-> (1) JavaThreadState を _thread_in_vm に変更
       -> ThreadStateTransition::trans_and_fence()
          -> (上述)
             (この場合は, まず _thread_blocked_trans に変更した後,
             Safepoint チェックを経て, 最終的に _thread_in_vm に変更される)
```


### ThreadInVMfromJavaNoAsyncException の処理
#### コンストラクタ
```
-> ThreadInVMfromJavaNoAsyncException::ThreadInVMfromJavaNoAsyncException()
   (1) JavaThreadState を _thread_in_vm に変更 (この場合は, 中間的な状態を経ることなく直接遷移)
       -> ThreadStateTransition::trans_from_java()
          -> (上述)
```

#### デストラクタ
```
-> ThreadInVMfromJavaNoAsyncException::~ThreadInVMfromJavaNoAsyncException()
   (1) JavaThreadState を _thread_in_Java に戻す. (Safepoint が始まっていればここでブロックする)
       -> ThreadStateTransition::trans()
          -> (上述)
             (この場合は, まず _thread_in_vm_trans に変更した後,
             Safepoint チェックを経て, 最終的に _thread_in_Java に変更される)
   (1) 処理対象のスレッドが例外を出していたりサスペンドされたりしていないかチェックし, もし何か起きていたら対処する.
       -> JavaThread::has_special_runtime_exit_condition()
       -> JavaThread::handle_special_runtime_exit_condition()  (<= 引数を false で呼び出す)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### ThreadInVMfromJava::ThreadInVMfromJava()
See: [here](no7882F2t.html) for details
### ThreadStateTransition::trans_from_java()
See: [here](no7882EKD.html) for details
### ThreadStateTransition::transition_from_java()
See: [here](no7882RUJ.html) for details
### JavaThread::set_thread_state()
See: [here](no31977pVc.html) for details

### ThreadInVMfromJava::~ThreadInVMfromJava()
See: [here](no7882SA0.html) for details
### ThreadStateTransition::trans()
See: [here](no7882roV.html) for details
### ThreadStateTransition::transition()
See: [here](no78824yb.html) for details
### os::write_memory_serialize_page()
See: [here](no7882sb0.html) for details
### JavaThread::has_special_runtime_exit_condition()
See: [here](no788245P.html) for details
### JavaThread::handle_special_runtime_exit_condition()
See: [here](no7882FEW.html) for details

### ThreadInVMfromUnknown::ThreadInVMfromUnknown()
See: [here](no78827DZ.html) for details

### ThreadInVMfromUnknown::~ThreadInVMfromUnknown()
See: [here](no7882IOf.html) for details

### ThreadInVMfromNative::ThreadInVMfromNative()
See: [here](no788256W.html) for details
### ThreadStateTransition::trans_from_native()
See: [here](no7882GFd.html) for details
### ThreadStateTransition::transition_from_native()
See: [here](no7882TPj.html) for details
### InterfaceSupport::serialize_memory()  (Linux の場合)
See: [here](no78826t1.html) for details
### InterfaceSupport::serialize_memory()  (Solaris の場合)
See: [here](no78825BL.html) for details
### InterfaceSupport::serialize_memory()  (Windows の場合)
See: [here](no7882GMR.html) for details
### os::win32::serialize_fault_filter()
See: [here](no7882TWX.html) for details
### JavaThread::is_suspend_after_native()
See: [here](no7882gZp.html) for details
### JavaThread::check_safepoint_and_suspend_for_native_trans()
See: [here](no7882tjv.html) for details
### ThreadInVMfromNative::~ThreadInVMfromNative()
See: [here](no7882ggd.html) for details
### ThreadStateTransition::trans_and_fence()
See: [here](no7882tqj.html) for details
### ThreadStateTransition::transition_and_fence()
See: [here](no788260p.html) for details

### ThreadToNativeFromVM::ThreadToNativeFromVM()
See: [here](no7882hvM.html) for details

### ThreadToNativeFromVM::~ThreadToNativeFromVM()
See: [here](no7882u5S.html) for details

### ThreadBlockInVM::ThreadBlockInVM()
See: [here](no7882HbA.html) for details

### ThreadBlockInVM::~ThreadBlockInVM()
See: [here](no7882UlG.html) for details

### ThreadInVMfromJavaNoAsyncException::ThreadInVMfromJavaNoAsyncException()
See: [here](no78826CS.html) for details

### ThreadInVMfromJavaNoAsyncException::~ThreadInVMfromJavaNoAsyncException()
See: [here](no7882HNY.html) for details






