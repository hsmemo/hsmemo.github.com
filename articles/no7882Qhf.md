---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp
### 説明(description)

```
  // Factory create and destroy methods.
```

### 名前(function name)
```
  static SynchronizedGCTaskQueue* create(GCTaskQueue* queue, Monitor * lock) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい SynchronizedGCTaskQueue オブジェクトを生成してリターンするだけ.
      ---------------------------------------- -}

	    return new SynchronizedGCTaskQueue(queue, lock);
	  }
	
```


