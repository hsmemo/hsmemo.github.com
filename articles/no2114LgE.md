---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPermGen.cpp

### 名前(function name)
```
void PSPermGen::compute_new_size(size_t used_before_collection) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 前回の GC 時からの増加量を計算して, _avg_size の値を更新しておく.
      ---------------------------------------- -}

	  // Update our padded average of objects allocated in perm
	  // gen between collections.
	  assert(used_before_collection >= _last_used,
	                                "negative allocation amount since last GC?");
	
	  const size_t alloc_since_last_gc = used_before_collection - _last_used;
	  _avg_size->sample(alloc_since_last_gc);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const size_t current_live = used_in_bytes();
	  // Stash away the current amount live for the next call to this method.
	  _last_used = current_live;
	
	  // We have different alignment constraints than the rest of the heap.
	  const size_t alignment = MAX2(MinPermHeapExpansion,
	                                virtual_space()->alignment());
	
  {- -------------------------------------------
  (1) 新しい領域長を以下のように計算する.
      (1) まず次の値を計算する
          「現在の使用量(GC後の使用量. 以下の current_live) 
            + 次のGCまでの間に増える量(の予測値. これまでの実績量の平均を使用. 以下の _avg_size->padded_average())」 
      (2) この値を, MinPermHeapExpansion または PSVirtualSpace::alignment() の大きい方の値(以下の alignment)で切り上げる
      (3) その結果が _max_gen_size を超えていたり _min_gen_size を下回っている場合には,  
          それぞれ _max_gen_size/_min_gen_size に切り詰める.
          (_min_gen_size ~ _max_gen_size の範囲に収まっていれば, (2) の結果をそのまま使用する)
      ---------------------------------------- -}

	  // Compute the desired size:
	  //  The free space is the newly computed padded average,
	  //  so the desired size is what's live + the free space.
	  size_t desired_size = current_live + (size_t)_avg_size->padded_average();
	  desired_size = align_size_up(desired_size, alignment);
	
	  // ...and no larger or smaller than our max and min allowed.
	  desired_size = MAX2(MIN2(desired_size, _max_gen_size), _min_gen_size);
	  assert(desired_size <= _max_gen_size, "just checking");
	
  {- -------------------------------------------
  (1) もし計算結果が現在の領域長と同じであれば, 何もすることはないので, ここでリターン.
      ---------------------------------------- -}

	  const size_t size_before = _virtual_space->committed_size();
	
	  if (desired_size == size_before) {
	    // no change, we're done
	    return;
	  }
	
  {- -------------------------------------------
  (1) ExpandHeap_lock ロックを保持した状態で, Perm 領域の大きさを変更する.
      (計算結果が現在の領域長よりも大きければ, PSOldGen::expand_by() で領域を拡大する.
       逆に, 現在の領域長よりも小さければ, PSOldGen::shrink() で領域を縮小する.)
      ---------------------------------------- -}

	  {
	    // We'll be growing or shrinking the heap:  in either case,
	    // we need to hold a lock.
	    MutexLocker x(ExpandHeap_lock);
	    if (desired_size > size_before) {
	      const size_t change_bytes = desired_size - size_before;
	      const size_t aligned_change_bytes =
	        align_size_up(change_bytes, alignment);
	      expand_by(aligned_change_bytes);
	    } else {
	      // Shrinking
	      const size_t change_bytes =
	        size_before - desired_size;
	      const size_t aligned_change_bytes = align_size_down(change_bytes, alignment);
	      shrink(aligned_change_bytes);
	    }
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  // While this code isn't controlled by AdaptiveSizePolicy, it's
	  // convenient to see all resizing decsions under the same flag.
	  if (PrintAdaptiveSizePolicy) {
	    ParallelScavengeHeap* heap = (ParallelScavengeHeap*)Universe::heap();
	    assert(heap->kind() == CollectedHeap::ParallelScavengeHeap, "Sanity");
	
	    gclog_or_tty->print_cr("AdaptiveSizePolicy::perm generation size: "
	                           "collection: %d "
	                           "(" SIZE_FORMAT ") -> (" SIZE_FORMAT ") ",
	                           heap->total_collections(),
	                           size_before, _virtual_space->committed_size());
	  }
	}
	
```


