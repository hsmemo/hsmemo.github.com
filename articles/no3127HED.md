---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp
### 説明(description)

```
  // It scans an object and visits its children.
```

### 名前(function name)
```
  void scan_object(oop obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(_nextMarkBitMap->isMarked((HeapWord*) obj), "invariant");
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	    if (_cm->verbose_high())
	      gclog_or_tty->print_cr("[%d] we're scanning object "PTR_FORMAT,
	                             _task_id, (void*) obj);
	
  {- -------------------------------------------
  (1) _words_scanned フィールドを増加させておく.
      ---------------------------------------- -}

	    size_t obj_size = obj->size();
	    _words_scanned += obj_size;
	
  {- -------------------------------------------
  (1) oop_iterate() を呼び出して, _cm_oop_closure を適用する.
      ---------------------------------------- -}

	    obj->oop_iterate(_oop_closure);
	    statsOnly( ++_objs_scanned );
	    check_limits();
	  }
	
```


