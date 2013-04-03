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
void GenCollectedHeap::gen_process_weak_roots(OopClosure* root_closure,
                                              CodeBlobClosure* code_roots,
                                              OopClosure* non_root_closure) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) SharedHeap::process_weak_roots() を呼び出す.
      ---------------------------------------- -}

	  SharedHeap::process_weak_roots(root_closure, code_roots, non_root_closure);

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // "Local" "weak" refs
	  for (int i = 0; i < _n_gens; i++) {
	    _gens[i]->ref_processor()->weak_oops_do(root_closure);
	  }
	}
	
```


