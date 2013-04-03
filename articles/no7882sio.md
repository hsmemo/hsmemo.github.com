---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/vm_operations.cpp

### 名前(function name)
```
void VM_Operation::set_calling_thread(Thread* thread, ThreadPriority priority) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _calling_thread フィールド, 及び _priority フィールドに
      引数で指定された値をセット.
      ---------------------------------------- -}

	  _calling_thread = thread;
	  assert(MinPriority <= priority && priority <= MaxPriority, "sanity check");
	  _priority = priority;
	}
	
```


