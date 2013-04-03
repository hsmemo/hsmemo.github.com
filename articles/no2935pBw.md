---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.cpp

### 名前(function name)
```
void ConcurrentGCThread::create_and_start() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) os::create_thread() を呼んで, 新しいスレッドを生成する.
      成功したら, さらに os::set_priority() を呼んで, 優先度の設定を行う.
      最後に, (_should_terminate フィールドや DisableStartThread オプションが false であれば)
      os::start_thread() を呼んでスレッドの実行を開始させる.
      ---------------------------------------- -}

	  if (os::create_thread(this, os::cgc_thread)) {
	    // XXX: need to set this to low priority
	    // unless "agressive mode" set; priority
	    // should be just less than that of VMThread.
	    os::set_priority(this, NearMaxPriority);
	    if (!_should_terminate && !DisableStartThread) {
	      os::start_thread(this);
	    }
	  }
	}
	
```


