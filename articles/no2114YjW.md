---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp

### 名前(function name)
```
void PSParallelCompact::enqueue_region_draining_tasks(GCTaskQueue* q,
                                                      uint parallel_gc_threads)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  TraceTime tm("drain task setup", print_phases(), true, gclog_or_tty);
	
  {- -------------------------------------------
  (1) 引数で指定された GC スレッド数(以下の parallel_gc_threads)分だけ
      DrainStacksCompactionTask をキューに追加.
      (なお, parallel_gc_threads が 0 だった場合には, 
       DrainStacksCompactionTask を1個だけキューに追加)
      ---------------------------------------- -}

	  const unsigned int task_count = MAX2(parallel_gc_threads, 1U);
	  for (unsigned int j = 0; j < task_count; j++) {
	    q->enqueue(new DrainStacksCompactionTask(j));
	  }
	
  {- -------------------------------------------
  (1) (以下で, 今すぐ処理できる region を探し, それを GC スレッドに割り当てていく.
  
       なお, region が昇順で埋まっていった方が効率がいいので 
       (See: PSParallelCompact::decrement_destination_counts())
       この処理はアドレスが高い領域から順に調べていく.
       また各領域内でもアドレスが大きい方から順に調べていく.
       (stack 式なので入れた順とは逆順に処理される))
      ---------------------------------------- -}

	  // Find all regions that are available (can be filled immediately) and
	  // distribute them to the thread stacks.  The iteration is done in reverse
	  // order (high to low) so the regions will be removed in ascending order.
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const ParallelCompactData& sd = PSParallelCompact::summary_data();
	
	  size_t fillable_regions = 0;   // A count for diagnostic purposes.
	  unsigned int which = 0;       // The worker thread number.
	
  {- -------------------------------------------
  (1) 以下の for ループ内で, Perm 以外の全ての領域(To, From, Eden, Old)について
      今すぐ処理できる region を探し
      GC スレッド (GCTaskThread) に割り当てていく.
    
      (各領域の dense prefix の終端(以下の beg_region)から 
       new_top の位置(以下の end_region) までに含まれる region を対象とし, 
       各 region に対して ParallelCompactData::RegionData::claim_unsafe() を呼び出して 
       今すぐ処理できるかどうかを調べていく.
       
       なお, ParallelCompactData::RegionData::claim_unsafe() は, 
       コンパクション先の region 数(destination count)が 0 なら true を返す.
       (つまり, コンパクション先が自分自身の中に収まっているか, あるいはそもそも live オブジェクトが1つもなければ true)
  
       ParallelCompactData::RegionData::claim_unsafe() が true を返した場合には, 
       その region を ParCompactionManager::push_region() で GC スレッドに割り当てる.
       なお, 登録先の ParCompactionManager は順繰りに(round-robin で)変えていく.
         (ある ParCompactionManager に1つ region を登録したら, 
          次の region は次の ParCompactionManager に登録する.
          最後の ParCompactionManager まで行き着いたら, また最初の ParCompactionManager に戻って繰り返す.)
      ---------------------------------------- -}

	  for (unsigned int id = to_space_id; id > perm_space_id; --id) {
	    SpaceInfo* const space_info = _space_info + id;
	    MutableSpace* const space = space_info->space();
	    HeapWord* const new_top = space_info->new_top();
	
	    const size_t beg_region = sd.addr_to_region_idx(space_info->dense_prefix());
	    const size_t end_region =
	      sd.addr_to_region_idx(sd.region_align_up(new_top));
	    assert(end_region > 0, "perm gen cannot be empty");
	
	    for (size_t cur = end_region - 1; cur >= beg_region; --cur) {
	      if (sd.region(cur)->claim_unsafe()) {
	        ParCompactionManager* cm = ParCompactionManager::manager_array(which);
	        cm->push_region(cur);
	
	        if (TraceParallelOldGCCompactionPhase && Verbose) {
	          const size_t count_mod_8 = fillable_regions & 7;
	          if (count_mod_8 == 0) gclog_or_tty->print("fillable: ");
	          gclog_or_tty->print(" " SIZE_FORMAT_W(7), cur);
	          if (count_mod_8 == 7) gclog_or_tty->cr();
	        }
	
	        NOT_PRODUCT(++fillable_regions;)
	
	        // Assign regions to threads in round-robin fashion.
	        if (++which == task_count) {
	          which = 0;
	        }
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceParallelOldGCCompactionPhase) {
	    if (Verbose && (fillable_regions & 7) != 0) gclog_or_tty->cr();
	    gclog_or_tty->print_cr("%u initially fillable regions", fillable_regions);
	  }
	}
	
```


