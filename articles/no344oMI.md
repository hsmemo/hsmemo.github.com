---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/utilities/vmError.cpp

### 名前(function name)
```
void VMError::report_java_out_of_memory() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) OnOutOfMemoryError オプションが指定されていれば, 
      VM_ReportJavaOutOfMemory クラスを使って OutOfMemoryError が発生したことを通知する.
      ---------------------------------------- -}

	  if (OnOutOfMemoryError && OnOutOfMemoryError[0]) {
	    MutexLocker ml(Heap_lock);
	    VM_ReportJavaOutOfMemory op(this);
	    VMThread::execute(&op);
	  }
	}
	
```


