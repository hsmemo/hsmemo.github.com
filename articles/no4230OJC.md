---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/synchronizer.cpp
### 説明(description)
(stack-locked であれば ObjectMonitor を新たに確保して inflated にさせる.
既に inflated であれば, 関連付けられている ObjectMonitor を返すだけ)


### 名前(function name)
```
ObjectMonitor * ATTR ObjectSynchronizer::inflate (Thread * Self, oop object) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Inflate mutates the heap ...
	  // Relaxing assertion for bug 6320749.
	  assert (Universe::verify_in_progress() ||
	          !SafepointSynchronize::is_at_safepoint(), "invariant") ;
	
  {- -------------------------------------------
  (1) (この関数の処理は, 以下の大きな for ループで構成されている.
       この中では, ロック状態に応じて以下のように処理を行う.
       * Inflated
         既に ObjectMonitor が対応付いているので, その ObjectMonitor を返すだけ.
       * INFLATING (他のスレッドが inflate 作業を行っている途中の状態)
         そのスレッドによる inflate 作業が終わるのを待ち, その後ループの先頭に戻って処理をやり直す.
       * Stack-locked
         新しい ObjectMonitor を確保し, CAS によってロック対象のオブジェクトに対応付ける. 
         CAS が成功したら, その ObjectMonitor を返す.
         失敗したら, ループの先頭に戻って処理をやり直す.
       * Neutral
         Stack-locked の場合とほぼ同様.
       * BIASED
         biased 状態でこの関数が呼ばれることはない.)
      ---------------------------------------- -}

	  for (;;) {

  {- -------------------------------------------
  (1) mark フィールドを取得する (以下で, mark フィールドに格納してあるロック状態に応じて処理を場合分け)
      ---------------------------------------- -}

	      const markOop mark = object->mark() ;

  {- -------------------------------------------
  (1) (assert)
      (biased 状態でこの関数が呼ばれることはない)
      ---------------------------------------- -}

	      assert (!mark->has_bias_pattern(), "invariant") ;
	
	      // The mark can be in one of the following states:
	      // *  Inflated     - just return
	      // *  Stack-locked - coerce it to inflated
	      // *  INFLATING    - busy wait for conversion to complete
	      // *  Neutral      - aggressively inflate the object.
	      // *  BIASED       - Illegal.  We should never see this
	
  {- -------------------------------------------
  (1) (以下は, inflated 状態の場合の処理)
      ---------------------------------------- -}

	      // CASE: inflated
	      if (mark->has_monitor()) {

    {- -------------------------------------------
  (1.1) 確保済みの ObjectMonitor を取得して返すだけ.
        ---------------------------------------- -}

	          ObjectMonitor * inf = mark->monitor() ;
	          assert (inf->header()->is_neutral(), "invariant");
	          assert (inf->object() == object, "invariant") ;
	          assert (ObjectSynchronizer::verify_objmon_isinpool(inf), "monitor is invalid");
	          return inf ;
	      }
	
  {- -------------------------------------------
  (1) (以下は, INFLATING 状態の場合の処理)
  
      (これは, 誰か他のスレッドが stack-locked から inflated へと状態遷移を実行している途中の場合
       (詳細は以下の "stack-locked 状態の場合の処理" 以降を参照).
       この場合には, 他のスレッドは inflate 作業が終了するまで待たなければならない.
       現状では, spin/yield/park and poll で inflate 作業の完了を待っている.
       polling については parking することにしてもいいかもしれない.)
      ---------------------------------------- -}

	      // CASE: inflation in progress - inflating over a stack-lock.
	      // Some other thread is converting from stack-locked to inflated.
	      // Only that thread can complete inflation -- other threads must wait.
	      // The INFLATING value is transient.
	      // Currently, we spin/yield/park and poll the markword, waiting for inflation to finish.
	      // We could always eliminate polling by parking the thread on some auxiliary list.
	      if (mark == markOopDesc::INFLATING()) {

    {- -------------------------------------------
  (1.1) ReadStableMark() で inflate 作業が完了するまで待つ. 
        終わったら, このループの先頭からもう一度やり直す.
        (ついでにTEVENTで(トレース出力)も出している)
        ---------------------------------------- -}

	         TEVENT (Inflate: spin while INFLATING) ;
	         ReadStableMark(object) ;
	         continue ;
	      }
	
  {- -------------------------------------------
  (1) (以下は, stack-locked 状態の場合の処理)
      
      (なお, コメントによると, 
         以前は, INFLATING 状態にしてから ObjectMonitor の確保を行っていたが, 
         critical section が伸びることで競合する可能性が高まるため, 
         現在は INFLATING 状態にする前に (投機的に) ObjectMonitor の確保を行っている.
  
         とはいえ, ObjectMonitor の確保処理については
         スレッド毎にフリーリストを用意して高速化するようになったため, 
         大域の ObjectMonitor リストからスレッド毎のリストに取ってくる処理さえ
         critical section 外にあれば, ObjectSynchronizer::omAlloc() 自体は 
         INFLATING 状態にしてから呼び出しても問題にならないだろう.
         ObjectSynchronizer::omAlloc() のコメントも参照のこと.
       とのこと.)
      ---------------------------------------- -}

	      // CASE: stack-locked
	      // Could be stack-locked either by this thread or by some other thread.
	      //
	      // Note that we allocate the objectmonitor speculatively, _before_ attempting
	      // to install INFLATING into the mark word.  We originally installed INFLATING,
	      // allocated the objectmonitor, and then finally STed the address of the
	      // objectmonitor into the mark.  This was correct, but artificially lengthened
	      // the interval in which INFLATED appeared in the mark, thus increasing
	      // the odds of inflation contention.
	      //
	      // We now use per-thread private objectmonitor free lists.
	      // These list are reprovisioned from the global free list outside the
	      // critical INFLATING...ST interval.  A thread can transfer
	      // multiple objectmonitors en-mass from the global free list to its local free list.
	      // This reduces coherency traffic and lock contention on the global free list.
	      // Using such local free lists, it doesn't matter if the omAlloc() call appears
	      // before or after the CAS(INFLATING) operation.
	      // See the comments in omAlloc().
	
	      if (mark->has_locker()) {

    {- -------------------------------------------
  (1.1) ObjectSynchronizer::omAlloc() で, 新しい ObjectMonitor オブジェクトを確保する.
        ついでに, オブジェクト内のフィールドの初期化も行う.
        ---------------------------------------- -}

	          ObjectMonitor * m = omAlloc (Self) ;
	          // Optimistically prepare the objectmonitor - anticipate successful CAS
	          // We do this before the CAS in order to minimize the length of time
	          // in which INFLATING appears in the mark.
	          m->Recycle();
	          m->_Responsible  = NULL ;
	          m->OwnerIsThread = 0 ;
	          m->_recursions   = 0 ;
	          m->_SpinDuration = ObjectMonitor::Knob_SpinLimit ;   // Consider: maintain by type/class
	
    {- -------------------------------------------
  (1.1) ロック対象のオブジェクトの mark フィールドを, CAS で INFLATING に書き換える.
        CAS が失敗したら, 確保した ObjectMonitor を開放し, ループの先頭に戻ってやり直し.
  
        (なお, なぜいきなり ObjectMonitor にせず INFLATING を経由するかというと,
         この inflate 作業と同時に stack-locked からのアンロック処理が行われた場合のため.
         この場合, タイミングが悪いと mark フィールドの値が完全に同期できず
         オブジェクトのハッシュコード(hashCode)が変わってしまったり, 
         あるいは短期間にちらつく(flicker)恐れがある.)
        ---------------------------------------- -}

	          markOop cmp = (markOop) Atomic::cmpxchg_ptr (markOopDesc::INFLATING(), object->mark_addr(), mark) ;
	          if (cmp != mark) {
	             omRelease (Self, m, true) ;
	             continue ;       // Interference -- just retry
	          }
	
	          // We've successfully installed INFLATING (0) into the mark-word.
	          // This is the only case where 0 will appear in a mark-work.
	          // Only the singular thread that successfully swings the mark-word
	          // to 0 can perform (or more precisely, complete) inflation.
	          //
	          // Why do we CAS a 0 into the mark-word instead of just CASing the
	          // mark-word from the stack-locked value directly to the new inflated state?
	          // Consider what happens when a thread unlocks a stack-locked object.
	          // It attempts to use CAS to swing the displaced header value from the
	          // on-stack basiclock back into the object header.  Recall also that the
	          // header value (hashcode, etc) can reside in (a) the object header, or
	          // (b) a displaced header associated with the stack-lock, or (c) a displaced
	          // header in an objectMonitor.  The inflate() routine must copy the header
	          // value from the basiclock on the owner's stack to the objectMonitor, all
	          // the while preserving the hashCode stability invariants.  If the owner
	          // decides to release the lock while the value is 0, the unlock will fail
	          // and control will eventually pass from slow_exit() to inflate.  The owner
	          // will then spin, waiting for the 0 value to disappear.   Put another way,
	          // the 0 causes the owner to stall if the owner happens to try to
	          // drop the lock (restoring the header from the basiclock to the object)
	          // while inflation is in-progress.  This protocol avoids races that might
	          // would otherwise permit hashCode values to change or "flicker" for an object.
	          // Critically, while object->mark is 0 mark->displaced_mark_helper() is stable.
	          // 0 serves as a "BUSY" inflate-in-progress indicator.
	
	
    {- -------------------------------------------
  (1.1) (以下は, INFLATING への CAS が成功した場合のパス)
        ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) ObjectMonitor 内に, INFLATING に書き換える前の mark フィールドの値(以下の mark)をセットする.
        ---------------------------------------- -}

	          // fetch the displaced mark from the owner's stack.
	          // The owner can't die or unwind past the lock while our INFLATING
	          // object is in the mark.  Furthermore the owner can't complete
	          // an unlock on the object, either.
	          markOop dmw = mark->displaced_mark_helper() ;
	          assert (dmw->is_neutral(), "invariant") ;
	
	          // Setup monitor fields to proper values -- prepare the monitor
	          m->set_header(dmw) ;
	
    {- -------------------------------------------
  (1.1) ObjectMonitor 内に, 現在ロックを取得しているスレッドの情報をセットする (ObjectMonitor::set_owner()).
        また, ロック対象のオブジェクトの情報もセットする (ObjectMonitor::set_object()).
    
        (なお, owner には本来はロックを取得しているスレッド自体へのポインタを格納するが, 
         stack-locked から inflated に変わる場合は, 
         対応する BasicObjectLock へのポインタを入れている.
         この状態は, ObjectMonitor::enter() や ObjectMonitor::exit() の処理中に是正される.)
        ---------------------------------------- -}

	          // Optimization: if the mark->locker stack address is associated
	          // with this thread we could simply set m->_owner = Self and
	          // m->OwnerIsThread = 1. Note that a thread can inflate an object
	          // that it has stack-locked -- as might happen in wait() -- directly
	          // with CAS.  That is, we can avoid the xchg-NULL .... ST idiom.
	          m->set_owner(mark->locker());
	          m->set_object(object);
	          // TODO-FIXME: assert BasicLock->dhw != 0.
	
    {- -------------------------------------------
  (1.1) oopDesc::release_set_mark() を呼んで, 
        ロック対象のオブジェクトの mark フィールドに ObjectMonitor をセットする.
        (oopDesc::release_set_mark() は, ついでにメモリバリアも張る. 
         これ以前にある ObjectMonitor の初期化用の store が mark フィールドの変更に抜かれるのは禁止)
        ---------------------------------------- -}

	          // Must preserve store ordering. The monitor state must
	          // be stable at the time of publishing the monitor address.
	          guarantee (object->mark() == markOopDesc::INFLATING(), "invariant") ;
	          object->release_set_mark(markOopDesc::encode(m));
	
    {- -------------------------------------------
  (1.1) (プロファイル情報の記録) (See: UsePerfData) (See: sun.rt._sync_Inflations)
        ---------------------------------------- -}

	          // Hopefully the performance counters are allocated on distinct cache lines
	          // to avoid false sharing on MP systems ...
	          if (ObjectMonitor::_sync_Inflations != NULL) ObjectMonitor::_sync_Inflations->inc() ;

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	          TEVENT(Inflate: overwrite stacklock) ;

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	          if (TraceMonitorInflation) {
	            if (object->is_instance()) {
	              ResourceMark rm;
	              tty->print_cr("Inflating object " INTPTR_FORMAT " , mark " INTPTR_FORMAT " , type %s",
	                (intptr_t) object, (intptr_t) object->mark(),
	                Klass::cast(object->klass())->external_name());
	            }
	          }

    {- -------------------------------------------
  (1.1) 確保した ObjectMonitor をリターン.
        ---------------------------------------- -}

	          return m ;
	      }
	
  {- -------------------------------------------
  (1) (以下は, neutral 状態の場合の処理)
      ---------------------------------------- -}

	      // CASE: neutral
	      // TODO-FIXME: for entry we currently inflate and then try to CAS _owner.
	      // If we know we're inflating for entry it's better to inflate by swinging a
	      // pre-locked objectMonitor pointer into the object header.   A successful
	      // CAS inflates the object *and* confers ownership to the inflating thread.
	      // In the current implementation we use a 2-step mechanism where we CAS()
	      // to inflate and then CAS() again to try to swing _owner from NULL to Self.
	      // An inflateTry() method that we could call from fast_enter() and slow_enter()
	      // would be useful.
	
	      assert (mark->is_neutral(), "invariant");

    {- -------------------------------------------
  (1.1) ObjectSynchronizer::omAlloc() で, 新しい ObjectMonitor オブジェクトを確保する.
        ついでに, オブジェクト内のフィールドの初期化も行う.
        ---------------------------------------- -}

	      ObjectMonitor * m = omAlloc (Self) ;
	      // prepare m for installation - set monitor to initial state
	      m->Recycle();
	      m->set_header(mark);
	      m->set_owner(NULL);
	      m->set_object(object);
	      m->OwnerIsThread = 1 ;
	      m->_recursions   = 0 ;
	      m->_Responsible  = NULL ;
	      m->_SpinDuration = ObjectMonitor::Knob_SpinLimit ;       // consider: keep metastats by type/class
	
    {- -------------------------------------------
  (1.1) ロック対象のオブジェクトの mark フィールドを, CAS で ObjedtMonitor に書き換える.
        CAS が失敗したら, 確保した ObjectMonitor を開放し, ループの先頭に戻ってやり直し.
        ---------------------------------------- -}

	      if (Atomic::cmpxchg_ptr (markOopDesc::encode(m), object->mark_addr(), mark) != mark) {
	          m->set_object (NULL) ;
	          m->set_owner  (NULL) ;
	          m->OwnerIsThread = 0 ;
	          m->Recycle() ;
	          omRelease (Self, m, true) ;
	          m = NULL ;
	          continue ;
	          // interference - the markword changed - just retry.
	          // The state-transitions are one-way, so there's no chance of
	          // live-lock -- "Inflated" is an absorbing state.
	      }
	
    {- -------------------------------------------
  (1.1) (以下は, CAS が成功した場合のパス)
        ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) (プロファイル情報の記録) (See: UsePerfData) (See: sun.rt._sync_Inflations)
        ---------------------------------------- -}

	      // Hopefully the performance counters are allocated on distinct
	      // cache lines to avoid false sharing on MP systems ...
	      if (ObjectMonitor::_sync_Inflations != NULL) ObjectMonitor::_sync_Inflations->inc() ;

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	      TEVENT(Inflate: overwrite neutral) ;

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	      if (TraceMonitorInflation) {
	        if (object->is_instance()) {
	          ResourceMark rm;
	          tty->print_cr("Inflating object " INTPTR_FORMAT " , mark " INTPTR_FORMAT " , type %s",
	            (intptr_t) object, (intptr_t) object->mark(),
	            Klass::cast(object->klass())->external_name());
	        }
	      }

    {- -------------------------------------------
  (1.1) 確保した ObjectMonitor をリターン.
        ---------------------------------------- -}

	      return m ;
	  }
	}
	
```


