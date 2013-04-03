---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/sharedHeap.cpp
### 説明(description)

```
// Unmarked shared Strings in the StringTable (which got there due to
// being in the constant pools of as-yet unloaded shared classes) were
// not marked and therefore did not have their mark words preserved.
// These entries are also deliberately not purged from the string
// table during unloading of unmarked strings. If an identity hash
// code was computed for any of these objects, it will not have been
// cleared to zero during the forwarding process or by the
// RecursiveAdjustSharedObjectClosure, and will be confused by the
// adjusting process as a forwarding pointer. We need to skip
// forwarding StringTable entries which contain unmarked shared
// Strings. Actually, since shared strings won't be moving, we can
// just skip adjusting any shared entries in the string table.

```

### 名前(function name)
```
void SharedHeap::process_weak_roots(OopClosure* root_closure,
                                    CodeBlobClosure* code_roots,
                                    OopClosure* non_root_closure) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Weak global JNI references に対して処理を行う.
      ---------------------------------------- -}

	  // Global (weak) JNI handles
	  JNIHandles::weak_oops_do(&always_true, root_closure);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  CodeCache::blobs_do(code_roots);
	  if (UseSharedSpaces && !DumpSharedSpaces) {
	    SkipAdjustingSharedStrings skip_closure(root_closure);
	    StringTable::oops_do(&skip_closure);
	  } else {
	    StringTable::oops_do(root_closure);
	  }
	}
	
```


