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
// Post vm_object_alloc event for vm allocated objects visible to java
// world.
```

### 名前(function name)
```
JvmtiVMObjectAllocEventCollector::~JvmtiVMObjectAllocEventCollector() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (_allocated != NULL) {
	    set_enabled(false);
	    for (int i = 0; i < _allocated->length(); i++) {
	      oop obj = _allocated->at(i);
	      if (ServiceUtil::visible_oop(obj)) {
	        JvmtiExport::post_vm_object_alloc(JavaThread::current(), obj);
	      }
	    }
	    delete _allocated;
	  }
	  unset_jvmti_thread_state();
	}
	
```


