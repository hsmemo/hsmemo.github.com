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
void VMThread::create() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) VMThread を生成する
      ---------------------------------------- -}

	  assert(vm_thread() == NULL, "we can only allocate one VMThread");
	  _vm_thread = new VMThread();
	
  {- -------------------------------------------
  (1) VMThread に VM Operation 要求を通知するための VMOperationQueue を生成する
      ---------------------------------------- -}

	  // Create VM operation queue
	  _vm_queue = new VMOperationQueue();
	  guarantee(_vm_queue != NULL, "just checking");
	
  {- -------------------------------------------
  (1) VMThread の終了を待つための Monitor を生成する
      (See: VMThread::wait_for_vm_thread_exit())
      ---------------------------------------- -}

	  _terminate_lock = new Monitor(Mutex::safepoint, "VMThread::_terminate_lock", true);
	
  {- -------------------------------------------
  (1) (プロファイル情報取得用の初期化処理) ("sun.threads.vmOperationTime") (See: PerfTraceTime)
      ---------------------------------------- -}

	  if (UsePerfData) {
	    // jvmstat performance counters
	    Thread* THREAD = Thread::current();
	    _perf_accumulated_vm_operation_time =
	                 PerfDataManager::create_counter(SUN_THREADS, "vmOperationTime",
	                                                 PerfData::U_Ticks, CHECK);
	  }
	}
	
```


