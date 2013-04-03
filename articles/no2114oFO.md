---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/parMarkBitMap.cpp

### 名前(function name)
```
ParMarkBitMap::IterationStatus
ParMarkBitMap::iterate(ParMarkBitMapClosure* live_closure,
                       idx_t range_beg, idx_t range_end) const
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  DEBUG_ONLY(verify_bit(range_beg);)
	  DEBUG_ONLY(verify_bit(range_end);)
	  assert(range_beg <= range_end, "live range invalid");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // The bitmap routines require the right boundary to be word-aligned.
	  const idx_t search_end = BitMap::word_align_up(range_end);
	
	  idx_t cur_beg = find_obj_beg(range_beg, search_end);

  {- -------------------------------------------
  (1) 処理対象範囲の終端(range_end)に達するまで, 以下の whlie ループを繰り返す.
      
      ループ内で行う処理は以下の通り.
      (1) 現在処理対象となっているオブジェクトに対して, 
          ParMarkBitMap::find_obj_end() でその終端を見つける.
          (もし, 終端が今回の処理範囲(range_end)を超えていたら, ここでリターン.
           返値は ParMarkBitMap::incomplete)
      (1) そのオブジェクトの処理を行うために
          引数で渡された ParMarkBitMapClosure (以下の live_closure) に対して
          ParMarkBitMapClosure::do_addr() を呼び出す.
          (もし, do_addr() の返値が ParMarkBitMap::incomplete でなかった場合, 
           もうコンパクション先が一杯で作業が続けられないということなので, ここでリターン.
           返値は, do_addr() の返値をそのまま使用.)
      (1) ParMarkBitMap::find_obj_beg() を呼び出して, 次のオブジェクトの先頭アドレスを見つける.
      ---------------------------------------- -}

	  while (cur_beg < range_end) {
	    const idx_t cur_end = find_obj_end(cur_beg, search_end);
	    if (cur_end >= range_end) {
	      // The obj ends outside the range.
	      live_closure->set_source(bit_to_addr(cur_beg));
	      return incomplete;
	    }
	
	    const size_t size = obj_size(cur_beg, cur_end);
	    IterationStatus status = live_closure->do_addr(bit_to_addr(cur_beg), size);
	    if (status != incomplete) {
	      assert(status == would_overflow || status == full, "sanity");
	      return status;
	    }
	
	    // Successfully processed the object; look for the next object.
	    cur_beg = find_obj_beg(cur_end + 1, search_end);
	  }
	
  {- -------------------------------------------
  (1) 引数で渡された ParMarkBitMapClosure (以下の live_closure) の _source フィールドに
      今回の処理対象範囲の終端アドレス(range_end)をセットしておく.
      ---------------------------------------- -}

	  live_closure->set_source(bit_to_addr(range_end));

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return complete;
	}
	
```


