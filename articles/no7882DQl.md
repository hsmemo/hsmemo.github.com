---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskThread.hpp
### 説明(description)

```
  // Factory create and destroy methods.
```

### 名前(function name)
```
  static GCTaskThread* create(GCTaskManager* manager,
                              uint           which,
                              uint           processor_id) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい GCTaskThread オブジェクトを生成してリターンする.
      ---------------------------------------- -}

	    return new GCTaskThread(manager, which, processor_id);
	  }
	
```


