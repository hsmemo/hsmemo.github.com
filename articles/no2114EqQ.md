---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/cardTableRS.cpp

### 名前(function name)
```
void ClearNoncleanCardWrapper::do_MemRegion(MemRegion mr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(mr.word_size() > 0, "Error");
	  assert(_ct->is_aligned(mr.start()), "mr.start() should be card aligned");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // mr.end() may not necessarily be card aligned.
	  jbyte* cur_entry = _ct->byte_for(mr.last());
	  const jbyte* limit = _ct->byte_for(mr.start());
	  HeapWord* end_of_non_clean = mr.end();
	  HeapWord* start_of_non_clean = end_of_non_clean;

  {- -------------------------------------------
  (1) 以下のループで引数で指定されたメモリ範囲を調べ, 
      dirty が連続する箇所を全て見つけだす.
    
      見つかった範囲については, (その範囲を表す MemRegion オブジェクトを引数として) 
      DirtyCardToOopClosure::do_MemRegion() を呼び出して処理を行う.
      ---------------------------------------- -}

	  while (cur_entry >= limit) {
	    HeapWord* cur_hw = _ct->addr_for(cur_entry);
	    if ((*cur_entry != CardTableRS::clean_card_val()) && clear_card(cur_entry)) {
	      // Continue the dirty range by opening the
	      // dirty window one card to the left.
	      start_of_non_clean = cur_hw;
	    } else {
	      // We hit a "clean" card; process any non-empty
	      // "dirty" range accumulated so far.
	      if (start_of_non_clean < end_of_non_clean) {
	        const MemRegion mrd(start_of_non_clean, end_of_non_clean);
	        _dirty_card_closure->do_MemRegion(mrd);
	      }
	      // Reset the dirty window, while continuing to look
	      // for the next dirty card that will start a
	      // new dirty window.
	      end_of_non_clean = cur_hw;
	      start_of_non_clean = cur_hw;
	    }
	    // Note that "cur_entry" leads "start_of_non_clean" in
	    // its leftward excursion after this point
	    // in the loop and, when we hit the left end of "mr",
	    // will point off of the left end of the card-table
	    // for "mr".
	    cur_entry--;
	  }

  {- -------------------------------------------
  (1) もし未処理な dirty range が残っていれば, その範囲も処理しておく.
      ---------------------------------------- -}

	  // If the first card of "mr" was dirty, we will have
	  // been left with a dirty window, co-initial with "mr",
	  // which we now process.
	  if (start_of_non_clean < end_of_non_clean) {
	    const MemRegion mrd(start_of_non_clean, end_of_non_clean);
	    _dirty_card_closure->do_MemRegion(mrd);
	  }
	}
	
```


