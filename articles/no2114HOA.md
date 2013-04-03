---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp

### 名前(function name)
```
void VmThreadCountClosure::do_thread(Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 「JavaThread で、かつ HotSpot が内部的に使用するスレッドでは無いもの」の数を数える.
      (この条件に当てはまるスレッドの場合のみ _count フィールドをインクリメントする)
      ---------------------------------------- -}

	  // exclude externally visible JavaThreads
	  if (thread->is_Java_thread() && !thread->is_hidden_from_external_view()) {
	    return;
	  }
	
	  _count++;
	}
	
```


