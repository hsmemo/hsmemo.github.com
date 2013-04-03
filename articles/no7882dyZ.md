---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.cpp

### 名前(function name)
```
GCTaskQueue* GCTaskQueue::create_on_c_heap() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい GCTaskQueue オブジェクトを (C ヒープ上に) 生成する.
      ---------------------------------------- -}

	  GCTaskQueue* result = new(ResourceObj::C_HEAP) GCTaskQueue(true);

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceGCTaskQueue) {
	    tty->print_cr("GCTaskQueue::create_on_c_heap()"
	                  " returns " INTPTR_FORMAT,
	                  result);
	  }

  {- -------------------------------------------
  (1) 結果をリターンする.
      ---------------------------------------- -}

	  return result;
	}
	
```


