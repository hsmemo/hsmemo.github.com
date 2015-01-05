---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止する側の処理
---
[Up](no7882dWU.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止する側の処理

--- 
## 概要(Summary)
Safepoint 停止に関する機能は SafepointSynchronize クラスの以下のメソッドに定義されている (See: SafepointSynchronize).

* SafepointSynchronize::begin() を呼ぶと safepoint 処理が開始される
* SafepointSynchronize::end() を呼ぶと safepoint 処理が終了する.

## 備考(Notes)
SafepointSynchronize::begin() と SafepointSynchronize::end() は
(ほとんど) VMThread からしか呼び出されない.
ただし, 例外的に ConcurrentGCThread からは呼ばれるパスが存在する.

(一瞬だけ stop the world しないといけない処理のために使っている模様)
(See: ConcurrentGCThread::stopWorldAndDo())

## 処理の流れ (概要)(Execution Flows : Summary)
### SafepointSynchronize::begin() の処理
<div class="flow-abst"><pre>
SafepointSynchronize::begin()
-&gt; (1) Threads_lock をロックしておく. (これは SafepointSynchronize::end() の中で開放する)
       -&gt; Monitor::lock()

   (1) SafepointSynchronize::_state を _synchronizing に変更

   (1) serialize page のメモリプロテクションを変化させる.
       -&gt; os::serialize_thread_states()

   (1) インタープリタ実行中のスレッドを止めるための処理を行う (dispatch table を Safepoint 用のものに置き換える, 等)
       -&gt; Interpreter::notice_safepoints() (またはそれをサブクラスがオーバーライドしたもの)

   (1) Safepoint Polling page をアクセス不可にしておく.
       -&gt; os::make_polling_page_unreadable()

   (1) 全ての JavaThread の ThreadSafepointState が running 以外の状態に変わるまで待機
       -&gt; ThreadSafepointState::examine_state_of_thread()
       -&gt; ThreadSafepointState::is_running()
       -&gt; SpinPause()  or  os::NakedYield()  or  os::yield_all()

   (1) _waiting_to_block が 0 になるまで待機
       -&gt; Monitior::wait()  (Safepoint_lock に対して. 起こす処理は SafepointSynchronize::block() で行われる)

   (1) SafepointSynchronize::_state を _synchronized に変更

   (1) 様々なクリーンアップ処理を実行
       -&gt; SafepointSynchronize::do_cleanup_tasks()
          -&gt; ObjectSynchronizer::deflate_idle_monitors()
          -&gt; InlineCacheBuffer::update_inline_caches()
          -&gt; CompilationPolicy::do_safepoint_work() (を各サブクラスがオーバーライドしたもの)
          -&gt; NMethodSweeper::scan_stacks()
</pre></div>

### SafepointSynchronize::end() の処理
<div class="flow-abst"><pre>
SafepointSynchronize::end()
-&gt; (1) Safepoint Polling page をアクセス可能な状態に戻す
       -&gt; os::make_polling_page_readable()

   (1) インタープリタ実行中のスレッドを再開させるための処理を行う (dispatch table を通常時用のものに戻す, 等)
       -&gt; Interpreter::ignore_safepoints() (またはそれをサブクラスがオーバーライドしたもの)

   (1) SafepointSynchronize::_state を _not_synchronized に戻す.

   (1) 全ての JavaThread を起床させる.
       -&gt; ThreadSafepointState::restart()

   (1) Threads_lock のロックを解除する.
       -&gt; Monitor::unlock()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### SafepointSynchronize::begin()
See: [here](no7882bMJ.html) for details
### SafepointSynchronize::deferred_initialize_stat()
See: [here](no7882D7T.html) for details
### os::serialize_thread_states()
See: [here](no7882QMO.html) for details
### TemplateInterpreter::notice_safepoints()
See: [here](no7882qZm.html) for details
### copy_table()
See: [here](no78823js.html) for details
### CppInterpreter::notice_safepoints()
See: [here](no7882dPg.html) for details
### os::make_polling_page_unreadable()  (Linux の場合)
See: [here](no7882Euy.html) for details
### os::make_polling_page_unreadable()  (Solaris の場合)
See: [here](no788223B.html) for details
### os::make_polling_page_unreadable()  (Windows の場合)
See: [here](no7882DCI.html) for details
### ThreadSafepointState::is_running()
See: [here](no7882R_s.html) for details
### ThreadSafepointState::examine_state_of_thread()
See: [here](no7882eJz.html) for details
### SafepointSynchronize::safepoint_safe()
See: [here](no7882SVQ.html) for details
### ThreadSafepointState::roll_forward()
See: [here](no7882QTC.html) for details
### SafepointSynchronize::signal_thread_at_safepoint()
See: [here](no7882ffW.html) for details
### ThreadSafepointState::set_has_called_back()
See: [here](no7882spc.html) for details
### SpinPause()  (Linux x86-32 の場合)
See: [here](no24825QTB.html) for details
### SpinPause()  (Linux x86-64 の場合)
See: [here](no24825R_r.html) for details
### SpinPause()  (Linux Sparc の場合)
(#Under Construction)

### SpinPause()  (Linux zero の場合)
See: [here](no24825ddH.html) for details
### SpinPause()  (Solaris Sparc の場合)
See: [here](no248253qf.html) for details
### SpinPause()  (Solaris x86 の場合)
(#Under Construction)

### SpinPause()  (Windows x86 の場合)
See: [here](no24825E1l.html) for details
### os::NakedYield()  (Linux の場合)
See: [here](no24825QMN.html) for details
### os::NakedYield()  (Solaris の場合)
See: [here](no24825dWT.html) for details
### os::NakedYield()  (Windows の場合)
See: [here](no2114ZRr.html) for details
### os::yield_all()  (Linux の場合)
See: [here](no7882ddI.html) for details
### os::yield_all()  (Solaris の場合)
See: [here](no78823xU.html) for details
### os::yield_all()  (Windows の場合)
See: [here](no7882qnO.html) for details
### RuntimeService::record_safepoint_begin()
(#Under Construction)

### RuntimeService::record_safepoint_synchronized()
(#Under Construction)

### SafepointSynchronize::begin_statistics()
See: [here](no7882E1m.html) for details
### SafepointSynchronize::update_statistics_on_spin_end()
See: [here](no7882QFa.html) for details
### SafepointSynchronize::update_statistics_on_sync_end()
See: [here](no7882qga.html) for details
### SafepointSynchronize::update_statistics_on_cleanup_end()
See: [here](no78823qg.html) for details
### SafepointSynchronize::do_cleanup_tasks()
See: [here](no28916uHl.html) for details
### SimpleThresholdPolicy::do_safepoint_work()
See: [here](no31977PBQ.html) for details
### NonTieredCompPolicy::do_safepoint_work()
See: [here](no3420xRn.html) for details
### SafepointSynchronize::end()
See: [here](no78822wN.html) for details
### SafepointSynchronize::end_statistics()
See: [here](no7882RGh.html) for details
### os::make_polling_page_readable()  (Linux の場合)
See: [here](no7882eQn.html) for details
### os::make_polling_page_readable()  (Solaris の場合)
See: [here](no7882rat.html) for details
### os::make_polling_page_readable()  (Windows の場合)
See: [here](no78824kz.html) for details
### TemplateInterpreter::ignore_safepoints()
See: [here](no788234I.html) for details
### CppInterpreter::ignore_safepoints()
See: [here](no7882quC.html) for details






