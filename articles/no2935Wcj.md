---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiExport.hpp
### 説明(description)

```
  // Collects vm internal objects for later event posting.
```

### 名前(function name)
```
  inline static void vm_object_alloc_event_collector(oop object) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (JVMTI のフック点)
      (VMObjectAlloc イベントが有効になっていれば, 
       JvmtiExport::record_vm_internal_object_allocation() を呼び出して通知を行う)
      ---------------------------------------- -}

	    if (should_post_vm_object_alloc()) {
	      record_vm_internal_object_allocation(object);
	    }
	  }
	
```


