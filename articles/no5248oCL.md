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
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; FlatProfiler::engage()
</pre></div>

#### (Linux の場合のみの処理) SR_signum 関係の初期化処理
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; os::init_2()
      -&gt; SR_initialize()
</pre></div>

### FlatProfilerTask による計測処理
<div class="flow-abst"><pre>
FlatProfilerTask::task()
-&gt; (1) ProfileVM オプションが指定されている場合, 以下の計測処理を行う.
       -&gt; FlatProfiler::record_vm_tick()
          -&gt; os::get_thread_pc()
             -&gt; * Linux の場合
                  * 停止させる側
                    -&gt; do_suspend()
                       -&gt; os::Linux::SuspendResume::set_suspend_action()
                       -&gt; pthread_kill()              (&lt;= SR_handler() を起動させて処理を停止させる)
                    -&gt; os::Linux::ucontext_get_pc()
                    -&gt; do_resume()
                       -&gt; os::Linux::SuspendResume::set_suspend_action()
                       -&gt; pthread_kill()              (&lt;= 処理を再開させる)

                  * 停止させられる側
                    -&gt; SR_handler()
                       -&gt; os::Linux::SuspendResume::set_suspended()
                       -&gt; sigsuspend()                (&lt;= 再開させられるまで待機)
                       -&gt; resume_clear_context()

                * Solaris の場合
                  * 停止させる側
                    -&gt; OSThread::Sync_Interrupt_Callback::interrupt()
                       -&gt; OSThread::set_interrupt_callback()
                       -&gt; thr_kill()       (&lt;= シグナルハンドラを起動させる)
                       -&gt; Monitor::wait()  (&lt;= 処理が終わるまで待機)
                       -&gt; OSThread::remove_interrupt_callback()
                    -&gt; GetThreadPC_Callback::addr()

                  * 停止させられる側
                    -&gt; JVM_handle_solaris_signal()
                       -&gt; OSThread::do_interrupt_callbacks_at_interrupt()
                          -&gt; GetThreadPC_Callback::execute()
                          -&gt; OSThread::Sync_Interrupt_Callback::leave_callback()
                             -&gt; Monitor::notify_all()  (&lt;= 停止させる側を起床させる)

                * Windows の場合
                  * 停止させる側
                    -&gt; GetThreadContext()

   (2) 何らかの VM_Operation が実行中の場合, 以下の計測処理を行う.
       -&gt; FlatProfiler::record_vm_operation()

   (3) Safepoint が開始されていない場合, 以下の計測処理を行う.
       -&gt; FlatProfiler::record_thread_ticks()
</pre></div>

### FlatProfiler の終了処理
<div class="flow-abst"><pre>
(See: <a href="no3059oro.html">here</a> for details)
-&gt; before_exit()
   -&gt; FlatProfiler::disengage()
   -&gt; FlatProfiler::print()
</pre></div>

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






