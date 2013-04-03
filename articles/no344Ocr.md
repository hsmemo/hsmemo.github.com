---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/defNewGeneration.cpp

### 名前(function name)
```
HeapWord* DefNewGeneration::allocate(size_t word_size,
                                     bool is_tlab) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) まずは, EdenSpace::par_allocate() で確保を試みる.
      成功すればここでリターン.
      ---------------------------------------- -}

	  // This is the slow-path allocation for the DefNewGeneration.
	  // Most allocations are fast-path in compiled code.
	  // We try to allocate from the eden.  If that works, we are happy.
	  // Note that since DefNewGeneration supports lock-free allocation, we
	  // have to use it here, as well.
	  HeapWord* result = eden()->par_allocate(word_size);
	  if (result != NULL) {
	    return result;
	  }

  {- -------------------------------------------
  (1) Eden の soft limit (soft_end()) が hard limit (end()) より小さければ, soft limit を増加させて確保を試みる.
      (以下, 確保が成功するか soft limit が hard limit に等しくなるまでループ) (詳細は以下参照)
      確保が成功すればここでリターン.
      ---------------------------------------- -}

	  do {

    {- -------------------------------------------
  (1.1) soft limit が hard limit より小さければ, soft limit を増加させる.
        ---------------------------------------- -}

	    HeapWord* old_limit = eden()->soft_end();
	    if (old_limit < eden()->end()) {
	      // Tell the next generation we reached a limit.
	      HeapWord* new_limit =
	        next_gen()->allocation_limit_reached(eden(), eden()->top(), word_size);
	      if (new_limit != NULL) {
	        Atomic::cmpxchg_ptr(new_limit, eden()->soft_end_addr(), old_limit);
	      } else {
	        assert(eden()->soft_end() == eden()->end(),
	               "invalid state after allocation_limit_reached returned null");
	      }

    {- -------------------------------------------
  (1.1) soft limit が hard limit と等しければ, ループを抜ける.
        ---------------------------------------- -}

	    } else {
	      // The allocation failed and the soft limit is equal to the hard limit,
	      // there are no reasons to do an attempt to allocate
	      assert(old_limit == eden()->end(), "sanity check");
	      break;
	    }

    {- -------------------------------------------
  (1.1) EdenSpace::par_allocate() で確保を試みる.
        ---------------------------------------- -}

	    // Try to allocate until succeeded or the soft limit can't be adjusted
	    result = eden()->par_allocate(word_size);
	  } while (result == NULL);
	
  {- -------------------------------------------
  (1) もしまだ確保が成功していなければ, 
      DefNewGeneration::allocate_from_space() で from 領域からの確保を試みる.
      ---------------------------------------- -}

	  // If the eden is full and the last collection bailed out, we are running
	  // out of heap space, and we try to allocate the from-space, too.
	  // allocate_from_space can't be inlined because that would introduce a
	  // circular dependency at compile time.
	  if (result == NULL) {
	    result = allocate_from_space(word_size);
	  }

  {- -------------------------------------------
  (1) 結果をリターンする
      ---------------------------------------- -}

	  return result;
	}
	
```


