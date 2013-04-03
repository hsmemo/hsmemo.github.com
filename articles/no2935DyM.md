---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/ptrQueue.cpp

### 名前(function name)
```
void PtrQueueSet::enqueue_complete_buffer(void** buf, size_t index) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以下の処理は _cbl_mon フィールドのモニターで排他して行う)
      ---------------------------------------- -}

	  MutexLockerEx x(_cbl_mon, Mutex::_no_safepoint_check_flag);

  {- -------------------------------------------
  (1) BufferNode::new_from_buffer() を呼んで, 
      buf 引数で指定されたバッファに対応する BufferNode オブジェクトを取得する.
    
      (なお, index の値は index 引数で指定されたものを使用)
      ---------------------------------------- -}

	  BufferNode* cbn = BufferNode::new_from_buffer(buf);
	  cbn->set_index(index);

  {- -------------------------------------------
  (1) 取得した BufferNode を _completed_buffers_tail の最後に追加する.
      (空だった場合は, さらに _completed_buffers_head もこの BufferNode を指すようにしておく)
    
      (ついでに _n_completed_buffers フィールドもインクリメントしておく)
      ---------------------------------------- -}

	  if (_completed_buffers_tail == NULL) {
	    assert(_completed_buffers_head == NULL, "Well-formedness");
	    _completed_buffers_head = cbn;
	    _completed_buffers_tail = cbn;
	  } else {
	    _completed_buffers_tail->set_next(cbn);
	    _completed_buffers_tail = cbn;
	  }
	  _n_completed_buffers++;
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (!_process_completed && _process_completed_threshold >= 0 &&
	      _n_completed_buffers >= _process_completed_threshold) {
	    _process_completed = true;
	    if (_notify_when_complete)
	      _cbl_mon->notify();
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  debug_only(assert_completed_buffer_list_len_correct_locked());
	}
	
```


