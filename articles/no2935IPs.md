---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiRawMonitor.cpp
### 説明(description)

```
// Any JavaThread will enter here with state _thread_blocked
```

### 名前(function name)
```
int JvmtiRawMonitor::raw_enter(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  TEVENT (raw_enter) ;

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  void * Contended ;
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // don't enter raw monitor if thread is being externally suspended, it will
	  // surprise the suspender if a "suspended" thread can still enter monitor
	  JavaThread * jt = (JavaThread *)THREAD;
	  if (THREAD->is_Java_thread()) {
	    jt->SR_lock()->lock_without_safepoint_check();
	    while (jt->is_external_suspend()) {
	      jt->SR_lock()->unlock();
	      jt->java_suspend_self();
	      jt->SR_lock()->lock_without_safepoint_check();
	    }
	    // guarded by SR_lock to avoid racing with new external suspend requests.
	    Contended = Atomic::cmpxchg_ptr (THREAD, &_owner, NULL) ;
	    jt->SR_lock()->unlock();
	  } else {
	    Contended = Atomic::cmpxchg_ptr (THREAD, &_owner, NULL) ;
	  }
	
	  if (Contended == THREAD) {
	     _recursions ++ ;
	     return OM_OK ;
	  }
	
	  if (Contended == NULL) {
	     guarantee (_owner == THREAD, "invariant") ;
	     guarantee (_recursions == 0, "invariant") ;
	     return OM_OK ;
	  }
	
	  THREAD->set_current_pending_monitor(this);
	
	  if (!THREAD->is_Java_thread()) {
	     // No other non-Java threads besides VM thread would acquire
	     // a raw monitor.
	     assert(THREAD->is_VM_thread(), "must be VM thread");
	     SimpleEnter (THREAD) ;
	   } else {
	     guarantee (jt->thread_state() == _thread_blocked, "invariant") ;
	     for (;;) {
	       jt->set_suspend_equivalent();
	       // cleared by handle_special_suspend_equivalent_condition() or
	       // java_suspend_self()
	       SimpleEnter (THREAD) ;
	
	       // were we externally suspended while we were waiting?
	       if (!jt->handle_special_suspend_equivalent_condition()) break ;
	
	       // This thread was externally suspended
	       //
	       // This logic isn't needed for JVMTI raw monitors,
	       // but doesn't hurt just in case the suspend rules change. This
	           // logic is needed for the JvmtiRawMonitor.wait() reentry phase.
	           // We have reentered the contended monitor, but while we were
	           // waiting another thread suspended us. We don't want to reenter
	           // the monitor while suspended because that would surprise the
	           // thread that suspended us.
	           //
	           // Drop the lock -
	       SimpleExit (THREAD) ;
	
	           jt->java_suspend_self();
	         }
	
	     assert(_owner == THREAD, "Fatal error with monitor owner!");
	     assert(_recursions == 0, "Fatal error with monitor recursions!");
	  }
	
	  THREAD->set_current_pending_monitor(NULL);
	  guarantee (_recursions == 0, "invariant") ;
	  return OM_OK;
	}
	
```


