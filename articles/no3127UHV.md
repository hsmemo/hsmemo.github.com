---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/satbQueue.cpp

### 名前(function name)
```
bool SATBMarkQueueSet::apply_closure_to_completed_buffer_work(bool par,
                                                              int worker) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  BufferNode* nd = NULL;

  {- -------------------------------------------
  (1) _completed_buffers_head の先頭要素を取り出す.
      (ついでに, ...)
  
      (なおこの処理は _cbl_mon で排他した状態で行う)
      ---------------------------------------- -}

	  {
	    MutexLockerEx x(_cbl_mon, Mutex::_no_safepoint_check_flag);
	    if (_completed_buffers_head != NULL) {
	      nd = _completed_buffers_head;
	      _completed_buffers_head = nd->next();
	      if (_completed_buffers_head == NULL) _completed_buffers_tail = NULL;
	      _n_completed_buffers--;
	      if (_n_completed_buffers == 0) _process_completed = false;
	    }
	  }

  {- -------------------------------------------
  (1) completed buffer が存在した場合は (= 取り出した BufferNode が NULL でなかった場合は)
      ObjPtrQueue::apply_closure_to_buffer() を呼び出して,
      取り出した BufferNode が格納していたバッファに対してクロージャーを適用する.
      (適用するクロージャーは, _par_closures[worker] または _closure)
      (なお, 処理し終わったバッファは deallocate_buffer() で開放している).
      その後 true をリターンする.
  
      completed buffer が存在しなかった場合は (= 取り出した BufferNode が NULL だった場合は), 単に false をリターンするだけ.
      ---------------------------------------- -}

	  ObjectClosure* cl = (par ? _par_closures[worker] : _closure);
	  if (nd != NULL) {
	    void **buf = BufferNode::make_buffer_from_node(nd);
	    ObjPtrQueue::apply_closure_to_buffer(cl, buf, 0, _sz);
	    deallocate_buffer(buf);
	    return true;
	  } else {
	    return false;
	  }
	}
	
```


