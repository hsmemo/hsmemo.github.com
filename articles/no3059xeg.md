---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp
### 説明(description)

```
// Thread start routine for all new Java threads
```

### 名前(function name)
```
static unsigned __stdcall java_start(Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _alloca() を呼んで, スタックの位置を多少増減させておく.
      (増減させる量は, 自分自身の process id と 
       static な int 変数(以下の counter)に基づいて決めている)
    
      (なおコメントによると, これはキャッシュ上での競合を減らすための措置.
       他のスレッドのスタックとキャッシュラインがかぶっていると, お互いに invalidate しあうことになるので.
       競合は同じ HotSpot プロセス内のスレッド同士でも起こりうるし, 
       別の HotSpot プロセス内のスレッドとも起こりうる.
       特に, Hyperthreading 環境ではずらすことによる効果が大きい, とのこと.)
      ---------------------------------------- -}

	  // Try to randomize the cache line index of hot stack frames.
	  // This helps when threads of the same stack traces evict each other's
	  // cache lines. The threads can be either from the same JVM instance, or
	  // from different JVM instances. The benefit is especially true for
	  // processors with hyperthreading technology.
	  static int counter = 0;
	  int pid = os::current_process_id();
	  _alloca(((pid ^ counter++) & 7) * 128);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  OSThread* osthr = thread->osthread();
	  assert(osthr->get_state() == RUNNABLE, "invalid os thread state");
	
  {- -------------------------------------------
  (1) UseNUMA オプションが指定されている場合は, 
      os::numa_get_group_id() で id を取得し, 
      カレントスレッドの lgrp_id フィールドにセットしておく.
      (See: [here](no28916ddy.html) for details)
      ---------------------------------------- -}

	  if (UseNUMA) {
	    int lgrp_id = os::numa_get_group_id();
	    if (lgrp_id != -1) {
	      thread->set_lgrp_id(lgrp_id);
	    }
	  }
	
	
  {- -------------------------------------------
  (1) Thread::run() (もしくはサブクラスでオーバーライドされたメソッド) を呼び出して, 
      スレッドのメイン処理を実行する.
    
      (なお, UseVectoredExceptions オプションが指定されている場合には, そのまま呼び出す.
       そうでなければ, SEH で囲んだ状態で呼び出す.
       SEH で囲む場合, 例外ハンドラは topLevelExceptionFilter() を使用)
      ---------------------------------------- -}

	  if (UseVectoredExceptions) {
	    // If we are using vectored exception we don't need to set a SEH
	    thread->run();
	  }
	  else {
	    // Install a win32 structured exception handler around every thread created
	    // by VM, so VM can genrate error dump when an exception occurred in non-
	    // Java thread (e.g. VM thread).
	    __try {
	       thread->run();
	    } __except(topLevelExceptionFilter(
	               (_EXCEPTION_POINTERS*)_exception_info())) {
	        // Nothing to do.
	    }
	  }
	
  {- -------------------------------------------
  (1) os::win32::_os_thread_count フィールドの値を減少させておく.
      (この値は何に使われる?? 現在の実装では意味のある使われ方をしていないような... #TODO)
      ---------------------------------------- -}

	  // One less thread is executing
	  // When the VMThread gets here, the main thread may have already exited
	  // which frees the CodeHeap containing the Atomic::add code
	  if (thread != VMThread::vm_thread() && VMThread::vm_thread() != NULL) {
	    Atomic::dec_ptr((intptr_t*)&os::win32::_os_thread_count);
	  }
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return 0;
	}
	
```


