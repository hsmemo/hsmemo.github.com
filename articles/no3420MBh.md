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
void SparsePRT::finish_cleanup_task(SparsePRTCleanupTask* sprt_cleanup_task) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(ParGCRareEvent_lock->owned_by_self(), "pre-condition");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  SparsePRT* head = sprt_cleanup_task->head();
	  SparsePRT* tail = sprt_cleanup_task->tail();

  {- -------------------------------------------
  (1) もし sprt_cleanup_task 引数で指定された SparsePRTCleanupTask 内に
      SparsePRT が入っていれば (= SparsePRT のリストが空でなければ), 
      そのリストを _head_expanded_list (これは static フィールド) の先頭につなぐ.
      ---------------------------------------- -}

	  if (head != NULL) {
	    assert(tail != NULL, "if head is not NULL, so should tail");
	
	    tail->set_next_expanded(_head_expanded_list);
	    _head_expanded_list = head;
	  } else {
	    assert(tail == NULL, "if head is NULL, so should tail");
	  }
	}
	
```


