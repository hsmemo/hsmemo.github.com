---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionManager.inline.hpp

### 名前(function name)
```
inline void PSPromotionManager::process_popped_location_depth(StarTask p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし処理対象のポインタが途中まで処理済みの配列を指していたら (See: PSPromotionManager::is_oop_masked()), 
      PSPromotionManager::process_array_chunk() で処理を行う.
      ---------------------------------------- -}

	  if (is_oop_masked(p)) {
	    assert(PSChunkLargeArrays, "invariant");
	    oop const old = unmask_chunked_array_oop(p);
	    process_array_chunk(old);

  {- -------------------------------------------
  (1) そうでなければ PSScavenge::copy_and_push_safe_barrier() で処理を行う.
      (なお, PSScavenge::copy_and_push_safe_barrier() は
       引数が narrowOop* でも oop* でも使えるように, 引数の型が template でパラメタライズされている.
       ここでの呼び出しでは, 対象のポインタを適切にキャストして呼び出し先を指定している.)
      ---------------------------------------- -}

	  } else {
	    if (p.is_narrow()) {
	      assert(UseCompressedOops, "Error");
	      PSScavenge::copy_and_push_safe_barrier(this, (narrowOop*)p);
	    } else {
	      PSScavenge::copy_and_push_safe_barrier(this, (oop*)p);
	    }
	  }
	}
	
```


