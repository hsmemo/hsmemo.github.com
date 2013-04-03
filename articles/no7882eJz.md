---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/safepoint.cpp

### 名前(function name)
```
void ThreadSafepointState::examine_state_of_thread() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(is_running(), "better be running or just have hit safepoint poll");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaThreadState state = _thread->thread_state();
	
  {- -------------------------------------------
  (1) 現在の JavaThreadState の値を _orig_thread_state フィールドに記録しておく.
      ---------------------------------------- -}

	  // Save the state at the start of safepoint processing.
	  _orig_thread_state = state;
	
  {- -------------------------------------------
  (1) 処理対象の JavaThread の suspend_type を, 以下のように変更する.
      (なお, suspend_type は ThreadSafepointState::is_running() に影響する.
       _running 以外の値になった場合, false が返されるようになる.
       See: ThreadSafepointState::is_running())
  
      * self-suspended しているスレッドの場合 (= _thread->is_ext_suspended() が true の場合):
        ThreadSafepointState::roll_forward() を呼んで, _at_safepoint に変更する.
        (ついでに, この場合には _waiting_to_block を減少させる処理も行われる)
  
        (なお, この時点で suspended されていれば, Safepoint 中に resume されることはない.
         これは, resume 処理では Threads_lock を確保しようとするが, 
         Safepoint 中は VMThread が Threads_lock を握りっぱなしにしているため)
    
        (なお, この確認処理で SR_lock を取るとデッドロックする危険性があるのでしてはいけない.
         また, そもそも以下の 2つの理由から SR_lock を取る必要が無い.
         * suspend flags は volatile であり, 変更も Atomic::cmpxchg() でなされるので, 
           ロック無しでも正しい値が見える.
         * この確認処理自体は, SafepointSynchronize::begin() のループ内で呼び出されている.
           というわけで, 何度も確認処理は行われるので, 多少見逃しても問題ない.)
  
      * そのスレッドを停止させても問題ない(あるいは既に停止している)場合 (= SafepointSynchronize::safepoint_safe() が true の場合):
        ThreadSafepointState::roll_forward() を呼んで, _at_safepoint に変更する.
        (ついでに, この場合には _waiting_to_block を減少させる処理も行われる)
  
      * JavaThreadState が _thread_in_vm の場合:
        ThreadSafepointState::roll_forward() を呼んで, _call_back に変更する.
  
      * それ以外の場合:
        変更しない (_running のままとする)
      ---------------------------------------- -}

	  // Check for a thread that is suspended. Note that thread resume tries
	  // to grab the Threads_lock which we own here, so a thread cannot be
	  // resumed during safepoint synchronization.
	
	  // We check to see if this thread is suspended without locking to
	  // avoid deadlocking with a third thread that is waiting for this
	  // thread to be suspended. The third thread can notice the safepoint
	  // that we're trying to start at the beginning of its SR_lock->wait()
	  // call. If that happens, then the third thread will block on the
	  // safepoint while still holding the underlying SR_lock. We won't be
	  // able to get the SR_lock and we'll deadlock.
	  //
	  // We don't need to grab the SR_lock here for two reasons:
	  // 1) The suspend flags are both volatile and are set with an
	  //    Atomic::cmpxchg() call so we should see the suspended
	  //    state right away.
	  // 2) We're being called from the safepoint polling loop; if
	  //    we don't see the suspended state on this iteration, then
	  //    we'll come around again.
	  //
	  bool is_suspended = _thread->is_ext_suspended();
	  if (is_suspended) {
	    roll_forward(_at_safepoint);
	    return;
	  }
	
	  // Some JavaThread states have an initial safepoint state of
	  // running, but are actually at a safepoint. We will happily
	  // agree and update the safepoint state here.
	  if (SafepointSynchronize::safepoint_safe(_thread, state)) {
	      roll_forward(_at_safepoint);
	      return;
	  }
	
	  if (state == _thread_in_vm) {
	    roll_forward(_call_back);
	    return;
	  }
	
	  // All other thread states will continue to run until they
	  // transition and self-block in state _blocked
	  // Safepoint polling in compiled code causes the Java threads to do the same.
	  // Note: new threads may require a malloc so they must be allowed to finish
	
	  assert(is_running(), "examine_state_of_thread on non-running thread");
	  return;
	}
	
```


