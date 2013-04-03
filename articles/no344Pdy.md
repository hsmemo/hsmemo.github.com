---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_interface/collectedHeap.inline.hpp
### 説明(description)

```
// Support for jvmti and dtrace
```

### 名前(function name)
```
inline void post_allocation_notify(KlassHandle klass, oop obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (JMM のフック点)
      ---------------------------------------- -}

	  // support low memory notifications (no-op if not enabled)
	  LowMemoryDetector::detect_low_memory_for_collected_pools();
	
  {- -------------------------------------------
  (1) (JVMTI のフック点)
      ---------------------------------------- -}

	  // support for JVMTI VMObjectAlloc event (no-op if not enabled)
	  JvmtiExport::vm_object_alloc_event_collector(obj);
	
  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  if (DTraceAllocProbes) {
	    // support for Dtrace object alloc event (no-op most of the time)
	    if (klass() != NULL && klass()->klass_part()->name() != NULL) {
	      SharedRuntime::dtrace_object_alloc(obj);
	    }
	  }
	}
	
```


