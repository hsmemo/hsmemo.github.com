---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiExport.cpp
### 説明(description)

```
// Collect all the vm internally allocated objects which are visible to java world
```

### 名前(function name)
```
void JvmtiExport::record_vm_internal_object_allocation(oop obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  Thread* thread = ThreadLocalStorage::thread();
	  if (thread != NULL && thread->is_Java_thread())  {
	    // Can not take safepoint here.
	    No_Safepoint_Verifier no_sfpt;
	    // Can not take safepoint here so can not use state_for to get
	    // jvmti thread state.
	    JvmtiThreadState *state = ((JavaThread*)thread)->jvmti_thread_state();
	    if (state != NULL ) {
	      // state is non NULL when VMObjectAllocEventCollector is enabled.
	      JvmtiVMObjectAllocEventCollector *collector;
	      collector = state->get_vm_object_alloc_event_collector();
	      if (collector != NULL && collector->is_enabled()) {
	        // Don't record classes as these will be notified via the ClassLoad
	        // event.
	        if (obj->klass() != SystemDictionary::Class_klass()) {
	          collector->record_allocation(obj);
	        }
	      }
	    }
	  }
	}
	
```


