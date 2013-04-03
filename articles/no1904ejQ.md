---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/javaCalls.cpp

### 名前(function name)
```
JavaCallWrapper::JavaCallWrapper(methodHandle callee_method, Handle receiver, JavaValue* result, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaThread* thread = (JavaThread *)THREAD;
	  bool clear_pending_exception = true;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee(thread->is_Java_thread(), "crucial check - the VM thread cannot and must not escape to Java code");
	  assert(!thread->owns_locks(), "must release all locks when leaving VM");
	  guarantee(!thread->is_Compiler_thread(), "cannot make java calls from the compiler");

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _result   = result;
	
  {- -------------------------------------------
  (1) 新しい JNI ローカル参照フレームを作成.
      ---------------------------------------- -}

	  // Allocate handle block for Java code. This must be done before we change thread_state to _thread_in_Java_or_stub,
	  // since it can potentially block.
	  JNIHandleBlock* new_handles = JNIHandleBlock::allocate_block(thread);
	
  {- -------------------------------------------
  (1) JavaThreadState を _thread_in_vm から _thread_in_Java に変更する.
      ---------------------------------------- -}

	  // After this, we are official in JavaCode. This needs to be done before we change any of the thread local
	  // info, since we cannot find oops before the new information is set up completely.
	  ThreadStateTransition::transition(thread, _thread_in_vm, _thread_in_Java);
	
  {- -------------------------------------------
  (1) JavaThread::has_special_runtime_exit_condition() を呼んで
      処理対象のスレッドが例外を出していたりサスペンドされたりしていないかチェックしておく.
      もし何か起きていたら, JavaThread::handle_special_runtime_exit_condition() を呼んで対処する.
      ---------------------------------------- -}

	  // Make sure that we handle asynchronous stops and suspends _before_ we clear all thread state
	  // in JavaCallWrapper::JavaCallWrapper(). This way, we can decide if we need to do any pd actions
	  // to prepare for stop/suspend (flush register windows on sparcs, cache sp, or other state).
	  if (thread->has_special_runtime_exit_condition()) {
	    thread->handle_special_runtime_exit_condition();
	    if (HAS_PENDING_EXCEPTION) {
	      clear_pending_exception = false;
	    }
	  }
	
	
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  // Make sure to set the oop's after the thread transition - since we can block there. No one is GC'ing
	  // the JavaCallWrapper before the entry frame is on the stack.
	  _callee_method = callee_method();
	  _receiver = receiver();
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef CHECK_UNHANDLED_OOPS 時にのみ実行) (See: UnhandledOops)
      ---------------------------------------- -}

	#ifdef CHECK_UNHANDLED_OOPS
	  THREAD->allow_unhandled_oop(&_callee_method);
	  THREAD->allow_unhandled_oop(&_receiver);
	#endif // CHECK_UNHANDLED_OOPS
	
  {- -------------------------------------------
  (1) フィールドの初期化
      (_handles は, 現在の JNI ローカル参照フレーム. ここで待避しておく)
      ---------------------------------------- -}

	  _thread       = (JavaThread *)thread;
	  _handles      = _thread->active_handles();    // save previous handle block & Java frame linkage
	
  {- -------------------------------------------
  (1) フィールドの初期化
      (現在の JavaFrameAnchor の値を待避しておく)
    
      (なお, プロファイラが見ているので last_Java_frame が一時的にでも不正な状態になるとまずい, とのこと)
      ---------------------------------------- -}

	  // For the profiler, the last_Java_frame information in thread must always be in
	  // legal state. We have no last Java frame if last_Java_sp == NULL so
	  // the valid transition is to clear _last_Java_sp and then reset the rest of
	  // the (platform specific) state.
	
	  _anchor.copy(_thread->frame_anchor());
	  _thread->frame_anchor()->clear();
	
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      ---------------------------------------- -}

	  debug_only(_thread->inc_java_call_counter());

  {- -------------------------------------------
  (1) 新しい JNI ローカル参照フレームをセットする
      ---------------------------------------- -}

	  _thread->set_active_handles(new_handles);     // install new handle block and reset Java frame linkage
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert (_thread->thread_state() != _thread_in_native, "cannot set native pc to NULL");
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // clear any pending exception in thread (native calls start with no exception pending)
	  if(clear_pending_exception) {
	    _thread->clear_pending_exception();
	  }
	
  {- -------------------------------------------
  (1) もし今回の呼び出しが
      スタックフレーム上での最初の Java メソッドのフレームになる場合は, 
      JavaThread::record_base_of_stack_pointer() を呼んで
      スタック領域関連のフィールドを初期化しておく.
      ---------------------------------------- -}

	  if (_anchor.last_Java_sp() == NULL) {
	    _thread->record_base_of_stack_pointer();
	  }
	}
	
```


