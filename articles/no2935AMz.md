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
void VM_G1CollectFull::doit() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectedHeap* g1h = G1CollectedHeap::heap();
	  GCCauseSetter x(g1h, _gc_cause);

  {- -------------------------------------------
  (1) G1CollectedHeap::do_full_collection() を呼び出すだけ.
      ---------------------------------------- -}

	  g1h->do_full_collection(false /* clear_all_soft_refs */);
	}
	
```


