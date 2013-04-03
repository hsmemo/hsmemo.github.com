---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/safepoint.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) _waiting_to_block フィールドの値を減少させるだけ.
      (この値は safepoint を開始していいかどうかの判定に用いられる.
       See: SafepointSynchronize::begin())
      ---------------------------------------- -}

	  static void   signal_thread_at_safepoint()              { _waiting_to_block--; }
	
```


