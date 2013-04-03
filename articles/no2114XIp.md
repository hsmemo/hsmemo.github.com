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
// A JavaThread is a normal Java thread

```

### 名前(function name)
```
void JavaThread::initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) フィールドの初期化
      (それ以外の処理も混じっている?? #TODO)
  
      (なお, ここの set_jni_functions() で 
       jni_functions() が返した値を _jni_environment.functions にセットしている.
       See: [here](no3059pIA.html) for details)
      ---------------------------------------- -}

	  // Initialize fields
	
	  // Set the claimed par_id to -1 (ie not claiming any par_ids)
	  set_claimed_par_id(-1);
	
	  set_saved_exception_pc(NULL);
	  set_threadObj(NULL);
	  _anchor.clear();
	  set_entry_point(NULL);
	  set_jni_functions(jni_functions());
	  set_callee_target(NULL);
	  set_vm_result(NULL);
	  set_vm_result_2(NULL);
	  set_vframe_array_head(NULL);
	  set_vframe_array_last(NULL);
	  set_deferred_locals(NULL);
	  set_deopt_mark(NULL);
	  set_deopt_nmethod(NULL);
	  clear_must_deopt_id();
	  set_monitor_chunks(NULL);
	  set_next(NULL);
	  set_thread_state(_thread_new);
	  _terminated = _not_terminated;
	  _privileged_stack_top = NULL;
	  _array_for_gc = NULL;
	  _suspend_equivalent = false;
	  _in_deopt_handler = 0;
	  _doing_unsafe_access = false;
	  _stack_guard_state = stack_guard_unused;
	  _exception_oop = NULL;
	  _exception_pc  = 0;
	  _exception_handler_pc = 0;
	  _exception_stack_size = 0;
	  _is_method_handle_return = 0;
	  _jvmti_thread_state= NULL;
	  _should_post_on_exceptions_flag = JNI_FALSE;
	  _jvmti_get_loaded_classes_closure = NULL;
	  _interp_only_mode    = 0;
	  _special_runtime_exit_condition = _no_async_condition;
	  _pending_async_exception = NULL;
	  _is_compiling = false;
	  _thread_stat = NULL;
	  _thread_stat = new ThreadStatistics();
	  _blocked_on_compilation = false;
	  _jni_active_critical = 0;
	  _do_not_unlock_if_synchronized = false;
	  _cached_monitor_info = NULL;
	  _parker = Parker::Allocate(this) ;
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifndef PRODUCT 時にのみ実行)
      ---------------------------------------- -}

	#ifndef PRODUCT
	  _jmp_ring_index = 0;
	  for (int ji = 0 ; ji < jump_ring_buffer_size ; ji++ ) {
	    record_jump(NULL, NULL, NULL, 0);
	  }
	#endif /* PRODUCT */
	
  {- -------------------------------------------
  (1) (プロファイル情報取得用の初期化処理) (See: ThreadProfiler)
      ---------------------------------------- -}

	  set_thread_profiler(NULL);
	  if (FlatProfiler::is_active()) {
	    // This is where we would decide to either give each thread it's own profiler
	    // or use one global one from FlatProfiler,
	    // or up to some count of the number of profiled threads, etc.
	    ThreadProfiler* pp = new ThreadProfiler();
	    pp->engage();
	    set_thread_profiler(pp);
	  }
	
  {- -------------------------------------------
  (1) フィールドの初期化
      (See: ThreadSafepointState::create())
      ---------------------------------------- -}

	  // Setup safepoint state info for this thread
	  ThreadSafepointState::create(this);
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (debug_only 時にのみ実行)
      ---------------------------------------- -}

	  debug_only(_java_call_counter = 0);
	
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  // JVMTI PopFrame support
	  _popframe_condition = popframe_inactive;
	  _popframe_preserved_args = NULL;
	  _popframe_preserved_args_size = 0;
	
  {- -------------------------------------------
  (1) JavaThread::pd_initialize() を呼び出して, 
      プラットフォーム固有の初期化処理を(もしそんなものがあれば)実行しておく.
      ---------------------------------------- -}

	  pd_initialize();
	}
	
```


