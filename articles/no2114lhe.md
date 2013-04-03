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
void JavaThread::send_thread_stop(oop java_throwable)  {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Thread::current()->is_VM_thread(), "should be in the vm thread");
	  assert(Threads_lock->is_locked(), "Threads_lock should be locked by safepoint code");
	  assert(SafepointSynchronize::is_at_safepoint(), "all threads are stopped");
	
  {- -------------------------------------------
  (1) 対象が CompilerThread の場合には, 何もせずにここでリターン.
      (CompilerThread に対しては非同期で例外を埋め込んだりすべきでない, とのこと.
       そもそも JavaThread のサブクラスになるべきでないので 1.4.2 で直す, 的なことが書かれているようだが...)
      ---------------------------------------- -}

	  // Do not throw asynchronous exceptions against the compiler thread
	  // (the compiler thread should not be a Java thread -- fix in 1.4.2)
	  if (is_Compiler_thread()) return;
	
  {- -------------------------------------------
  (1) JavaThread::set_pending_async_exception() で, 
      引数で指定された例外(java_throwable)を
      対象のスレッド内に発生させる.
    
      (ただし, 対象のスレッド内にすでに ThreadDeath 例外が起きている場合には, 何もしない)
    
      (なお, 対象のスレッドがスタックフレームの一番先頭に runtime stub を持っている場合には, 
       compiled code から OptoRuntime の呼び出しを行っている最中ということになる.
       この場合には, Deoptimization::deoptimize() で runtime stub を呼び出したフレームを deopt している.
       これは, 幾つかの runtime stub (new, monitor_exit, etc) については
       compiled code の exception handler table が正しくないため.)
  
      (なお, JavaThread::set_pending_async_exception() の呼び出しを行った場合には
       ついでに(トレース出力)も出す.)
      ---------------------------------------- -}

	  {
	    // Actually throw the Throwable against the target Thread - however
	    // only if there is no thread death exception installed already.
	    if (_pending_async_exception == NULL || !_pending_async_exception->is_a(SystemDictionary::ThreadDeath_klass())) {
	      // If the topmost frame is a runtime stub, then we are calling into
	      // OptoRuntime from compiled code. Some runtime stubs (new, monitor_exit..)
	      // must deoptimize the caller before continuing, as the compiled  exception handler table
	      // may not be valid
	      if (has_last_Java_frame()) {
	        frame f = last_frame();
	        if (f.is_runtime_frame() || f.is_safepoint_blob_frame()) {
	          // BiasedLocking needs an updated RegisterMap for the revoke monitors pass
	          RegisterMap reg_map(this, UseBiasedLocking);
	          frame compiled_frame = f.sender(&reg_map);
	          if (compiled_frame.can_be_deoptimized()) {
	            Deoptimization::deoptimize(this, compiled_frame, &reg_map);
	          }
	        }
	      }
	
	      // Set async. pending exception in thread.
	      set_pending_async_exception(java_throwable);
	
	      if (TraceExceptions) {
	       ResourceMark rm;
	       tty->print_cr("Pending Async. exception installed of type: %s", instanceKlass::cast(_pending_async_exception->klass())->external_name());
	      }
	      // for AbortVMOnException flag
	      NOT_PRODUCT(Exceptions::debug_check_abort(instanceKlass::cast(_pending_async_exception->klass())->external_name()));
	    }
	  }
	
	
  {- -------------------------------------------
  (1) 処理対象のスレッドがブロックしていたら
      例外を埋め込んでもすぐには気づかない可能性があるので, 
      念のため Thread::interrupt() を呼んでおく.
      ---------------------------------------- -}

	  // Interrupt thread so it will wake up from a potential wait()
	  Thread::interrupt(this);
	}
	
```


