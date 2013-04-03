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
void CMTask::regular_clock_call() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 何らかの理由でこの CMTask が中断されていたら, ここでリターン
      ---------------------------------------- -}

	  if (has_aborted())
	    return;
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // First, we need to recalculate the words scanned and refs reached
	  // limits for the next clock call.
	  recalculate_limits();
	
  {- -------------------------------------------
  (1) 以下のチェックを行う.
      (1) ...
      (2) ...
      (3) ...
      (4) ...
      (5) ...
      (6) ...
      ---------------------------------------- -}

	  // During the regular clock call we do the following
	
	  // (1) If an overflow has been flagged, then we abort.
	  if (_cm->has_overflown()) {
	    set_has_aborted();
	    return;
	  }
	
	  // If we are not concurrent (i.e. we're doing remark) we don't need
	  // to check anything else. The other steps are only needed during
	  // the concurrent marking phase.
	  if (!concurrent())
	    return;
	
	  // (2) If marking has been aborted for Full GC, then we also abort.
	  if (_cm->has_aborted()) {
	    set_has_aborted();
	    statsOnly( ++_aborted_cm_aborted );
	    return;
	  }
	
	  double curr_time_ms = os::elapsedVTime() * 1000.0;
	
	  // (3) If marking stats are enabled, then we update the step history.
	#if _MARKING_STATS_
	  if (_words_scanned >= _words_scanned_limit)
	    ++_clock_due_to_scanning;
	  if (_refs_reached >= _refs_reached_limit)
	    ++_clock_due_to_marking;
	
	  double last_interval_ms = curr_time_ms - _interval_start_time_ms;
	  _interval_start_time_ms = curr_time_ms;
	  _all_clock_intervals_ms.add(last_interval_ms);
	
	  if (_cm->verbose_medium()) {
	    gclog_or_tty->print_cr("[%d] regular clock, interval = %1.2lfms, "
	                           "scanned = %d%s, refs reached = %d%s",
	                           _task_id, last_interval_ms,
	                           _words_scanned,
	                           (_words_scanned >= _words_scanned_limit) ? " (*)" : "",
	                           _refs_reached,
	                           (_refs_reached >= _refs_reached_limit) ? " (*)" : "");
	  }
	#endif // _MARKING_STATS_
	
	  // (4) We check whether we should yield. If we have to, then we abort.
	  if (_cm->should_yield()) {
	    // We should yield. To do this we abort the task. The caller is
	    // responsible for yielding.
	    set_has_aborted();
	    statsOnly( ++_aborted_yield );
	    return;
	  }
	
	  // (5) We check whether we've reached our time quota. If we have,
	  // then we abort.
	  double elapsed_time_ms = curr_time_ms - _start_time_ms;
	  if (elapsed_time_ms > _time_target_ms) {
	    set_has_aborted();
	    _has_timed_out = true;
	    statsOnly( ++_aborted_timed_out );
	    return;
	  }
	
	  // (6) Finally, we check whether there are enough completed STAB
	  // buffers available for processing. If there are, we abort.
	  SATBMarkQueueSet& satb_mq_set = JavaThread::satb_mark_queue_set();
	  if (!_draining_satb_buffers && satb_mq_set.process_completed_buffers()) {
	    if (_cm->verbose_low())
	      gclog_or_tty->print_cr("[%d] aborting to deal with pending SATB buffers",
	                             _task_id);
	    // we do need to process SATB buffers, we'll abort and restart
	    // the marking task to do so
	    set_has_aborted();
	    statsOnly( ++_aborted_satb );
	    return;
	  }
	}
	
```


