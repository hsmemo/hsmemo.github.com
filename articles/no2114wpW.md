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
// Base class for all threads: VMThread, WatcherThread, ConcurrentMarkSweepThread,
// JavaThread


```

### 名前(function name)
```
Thread::Thread() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (フィールドの初期化)
      ---------------------------------------- -}

	  // stack and get_thread
	  set_stack_base(NULL);
	  set_stack_size(0);
	  set_self_raw_id(0);
	  set_lgrp_id(-1);
	
	  // allocated data structures
	  set_osthread(NULL);
	  set_resource_area(new ResourceArea());
	  set_handle_area(new HandleArea(NULL));
	  set_active_handles(NULL);
	  set_free_handle_block(NULL);
	  set_last_handle_mark(NULL);
	
	  // This initial value ==> never claimed.
	  _oops_do_parity = 0;
	
	  // the handle mark links itself to last_handle_mark
	  new HandleMark(this);
	
	  // plain initialization
	  debug_only(_owned_locks = NULL;)
	  debug_only(_allow_allocation_count = 0;)
	  NOT_PRODUCT(_allow_safepoint_count = 0;)
	  NOT_PRODUCT(_skip_gcalot = false;)
	  CHECK_UNHANDLED_OOPS_ONLY(_gc_locked_out_count = 0;)
	  _jvmti_env_iteration_count = 0;
	  set_allocated_bytes(0);
	  _vm_operation_started_count = 0;
	  _vm_operation_completed_count = 0;
	  _current_pending_monitor = NULL;
	  _current_pending_monitor_is_from_java = true;
	  _current_waiting_monitor = NULL;
	  _num_nested_signal = 0;
	  omFreeList = NULL ;
	  omFreeCount = 0 ;
	  omFreeProvision = 32 ;
	  omInUseList = NULL ;
	  omInUseCount = 0 ;
	
	  _SR_lock = new Monitor(Mutex::suspend_resume, "SR_lock", true);
	  _suspend_flags = 0;
	
	  // thread-specific hashCode stream generator state - Marsaglia shift-xor form
	  _hashStateX = os::random() ;
	  _hashStateY = 842502087 ;
	  _hashStateZ = 0x8767 ;    // (int)(3579807591LL & 0xffff) ;
	  _hashStateW = 273326509 ;
	
	  _OnTrap   = 0 ;
	  _schedctl = NULL ;
	  _Stalled  = 0 ;
	  _TypeTag  = 0x2BAD ;
	
	  // Many of the following fields are effectively final - immutable
	  // Note that nascent threads can't use the Native Monitor-Mutex
	  // construct until the _MutexEvent is initialized ...
	  // CONSIDER: instead of using a fixed set of purpose-dedicated ParkEvents
	  // we might instead use a stack of ParkEvents that we could provision on-demand.
	  // The stack would act as a cache to avoid calls to ParkEvent::Allocate()
	  // and ::Release()
	  _ParkEvent   = ParkEvent::Allocate (this) ;
	  _SleepEvent  = ParkEvent::Allocate (this) ;
	  _MutexEvent  = ParkEvent::Allocate (this) ;
	  _MuxEvent    = ParkEvent::Allocate (this) ;
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef CHECK_UNHANDLED_OOPS 時にのみ実行) (See: UnhandledOops)
      ---------------------------------------- -}

	#ifdef CHECK_UNHANDLED_OOPS
	  if (CheckUnhandledOops) {
	    _unhandled_oops = new UnhandledOops(this);
	  }
	#endif // CHECK_UNHANDLED_OOPS

  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  if (UseBiasedLocking) {
	    assert((((uintptr_t) this) & (markOopDesc::biased_lock_alignment - 1)) == 0, "forced alignment of thread object failed");
	    assert(this == _real_malloc_address ||
	           this == (void*) align_size_up((intptr_t) _real_malloc_address, markOopDesc::biased_lock_alignment),
	           "bug in forced alignment of thread objects");
	  }
	#endif /* ASSERT */
	}
	
```


