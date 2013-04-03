---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/safepoint.cpp

### 名前(function name)
```
void SafepointSynchronize::update_statistics_on_sync_end(jlong end_time) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _safepoint_stats[_cur_stat_index] に入っている統計情報を更新する.
      ---------------------------------------- -}

	  SafepointStats *spstat = &_safepoint_stats[_cur_stat_index];
	
	  if (spstat->_nof_threads_wait_to_block != 0) {
	    spstat->_time_to_wait_to_block = end_time -
	      spstat->_time_to_wait_to_block;
	  }
	
	  // Records the end time of sync which will be used to calculate the total
	  // vm operation time. Again, the real time spending in syncing will be deducted
	  // from the start of the sync time later when end_statistics is called.
	  spstat->_time_to_sync = end_time - _safepoint_begin_time;
	  if (spstat->_time_to_sync > _max_sync_time) {
	    _max_sync_time = spstat->_time_to_sync;
	  }
	
	  spstat->_time_to_do_cleanups = end_time;
	}
	
```


