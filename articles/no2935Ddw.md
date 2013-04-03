---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/dirtyCardQueue.cpp

### 名前(function name)
```
bool DirtyCardQueueSet::
apply_closure_to_completed_buffer_helper(CardTableEntryClosure* cl,
                                         int worker_i,
                                         BufferNode* nd) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) nd 引数が NULL でなければ, 
      その BufferNode を処理し, true もしくは false をリターンする.
      ---------------------------------------- -}

	  if (nd != NULL) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    void **buf = BufferNode::make_buffer_from_node(nd);
	    size_t index = nd->index();

    {- -------------------------------------------
  (1.1) DirtyCardQueue::apply_closure_to_buffer() を呼んで, 
        対象のバッファに対して, cl 引数で指定された Closure を適用する.
    
        もし処理が完了したら, 
        処理し終わったバッファを PtrQueueSet::deallocate_buffer() で開放し, 
        true をリターンする.
    
        逆に, 処理が途中で失敗した場合は, 
        PtrQueueSet::enqueue_complete_buffer() を呼んで
        もう一度バッファを登録しなおした後, 
        false をリターンする.
        ---------------------------------------- -}

	    bool b =
	      DirtyCardQueue::apply_closure_to_buffer(cl, buf,
	                                              index, _sz,
	                                              true, worker_i);
	    if (b) {
	      deallocate_buffer(buf);
	      return true;  // In normal case, go on to next buffer.
	    } else {
	      enqueue_complete_buffer(buf, index);
	      return false;
	    }

  {- -------------------------------------------
  (1) nd 引数が NULL の場合には, false をリターンするだけ.
      ---------------------------------------- -}

	  } else {
	    return false;
	  }
	}
	
```


