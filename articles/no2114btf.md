---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp
### 説明(description)
この関数は, dense prefix の終端とするのに最も適当なアドレスを返す 
(つまり, コンパクション処理の対象となる最初の位置のアドレスを返す).
なお, リターンされるアドレスは, 常に region 同士の境界地点になっている.
 
このアドレスは, 以下のようにして選ばれる.

  1. まず, 領域の先頭にある「完全に live オブジェクトだけで埋まっている region」は, 
     必ず dense prefix に含まれるものとする
     (なぜなら, これらはコンパクション後にも移動しないのは明らかなので).
  2. 次に, コンパクションせずに残しておいてもよいゴミ(死んでいるオブジェクト)の量を計算する
     (死んでいるオブジェクトでもコンパクションせずにそのままにしておくと, 
      コンパクション処理が減るため, オーバーヘッドが小さくなる).
  3. ゴミの量が 2. の結果に達するまでの全 region について, 
     その region を dense prefix の終端にした場合の効果を
     PSParallelCompact::reclaimed_ratio() で計算し, 
     最も効果が高いものを dense prefix の終端とする.

```
// Return the address of the end of the dense prefix, a.k.a. the start of the
// compacted region.  The address is always on a region boundary.
//
// Completely full regions at the left are skipped, since no compaction can
// occur in those regions.  Then the maximum amount of dead wood to allow is
// computed, based on the density (amount live / capacity) of the generation;
// the region with approximately that amount of dead space to the left is
// identified as the limit region.  Regions between the last completely full
// region and the limit region are scanned and the one that has the best
// (maximum) reclaimed_ratio() is selected.
```

