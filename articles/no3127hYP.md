---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp

### 名前(function name)
```
  template <class T> void do_oop_work(T* p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert( _g1h->is_in_g1_reserved((HeapWord*) p), "invariant");
	    assert(!_g1h->is_on_master_free_list(
	                    _g1h->heap_region_containing((HeapWord*) p)), "invariant");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    oop obj = oopDesc::load_decode_heap_oop(p);

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	    if (_cm->verbose_high())
	      gclog_or_tty->print_cr("[%d] we're looking at location "
	                             "*"PTR_FORMAT" = "PTR_FORMAT,
	                             _task->task_id(), p, (void*) obj);

  {- -------------------------------------------
  (1) CMTask::deal_with_reference() を呼び出して処理を行う.
      ---------------------------------------- -}

	    _task->deal_with_reference(obj);
	  }
	
```


