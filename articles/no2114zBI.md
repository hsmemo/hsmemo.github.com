---
layout: default
title: (#WIP) Thread の suspend/resume 処理 (java.lang.Thread.suspend(), java.lang.Thread.resume() の処理)  
---
[Up](no1IkYYOWe.html) [Top](../index.html)

#### (#WIP) Thread の suspend/resume 処理 (java.lang.Thread.suspend(), java.lang.Thread.resume() の処理)  

--- 
#Under Construction

## 概要(Summary)
(#Under Construction)


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
      // ***************************************************************
      // Suspend and resume support
      // ***************************************************************
      //
      // VM suspend/resume no longer exists - it was once used for various
      // things including safepoints but was deprecated and finally removed
      // in Java 7. Because VM suspension was considered "internal" Java-level
      // suspension was considered "external", and this legacy naming scheme
      // remains.
      //
      // External suspend/resume requests come from JVM_SuspendThread,
      // JVM_ResumeThread, JVMTI SuspendThread, and finally JVMTI
      // ResumeThread. External
      // suspend requests cause _external_suspend to be set and external
      // resume requests cause _external_suspend to be cleared.
      // External suspend requests do not nest on top of other external
      // suspend requests. The higher level APIs reject suspend requests
      // for already suspended threads.
      //
      // The external_suspend
      // flag is checked by has_special_runtime_exit_condition() and java thread
      // will self-suspend when handle_special_runtime_exit_condition() is
      // called. Most uses of the _thread_blocked state in JavaThreads are
      // considered the same as being externally suspended; if the blocking
      // condition lifts, the JavaThread will self-suspend. Other places
      // where VM checks for external_suspend include:
      //   + mutex granting (do not enter monitors when thread is suspended)
      //   + state transitions from _thread_in_native
      //
      // In general, java_suspend() does not wait for an external suspend
      // request to complete. When it returns, the only guarantee is that
      // the _external_suspend field is true.
      //
      // wait_for_ext_suspend_completion() is used to wait for an external
      // suspend request to complete. External suspend requests are usually
      // followed by some other interface call that requires the thread to
      // be quiescent, e.g., GetCallTrace(). By moving the "wait time" into
      // the interface that requires quiescence, we give the JavaThread a
      // chance to self-suspend before we need it to be quiescent. This
      // improves overall suspend/query performance.
      //
      // _suspend_flags controls the behavior of java_ suspend/resume.
      // It must be set under the protection of SR_lock. Read from the flag is
      // OK without SR_lock as long as the value is only used as a hint.
      // (e.g., check _external_suspend first without lock and then recheck
      // inside SR_lock and finish the suspension)
      //
      // _suspend_flags is also overloaded for other "special conditions" so
      // that a single check indicates whether any special action is needed
      // eg. for async exceptions.
      // -------------------------------------------------------------------
      // Notes:
      // 1. The suspend/resume logic no longer uses ThreadState in OSThread
      // but we still update its value to keep other part of the system (mainly
      // JVMTI) happy. ThreadState is legacy code (see notes in
      // osThread.hpp).
      //
      // 2. It would be more natural if set_external_suspend() is private and
      // part of java_suspend(), but that probably would affect the suspend/query
      // performance. Need more investigation on this.
      //
```


suspend 状態になると, 各スレッドの Thread::_suspend_flags というフィールドが 0 ではない値になる.

このフィールドの値の中で _external_suspend と名付けられているビットが 1 になると, 以下の関数が true を返すようになる.
この値を見てスレッドは自分で眠りにつく.

  * JavaThread::is_external_suspend()

    JavaThread::java_suspend_self() 内でチェックされている.

    この関数は transition の際やロックを取る際に呼び出される関数で,
    JavaThread::is_external_suspend() が true なら
    SR_lock に対して wait() して眠りにつく.

    (See: [here](no2114aSy.html) for details)
    (See: JavaThread::check_safepoint_and_suspend_for_native_trans()
          Parker::park()
          JvmtiRawMonitor::raw_enter()
          Monitor::wait()
          ObjectMonitor::enter()
          ObjectMonitor::ReenterI()
          ObjectMonitor::wait())

  * JavaThread::is_suspend_after_native()

    (See: transition_from_native())

なお, この処理に関する排他制御のために Thread::_SR_lock というロックが用意されている.

## 備考(Notes)
なお, Thread::_suspend_flags フィールドには
Thread.suspend()/Thread.resume() 以外の原因による suspend 状態も記録されている.

そのため, _suspend_flags はそれぞれの種別ごとの bit を集めた bit map になっている.
Thread.suspend()/Thread.resume() では, その中で _external_suspend と名づけられている bit のみを使用する.


## 処理の流れ (概要)(Execution Flows : Summary)
### suspend の処理
```
java.lang.Thread.suspend()
-> java.lang.Thread.suspend0()
   -> JVM_SuspendThread()
      -> JavaThread::set_external_suspend()
         -> Thread::set_suspend_flag()     (_external_suspend bit を立てる)
      -> JavaThread::java_suspend()
         -> きちんと止めるために, VM_ForceSafepoint による safepoint 停止が行われる.
            (強制的にここで transition させて suspend 処理に引っ掛ける)
```

### resume の処理
```
java.lang.Thread.resume()
-> java.lang.Thread.resume0()
   -> JVM_ResumeThread()
      -> JavaThread::java_resume()
         -> JavaThread::clear_external_suspend()
            -> Thread::clear_suspend_flag()  (_external_suspend bit をクリアする)
         -> JavaThread::clear_ext_suspended()
         -> Monitor::notify_all()     (<= SR_lock に対して notify_all し, suspend していたものを全て起こす)
```


## 処理の流れ (詳細)(Execution Flows : Details)







