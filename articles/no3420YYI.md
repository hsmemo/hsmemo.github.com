---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/sparsePRT.cpp

### 名前(function name)
```
void SparsePRT::do_cleanup_work(SparsePRTCleanupTask* sprt_cleanup_task) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もしこの SparsePRT が拡張されていれば, 
      SparsePRTCleanupTask::add() を呼んで
      sprt_cleanup_task 引数で指定された SparsePRTCleanupTask オブジェクトに
      この SparsePRT を登録しておく.
      ---------------------------------------- -}

	  if (should_be_on_expanded_list()) {
	    sprt_cleanup_task->add(this);
	  }
	}
	
```


