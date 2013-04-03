---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp

### 名前(function name)
```
bool PSScavenge::should_attempt_scavenge() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ParallelScavengeHeap* heap = (ParallelScavengeHeap*)Universe::heap();
	  assert(heap->kind() == CollectedHeap::ParallelScavengeHeap, "Sanity");
	  PSGCAdaptivePolicyCounters* counters = heap->gc_policy_counters();
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録) (See: PSGCAdaptivePolicyCounters)
      ---------------------------------------- -}

	  if (UsePerfData) {
	    counters->update_scavenge_skipped(not_skipped);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  PSYoungGen* young_gen = heap->young_gen();
	  PSOldGen* old_gen = heap->old_gen();
	
  {- -------------------------------------------
  (1) もし To 領域が空でなければ, Scavenge GC では駄目だと判断して, false をリターンする.
      (ついでに, _consecutive_skipped_scavenges フィールドの値をインクリメントしておく.
       UsePerfData オプションが指定されている場合には, (プロファイル情報の記録)も行っておく. (See: PSGCAdaptivePolicyCounters))
    
      (ただし, ScavengeWithObjectsInToSpace オプションが指定されていれば, 
       To 領域が空かどうかに関わらず, 
       ここでの処理は全てスキップしてこのまま処理を続ける)
      ---------------------------------------- -}

	  if (!ScavengeWithObjectsInToSpace) {
	    // Do not attempt to promote unless to_space is empty
	    if (!young_gen->to_space()->is_empty()) {
	      _consecutive_skipped_scavenges++;
	      if (UsePerfData) {
	        counters->update_scavenge_skipped(to_space_not_empty);
	      }
	      return false;
	    }
	  }
	
  {- -------------------------------------------
  (1) 昇格すると予想される量を計算し (以下の promotion_estimate), 
      現在の Old 領域の空き容量と比較する (比較結果は以下の result).
  
      (この比較結果が最終的な返値となる)
      
      なお, 昇格すると予想される量は, 以下の2つの値の小さい方とする.
        * これまでの GC における昇格量の平均値
          (より具体的には, PSAdaptiveSizePolicy::padded_average_promoted_in_bytes() が返す値)
        * 現在の New 領域の使用量   
          (原理的に昇格量がこれを超えることはあり得ない)
      ---------------------------------------- -}

	  // Test to see if the scavenge will likely fail.
	  PSAdaptiveSizePolicy* policy = heap->size_policy();
	
	  // A similar test is done in the policy's should_full_GC().  If this is
	  // changed, decide if that test should also be changed.
	  size_t avg_promoted = (size_t) policy->padded_average_promoted_in_bytes();
	  size_t promotion_estimate = MIN2(avg_promoted, young_gen->used_in_bytes());
	  bool result = promotion_estimate < old_gen->free_in_bytes();
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (PrintGCDetails && Verbose) {
	    gclog_or_tty->print(result ? "  do scavenge: " : "  skip scavenge: ");
	    gclog_or_tty->print_cr(" average_promoted " SIZE_FORMAT
	      " padded_average_promoted " SIZE_FORMAT
	      " free in old gen " SIZE_FORMAT,
	      (size_t) policy->average_promoted_in_bytes(),
	      (size_t) policy->padded_average_promoted_in_bytes(),
	      old_gen->free_in_bytes());
	    if (young_gen->used_in_bytes() <
	        (size_t) policy->padded_average_promoted_in_bytes()) {
	      gclog_or_tty->print_cr(" padded_promoted_average is greater"
	        " than maximum promotion = " SIZE_FORMAT, young_gen->used_in_bytes());
	    }
	  }
	
  {- -------------------------------------------
  (1) 比較結果に基づいて _consecutive_skipped_scavenges フィールドの値を変更しておく.
        * Scavenge GC で問題なさそうなら, _consecutive_skipped_scavenges を 0 にリセットする.
        * Scavenge GC では駄目そうなら, _consecutive_skipped_scavenges をインクリメントする.
          (この場合, UsePerfData オプションが指定されている場合には(プロファイル情報の記録)も行っておく. 
           (See: PSGCAdaptivePolicyCounters))
      ---------------------------------------- -}

	  if (result) {
	    _consecutive_skipped_scavenges = 0;
	  } else {
	    _consecutive_skipped_scavenges++;
	    if (UsePerfData) {
	      counters->update_scavenge_skipped(promoted_too_large);
	    }
	  }

  {- -------------------------------------------
  (1) 比較結果をリターン
      ---------------------------------------- -}

	  return result;
	}
	
```


