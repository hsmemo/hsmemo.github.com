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
void SafepointSynchronize::end_statistics(jlong vmop_end_time) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _safepoint_stats[_cur_stat_index] 及び _max_vmop_time に入っている統計情報を更新する.
      また, 以下の条件が成り立てば, SafepointSynchronize::print_statistics() で統計情報を出力する.
      * #TODO
      * #TODO
  
      (なお, 必要があれば _cur_stat_index の値も更新しておく)
      ---------------------------------------- -}

	  SafepointStats *spstat = &_safepoint_stats[_cur_stat_index];
	
	  // Update the vm operation time.
	  spstat->_time_to_exec_vmop = vmop_end_time -  cleanup_end_time;
	  if (spstat->_time_to_exec_vmop > _max_vmop_time) {
	    _max_vmop_time = spstat->_time_to_exec_vmop;
	  }
	  // Only the sync time longer than the specified
	  // PrintSafepointStatisticsTimeout will be printed out right away.
	  // By default, it is -1 meaning all samples will be put into the list.
	  if ( PrintSafepointStatisticsTimeout > 0) {
	    if (spstat->_time_to_sync > PrintSafepointStatisticsTimeout * MICROUNITS) {
	      print_statistics();
	    }
	  } else {
	    // The safepoint statistics will be printed out when the _safepoin_stats
	    // array fills up.
	    if (_cur_stat_index == PrintSafepointStatisticsCount - 1) {
	      print_statistics();
	      _cur_stat_index = 0;
	    } else {
	      _cur_stat_index++;
	    }
	  }
	}
	
```


