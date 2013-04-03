---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp

### 名前(function name)
```
ConcurrentMark::ConcurrentMark(ReservedSpace rs,
                               int max_regions) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  _markBitMap1(rs, MinObjAlignment - 1),
	  _markBitMap2(rs, MinObjAlignment - 1),
	
	  _parallel_marking_threads(0),
	  _sleep_factor(0.0),
	  _marking_task_overhead(1.0),
	  _cleanup_sleep_factor(0.0),
	  _cleanup_task_overhead(1.0),
	  _cleanup_list("Cleanup List"),
	  _region_bm(max_regions, false /* in_resource_area*/),
	  _card_bm((rs.size() + CardTableModRefBS::card_size - 1) >>
	           CardTableModRefBS::card_shift,
	           false /* in_resource_area*/),
	  _prevMarkBitMap(&_markBitMap1),
	  _nextMarkBitMap(&_markBitMap2),
	  _at_least_one_mark_complete(false),
	
	  _markStack(this),
	  _regionStack(),
	  // _finger set in set_non_marking_state
	
	  _max_task_num(MAX2(ParallelGCThreads, (size_t)1)),
	  // _active_tasks set in set_non_marking_state
	  // _tasks set inside the constructor
	  _task_queues(new CMTaskQueueSet((int) _max_task_num)),
	  _terminator(ParallelTaskTerminator((int) _max_task_num, _task_queues)),
	
	  _has_overflown(false),
	  _concurrent(false),
	  _has_aborted(false),
	  _restart_for_overflow(false),
	  _concurrent_marking_in_progress(false),
	  _should_gray_objects(false),
	
	  // _verbose_level set below
	
	  _init_times(),
	  _remark_times(), _remark_mark_times(), _remark_weak_ref_times(),
	  _cleanup_times(),
	  _total_counting_time(0.0),
	  _total_rs_scrub_time(0.0),
	
	  _parallel_workers(NULL)
	{
	  CMVerboseLevel verbose_level =
	    (CMVerboseLevel) G1MarkingVerboseLevel;
	  if (verbose_level < no_verbose)
	    verbose_level = no_verbose;
	  if (verbose_level > high_verbose)
	    verbose_level = high_verbose;
	  _verbose_level = verbose_level;
	
	  if (verbose_low())
	    gclog_or_tty->print_cr("[global] init, heap start = "PTR_FORMAT", "
	                           "heap end = "PTR_FORMAT, _heap_start, _heap_end);
	
	  _markStack.allocate(MarkStackSize);
	  _regionStack.allocate(G1MarkRegionStackSize);
	
	  // Create & start a ConcurrentMark thread.
	  _cmThread = new ConcurrentMarkThread(this);
	  assert(cmThread() != NULL, "CM Thread should have been created");
	  assert(cmThread()->cm() != NULL, "CM Thread should refer to this cm");
	
	  _g1h = G1CollectedHeap::heap();
	  assert(CGC_lock != NULL, "Where's the CGC_lock?");
	  assert(_markBitMap1.covers(rs), "_markBitMap1 inconsistency");
	  assert(_markBitMap2.covers(rs), "_markBitMap2 inconsistency");
	
	  SATBMarkQueueSet& satb_qs = JavaThread::satb_mark_queue_set();
	  satb_qs.set_buffer_size(G1SATBBufferSize);
	
	  _tasks = NEW_C_HEAP_ARRAY(CMTask*, _max_task_num);
	  _accum_task_vtime = NEW_C_HEAP_ARRAY(double, _max_task_num);
	
	  // so that the assertion in MarkingTaskQueue::task_queue doesn't fail
	  _active_tasks = _max_task_num;
	  for (int i = 0; i < (int) _max_task_num; ++i) {
	    CMTaskQueue* task_queue = new CMTaskQueue();
	    task_queue->initialize();
	    _task_queues->register_queue(i, task_queue);
	
	    _tasks[i] = new CMTask(i, this, task_queue, _task_queues);
	    _accum_task_vtime[i] = 0.0;
	  }
	
	  if (ConcGCThreads > ParallelGCThreads) {
	    vm_exit_during_initialization("Can't have more ConcGCThreads "
	                                  "than ParallelGCThreads.");
	  }
	  if (ParallelGCThreads == 0) {
	    // if we are not running with any parallel GC threads we will not
	    // spawn any marking threads either
	    _parallel_marking_threads =   0;
	    _sleep_factor             = 0.0;
	    _marking_task_overhead    = 1.0;
	  } else {
	    if (ConcGCThreads > 0) {
	      // notice that ConcGCThreads overwrites G1MarkingOverheadPercent
	      // if both are set
	
	      _parallel_marking_threads = ConcGCThreads;
	      _sleep_factor             = 0.0;
	      _marking_task_overhead    = 1.0;
	    } else if (G1MarkingOverheadPercent > 0) {
	      // we will calculate the number of parallel marking threads
	      // based on a target overhead with respect to the soft real-time
	      // goal
	
	      double marking_overhead = (double) G1MarkingOverheadPercent / 100.0;
	      double overall_cm_overhead =
	        (double) MaxGCPauseMillis * marking_overhead /
	        (double) GCPauseIntervalMillis;
	      double cpu_ratio = 1.0 / (double) os::processor_count();
	      double marking_thread_num = ceil(overall_cm_overhead / cpu_ratio);
	      double marking_task_overhead =
	        overall_cm_overhead / marking_thread_num *
	                                                (double) os::processor_count();
	      double sleep_factor =
	                         (1.0 - marking_task_overhead) / marking_task_overhead;
	
	      _parallel_marking_threads = (size_t) marking_thread_num;
	      _sleep_factor             = sleep_factor;
	      _marking_task_overhead    = marking_task_overhead;
	    } else {
	      _parallel_marking_threads = MAX2((ParallelGCThreads + 2) / 4, (size_t)1);
	      _sleep_factor             = 0.0;
	      _marking_task_overhead    = 1.0;
	    }
	
	    if (parallel_marking_threads() > 1)
	      _cleanup_task_overhead = 1.0;
	    else
	      _cleanup_task_overhead = marking_task_overhead();
	    _cleanup_sleep_factor =
	                     (1.0 - cleanup_task_overhead()) / cleanup_task_overhead();
	
	#if 0
	    gclog_or_tty->print_cr("Marking Threads          %d", parallel_marking_threads());
	    gclog_or_tty->print_cr("CM Marking Task Overhead %1.4lf", marking_task_overhead());
	    gclog_or_tty->print_cr("CM Sleep Factor          %1.4lf", sleep_factor());
	    gclog_or_tty->print_cr("CL Marking Task Overhead %1.4lf", cleanup_task_overhead());
	    gclog_or_tty->print_cr("CL Sleep Factor          %1.4lf", cleanup_sleep_factor());
	#endif
	
	    guarantee(parallel_marking_threads() > 0, "peace of mind");
	    _parallel_workers = new FlexibleWorkGang("G1 Parallel Marking Threads",
	         (int) _parallel_marking_threads, false, true);
	    if (_parallel_workers == NULL) {
	      vm_exit_during_initialization("Failed necessary allocation.");
	    } else {
	      _parallel_workers->initialize_workers();
	    }
	  }
	
	  // so that the call below can read a sensible value
	  _heap_start = (HeapWord*) rs.base();
	  set_non_marking_state();
	}
	
```


