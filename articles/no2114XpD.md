---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psCompactionManager.cpp

### 名前(function name)
```
void ParCompactionManager::reset() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  for(uint i = 0; i < ParallelGCThreads + 1; i++) {
	    assert(manager_array(i)->revisit_klass_stack()->is_empty(), "sanity");
	    assert(manager_array(i)->revisit_mdo_stack()->is_empty(), "sanity");
	  }
	}
	
```


