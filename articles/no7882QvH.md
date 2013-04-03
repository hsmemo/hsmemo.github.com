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
NoopGCTask* NoopGCTask::create_on_c_heap() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい NoopGCTask オブジェクトを (C ヒープ上に) 生成してリターンする.
      ---------------------------------------- -}

	  NoopGCTask* result = new(ResourceObj::C_HEAP) NoopGCTask(true);
	  return result;
	}
	
```


