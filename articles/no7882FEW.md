---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp

### 名前(function name)
```
void JavaThread::handle_special_runtime_exit_condition(bool check_asyncs) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JavaThread::is_external_suspend_with_lock() の値 (以下の do_self_suspend) を確認し, 
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

	  //
	  // Check for pending external suspend. Internal suspend requests do
	  // not use handle_special_runtime_exit_condition().
	  // If JNIEnv proxies are allowed, don't self-suspend if the target
	  // thread is not the current thread. In older versions of jdbx, jdbx
	  // threads could call into the VM with another thread's JNIEnv so we
	  // can be here operating on behalf of a suspended thread (4432884).
	  bool do_self_suspend = is_external_suspend_with_lock();
	  if (do_self_suspend && (!AllowJNIEnvProxy || this == JavaThread::current())) {
	    //
	    // Because thread is external suspended the safepoint code will count
	    // thread as at a safepoint. This can be odd because we can be here
	    // as _thread_in_Java which would normally transition to _thread_blocked
	    // at a safepoint. We would like to mark the thread as _thread_blocked
	    // before calling java_suspend_self like all other callers of it but
	    // we must then observe proper safepoint protocol. (We can't leave
	    // _thread_blocked with a safepoint in progress). However we can be
	    // here as _thread_in_native_trans so we can't use a normal transition
	    // constructor/destructor pair because they assert on that type of
	    // transition. We could do something like:
	    //
	    // JavaThreadState state = thread_state();
	    // set_thread_state(_thread_in_vm);
	    // {
	    //   ThreadBlockInVM tbivm(this);
	    //   java_suspend_self()
	    // }
	    // set_thread_state(_thread_in_vm_trans);
	    // if (safepoint) block;
	    // set_thread_state(state);
	    //
	    // but that is pretty messy. Instead we just go with the way the
	    // code has worked before and note that this is the only path to
	    // java_suspend_self that doesn't put the thread in _thread_blocked
	    // mode.
	
	    frame_anchor()->make_walkable(this);
	    java_suspend_self();
	
	    // We might be here for reasons in addition to the self-suspend request
	    // so check for other async requests.
	  }
	
  {- -------------------------------------------
  (1) check_asyncs 引数が true の場合は, 
      JavaThread::check_and_handle_async_exceptions() を呼び出して
      Asynchronous Exception のチェックをしておく.
      (もし Asynchronous Exception が存在していれば, この中で pending_exception にセットされる)
      ---------------------------------------- -}

	  if (check_asyncs) {
	    check_and_handle_async_exceptions();
	  }
	}
	
```


