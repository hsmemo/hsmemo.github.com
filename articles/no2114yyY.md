---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp

### 名前(function name)
```
int os::sleep(Thread* thread, jlong millis, bool interruptible) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(thread == Thread::current(),  "thread consistency check");
	
  {- -------------------------------------------
  (1) #TODO
      ---------------------------------------- -}

	  // TODO-FIXME: this should be removed.
	  // On Solaris machines (especially 2.5.1) we found that sometimes the VM gets into a live lock
	  // situation with a JavaThread being starved out of a lwp. The kernel doesn't seem to generate
	  // a SIGWAITING signal which would enable the threads library to create a new lwp for the starving
	  // thread. We suspect that because the Watcher thread keeps waking up at periodic intervals the kernel
	  // is fooled into believing that the system is making progress. In the code below we block the
	  // the watcher thread while safepoint is in progress so that it would not appear as though the
	  // system is making progress.
	  if (!Solaris::T2_libthread() &&
	      thread->is_Watcher_thread() && SafepointSynchronize::is_synchronizing() && !Arguments::has_profile()) {
	    // We now try to acquire the threads lock. Since this lock is held by the VM thread during
	    // the entire safepoint, the watcher thread will  line up here during the safepoint.
	    Threads_lock->lock_without_safepoint_check();
	    Threads_lock->unlock();
	  }
	
  {- -------------------------------------------
  (1) (以下の処理は, 処理対象のスレッドが JavaThread か否かに応じて, 2通りに分岐)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下が, JavaThread の場合の処理)
  
      ThreadBlockInVM で JavaThread の状態を変更した後, 
      以下のどちらかの待機処理を行い, 
      結果をリターンする.
      * スリープ時間が 0 以下であれば, thr_yield() システムコールを呼んで yield する. (この場合, 結果は常に 0)
        (なおこちらのケースについては, 最初のバージョンの実装が 
         OSThreadWaitState を使っていなかったので, 現在でも使っていないとのこと)
      * そうでなければ, os_sleep() で眠りにつく. (この場合, 結果は os_sleep() の返値)
        (ついでに, 眠りにつく前に OSThreadWaitState で OSThread の状態変更も行っている)
  
      (なお, 寝ている間に java.lang.Thread.suspend() で suspend 状態にされているかもしれないので, 
       目が覚めた後に JavaThread::check_and_wait_while_suspended() でチェックを行っている.
       もし suspend されていれば, すぐにはリターンせず, resume されるまでこの中で待機する
       (See: java.lang.Thread.suspend()))
  
  
      #TODO  JavaThread::set_suspend_equivalent() はどういう意味がある??
      ---------------------------------------- -}

	  if (thread->is_Java_thread()) {
	    // This is a JavaThread so we honor the _thread_blocked protocol
	    // even for sleeps of 0 milliseconds. This was originally done
	    // as a workaround for bug 4338139. However, now we also do it
	    // to honor the suspend-equivalent protocol.
	
	    JavaThread *jt = (JavaThread *) thread;
	    ThreadBlockInVM tbivm(jt);
	
	    jt->set_suspend_equivalent();
	    // cleared by handle_special_suspend_equivalent_condition() or
	    // java_suspend_self() via check_and_wait_while_suspended()
	
	    int ret_code;
	    if (millis <= 0) {
	      thr_yield();
	      ret_code = 0;
	    } else {
	      // The original sleep() implementation did not create an
	      // OSThreadWaitState helper for sleeps of 0 milliseconds.
	      // I'm preserving that decision for now.
	      OSThreadWaitState osts(jt->osthread(), false /* not Object.wait() */);
	
	      ret_code = os_sleep(millis, interruptible);
	    }
	
	    // were we externally suspended while we were waiting?
	    jt->check_and_wait_while_suspended();
	
	    return ret_code;
	  }
	
  {- -------------------------------------------
  (1) (以下が, JavaThread 以外の場合の処理)
  
      以下のどちらかの処理を行う.
      * スリープ時間が 0 以下であれば, thr_yield() システムコールを呼んで yield した後, リターン.
      * そうでなければ, os_sleep() で眠りについた後, リターン.
        (ついでに, 眠りにつく前に OSThreadWaitState で OSThread の状態変更も行っている)
      ---------------------------------------- -}

	  // non-JavaThread from this point on:
	
	  if (millis <= 0) {
	    thr_yield();
	    return 0;
	  }
	
	  OSThreadWaitState osts(thread->osthread(), false /* not Object.wait() */);
	
	  return os_sleep(millis, interruptible);
	}
	
```


