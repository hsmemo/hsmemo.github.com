---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/genCollectedHeap.cpp

### 名前(function name)
```
void GenCollectedHeap::object_iterate_since_last_GC(ObjectClosure* cl) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) GenCollectedHeap 内の各 Generaiton オブジェクトに対して, 
      Generation::object_iterate_since_last_GC() を呼び出す.
      ---------------------------------------- -}

	  for (int i = 0; i < _n_gens; i++) {
	    _gens[i]->object_iterate_since_last_GC(cl);
	  }
	}
	
```


