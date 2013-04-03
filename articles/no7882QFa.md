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
void SafepointSynchronize::update_statistics_on_spin_end() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _safepoint_stats[_cur_stat_index] に入っている統計情報を更新する.
      ---------------------------------------- -}

	  SafepointStats *spstat = &_safepoint_stats[_cur_stat_index];
	
	  jlong cur_time = os::javaTimeNanos();
	
	  spstat->_nof_threads_wait_to_block = _waiting_to_block;
	  if (spstat->_nof_initial_running_threads != 0) {
	    spstat->_time_to_spin = cur_time - spstat->_time_to_spin;
	  }
	
	  if (need_to_track_page_armed_status) {
	    spstat->_page_armed = (PageArmed == 1);
	  }
	
	  // Records the start time of waiting for to block. Updated when block is done.
	  if (_waiting_to_block != 0) {
	    spstat->_time_to_wait_to_block = cur_time;
	  } else {
	    spstat->_time_to_wait_to_block = 0;
	  }
	}
	
```


