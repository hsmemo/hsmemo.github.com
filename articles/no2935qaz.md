---
layout: default
title: Runtime による Thread の一時停止処理 ： VM Operation の起動処理  
---
[Up](no2480eqy.html) [Top](../index.html)

#### Runtime による Thread の一時停止処理 ： VM Operation の起動処理  

--- 
## 概要(Summary)
VM Operation は, VMThread::execute() が呼び出されることで開始される
(なお, 要求を出される VMThread 自身は, HotSpot の起動時に作成される (See: [here](no-la6kE9R.html) for details)).

生成された VMThread は, VM Operation 要求が来るまで無限ループで待機している
(より正確には, このループ内で VMOperationQueue_lock という Monitor に wait() して眠っている).

VMThread::execute() が呼び出されると,
VMThread::_vm_queue という VMOperationQueue に VM Operation 要求が追加される.
さらに VMOperationQueue_lock に notify() が通知される.
この notify() によって wait() していた VMThread が起床し,
VMThread::_vm_queue から VM Operation オブジェクトを取得し, 
VMThread::evaluate_operation() で VM Operation を実行する.


## 処理の流れ (概要)(Execution Flows : Summary)
### VMThread の処理の流れ
```
VMThread::run()
-> (1) VMThread の初期化を行う
       -> Thread::initialize_thread_local_storage()
       -> Thread::record_stack_base_and_size()
       -> JNIHandleBlock::allocate_block()
       -> Thread::set_active_handles()
       -> os::set_native_priority()

   (1) VMThread のメイン処理を実行
       -> VMThread::loop()
          (以下の処理を無限ループ)
          -> (1) VM Operation 要求が来るまで待機する
                 -> VMOperationQueue::remove_next()
                 -> Monitor::wait()

             (1) Safepoint 停止が必要であれば Safepoint 処理を開始させる
                 -> SafepointSynchronize::begin()   (← 必要に応じて)
                    -> (See: [here](noFCZ0Hp3S.html) for details)

             (1) 取得した VM Operation 要求を実行する
                 (なお safepoint が必要な VM Operation だった場合には,
                 safepoint 停止回数ができるだけ少なくなるように
                 VMThread::_vm_queue 内の他の safepoint を要求する VM Operation も全部実行する.)
                 -> VMThread::evaluate_operation()
                    -> VM_Operation::evaluate()
                       -> VM_Operation::doit()  (← ここで実際の VM Operation 処理が行われる. 
                                                    doit() は各サブクラスでオーバーライドされている)

             (1) Safepoint 停止した場合には Safepoint を解除する
                 -> SafepointSynchronize::end()     (← 必要に応じて)
                    -> (See: [here](noFCZ0Hp3S.html) for details)

             (1) VM Operation 要求を出したスレッドが完了を待っているかもしれないため, 起床させておく
                 -> Monitor::notify_all()

   (1) VMThread の終了処理を行う
       -> SafepointSynchronize::begin()
       -> CompileBroker::set_should_block()
       -> VM_Exit::wait_for_threads_in_native_to_block()
```

### VM Operation 要求を出す側の流れ
```
VMThread::execute()
-> (VMThread自身から呼び出されたかどうかで, 以下の２通りのパスが存在)
   * VMThread 以外から呼び出されたケース (nested VM Operation ではない場合)
     -> VM_Operation::doit_prologue() (をサブクラスがオーバーライドしたもの)
     -> _vm_queue に VM_Operation オブジェクトを追加
     -> VMOperationQueue_lock に対して Monitor::notify() を呼び出す
     -> (完了するまで待つように指定されている VM Operation の場合には)
        VMOperationRequest_lock に対して Monitor::wait() して VM Operation の完了を待つ
     -> (VMThread に delete されない種類の VM_Operation であれば)
        VM_Operation::doit_epilogue() (をサブクラスがオーバーライドしたもの)

   * VMThread から呼び出されたケース (nested VM Operation 等の場合)
     -> SafepointSynchronize::begin()   (← 必要に応じて)
        -> (See: [here](noFCZ0Hp3S.html) for details)
     -> VM_Operation::evaluate()
        -> VM_Operation::doit()  (← ここで実際の VM Operation 処理が行われる. 
                                     doit() は各サブクラスでオーバーライドされている)
     -> SafepointSynchronize::end()     (← 必要に応じて)
        -> (See: [here](noFCZ0Hp3S.html) for details)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### VMThread::run()
See: [here](no3059ymb.html) for details
### Thread::initialize_thread_local_storage()
See: [here](no17119iiR.html) for details
### ThreadLocalStorage::set_thread()
See: [here](no30596Wi.html) for details
### Thread::record_stack_base_and_size()
See: [here](no30597Xp.html) for details
### os::current_stack_base()  (Linux x86 の場合)
See: [here](no3059Iiv.html) for details
### current_stack_region()  (Linux x86 の場合)
See: [here](no3059Vs1.html) for details
### os::Linux::is_initial_thread()
See: [here](no3059H2E.html) for details
### os::Linux::initial_thread_stack_bottom()
See: [here](no3059UAL.html) for details
### os::Linux::initial_thread_stack_size()
See: [here](no3059hKR.html) for details
### os::current_stack_size()  (Linux x86 の場合)
See: [here](no3059uUX.html) for details
### JNIHandleBlock::allocate_block()
See: [here](no3718fXI.html) for details
### Thread::set_active_handles()
See: [here](no3059oAF.html) for details
### os::set_native_priority()  (Linux の場合)
See: [here](no2114xxR.html) for details
### os::set_native_priority()  (Solaris の場合)
See: [here](no2114YQk.html) for details
### set_lwp_priority()
See: [here](no3059KAO.html) for details
### priocntl_stub()
See: [here](no3059kUa.html) for details
### os::set_native_priority()  (Windows の場合)
See: [here](no2114-7X.html) for details

### VMThread::loop()
See: [here](no2480vbZ.html) for details
### VMOperationQueue::remove_next()
(#Under Construction)

### VMThread::should_terminate()
See: [here](no3059M7n.html) for details
### SafepointSynchronize::is_cleanup_needed()
(#Under Construction)

### os::hint_no_preempt() (Linux の場合)
See: [here](no31977Q7i.html) for details
### os::hint_no_preempt() (Solaris の場合)
See: [here](no31977qPv.html) for details
### os::hint_no_preempt() (Windows の場合)
See: [here](no31977dFp.html) for details

### VMThread::evaluate_operation()
See: [here](no24808lf.html) for details
### VM_Operation::evaluate()
See: [here](no344LSi.html) for details
### VMThread::execute()
See: [here](no344Y67.html) for details
### VM_Operation::set_calling_thread()
See: [here](no7882sio.html) for details
### SafepointSynchronize::last_non_safepoint_interval()
See: [here](no7882fYi.html) for details
### CompileBroker::set_should_block()
See: [here](no3059YZD.html) for details
### VM_Exit::wait_for_threads_in_native_to_block()
See: [here](no3059ljJ.html) for details






