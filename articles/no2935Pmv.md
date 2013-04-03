---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp

### 名前(function name)
```
bool G1NoteEndOfConcMarkClosure::doHeapRegion(HeapRegion *hr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // We use a claim value of zero here because all regions
	  // were claimed with value 1 in the FinalCount task.
	  hr->reset_gc_time_stamp();

  {- -------------------------------------------
  (1) (処理対象が Humongous オブジェクトの途中に当たる HeapRegion の場合は, 
       特にすることはないので, 以下のブロック内の処理は省略)
      ---------------------------------------- -}

	  if (!hr->continuesHumongous()) {

    {- -------------------------------------------
  (1.1) HeapRegion::note_end_of_marking() を呼んで, 
        next TAMS と prev TAMS を入れ替える.
  
        また, G1CollectedHeap::free_region_if_empty() を呼んで
        空の HeapRegion の回収を行う.
        (ついでに, 空でない HeapRegion については,
         expand された SparsePRT オブジェクトの情報を収集する)
  
        また, この G1NoteEndOfConcMarkClosure オブジェクトの以下のフィールド内に
        処理した HeapRegion に関する情報を蓄積していく.
        * _regions_claimed : 処理した HeapRegion の個数
        * _max_live_bytes : 処理した HeapRegion 中の live object 量の合計値
        * _claimed_region_time : 処理に掛かった合計時間
        * _max_region_time : 1 HeapRegion あたりの処理時間の最大値
        ---------------------------------------- -}

	    double start = os::elapsedTime();
	    _regions_claimed++;
	    hr->note_end_of_marking();
	    _max_live_bytes += hr->max_live_bytes();
	    _g1->free_region_if_empty(hr,
	                              &_freed_bytes,
	                              _local_cleanup_list,
	                              _humongous_proxy_set,
	                              _hrrs_cleanup_task,
	                              true /* par */);
	    double region_time = (os::elapsedTime() - start);
	    _claimed_region_time += region_time;
	    if (region_time > _max_region_time) _max_region_time = region_time;
	  }

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return false;
	}
	
```


