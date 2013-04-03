---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp
### 説明(description)

```
// Slow path when the native==>VM/Java barriers detect a safepoint is in
// progress or when _suspend_flags is non-zero.
// Current thread needs to self-suspend if there is a suspend request and/or
// block if a safepoint is in progress.
// Async exception ISN'T checked.
// Note only the ThreadInVMfromNative transition can call this function
// directly and when thread state is _thread_in_native_trans
```

### 名前(function name)
```
void JavaThread::check_safepoint_and_suspend_for_native_trans(JavaThread *thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(thread->thread_state() == _thread_in_native_trans, "wrong state");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaThread *curJT = JavaThread::current();
	  bool do_self_suspend = thread->is_external_suspend();
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!curJT->has_last_Java_frame() || curJT->frame_anchor()->walkable(), "Unwalkable stack in native->vm transition");
	
  {- -------------------------------------------
  (1) JavaThread::is_external_suspend() の値 (以下の do_self_suspend) を確認し, 
      もし suspend されていた場合は 
      JavaThread::java_suspend_self() を呼んで処理対象のスレッドを待機させる.
    
      (ただし, AllowJNIEnvProxy オプションが指定されている場合は, 処理対象のスレッドがカレントスレッドでなければこの処理は行わない)
    
      (JavaThread::java_suspend_self() による待機が解けた後は, JavaThreadState を待機前の状態に戻す.
       さらに, MP 環境の場合には
       VM Thread 側への visibility (sequential consistency) を保証する必要があるので, 
       変更後に以下のどちらかを行っている (See: SafepointSynchronize::begin()).
       * UseMembar オプションが指定されている場合:
         OrderAccess::fence() でメモリバリアを張る.
       * そうではない場合: 
         InterfaceSupport::serialize_memory() で serialize page への書き込みを行う.)
      ---------------------------------------- -}

	  // If JNIEnv proxies are allowed, don't self-suspend if the target
	  // thread is not the current thread. In older versions of jdbx, jdbx
	  // threads could call into the VM with another thread's JNIEnv so we
	  // can be here operating on behalf of a suspended thread (4432884).
	  if (do_self_suspend && (!AllowJNIEnvProxy || curJT == thread)) {
	    JavaThreadState state = thread->thread_state();
	
	    // We mark this thread_blocked state as a suspend-equivalent so
	    // that a caller to is_ext_suspend_completed() won't be confused.
	    // The suspend-equivalent state is cleared by java_suspend_self().
	    thread->set_suspend_equivalent();
	
	    // If the safepoint code sees the _thread_in_native_trans state, it will
	    // wait until the thread changes to other thread state. There is no
	    // guarantee on how soon we can obtain the SR_lock and complete the
	    // self-suspend request. It would be a bad idea to let safepoint wait for
	    // too long. Temporarily change the state to _thread_blocked to
	    // let the VM thread know that this thread is ready for GC. The problem
	    // of changing thread state is that safepoint could happen just after
	    // java_suspend_self() returns after being resumed, and VM thread will
	    // see the _thread_blocked state. We must check for safepoint
	    // after restoring the state and make sure we won't leave while a safepoint
	    // is in progress.
	    thread->set_thread_state(_thread_blocked);
	    thread->java_suspend_self();
	    thread->set_thread_state(state);
	    // Make sure new state is seen by VM thread
	    if (os::is_MP()) {
	      if (UseMembar) {
	        // Force a fence between the write above and read below
	        OrderAccess::fence();
	      } else {
	        // Must use this rather than serialization page in particular on Windows
	        InterfaceSupport::serialize_memory(thread);
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) もし Safepoint が開始されていたら (= SafepointSynchronize::do_call_back() が true ならば), 
      SafepointSynchronize::block() を呼んでブロックする.
      ---------------------------------------- -}

	  if (SafepointSynchronize::do_call_back()) {
	    // If we are safepointing, then block the caller which may not be
	    // the same as the target thread (see above).
	    SafepointSynchronize::block(curJT);
	  }
	
  {- -------------------------------------------
  (1) #TODO
      ---------------------------------------- -}

	  if (thread->is_deopt_suspend()) {
	    thread->clear_deopt_suspend();
	    RegisterMap map(thread, false);
	    frame f = thread->last_frame();
	    while ( f.id() != thread->must_deopt_id() && ! f.is_first_frame()) {
	      f = f.sender(&map);
	    }
	    if (f.id() == thread->must_deopt_id()) {
	      thread->clear_must_deopt_id();
	      f.deoptimize(thread);
	    } else {
	      fatal("missed deoptimization!");
	    }
	  }
	}
	
```


