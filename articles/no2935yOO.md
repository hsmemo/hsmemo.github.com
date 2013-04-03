---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp
### 説明(description)

```
/*****************************************************************************

    The do_marking_step(time_target_ms) method is the building block
    of the parallel marking framework. It can be called in parallel
    with other invocations of do_marking_step() on different tasks
    (but only one per task, obviously) and concurrently with the
    mutator threads, or during remark, hence it eliminates the need
    for two versions of the code. When called during remark, it will
    pick up from where the task left off during the concurrent marking
    phase. Interestingly, tasks are also claimable during evacuation
    pauses too, since do_marking_step() ensures that it aborts before
    it needs to yield.

    The data structures that is uses to do marking work are the
    following:

      (1) Marking Bitmap. If there are gray objects that appear only
      on the bitmap (this happens either when dealing with an overflow
      or when the initial marking phase has simply marked the roots
      and didn't push them on the stack), then tasks claim heap
      regions whose bitmap they then scan to find gray objects. A
      global finger indicates where the end of the last claimed region
      is. A local finger indicates how far into the region a task has
      scanned. The two fingers are used to determine how to gray an
      object (i.e. whether simply marking it is OK, as it will be
      visited by a task in the future, or whether it needs to be also
      pushed on a stack).

      (2) Local Queue. The local queue of the task which is accessed
      reasonably efficiently by the task. Other tasks can steal from
      it when they run out of work. Throughout the marking phase, a
      task attempts to keep its local queue short but not totally
      empty, so that entries are available for stealing by other
      tasks. Only when there is no more work, a task will totally
      drain its local queue.

      (3) Global Mark Stack. This handles local queue overflow. During
      marking only sets of entries are moved between it and the local
      queues, as access to it requires a mutex and more fine-grain
      interaction with it which might cause contention. If it
      overflows, then the marking phase should restart and iterate
      over the bitmap to identify gray objects. Throughout the marking
      phase, tasks attempt to keep the global mark stack at a small
      length but not totally empty, so that entries are available for
      popping by other tasks. Only when there is no more work, tasks
      will totally drain the global mark stack.

      (4) Global Region Stack. Entries on it correspond to areas of
      the bitmap that need to be scanned since they contain gray
      objects. Pushes on the region stack only happen during
      evacuation pauses and typically correspond to areas covered by
      GC LABS. If it overflows, then the marking phase should restart
      and iterate over the bitmap to identify gray objects. Tasks will
      try to totally drain the region stack as soon as possible.

      (5) SATB Buffer Queue. This is where completed SATB buffers are
      made available. Buffers are regularly removed from this queue
      and scanned for roots, so that the queue doesn't get too
      long. During remark, all completed buffers are processed, as
      well as the filled in parts of any uncompleted buffers.

    The do_marking_step() method tries to abort when the time target
    has been reached. There are a few other cases when the
    do_marking_step() method also aborts:

      (1) When the marking phase has been aborted (after a Full GC).

      (2) When a global overflow (either on the global stack or the
      region stack) has been triggered. Before the task aborts, it
      will actually sync up with the other tasks to ensure that all
      the marking data structures (local queues, stacks, fingers etc.)
      are re-initialised so that when do_marking_step() completes,
      the marking phase can immediately restart.

      (3) When enough completed SATB buffers are available. The
      do_marking_step() method only tries to drain SATB buffers right
      at the beginning. So, if enough buffers are available, the
      marking step aborts and the SATB buffers are processed at
      the beginning of the next invocation.

      (4) To yield. when we have to yield then we abort and yield
      right at the end of do_marking_step(). This saves us from a lot
      of hassle as, by yielding we might allow a Full GC. If this
      happens then objects will be compacted underneath our feet, the
      heap might shrink, etc. We save checking for this by just
      aborting and doing the yield right at the end.

    From the above it follows that the do_marking_step() method should
    be called in a loop (or, otherwise, regularly) until it completes.

    If a marking step completes without its has_aborted() flag being
    true, it means it has completed the current marking phase (and
    also all other marking tasks have done so and have all synced up).

    A method called regular_clock_call() is invoked "regularly" (in
    sub ms intervals) throughout marking. It is this clock method that
    checks all the abort conditions which were mentioned above and
    decides when the task should abort. A work-based scheme is used to
    trigger this clock method: when the number of object words the
    marking phase has scanned or the number of references the marking
    phase has visited reach a given limit. Additional invocations to
    the method clock have been planted in a few other strategic places
    too. The initial reason for the clock method was to avoid calling
    vtime too regularly, as it is quite expensive. So, once it was in
    place, it was natural to piggy-back all the other conditions on it
    too and not constantly check them throughout the code.

 *****************************************************************************/

```

