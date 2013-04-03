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
bool DirtyCardQueueSet::apply_closure_to_completed_buffer(CardTableEntryClosure* cl,
                                                          int worker_i,
                                                          int stop_at,
                                                          bool during_pause) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!during_pause || stop_at == 0, "Should not leave any completed buffers during a pause");

  {- -------------------------------------------
  (1) DirtyCardQueueSet::get_completed_buffer() を呼んで, 次の BufferNode を取得する.
      ---------------------------------------- -}

	  BufferNode* nd = get_completed_buffer(stop_at);

  {- -------------------------------------------
  (1) DirtyCardQueueSet::apply_closure_to_completed_buffer_helper() を呼んで, 
      取得した BufferNode に対して, cl 引数で指定された Closure を適用する.
    
      処理が成功した場合は (= 返値が true だった場合は), 
      _processed_buffers_rs_thread の値をアトミックに1増加させておく.
      ---------------------------------------- -}

	  bool res = apply_closure_to_completed_buffer_helper(cl, worker_i, nd);
	  if (res) Atomic::inc(&_processed_buffers_rs_thread);

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return res;
	}
	
```


