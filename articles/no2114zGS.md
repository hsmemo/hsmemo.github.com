---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/space.cpp

### 名前(function name)
```
bool CompactibleSpace::insert_deadspace(size_t& allowed_deadspace_words,
                                        HeapWord* q, size_t deadlength) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数の allowed_deadspace_words が deadlength 未満の場合には
      allowed_deadspace_words を 0 にするだけで, 即リターン.
      (なお, 参照渡しなので, 呼び出し元の値も 0 になる), 
      (この場合, false をリターンする)
      
      逆に, 引数の allowed_deadspace_words が deadlength 以上の場合には, 
      CollectedHeap::fill_with_object() で deadlength 分だけのダミーオブジェクトを埋める.
      (このダミーオブジェクトは, oopDesc::set_mark() を呼んで, マーク済みの状態にしておく)
      (ついでに, allowed_deadspace_words の値も deadlength 分だけ小さくしておく.
       これは, 参照渡しなので, 呼び出し元の値も小さくなる)
      (この場合, true をリターンする)
      ---------------------------------------- -}

	  if (allowed_deadspace_words >= deadlength) {
	    allowed_deadspace_words -= deadlength;
	    CollectedHeap::fill_with_object(q, deadlength);
	    oop(q)->set_mark(oop(q)->mark()->set_marked());
	    assert((int) deadlength == oop(q)->size(), "bad filler object size");
	    // Recall that we required "q == compaction_top".
	    return true;
	  } else {
	    allowed_deadspace_words = 0;
	    return false;
	  }
	}
	
```


