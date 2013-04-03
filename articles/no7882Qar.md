---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.cpp

### 名前(function name)
```
void GCTaskManager::initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceGCTaskManager) {
	    tty->print_cr("GCTaskManager::initialize: workers: %u", workers());
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(workers() != 0, "no workers");

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _monitor = new Monitor(Mutex::barrier,                // rank
	                         "GCTaskManager monitor",       // name
	                         Mutex::_allow_vm_block_flag);  // allow_vm_block
	  // The queue for the GCTaskManager must be a CHeapObj.
	  GCTaskQueue* unsynchronized_queue = GCTaskQueue::create_on_c_heap();
	  _queue = SynchronizedGCTaskQueue::create(unsynchronized_queue, lock());
	  _noop_task = NoopGCTask::create_on_c_heap();
	  _resource_flag = NEW_C_HEAP_ARRAY(bool, workers());

    {- -------------------------------------------
  (1.1) (以下のブロック内で, GCTaskThread を入れるための配列の確保, 
         及び GCTaskThread オブジェクトの生成を行っている)
        ---------------------------------------- -}

	  {
	    // Set up worker threads.
	    //     Distribute the workers among the available processors,
	    //     unless we were told not to, or if the os doesn't want to.
	    uint* processor_assignment = NEW_C_HEAP_ARRAY(uint, workers());
	    if (!BindGCTaskThreadsToCPUs ||
	        !os::distribute_processes(workers(), processor_assignment)) {
	      for (uint a = 0; a < workers(); a += 1) {
	        processor_assignment[a] = sentinel_worker();
	      }
	    }
	    _thread = NEW_C_HEAP_ARRAY(GCTaskThread*, workers());
	    for (uint t = 0; t < workers(); t += 1) {
	      set_thread(t, GCTaskThread::create(this, t, processor_assignment[t]));
	    }
	    if (TraceGCTaskThread) {
	      tty->print("GCTaskManager::initialize: distribution:");
	      for (uint t = 0; t < workers(); t += 1) {
	        tty->print("  %u", processor_assignment[t]);
	      }
	      tty->cr();
	    }
	    FREE_C_HEAP_ARRAY(uint, processor_assignment);
	  }
	  reset_busy_workers();
	  set_unblocked();
	  for (uint w = 0; w < workers(); w += 1) {
	    set_resource_flag(w, false);
	  }
	  reset_delivered_tasks();
	  reset_completed_tasks();
	  reset_noop_tasks();
	  reset_barriers();
	  reset_emptied_queue();

  {- -------------------------------------------
  (1) 生成した全ての GCTaskThread に対して GCTaskThread::start() を呼び出し, 実行を開始させる.
      ---------------------------------------- -}

	  for (uint s = 0; s < workers(); s += 1) {
	    thread(s)->start();
	  }
	}
	
```