### 名前(function name)
```
HeapWord*
PSParallelCompact::compute_dense_prefix(const SpaceId id,
                                        bool maximum_compaction)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)?? #TODO
      (ParallelOldGCSplitALot オプションが指定されている場合は, 既に計算済みなので??, ここでリターン)
      ---------------------------------------- -}

	  if (ParallelOldGCSplitALot) {
	    if (_space_info[id].dense_prefix() != _space_info[id].space()->bottom()) {
	      // The value was chosen to provoke splitting a young gen space; use it.
	      return _space_info[id].dense_prefix();
	    }
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const size_t region_size = ParallelCompactData::RegionSize;
	  const ParallelCompactData& sd = summary_data();
	
	  const MutableSpace* const space = _space_info[id].space();
	  HeapWord* const top = space->top();
	  HeapWord* const top_aligned_up = sd.region_align_up(top);
	  HeapWord* const new_top = _space_info[id].new_top();
	  HeapWord* const new_top_aligned_up = sd.region_align_up(new_top);
	  HeapWord* const bottom = space->bottom();
	  const RegionData* const beg_cp = sd.addr_to_region_ptr(bottom);
	  const RegionData* const top_cp = sd.addr_to_region_ptr(top_aligned_up);
	  const RegionData* const new_top_cp =
	    sd.addr_to_region_ptr(new_top_aligned_up);
	
  {- -------------------------------------------
  (1) PSParallelCompact::first_dead_space_region() を呼んで, 
      処理対象の Space の先頭にある「完全に live オブジェクトだけで埋まっている region」の数を取得する (以下の full_cp).
      (これらはコンパクションしても明らかに移動しないので, dense prefix は最低限これらを含んだ大きさでないといけない)
      ---------------------------------------- -}

	  // Skip full regions at the beginning of the space--they are necessarily part
	  // of the dense prefix.
	  const RegionData* const full_cp = first_dead_space_region(beg_cp, new_top_cp);
	  assert(full_cp->destination() == sd.region_to_addr(full_cp) ||
	         space->is_empty(), "no dead space allowed to the left");
	  assert(full_cp->data_size() < region_size || full_cp == new_top_cp - 1,
	         "region must have dead space");
	
  {- -------------------------------------------
  (1) 毎回全部コンパクションしていると遅いので, 普段は完全なコンパクションはしない.
      (完全なコンパクションをしない場合, 処理対象の領域の先頭にある程度の量のゴミが残っていてよいものとする)
      完全にコンパクションする場合には, full_cp に対応するアドレスを返値として, ここでリターン.
      (なおこの場合は, ついでに _maximum_compaction_gc_num の値を total_invocations() に更新する処理を行ってからリターンする)
  
      完全にコンパクションするかどうかは, 以下の条件によって決まる.
      (これらのうち, どれか1つでも満たされていれば完全にコンパクションする)
        * 直近の完全にコンパクションを行った時点 (_maximum_compaction_gc_num に記録されている) から数えて, 
          HeapMaximumCompactionInterval 回の Full GC が実行された場合
          (= 以下の gcs_since_max > HeapMaximumCompactionInterval が成り立つ場合)
        * Full GC の実行回数が, 初めて HeapFirstMaximumCompactionCount 回に到達した場合
        * 引数の maximum_compaction が true の場合 (= 空き容量が逼迫している場合)
        * dead オブジェクトが全く存在しない場合 (以下の full_cp == top_cp が成り立つ場合)
      ---------------------------------------- -}

	  // The gc number is saved whenever a maximum compaction is done, and used to
	  // determine when the maximum compaction interval has expired.  This avoids
	  // successive max compactions for different reasons.
	  assert(total_invocations() >= _maximum_compaction_gc_num, "sanity");
	  const size_t gcs_since_max = total_invocations() - _maximum_compaction_gc_num;
	  const bool interval_ended = gcs_since_max > HeapMaximumCompactionInterval ||
	    total_invocations() == HeapFirstMaximumCompactionCount;
	  if (maximum_compaction || full_cp == top_cp || interval_ended) {
	    _maximum_compaction_gc_num = total_invocations();
	    return sd.region_to_addr(full_cp);
	  }
	
  {- -------------------------------------------
  (1) 完全なコンパクションをしない場合は, 
      引き続いて, 残っていてよいゴミの量 (以下の dead_wood_limit) を計算する.
  
      (dead_wood_limit は, 処理対象の領域の大きさ(以下の space_capacity)に, 
       PSParallelCompact::dead_wood_limiter() で求めた比率(以下の limiter)を掛けたものとする.
  
       ただし, この計算結果が
       処理対象の領域内にある dead オブジェクトの合計量 (以下の dead_wood_max) を超える場合は, 
       (合計量よりも多くのゴミが残ることはあり得ないので) dead_wood_max に切り詰めておく)
      ---------------------------------------- -}

	  const size_t space_live = pointer_delta(new_top, bottom);
	  const size_t space_used = space->used_in_words();
	  const size_t space_capacity = space->capacity_in_words();
	
	  const double density = double(space_live) / double(space_capacity);
	  const size_t min_percent_free =
	          id == perm_space_id ? PermMarkSweepDeadRatio : MarkSweepDeadRatio;
	  const double limiter = dead_wood_limiter(density, min_percent_free);
	  const size_t dead_wood_max = space_used - space_live;
	  const size_t dead_wood_limit = MIN2(size_t(space_capacity * limiter),
	                                      dead_wood_max);
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceParallelOldGCDensePrefix) {
	    tty->print_cr("space_live=" SIZE_FORMAT " " "space_used=" SIZE_FORMAT " "
	                  "space_cap=" SIZE_FORMAT,
	                  space_live, space_used,
	                  space_capacity);
	    tty->print_cr("dead_wood_limiter(%6.4f, %d)=%6.4f "
	                  "dead_wood_max=" SIZE_FORMAT " dead_wood_limit=" SIZE_FORMAT,
	                  density, min_percent_free, limiter,
	                  dead_wood_max, dead_wood_limit);
	  }
	
  {- -------------------------------------------
  (1) PSParallelCompact::dead_wood_limit_region() を呼んで, 
      「その region より前に存在する dead オブジェクトの総量が
      dead_wood_limit を超える最初の region」を求める (以下の limit_cp).
      ---------------------------------------- -}

	  // Locate the region with the desired amount of dead space to the left.
	  const RegionData* const limit_cp =
	    dead_wood_limit_region(full_cp, top_cp, dead_wood_limit);
	
  {- -------------------------------------------
  (1) 実際の dense prefix の長さは, 
      以上で求めた full_cp から limit_cp の間で, 一番効果が高いものとする.
      以下の for ループ内で, full_cp から limit_cp までの値に対して
      PSParallelCompact::reclaimed_ratio() で効果を推定し, 
      最も効果が高いものを選ぶ (以下の best_cp).
      ---------------------------------------- -}

	  // Scan from the first region with dead space to the limit region and find the
	  // one with the best (largest) reclaimed ratio.
	  double best_ratio = 0.0;
	  const RegionData* best_cp = full_cp;
	  for (const RegionData* cp = full_cp; cp < limit_cp; ++cp) {
	    double tmp_ratio = reclaimed_ratio(cp, bottom, top, new_top);
	    if (tmp_ratio > best_ratio) {
	      best_cp = cp;
	      best_ratio = tmp_ratio;
	    }
	  }
	
  {- -------------------------------------------
  (1) (ここまでで計算した best_cp が full_cp にかなり近い場合(4 region 未満の距離の場合)には 
       dense prefix の終端を full_cp にしてしまう, という処理が入っていたようだが 
       #if 0 で消されている...)
      ---------------------------------------- -}

	#if     0
	  // Something to consider:  if the region with the best ratio is 'close to' the
	  // first region w/free space, choose the first region with free space
	  // ("first-free").  The first-free region is usually near the start of the
	  // heap, which means we are copying most of the heap already, so copy a bit
	  // more to get complete compaction.
	  if (pointer_delta(best_cp, full_cp, sizeof(RegionData)) < 4) {
	    _maximum_compaction_gc_num = total_invocations();
	    best_cp = full_cp;
	  }
	#endif  // #if 0
	
  {- -------------------------------------------
  (1) best_cp に対応するアドレスをリターン.
      ---------------------------------------- -}

	  return sd.region_to_addr(best_cp);
	}
	
```


