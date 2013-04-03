---
layout: default
title: Serviceability 機能 ： -Xprof によるプロファイル機能  
---
[Up](noOQc_VTg2.html) [Top](../index.html)

#### Serviceability 機能 ： -Xprof によるプロファイル機能  

--- 
## 概要(Summary)
内部的には FlatProfiler クラスおよび FlatProfilerTask クラスによって実現されている
(See: FlatProfiler) (See: FlatProfilerTask).

起動時に FlatProfiler クラスによって FlatProfilerTask オブジェクトが生成され, 
以後は定期間隔で FlatProfilerTask から以下の関数が呼び出される.

  * FlatProfiler::record_vm_tick()       (<= ProfileVM オプションが指定されている場合にのみ呼び出す)
  * FlatProfiler::record_vm_operation()  (<= 何らかの VM_Operation が実行中の場合にのみ呼び出す)
  * FlatProfiler::record_thread_ticks()  (<= Safepoint が開始されていない場合にのみ呼び出す)

これらの関数で集められた統計情報は, 
HotSpot の終了時に FlatProfiler クラスによって出力される.

## 備考(Notes)
FlatProfiler::record_vm_tick() では,
プロファイリング情報取得のタイミングで実行中だった関数名を取得する.

この際には OS 依存の方法で対象のスレッドを停止させている 
(なお, これらのスレッド停止手法は現状ではここでしか使われていない).

  * Linux なら SR_signum を使った suspend/resume
  * Solaris なら OSThread::do_interrupt_callbacks_at_interrupt() によるコールバック機構


## 処理の流れ (概要)(Execution Flows : Summary)
### FlatProfiler の起動処理
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> FlatProfiler::engage()
```

#### (Linux の場合のみの処理) SR_signum 関係の初期化処理
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> os::init_2()
      -> SR_initialize()
```

### FlatProfilerTask による計測処理
```
FlatProfilerTask::task()
-> (1) ProfileVM オプションが指定されている場合, 以下の計測処理を行う.
       -> FlatProfiler::record_vm_tick()
          -> os::get_thread_pc()
             -> * Linux の場合
                  * 停止させる側
                    -> do_suspend()
                       -> os::Linux::SuspendResume::set_suspend_action()
                       -> pthread_kill()              (<= SR_handler() を起動させて処理を停止させる)
                    -> os::Linux::ucontext_get_pc()
                    -> do_resume()
                       -> os::Linux::SuspendResume::set_suspend_action()
                       -> pthread_kill()              (<= 処理を再開させる)

                  * 停止させられる側
                    -> SR_handler()
                       -> os::Linux::SuspendResume::set_suspended()
                       -> sigsuspend()                (<= 再開させられるまで待機)
                       -> resume_clear_context()

                * Solaris の場合
                  * 停止させる側
                    -> OSThread::Sync_Interrupt_Callback::interrupt()
                       -> OSThread::set_interrupt_callback()
                       -> thr_kill()       (<= シグナルハンドラを起動させる)
                       -> Monitor::wait()  (<= 処理が終わるまで待機)
                       -> OSThread::remove_interrupt_callback()
                    -> GetThreadPC_Callback::addr()

                  * 停止させられる側
                    -> JVM_handle_solaris_signal()
                       -> OSThread::do_interrupt_callbacks_at_interrupt()
                          -> GetThreadPC_Callback::execute()
                          -> OSThread::Sync_Interrupt_Callback::leave_callback()
                             -> Monitor::notify_all()  (<= 停止させる側を起床させる)

                * Windows の場合
                  * 停止させる側
                    -> GetThreadContext()

   (2) 何らかの VM_Operation が実行中の場合, 以下の計測処理を行う.
       -> FlatProfiler::record_vm_operation()

   (3) Safepoint が開始されていない場合, 以下の計測処理を行う.
       -> FlatProfiler::record_thread_ticks()
```

### FlatProfiler の終了処理
```
(See: [here](no3059oro.html) for details)
-> before_exit()
   -> FlatProfiler::disengage()
   -> FlatProfiler::print()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### SR_initialize()
See: [here](no5248cyX.html) for details

### FlatProfilerTask::task()
See: [here](no5248CXX.html) for details
### FlatProfiler::record_vm_tick()
See: [here](no5248Phd.html) for details
### os::get_thread_pc() (Linux の場合)
See: [here](no5248p1p.html) for details
### do_suspend() (Linux の場合)
See: [here](no52481TF.html) for details
### do_resume() (Linux の場合)
See: [here](no5248CeL.html) for details
### SR_handler() (Linux の場合)
See: [here](no5248PoR.html) for details
### resume_clear_context() (Linux の場合)
See: [here](no5248p8d.html) for details
### os::get_thread_pc() (Solaris の場合)
See: [here](no52482_v.html) for details

### os::get_thread_pc() (Windows の場合)
See: [here](no5248DK2.html) for details
### FlatProfiler::record_vm_operation()
(#Under Construction)
See: [here](no5248crj.html) for details
### FlatProfiler::record_thread_ticks()
(#Under Construction)

=






