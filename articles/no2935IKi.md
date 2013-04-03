---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/code/nmethod.cpp

### 名前(function name)
```
void nmethod::post_compiled_method_unload() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (unload_reported()) {
	    // During unloading we transition to unloaded and then to zombie
	    // and the unloading is reported during the first transition.
	    return;
	  }
	
	  assert(_method != NULL && !is_unloaded(), "just checking");
	  DTRACE_METHOD_UNLOAD_PROBE(method());
	
	  // If a JVMTI agent has enabled the CompiledMethodUnload event then
	  // post the event. Sometime later this nmethod will be made a zombie
	  // by the sweeper but the methodOop will not be valid at that point.
	  // If the _jmethod_id is null then no load event was ever requested
	  // so don't bother posting the unload.  The main reason for this is
	  // that the jmethodID is a weak reference to the methodOop so if
	  // it's being unloaded there's no way to look it up since the weak
	  // ref will have been cleared.
	  if (_jmethod_id != NULL && JvmtiExport::should_post_compiled_method_unload()) {
	    assert(!unload_reported(), "already unloaded");
	    JvmtiDeferredEvent event =
	      JvmtiDeferredEvent::compiled_method_unload_event(this,
	          _jmethod_id, insts_begin());
	    if (SafepointSynchronize::is_at_safepoint()) {
	      // Don't want to take the queueing lock. Add it as pending and
	      // it will get enqueued later.
	      JvmtiDeferredEventQueue::add_pending_event(event);
	    } else {
	      MutexLockerEx ml(Service_lock, Mutex::_no_safepoint_check_flag);
	      JvmtiDeferredEventQueue::enqueue(event);
	    }
	  }
	
	  // The JVMTI CompiledMethodUnload event can be enabled or disabled at
	  // any time. As the nmethod is being unloaded now we mark it has
	  // having the unload event reported - this will ensure that we don't
	  // attempt to report the event in the unlikely scenario where the
	  // event is enabled at the time the nmethod is made a zombie.
	  set_unload_reported();
	}
	
```


