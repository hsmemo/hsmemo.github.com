---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/fprofiler.cpp

### 名前(function name)
```
void FlatProfilerTask::task() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  FlatProfiler::received_ticks += 1;
	
  {- -------------------------------------------
  (1) ProfileVM オプションが指定されている場合は, 
      FlatProfiler::record_vm_tick() を呼んで 
      VMThread に関するプロファイル情報を更新する.
      ---------------------------------------- -}

	  if (ProfileVM) {
	    FlatProfiler::record_vm_tick();
	  }
	
  {- -------------------------------------------
  (1) VM_Operation を実行中であれば 
      FlatProfiler::record_vm_operation() を呼んで
      VM Operation 関連のプロファイル情報をインクリメントしておく.
  
      (なお, Safepoint が開始されていたら, 
       FlatProfiler::record_vm_operation() を呼んだ後にリターン)
      ---------------------------------------- -}

	  VM_Operation* op = VMThread::vm_operation();
	  if (op != NULL) {
	    FlatProfiler::record_vm_operation();
	    if (SafepointSynchronize::is_at_safepoint()) {
	      return;
	    }
	  }

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  FlatProfiler::record_thread_ticks();
	}
	
```


