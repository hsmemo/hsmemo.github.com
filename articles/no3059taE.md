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
// The first routine called by a new Java thread
```

### 名前(function name)
```
void JavaThread::run() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JavaThread::initialize_tlab() を呼んで, TLAB の初期化を行う.
      ---------------------------------------- -}

	  // initialize thread-local alloc buffer related fields
	  this->initialize_tlab();
	
  {- -------------------------------------------
  (1) JavaThread::record_base_of_stack_pointer() を呼んで, スタック領域関連のフィールドを初期化する.
      ---------------------------------------- -}

	  // used to test validitity of stack trace backs
	  this->record_base_of_stack_pointer();
	
  {- -------------------------------------------
  (1) Thread::record_stack_base_and_size() を呼んで, スタック領域関連のフィールドを初期化する.
      ---------------------------------------- -}

	  // Record real stack base and size.
	  this->record_stack_base_and_size();
	
  {- -------------------------------------------
  (1) Thread::initialize_thread_local_storage() を呼んで, 
      現在実行中のネイティブスレッドにこの JavaThread オブジェクトを対応付けておく.
      ---------------------------------------- -}

	  // Initialize thread local storage; set before calling MutexLocker
	  this->initialize_thread_local_storage();
	
  {- -------------------------------------------
  (1) JavaThread::create_stack_guard_pages() を呼んで, 
      カレントスレッドのスタック上に guard page を設定しておく.
      ---------------------------------------- -}

	  this->create_stack_guard_pages();
	
  {- -------------------------------------------
  (1) JavaThread::cache_global_variables() を呼んで, 
      プラットフォーム固有のフィールドを(もしそんなものがあれば)初期化しておく.
      ---------------------------------------- -}

	  this->cache_global_variables();
	
  {- -------------------------------------------
  (1) この時点で, カレントスレッドは Safepoint 停止も実行できる程度に初期化されている.
      そこで, ThreadStateTransition::transition_and_fence() を呼んで
      カレントスレッドの JavaThreadState を _thread_new から _thread_in_vm に変えておく.
      ---------------------------------------- -}

	  // Thread is now sufficient initialized to be handled by the safepoint code as being
	  // in the VM. Change thread state from _thread_new to _thread_in_vm
	  ThreadStateTransition::transition_and_fence(this, _thread_new, _thread_in_vm);
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(JavaThread::current() == this, "sanity check");
	  assert(!Thread::current()->owns_locks(), "sanity check");
	
  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_THREAD_PROBE(start, this);
	
  {- -------------------------------------------
  (1) カレントスレッドの JNI ローカル参照フレームを作成しておく.
      (JNIHandleBlock::allocate_block() で新しい JNIHandleBlock を作成し, 
       それを Thread::set_active_handles() でカレントスレッドにセットする.
       See: [here](no3059hRF.html) for details)
      ---------------------------------------- -}

	  // This operation might block. We call that after all safepoint checks for a new thread has
	  // been completed.
	  this->set_active_handles(JNIHandleBlock::allocate_block());
	
  {- -------------------------------------------
  (1) (JVMTI のフック点)
      ---------------------------------------- -}

	  if (JvmtiExport::should_post_thread_life()) {
	    JvmtiExport::post_thread_start(this);
	  }
	
  {- -------------------------------------------
  (1) JavaThread::thread_main_inner() を呼んで, 
      カレントスレッドのメイン処理(及びメイン処理終了後の後片付け処理)を行う.
    
      (なお, JavaThread::thread_main_inner() を呼び出した先(= フレームを積んだ先)でメイン処理を行うので, 
       メイン処理内で使われるスタックの底は実際に確保したアドレスよりは少し下になる)
      ---------------------------------------- -}

	  // We call another function to do the rest so we are sure that the stack addresses used
	  // from there will be lower than the stack base just computed
	  thread_main_inner();
	
  {- -------------------------------------------
  (1) (この時点では, もうこのスレッドは有効な状態にはない)
      ---------------------------------------- -}

	  // Note, thread is no longer valid at this point!
	}
	
```


