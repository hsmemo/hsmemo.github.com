---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/synchronizer.cpp

### 名前(function name)
```
void ObjectSynchronizer::monitors_iterate(MonitorClosure* closure) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ObjectMonitor* block = gBlockList;
	  ObjectMonitor* mid;

  {- -------------------------------------------
  (1) 重量ロック状態になっている全てのオブジェクトに対して, 引数の OopClosure を適用する.
      (See: ObjectMonitor)
  
      (現在使用中の ObjectMonitor オブジェクトは gBlockList フィールドから始まるリストにつながれているので, 
       このリストを先頭から最後まで見て, (ObjectMonitor オブジェクトの object フィールドが NULL でないものについて)
       MonitorClosure::do_monitor() を適用していくだけ.)
      ---------------------------------------- -}

	  while (block) {
	    assert(block->object() == CHAINMARKER, "must be a block header");
	    for (int i = _BLOCKSIZE - 1; i > 0; i--) {
	      mid = block + i;
	      oop object = (oop) mid->object();
	      if (object != NULL) {
	        closure->do_monitor(mid);
	      }
	    }
	    block = (ObjectMonitor*) block->FreeNext;
	  }
	}
	
```


