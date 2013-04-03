---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/vm_operations_g1.cpp

### 名前(function name)
```
bool VM_CGC_Operation::doit_prologue() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Heap_lock を確保する 
      (Concurrent な処理はこのロックで排他して行う. 
       開放するのは VM_CGC_Operation::doit_epilogue() 内)
  
      (ついでに SharedHeap::thread_holds_heap_lock_for_gc フィールドの値も変更しているが, 
       これはデバッグ用の処理である模様(??).
       現状では SharedHeap::heap_lock_held_for_gc() の返値にしか影響しないが, 
       この関数が assert() 内でしか使用されていないので...)
      ---------------------------------------- -}

	  Heap_lock->lock();
	  SharedHeap::heap()->_thread_holds_heap_lock_for_gc = true;
	  return true;
	}
	
```