### 名前(function name)
```
void CMTask::do_marking_step(double time_target_ms,
                             bool do_stealing,
                             bool do_termination) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(time_target_ms >= 1.0, "minimum granularity is 1ms");
	  assert(concurrent() == _cm->concurrent(), "they should be the same");
	
	  assert(concurrent() || _cm->region_stack_empty(),
	         "the region stack should have been cleared before remark");
	  assert(concurrent() || !_cm->has_aborted_regions(),
	         "aborted regions should have been cleared before remark");
	  assert(_region_finger == NULL,
	         "this should be non-null only when a region is being scanned");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectorPolicy* g1_policy = _g1h->g1_policy();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_task_queues != NULL, "invariant");
	  assert(_task_queue != NULL, "invariant");
	  assert(_task_queues->queue(_task_id) == _task_queue, "invariant");
	
	  assert(!_claimed,
	         "only one thread should claim this task at any one time");
	
  {- -------------------------------------------
  (1) (ここからが _claimed が true の区間)
      ---------------------------------------- -}

	  // OK, this doesn't safeguard again all possible scenarios, as it is
	  // possible for two threads to set the _claimed flag at the same
	  // time. But it is only for debugging purposes anyway and it will
	  // catch most problems.
	  _claimed = true;
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  _start_time_ms = os::elapsedVTime() * 1000.0;
	  statsOnly( _interval_start_time_ms = _start_time_ms );
	
	  double diff_prediction_ms =
	    g1_policy->get_new_prediction(&_marking_step_diffs_ms);
	  _time_target_ms = time_target_ms - diff_prediction_ms;
	
	  // set up the variables that are used in the work-based scheme to
	  // call the regular clock method
	  _words_scanned = 0;
	  _refs_reached  = 0;
	  recalculate_limits();
	
	  // clear all flags
	  clear_has_aborted();
	  _has_timed_out = false;
	  _draining_satb_buffers = false;
	
	  ++_calls;
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (_cm->verbose_low())
	    gclog_or_tty->print_cr("[%d] >>>>>>>>>> START, call = %d, "
	                           "target = %1.2lfms >>>>>>>>>>",
	                           _task_id, _calls, _time_target_ms);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // Set up the bitmap and oop closures. Anything that uses them is
	  // eventually called from this method, so it is OK to allocate these
	  // statically.
	  CMBitMapClosure bitmap_closure(this, _cm, _nextMarkBitMap);
	  CMOopClosure    oop_closure(_g1h, _cm, this);

  {- -------------------------------------------
  (1) CMOopClosure をセットしておく.
      (See: CMTask::scan_object())
      ---------------------------------------- -}

	  set_oop_closure(&oop_closure);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (_cm->has_overflown()) {
	    // This can happen if the region stack or the mark stack overflows
	    // during a GC pause and this task, after a yield point,
	    // restarts. We have to abort as we need to get into the overflow
	    // protocol which happens right at the end of this task.
	    set_has_aborted();
	  }
	
  {- -------------------------------------------
  (1) まずは JavaThread::satb_mark_queue_set() に溜まっている completed buffer を処理する.
      (SATB buffers を見るのはここだけで, この後は次に呼び出された時まで見ない,
      と書いてあるが正しいか?? 後ろの方でも見ているような...)
      その後, local queue と global stack を処理する.
      ---------------------------------------- -}

	  // First drain any available SATB buffers. After this, we will not
	  // look at SATB buffers before the next invocation of this method.
	  // If enough completed SATB buffers are queued up, the regular clock
	  // will abort this task so that it restarts.
	  drain_satb_buffers();
	  // ...then partially drain the local queue and the global stack
	  drain_local_queue(true);
	  drain_global_stack(true);
	
  {- -------------------------------------------
  (1) 次に, CMTask::drain_region_stack() を呼んで...
      その後, local queue と global stack を処理する.
      ---------------------------------------- -}

	  // Then totally drain the region stack.  We will not look at
	  // it again before the next invocation of this method. Entries on
	  // the region stack are only added during evacuation pauses, for
	  // which we have to yield. When we do, we abort the task anyway so
	  // it will look at the region stack again when it restarts.
	  bitmap_closure.set_scanning_heap_region(false);
	  drain_region_stack(&bitmap_closure);
	  // ...then partially drain the local queue and the global stack
	  drain_local_queue(true);
	  drain_global_stack(true);
	
  {- -------------------------------------------
  (1) (以下の do...while ループで,
      グレー状態のオブジェクトを探し, そこから辿れる範囲を再帰的にマークする処理を行う.
  
      この処理は, claim_region で処理対象を取得し BitMap::iterate() で処理する, という処理の繰り返し.
      ただし, 前回の処理で取得した後に abort した可能性があるので,
      ループ内での処理の順番は, もし処理対象があれば BitMap::iterate() で処理してから claim_region で処理対象を取得, という順になっている.)
      ---------------------------------------- -}

	  do {

    {- -------------------------------------------
  (1.1) 前回取得したリージョンが残っており(= _curr_region が NULL ではない)
        かつまだ abort もされていない場合は,
        以下の if ブロック内で処理を行う.
  
        (_curr_region はこの関数内で呼び出される setup_for_region() でのみセットされる模様)
        ---------------------------------------- -}

	    if (!has_aborted() && _curr_region != NULL) {

      {- -------------------------------------------
  (1.1.1) (assert)
          ---------------------------------------- -}

	      // This means that we're already holding on to a region.
	      assert(_finger != NULL, "if region is not NULL, then the finger "
	             "should not be NULL either");
	
      {- -------------------------------------------
  (1.1.1) CMTask::update_region_limit() を呼んで, _finger と _region_limit の値を更新する.
  
          (もしかしたら, 今回の marking 処理は, マーキングを途中まで行っていたのに一旦中断されて再開された状態かもしれない.
           その場合, 中断されている間に evacuation pause が発生し, 
           調査していた HeapRegion がゴミだけになっているかもしれないので, 
           CMTask::update_region_limit() でチェックしておく必要がある)
          ---------------------------------------- -}

	      // We might have restarted this task after an evacuation pause
	      // which might have evacuated the region we're holding on to
	      // underneath our feet. Let's read its limit again to make sure
	      // that we do not iterate over a region of the heap that
	      // contains garbage (update_region_limit() will also move
	      // _finger to the start of the region if it is found empty).
	      update_region_limit();

      {- -------------------------------------------
  (1.1.1) (変数宣言など)
  
          (もしかしたら, 今回の marking 処理は, マーキングを途中まで行っていたのに一旦中断されて再開された状態かもしれない.
           その場合, 前回の作業時に _finger までは見ているので, 
           HeapRegion の先頭からではなく _finger から始めている.
           なお, 途中からの再開でない場合は _finger は HeapRegion の先頭を指している.)
          ---------------------------------------- -}

	      // We will start from _finger not from the start of the region,
	      // as we might be restarting this task after aborting half-way
	      // through scanning this region. In this case, _finger points to
	      // the address where we last found a marked object. If this is a
	      // fresh region, _finger points to start().
	      MemRegion mr = MemRegion(_finger, _region_limit);
	
      {- -------------------------------------------
  (1.1.1) (トレース出力)
          ---------------------------------------- -}

	      if (_cm->verbose_low())
	        gclog_or_tty->print_cr("[%d] we're scanning part "
	                               "["PTR_FORMAT", "PTR_FORMAT") "
	                               "of region "PTR_FORMAT,
	                               _task_id, _finger, _region_limit, _curr_region);
	
      {- -------------------------------------------
  (1.1.1) 対象の mr が空でなければ, BitMap::iterate() で対象の mr に対して CMBitMapClosure を適用して処理を行う.
          結果に応じて以下のどれかを行う.
  
          * 対象の mr が空だった場合:
            giveup_current_region() をよんで, この CMTask の状態をリセットする
            (これにより, この CMTask は処理中の HeapRegion を持たない状態になる)
            (ついでに, regular_clock_call() も呼んで...しておく)
  
          * BitMap::iterate() による処理が成功した場合:
            giveup_current_region() をよんで, この CMTask の状態をリセットする
            (これにより, この CMTask は処理中の HeapRegion を持たない状態になる)
            (ついでに, regular_clock_call() も呼んで...しておく)
  
          * BitMap::iterate() による処理が失敗した場合:
            nextWord() で次の処理対象を取得し, move_finger_to で finger にセットしておく.
            (ただし, 次の処理対象が _region_limit を越えていたら,
             move_finger_to() はせず,
             代わりに giveup_current_region() をよんで, この CMTask の状態をリセットする
             (これにより, この CMTask は処理中の HeapRegion を持たない状態になる))
          ---------------------------------------- -}

	      // Let's iterate over the bitmap of the part of the
	      // region that is left.
	      bitmap_closure.set_scanning_heap_region(true);
	      if (mr.is_empty() ||
	          _nextMarkBitMap->iterate(&bitmap_closure, mr)) {
	        // We successfully completed iterating over the region. Now,
	        // let's give up the region.
	        giveup_current_region();
	        regular_clock_call();
	      } else {
	        assert(has_aborted(), "currently the only way to do so");
	        // The only way to abort the bitmap iteration is to return
	        // false from the do_bit() method. However, inside the
	        // do_bit() method we move the _finger to point to the
	        // object currently being looked at. So, if we bail out, we
	        // have definitely set _finger to something non-null.
	        assert(_finger != NULL, "invariant");
	
	        // Region iteration was actually aborted. So now _finger
	        // points to the address of the object we last scanned. If we
	        // leave it there, when we restart this task, we will rescan
	        // the object. It is easy to avoid this. We move the finger by
	        // enough to point to the next possible object header (the
	        // bitmap knows by how much we need to move it as it knows its
	        // granularity).
	        assert(_finger < _region_limit, "invariant");
	        HeapWord* new_finger = _nextMarkBitMap->nextWord(_finger);
	        // Check if bitmap iteration was aborted while scanning the last object
	        if (new_finger >= _region_limit) {
	            giveup_current_region();
	        } else {
	            move_finger_to(new_finger);
	        }
	      }
	    }

    {- -------------------------------------------
  (1.1) local queue と global stack を処理しておく.
        ---------------------------------------- -}

	    // At this point we have either completed iterating over the
	    // region we were holding on to, or we have aborted.
	
	    // We then partially drain the local queue and the global stack.
	    // (Do we really need this?)
	    drain_local_queue(true);
	    drain_global_stack(true);
	
    {- -------------------------------------------
  (1.1) 以下の while ブロック内で, 次の処理対象となる HeapRegion を取得する.
  
        なお失敗する可能性があるのでループになっている模様
        (これについては while の直前にいろいろ説明が書いてあるようだが...)
        ---------------------------------------- -}

	    // Read the note on the claim_region() method on why it might
	    // return NULL with potentially more regions available for
	    // claiming and why we have to check out_of_regions() to determine
	    // whether we're done or not.
	    while (!has_aborted() && _curr_region == NULL && !_cm->out_of_regions()) {

      {- -------------------------------------------
  (1.1.1) (assert)
          ---------------------------------------- -}

	      // We are going to try to claim a new region. We should have
	      // given up on the previous one.
	      // Separated the asserts so that we know which one fires.
	      assert(_curr_region  == NULL, "invariant");
	      assert(_finger       == NULL, "invariant");
	      assert(_region_limit == NULL, "invariant");

      {- -------------------------------------------
  (1.1.1) (トレース出力)
          ---------------------------------------- -}

	      if (_cm->verbose_low())
	        gclog_or_tty->print_cr("[%d] trying to claim a new region", _task_id);

      {- -------------------------------------------
  (1.1.1) ConcurrentMark::claim_region() で
          次の処理対象の HeapRegion を取得し, setup_for_region() で _curr_region にセットする.
          
          (_finger フィールドのアドレスを含んでいる HeapRegion が取得される模様)
          ---------------------------------------- -}

	      HeapRegion* claimed_region = _cm->claim_region(_task_id);
	      if (claimed_region != NULL) {
	        // Yes, we managed to claim one
	        statsOnly( ++_regions_claimed );
	
	        if (_cm->verbose_low())
	          gclog_or_tty->print_cr("[%d] we successfully claimed "
	                                 "region "PTR_FORMAT,
	                                 _task_id, claimed_region);
	
	        setup_for_region(claimed_region);
	        assert(_curr_region == claimed_region, "invariant");
	      }

      {- -------------------------------------------
  (1.1.1) regular_clock_call() で abort 条件をチェックしておく.
          ---------------------------------------- -}

	      // It is important to call the regular clock here. It might take
	      // a while to claim a region if, for example, we hit a large
	      // block of empty regions. So we need to call the regular clock
	      // method once round the loop to make sure it's called
	      // frequently enough.
	      regular_clock_call();
	    }
	
    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	    if (!has_aborted() && _curr_region == NULL) {
	      assert(_cm->out_of_regions(),
	             "at this point we should be out of regions");
	    }

  {- -------------------------------------------
  (1) (ここまでが, グレー状態のオブジェクトから辿れる範囲の再帰的なマーク処理)
      ---------------------------------------- -}

	  } while ( _curr_region != NULL && !has_aborted());
	
  {- -------------------------------------------
  (1) まだ abort してなければ drain_satb_buffers() しておく.
      (ついでに(トレース出力)も出している)
      ---------------------------------------- -}

	  if (!has_aborted()) {
	    // We cannot check whether the global stack is empty, since other
	    // tasks might be pushing objects to it concurrently. We also cannot
	    // check if the region stack is empty because if a thread is aborting
	    // it can push a partially done region back.
	    assert(_cm->out_of_regions(),
	           "at this point we should be out of regions");
	
	    if (_cm->verbose_low())
	      gclog_or_tty->print_cr("[%d] all regions claimed", _task_id);
	
	    // Try to reduce the number of available SATB buffers so that
	    // remark has less work to do.
	    drain_satb_buffers();
	  }
	
  {- -------------------------------------------
  (1) local queue と global stack を処理しておく.
      ---------------------------------------- -}

	  // Since we've done everything else, we can now totally drain the
	  // local queue and global stack.
	  drain_local_queue(false);
	  drain_global_stack(false);
	
  {- -------------------------------------------
  (1) work stealing するように指定されており (= do_stealing 引数が true),
      かつ !has_aborted() なら,
      以下の if ブロック内で他の task の仕事を処理する.
      ---------------------------------------- -}

	  // Attempt at work stealing from other task's queues.
	  if (do_stealing && !has_aborted()) {

    {- -------------------------------------------
  (1.1) (abort してないと言うことは, 自分の仕事は全てやり終えたと言うこと)
        ---------------------------------------- -}

	    // We have not aborted. This means that we have finished all that
	    // we could. Let's try to do some stealing...
	
    {- -------------------------------------------
  (1.1) (assert)
        (なおコメントによると,
         global stack が空かどうかはチェック出来ない (他のタスクが並行して push するかもしれないので).
         また region stack が空かどうかもチェックできない (誰かが abort したらそいつが残りの仕事を push するので)
         とのこと)
        ---------------------------------------- -}

	    // We cannot check whether the global stack is empty, since other
	    // tasks might be pushing objects to it concurrently. We also cannot
	    // check if the region stack is empty because if a thread is aborting
	    // it can push a partially done region back.
	    assert(_cm->out_of_regions() && _task_queue->size() == 0,
	           "only way to reach here");
	
    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    if (_cm->verbose_low())
	      gclog_or_tty->print_cr("[%d] starting to steal", _task_id);
	
    {- -------------------------------------------
  (1.1) abort するか try_stealing() が失敗するまで, 以下の while ブロックで work stealing を繰り返す.
        ---------------------------------------- -}

	    while (!has_aborted()) {

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	      oop obj;
	      statsOnly( ++_steal_attempts );
	
	      if (_cm->try_stealing(_task_id, &_hash_seed, obj)) {

      {- -------------------------------------------
  (1.1.1) (トレース出力)
          ---------------------------------------- -}

	        if (_cm->verbose_medium())
	          gclog_or_tty->print_cr("[%d] stolen "PTR_FORMAT" successfully",
	                                 _task_id, (void*) obj);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	        statsOnly( ++_steals );
	
      {- -------------------------------------------
  (1.1.1) (assert)
          ---------------------------------------- -}

	        assert(_nextMarkBitMap->isMarked((HeapWord*) obj),
	               "any stolen object should be marked");

      {- -------------------------------------------
  (1.1.1) scan_object() で処理し, local queue と global stack も処理.
          ---------------------------------------- -}

	        scan_object(obj);
	
	        // And since we're towards the end, let's totally drain the
	        // local queue and global stack.
	        drain_local_queue(false);
	        drain_global_stack(false);
	      } else {

      {- -------------------------------------------
  (1.1.1) try_stealing() が失敗したら脱出
          ---------------------------------------- -}

	        break;
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: ForceOverflowSettings)
      ---------------------------------------- -}

	  // If we are about to wrap up and go into termination, check if we
	  // should raise the overflow flag.
	  if (do_termination && !has_aborted()) {
	    if (_cm->force_overflow()->should_force()) {
	      _cm->set_has_overflown();
	      regular_clock_call();
	    }
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // We still haven't aborted. Now, let's try to get into the
	  // termination protocol.
	  if (do_termination && !has_aborted()) {

    {- -------------------------------------------
  (1.1) (assert)
        (なおコメントによると,
         global stack が空かどうかはチェック出来ない (他のタスクが並行して push するかもしれないので).
         また region stack が空かどうかもチェックできない (誰かが abort したらそいつが残りの仕事を push するので)
         とのこと)
        ---------------------------------------- -}

	    // We cannot check whether the global stack is empty, since other
	    // tasks might be concurrently pushing objects on it. We also cannot
	    // check if the region stack is empty because if a thread is aborting
	    // it can push a partially done region back.
	    // Separated the asserts so that we know which one fires.
	    assert(_cm->out_of_regions(), "only way to reach here");
	    assert(_task_queue->size() == 0, "only way to reach here");
	
    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    if (_cm->verbose_low())
	      gclog_or_tty->print_cr("[%d] starting termination protocol", _task_id);
	
    {- -------------------------------------------
  (1.1) offer_termination() を呼んで終了判定を行う.
        (ついでに, その前後で時間をはかり, 掛かった時間を _termination_time_ms に足し込んでもいる)
        * 終了した場合
          自分が _task_id == 0 であり, かつ concurrent() が true であれば,
          clear_concurrent_marking_in_progress() を呼んで...
          (そうでなければ, 終了した場合には何もしない. トレース出力くらいは出すが...)
        * 終了していない場合
          set_has_aborted() を呼んで, 残りの処理が後で呼び出されるようにしておく.
          (ついでにトレース...)
          (ついでに, statsOnly で _aborted_termination をインクリメント)
        ---------------------------------------- -}

	    _termination_start_time_ms = os::elapsedVTime() * 1000.0;
	    // The CMTask class also extends the TerminatorTerminator class,
	    // hence its should_exit_termination() method will also decide
	    // whether to exit the termination protocol or not.
	    bool finished = _cm->terminator()->offer_termination(this);
	    double termination_end_time_ms = os::elapsedVTime() * 1000.0;
	    _termination_time_ms +=
	      termination_end_time_ms - _termination_start_time_ms;
	
	    if (finished) {
	      // We're all done.
	
	      if (_task_id == 0) {
	        // let's allow task 0 to do this
	        if (concurrent()) {
	          assert(_cm->concurrent_marking_in_progress(), "invariant");
	          // we need to set this to false before the next
	          // safepoint. This way we ensure that the marking phase
	          // doesn't observe any more heap expansions.
	          _cm->clear_concurrent_marking_in_progress();
	        }
	      }
	
	      // We can now guarantee that the global stack is empty, since
	      // all other tasks have finished. We separated the guarantees so
	      // that, if a condition is false, we can immediately find out
	      // which one.
	      guarantee(_cm->out_of_regions(), "only way to reach here");
	      guarantee(_aborted_region.is_empty(), "only way to reach here");
	      guarantee(_cm->region_stack_empty(), "only way to reach here");
	      guarantee(_cm->mark_stack_empty(), "only way to reach here");
	      guarantee(_task_queue->size() == 0, "only way to reach here");
	      guarantee(!_cm->has_overflown(), "only way to reach here");
	      guarantee(!_cm->mark_stack_overflow(), "only way to reach here");
	      guarantee(!_cm->region_stack_overflow(), "only way to reach here");
	
	      if (_cm->verbose_low())
	        gclog_or_tty->print_cr("[%d] all tasks terminated", _task_id);
	    } else {
	      // Apparently there's more work to do. Let's abort this task. It
	      // will restart it and we can hopefully find more things to do.
	
	      if (_cm->verbose_low())
	        gclog_or_tty->print_cr("[%d] apparently there is more work to do", _task_id);
	
	      set_has_aborted();
	      statsOnly( ++_aborted_termination );
	    }
	  }
	
  {- -------------------------------------------
  (1) set_oop_closure() を呼んで closure を NULL にリセットしておく.
      (これは念のための処理.
      スタック上の局所変数のアドレスをセットしているのでリターン前に消しておいた方が無難, という判断)
      ---------------------------------------- -}

	  // Mainly for debugging purposes to make sure that a pointer to the
	  // closure which was statically allocated in this frame doesn't
	  // escape it by accident.
	  set_oop_closure(NULL);

  {- -------------------------------------------
  (1) 処理に掛かった時間を _step_times_ms に add()
      ---------------------------------------- -}

	  double end_time_ms = os::elapsedVTime() * 1000.0;
	  double elapsed_time_ms = end_time_ms - _start_time_ms;
	  // Update the step history.
	  _step_times_ms.add(elapsed_time_ms);
	
  {- -------------------------------------------
  (1) 何らかの理由で abort していた場合は, 原因を調べて後始末をしておく.
      ---------------------------------------- -}

	  if (has_aborted()) {
	    // The task was aborted for some reason.
	
	    statsOnly( ++_aborted );
	
    {- -------------------------------------------
  (1.1) タイムアウトによる場合:
        _marking_step_diffs_ms に処理時間を足し込むだけ.
        ---------------------------------------- -}

	    if (_has_timed_out) {
	      double diff_ms = elapsed_time_ms - _time_target_ms;
	      // Keep statistics of how well we did with respect to hitting
	      // our target only if we actually timed out (if we aborted for
	      // other reasons, then the results might get skewed).
	      _marking_step_diffs_ms.add(diff_ms);
	    }
	
    {- -------------------------------------------
  (1.1) オーバーフローによる場合:
        ...#TODO
        ---------------------------------------- -}

	    if (_cm->has_overflown()) {
	      // This is the interesting one. We aborted because a global
	      // overflow was raised. This means we have to restart the
	      // marking phase and start iterating over regions. However, in
	      // order to do this we have to make sure that all tasks stop
	      // what they are doing and re-initialise in a safe manner. We
	      // will achieve this with the use of two barrier sync points.
	
	      if (_cm->verbose_low())
	        gclog_or_tty->print_cr("[%d] detected overflow", _task_id);
	
	      _cm->enter_first_sync_barrier(_task_id);
	      // When we exit this sync barrier we know that all tasks have
	      // stopped doing marking work. So, it's now safe to
	      // re-initialise our data structures. At the end of this method,
	      // task 0 will clear the global data structures.
	
	      statsOnly( ++_aborted_overflow );
	
	      // We clear the local state of this task...
	      clear_region_fields();
	
	      // ...and enter the second barrier.
	      _cm->enter_second_sync_barrier(_task_id);
	      // At this point everything has bee re-initialised and we're
	      // ready to restart.
	    }
	
    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    if (_cm->verbose_low()) {
	      gclog_or_tty->print_cr("[%d] <<<<<<<<<< ABORTING, target = %1.2lfms, "
	                             "elapsed = %1.2lfms <<<<<<<<<<",
	                             _task_id, _time_target_ms, elapsed_time_ms);
	      if (_cm->has_aborted())
	        gclog_or_tty->print_cr("[%d] ========== MARKING ABORTED ==========",
	                               _task_id);
	    }

  {- -------------------------------------------
  (1) 以下は abort していない場合.
     トレース出力を出すだけ.
      ---------------------------------------- -}

	  } else {
	    if (_cm->verbose_low())
	      gclog_or_tty->print_cr("[%d] <<<<<<<<<< FINISHED, target = %1.2lfms, "
	                             "elapsed = %1.2lfms <<<<<<<<<<",
	                             _task_id, _time_target_ms, elapsed_time_ms);
	  }
	
  {- -------------------------------------------
  (1) (ここまでが _claimed が true の区間)
      ---------------------------------------- -}

	  _claimed = false;
	}
	
```


