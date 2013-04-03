---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiExport.cpp

### 名前(function name)
```
void JvmtiExport::post_dynamic_code_generated(const char *name, const void *code_begin, const void *code_end) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  jvmtiPhase phase = JvmtiEnv::get_phase();
	  if (phase == JVMTI_PHASE_PRIMORDIAL || phase == JVMTI_PHASE_START) {
	    post_dynamic_code_generated_internal(name, code_begin, code_end);
	  } else {
	    // It may not be safe to post the event from this thread.  Defer all
	    // postings to the service thread so that it can perform them in a safe
	    // context and in-order.
	    MutexLockerEx ml(Service_lock, Mutex::_no_safepoint_check_flag);
	    JvmtiDeferredEvent event = JvmtiDeferredEvent::dynamic_code_generated_event(
	        name, code_begin, code_end);
	    JvmtiDeferredEventQueue::enqueue(event);
	  }
	}
	
```


