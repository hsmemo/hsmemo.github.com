---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/vmThread.cpp

### 名前(function name)
```
void VMThread::run() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(this == vm_thread(), "check");
	
  {- -------------------------------------------
  (1) Thread::initialize_thread_local_storage() を呼んで, 
      現在実行中のネイティブスレッドにこの VMThread オブジェクトを対応付けておく.
      ---------------------------------------- -}

	  this->initialize_thread_local_storage();

  {- -------------------------------------------
  (1) Thread::record_stack_base_and_size() を呼んで, スタック領域関連のフィールドを初期化する.
      ---------------------------------------- -}

	  this->record_stack_base_and_size();

  {- -------------------------------------------
  (1) カレントスレッドの JNI ローカル参照フレームを作成しておく.
      (JNIHandleBlock::allocate_block() で新しい JNIHandleBlock を作成し, 
       それを Thread::set_active_handles() でカレントスレッドにセットする.
       See: [here](no3059hRF.html) for details)
      ---------------------------------------- -}

	  // Notify_lock wait checks on active_handles() to rewait in
	  // case of spurious wakeup, it should wait on the last
	  // value set prior to the notify
	  this->set_active_handles(JNIHandleBlock::allocate_block());
	
  {- -------------------------------------------
  (1) Notify_lock に対して Monitor::notify() を呼んでおく.
  
      (これは VMThread の作成元である Threads::create_vm() との同期処理.
       See: Threads::create_vm())
      ---------------------------------------- -}

	  {
	    MutexLocker ml(Notify_lock);
	    Notify_lock->notify();
	  }
	  // Notify_lock is destroyed by Threads::create_vm()
	
  {- -------------------------------------------
  (1) os::set_native_priority() を呼んで, VMThread の優先度を変更しておく.
      ---------------------------------------- -}

	  int prio = (VMThreadPriority == -1)
	    ? os::java_to_os_priority[NearMaxPriority]
	    : VMThreadPriority;
	  // Note that I cannot call os::set_priority because it expects Java
	  // priorities and I am *explicitly* using OS priorities so that it's
	  // possible to set the VM thread priority higher than any Java thread.
	  os::set_native_priority( this, prio );
	
  {- -------------------------------------------
  (1) VMThread::loop() を呼んで, VMThread のメイン処理を開始する.
      ---------------------------------------- -}

	  // Wait for VM_Operations until termination
	  this->loop();
	
  {- -------------------------------------------
  (1) (トレース出力)
  
      (なお, ここでトレースを出すのには, 
       他の箇所から大量の tty 出力があった場合に
       それが完了するのを待ち受けるという意味もある.
  
       実際, 大量の出力がある場合に待ち受けを行わないと, 
       以降の SafepointSynchronize がタイムアウトして異常終了する恐れがある.
       Bug 6295565 も参照)
      ---------------------------------------- -}

	  // Note the intention to exit before safepointing.
	  // 6295565  This has the effect of waiting for any large tty
	  // outputs to finish.
	  if (xtty != NULL) {
	    ttyLocker ttyl;
	    xtty->begin_elem("destroy_vm");
	    xtty->stamp();
	    xtty->end_elem();
	    assert(should_terminate(), "termination flag must be set");
	  }
	
  {- -------------------------------------------
  (1) この後の終了処理で様々なリソースを開放していくので, 
      解放後のリソースにスレッドがうっかりアクセスしてしまわないように, 
      SafepointSynchronize::begin() で HotSpot を Safepoint 状態にしておく.
      (Bug 4526887 も参照)
      ---------------------------------------- -}

	  // 4526887 let VM thread exit at Safepoint
	  SafepointSynchronize::begin();
	
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	  if (VerifyBeforeExit) {
	    HandleMark hm(VMThread::vm_thread());
	    // Among other things, this ensures that Eden top is correct.
	    Universe::heap()->prepare_for_verify();
	    os::check_heap();
	    // Silent verification so as not to pollute normal output,
	    // unless we really asked for it.
	    Universe::verify(true, !(PrintGCDetails || Verbose));
	  }
	
  {- -------------------------------------------
  (1) CompileBroker::set_should_block() を呼んで, CompilerThread に停止要請を通知する.
    
      (CompileBroker::set_should_block() を呼ぶと
       CompileBroker::maybe_block() が true を返すようになり, 
       これによって CompilerThread が停止するようになる.
  
       See: CompileBroker::maybe_block())
      ---------------------------------------- -}

	  CompileBroker::set_should_block();
	
  {- -------------------------------------------
  (1) VM_Exit::wait_for_threads_in_native_to_block() を呼んで, 
      native method を実行しているスレッドが停止するまで, 少しのあいだ待ってみる.
      ---------------------------------------- -}

	  // wait for threads (compiler threads or daemon threads) in the
	  // _thread_in_native state to block.
	  VM_Exit::wait_for_threads_in_native_to_block();
	
  {- -------------------------------------------
  (1) VMThread が終了したことを, jni_DestroyJavaVM() 内で 
      VMThread の終了を待っているスレッドに通知する.
  
      (_terminated の値を変更することで, 
       VMThread::is_terminated() が true を返すようになり, 
       終了したことが相手に伝わる.
  
       See: VMThread::wait_for_vm_thread_exit())
      ---------------------------------------- -}

	  // signal other threads that VM process is gone
	  {
	    // Note: we must have the _no_safepoint_check_flag. Mutex::lock() allows
	    // VM thread to enter any lock at Safepoint as long as its _owner is NULL.
	    // If that happens after _terminate_lock->wait() has unset _owner
	    // but before it actually drops the lock and waits, the notification below
	    // may get lost and we will have a hang. To avoid this, we need to use
	    // Mutex::lock_without_safepoint_check().
	    MutexLockerEx ml(_terminate_lock, Mutex::_no_safepoint_check_flag);
	    _terminated = true;
	    _terminate_lock->notify();
	  }
	
  {- -------------------------------------------
  (1) (VMThread の開放処理は, JNI の DestroyJavaVM() を呼び出したスレッドが, 同期的に行わないといけない.
       これは, CodeHeap を開放する前に VMThread の削除が終わることを保証したいため.)
      ---------------------------------------- -}

	  // Deletion must be done synchronously by the JNI DestroyJavaVM thread
	  // so that the VMThread deletion completes before the main thread frees
	  // up the CodeHeap.
	
	}
	
```


