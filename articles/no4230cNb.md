---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/objectMonitor.cpp
### 説明(description)

```
// -----------------------------------------------------------------------------
// Exit support
//
// exit()
// ~~~~~~
// Note that the collector can't reclaim the objectMonitor or deflate
// the object out from underneath the thread calling ::exit() as the
// thread calling ::exit() never transitions to a stable state.
// This inhibits GC, which in turn inhibits asynchronous (and
// inopportune) reclamation of "this".
//
// We'd like to assert that: (THREAD->thread_state() != _thread_blocked) ;
// There's one exception to the claim above, however.  EnterI() can call
// exit() to drop a lock if the acquirer has been externally suspended.
// In that case exit() is called with _thread_state as _thread_blocked,
// but the monitor's _count field is > 0, which inhibits reclamation.
//
// 1-0 exit
// ~~~~~~~~
// ::exit() uses a canonical 1-1 idiom with a MEMBAR although some of
// the fast-path operators have been optimized so the common ::exit()
// operation is 1-0.  See i486.ad fast_unlock(), for instance.
// The code emitted by fast_unlock() elides the usual MEMBAR.  This
// greatly improves latency -- MEMBAR and CAS having considerable local
// latency on modern processors -- but at the cost of "stranding".  Absent the
// MEMBAR, a thread in fast_unlock() can race a thread in the slow
// ::enter() path, resulting in the entering thread being stranding
// and a progress-liveness failure.   Stranding is extremely rare.
// We use timers (timed park operations) & periodic polling to detect
// and recover from stranding.  Potentially stranded threads periodically
// wake up and poll the lock.  See the usage of the _Responsible variable.
//
// The CAS() in enter provides for safety and exclusion, while the CAS or
// MEMBAR in exit provides for progress and avoids stranding.  1-0 locking
// eliminates the CAS/MEMBAR from the exist path, but it admits stranding.
// We detect and recover from stranding with timers.
//
// If a thread transiently strands it'll park until (a) another
// thread acquires the lock and then drops the lock, at which time the
// exiting thread will notice and unpark the stranded thread, or, (b)
// the timer expires.  If the lock is high traffic then the stranding latency
// will be low due to (a).  If the lock is low traffic then the odds of
// stranding are lower, although the worst-case stranding latency
// is longer.  Critically, we don't want to put excessive load in the
// platform's timer subsystem.  We want to minimize both the timer injection
// rate (timers created/sec) as well as the number of timers active at
// any one time.  (more precisely, we want to minimize timer-seconds, which is
// the integral of the # of active timers at any instant over time).
// Both impinge on OS scalability.  Given that, at most one thread parked on
// a monitor will use a timer.

```

### 名前(function name)
```
void ATTR ObjectMonitor::exit(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし ObjectMonitor 内の _owner フィールドがカレントスレッドを指していない場合, 
      _owner をカレントスレッドに変更しておく.
  
      (stack-locked 状態のロックが inflate された直後は, _owner に正しい値が入っていない.
       この状態になるのは, ロックを取った時には stack-locked だったものが, 
       開放までの間に何らかの理由で inflate された場合.
       つまりこの場合は, ロックは持っているが単に _owner に書かれていないというだけの状態.
       See: ObjectSynchronizer::inflate())
  
      (一応, ちゃんとロックを持っていること(= is_lock_owned() が true になること)を確認してはいる.
       が, コメントによると, 
       native method の場合にはロックとアンロックのバランスが取れていなければ
       IllegalMonitorStateException にするが, 
       この場合は (HotSpot の実装にバグがなければ) ロックを持っていないことはあり得ないのでどうするか, 
       とのこと.
       現状では, スレッドがロックを保持していなければ assert failure になっている (つまり製品版では何もしない).)
      ---------------------------------------- -}

	   Thread * Self = THREAD ;
	   if (THREAD != _owner) {
	     if (THREAD->is_lock_owned((address) _owner)) {
	       // Transmute _owner from a BasicLock pointer to a Thread address.
	       // We don't need to hold _mutex for this transition.
	       // Non-null to Non-null is safe as long as all readers can
	       // tolerate either flavor.
	       assert (_recursions == 0, "invariant") ;
	       _owner = THREAD ;
	       _recursions = 0 ;
	       OwnerIsThread = 1 ;

    {- -------------------------------------------
  (1.1) (スレッドがロックを保持していなければ assert failure)
        ---------------------------------------- -}

	     } else {
	       // NOTE: we need to handle unbalanced monitor enter/exit
	       // in native code by throwing an exception.
	       // TODO: Throw an IllegalMonitorStateException ?
	       TEVENT (Exit - Throw IMSX) ;
	       assert(false, "Non-balanced monitor enter/exit!");
	       if (false) {
	          THROW(vmSymbols::java_lang_IllegalMonitorStateException());
	       }
	       return;
	     }
	   }
	
  {- -------------------------------------------
  (1) もし _recursions が 1 以上であれば, _recursions をデクリメントするだけでいい. 
      (= 自分が複数回ロックしているオブジェクトであれば, そのロック回数を減らすだけでいい)
      この場合, ここでリターン.
      ---------------------------------------- -}

	   if (_recursions != 0) {
	     _recursions--;        // this is simple recursive enter
	     TEVENT (Inflated exit - recursive) ;
	     return ;
	   }
	
  {- -------------------------------------------
  (1) この ObjectMonitor オブジェクトの _Responsible を NULL に戻しておく.
      (See: 1-0 model)
  
      (なお, この最適化(?)は SyncFlags の 3bit目(4)が立っていない場合にのみ行われる)
  
      (なおコメントによると, 
       Responsible を NULL に戻したスレッドは, 
       その後に待ちキュー(EntryList|cxq)を読むまでにメモリバリアを入れておかなければならない, 
       とのこと.
       これについては, ObjectMonitor::EnterI() 中のコメントも参照のこと.)
  
      (_Responsible が自分かどうか等は特に確認してないが, 
       とにかく同一のスレッドが以下の順番で処理を行えばいいわけだから問題は無いか(?)
         ST Responsible=null; MEMBAR; LD cxq|EntryList; wakeup 処理)
      ---------------------------------------- -}

	   // Invariant: after setting Responsible=null an thread must execute
	   // a MEMBAR or other serializing instruction before fetching EntryList|cxq.
	   if ((SyncFlags & 4) == 0) {
	      _Responsible = NULL ;
	   }
	
  {- -------------------------------------------
  (1) (以下の for ループ内で, 実際にロックを解放する処理を行う.
       またロック待ちをしているスレッドがいれば, その起床処理も行う.
       以上の処理が完了するまで, このループからは出ない.)
  
      (ループ内では, 以下のように処理が行われる.
       (1) ロックの解放処理
           この処理は, Knob_ExitPolicy の値に応じて 2通りに分岐する.
           (といっても, どちらもそんなに変わらない)
  
       (2) ロック待ちをしているスレッドの起床処理 (ロック待ちをしているスレッドがいなければ行われない)
           この処理は, Knob_QMode の値に応じて 5 通りに分岐する.
           どの場合も, 最終的には ObjectMonitor::ExitEpilog() で実際の起床処理が行われる.
  
           * Knob_QMode が下記以外の値の場合
             (cxq よりも EntryList 内のスレッドの方を優先するポリシー)
             EntryList が空でなければ, その先頭のスレッドを起床させる.
             EntryList が空であれば, cxq を順番を保って EntryList に移した後, その先頭のスレッドを起床させる.
  
           * Knob_QMode が 1 の場合
             (cxq よりも EntryList 内のスレッドの方を優先するポリシー.)
             EntryList が空でなければ, その先頭のスレッドを起床させる.
             EntryList が空であれば, cxq を逆順にして EntryList に移した後, その先頭のスレッドを起床させる.
  
           * Knob_QMode が 2 の場合
             (EntryList よりも cxq 内のスレッドの方を優先するポリシー)
             cxq が空でなければ, その先頭のスレッドを起床させる.
             cxq が空であれば, EntryList の先頭のスレッドを起床させる.
  
           * Knob_QMode が 3 の場合
             (cxq よりも EntryList 内のスレッドの方を優先するポリシー)
             cxq の要素を EntryList の末尾に(順番を保って)移した後, 
             EntryList の先頭にあるスレッドを起床させる.
             (Knob_QMode が 2 の場合と同じことになりそうな気もするが???#TODO)
  
           * Knob_QMode が 4 の場合
             (EntryList よりも cxq 内のスレッドの方を優先するポリシー)
             cxq の要素を EntryList の頭に(順番を保って)移動させた後, 
             EntryList の先頭にあるスレッドを起床させる.
      ---------------------------------------- -}

	   for (;;) {

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	      assert (THREAD == _owner, "invariant") ;
	
	
    {- -------------------------------------------
  (1.1) (以下で, まずロックの解放処理を行う)
        ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) (以下は Knob_ExitPolicy が 0 の場合のロック解放処理)
        ---------------------------------------- -}

	      if (Knob_ExitPolicy == 0) {

      {- -------------------------------------------
  (1.1.1) ロックを解放する (= owner を NULL にする).
          ついでに, メモリバリアも張っている.
          (owner を NULL にするまでが critical section なので.
           critical section 内の load/store が critical section 後に re-order されるのは禁止)
  
          (なおコメントによると, 
           OrderAccess::release() とか OrderAccess::loadstore(); OrderAccess::storestore(); を使いたかったけど, 
           現状の実装だと内部で不要な store をしてくれて無駄なので release_store() を使っている, 
           とのこと)
          ---------------------------------------- -}

	         // release semantics: prior loads and stores from within the critical section
	         // must not float (reorder) past the following store that drops the lock.
	         // On SPARC that requires MEMBAR #loadstore|#storestore.
	         // But of course in TSO #loadstore|#storestore is not required.
	         // I'd like to write one of the following:
	         // A.  OrderAccess::release() ; _owner = NULL
	         // B.  OrderAccess::loadstore(); OrderAccess::storestore(); _owner = NULL;
	         // Unfortunately OrderAccess::release() and OrderAccess::loadstore() both
	         // store into a _dummy variable.  That store is not needed, but can result
	         // in massive wasteful coherency traffic on classic SMP systems.
	         // Instead, I use release_store(), which is implemented as just a simple
	         // ST on x64, x86 and SPARC.
	         OrderAccess::release_store_ptr (&_owner, NULL) ;   // drop the lock

      {- -------------------------------------------
  (1.1.1) 待ちキュー(EntryList や cxq)が空の場合, あるいは 
          _succ が空でない(既に誰かがスレッドが一つ起こしている or 現在スピンしているスレッドがいる)場合は, 
          次のスレッドを起こす必要はないので, ここで終了.
  
          (スピンしているスレッドのケースについては,
           ObjectMonitor::TrySpin_VaryDuration() 参照)
  
          (なおここにもメモリバリアを張っている. 
           待ちキューや _succ を確認するための load がロックを落とす前(critical section 内)に re-order されるのは禁止)
          ---------------------------------------- -}

	         OrderAccess::storeload() ;                         // See if we need to wake a successor
	         if ((intptr_t(_EntryList)|intptr_t(_cxq)) == 0 || _succ != NULL) {
	            TEVENT (Inflated exit - simple egress) ;
	            return ;
	         }

      {- -------------------------------------------
  (1.1.1) (ここから先は, 次のスレッドを起こす必要があったケース)
          ---------------------------------------- -}

      {- -------------------------------------------
  (1.1.1) (トレース出力)
          ---------------------------------------- -}

	         TEVENT (Inflated exit - complex egress) ;
	
      {- -------------------------------------------
  (1.1.1) 次のスレッドを起こす必要がある場合, もう一度 owner に CAS してロックを取得し, 
          EntryList や cxq をいじる権利を得ておく.
          もしロックが取れなかったら, ロックを取った他のスレッドに責任が移譲されたことになるので, ここで終了.
          ---------------------------------------- -}

	         // Normally the exiting thread is responsible for ensuring succession,
	         // but if other successors are ready or other entering threads are spinning
	         // then this thread can simply store NULL into _owner and exit without
	         // waking a successor.  The existence of spinners or ready successors
	         // guarantees proper succession (liveness).  Responsibility passes to the
	         // ready or running successors.  The exiting thread delegates the duty.
	         // More precisely, if a successor already exists this thread is absolved
	         // of the responsibility of waking (unparking) one.
	         //
	         // The _succ variable is critical to reducing futile wakeup frequency.
	         // _succ identifies the "heir presumptive" thread that has been made
	         // ready (unparked) but that has not yet run.  We need only one such
	         // successor thread to guarantee progress.
	         // See http://www.usenix.org/events/jvm01/full_papers/dice/dice.pdf
	         // section 3.3 "Futile Wakeup Throttling" for details.
	         //
	         // Note that spinners in Enter() also set _succ non-null.
	         // In the current implementation spinners opportunistically set
	         // _succ so that exiting threads might avoid waking a successor.
	         // Another less appealing alternative would be for the exiting thread
	         // to drop the lock and then spin briefly to see if a spinner managed
	         // to acquire the lock.  If so, the exiting thread could exit
	         // immediately without waking a successor, otherwise the exiting
	         // thread would need to dequeue and wake a successor.
	         // (Note that we'd need to make the post-drop spin short, but no
	         // shorter than the worst-case round-trip cache-line migration time.
	         // The dropped lock needs to become visible to the spinner, and then
	         // the acquisition of the lock by the spinner must become visible to
	         // the exiting thread).
	         //
	
	         // It appears that an heir-presumptive (successor) must be made ready.
	         // Only the current lock owner can manipulate the EntryList or
	         // drain _cxq, so we need to reacquire the lock.  If we fail
	         // to reacquire the lock the responsibility for ensuring succession
	         // falls to the new owner.
	         //
	         if (Atomic::cmpxchg_ptr (THREAD, &_owner, NULL) != NULL) {
	            return ;
	         }

      {- -------------------------------------------
  (1.1.1) (トレース出力)
          ---------------------------------------- -}

	         TEVENT (Exit - Reacquired) ;

    {- -------------------------------------------
  (1.1) (以下は Knob_ExitPolicy が 0 ではない場合のロック解放処理)
        ---------------------------------------- -}

	      } else {

      {- -------------------------------------------
  (1.1.1) まず次のスレッドを起こす必要があるかどうかを確認する.
          (EntryList や cxq が空の場合, あるいは _succ が空でない場合には必要ない)
          * 起こす必要がない場合
            以下の if の then 節でアンロック処理を行う.
          * 〃 がある場合
            この場合は, 今アンロックしても無駄 (スレッドを起こすためにはロックを持っている必要がある).
            以下の if の else 節に進んで, そのまま起床処理にフォールスルー.
          ---------------------------------------- -}

	         if ((intptr_t(_EntryList)|intptr_t(_cxq)) == 0 || _succ != NULL) {

      {- -------------------------------------------
  (1.1.1) (メモリバリアを張りながら) ロックを解放する (= owner を NULL にする).
          ---------------------------------------- -}

	            OrderAccess::release_store_ptr (&_owner, NULL) ;   // drop the lock

      {- -------------------------------------------
  (1.1.1) (メモリバリアを張りながら) 次のスレッドを起こす必要があるかどうかをもう一度確認する. 
          やっぱり必要なければ, ここでリターン.
          ---------------------------------------- -}

	            OrderAccess::storeload() ;
	            // Ratify the previously observed values.
	            if (_cxq == NULL || _succ != NULL) {
	                TEVENT (Inflated exit - simple egress) ;
	                return ;
	            }
	
      {- -------------------------------------------
  (1.1.1) 次のスレッドを起こす必要がある場合, もう一度 owner に CAS してロックを取得し, 
          EntryList や cxq をいじる権利を得ておく.
          もしロックが取れなかったら, ロックを取った他のスレッドに責任が移譲されたことになるので, ここで終了.
          ---------------------------------------- -}

	            // inopportune interleaving -- the exiting thread (this thread)
	            // in the fast-exit path raced an entering thread in the slow-enter
	            // path.
	            // We have two choices:
	            // A.  Try to reacquire the lock.
	            //     If the CAS() fails return immediately, otherwise
	            //     we either restart/rerun the exit operation, or simply
	            //     fall-through into the code below which wakes a successor.
	            // B.  If the elements forming the EntryList|cxq are TSM
	            //     we could simply unpark() the lead thread and return
	            //     without having set _succ.
	            if (Atomic::cmpxchg_ptr (THREAD, &_owner, NULL) != NULL) {
	               TEVENT (Inflated exit - reacquired succeeded) ;
	               return ;
	            }
	            TEVENT (Inflated exit - reacquired failed) ;

      {- -------------------------------------------
  (1.1.1) (次のスレッドを起こす必要がある場合は, このまま起床処理にフォールスルー)
          ---------------------------------------- -}

	         } else {
	            TEVENT (Inflated exit - complex egress) ;
	         }
	      }
	
    {- -------------------------------------------
  (1.1) (ここまでが, ロックの解放処理)
        (以降は, ロック待ちをしているスレッドを起床させる処理.
         ロック待ちしているスレッドがいない場合は, ここには到達しないはず.)
        ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	      guarantee (_owner == THREAD, "invariant") ;
	
    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	      ObjectWaiter * w = NULL ;
	      int QMode = Knob_QMode ;
	
    {- -------------------------------------------
  (1.1) Knob_QMode が 2 の場合は, 
        cxq が空でなければ, その先頭要素を ObjectMonitor::ExitEpilog() で起床させるだけ.
        この場合, ここでリターン.
  
        (なお, この時点では起こされたスレッド自体は cxq に含まれたままになっている.
         これは, 起床されてもすぐにロックが取れるとは限らないためだと思われる.
         実際に cxq から外す作業は, 起こされたスレッドの側で行っている.
         See: ObjectMonitor::UnlinkAfterAcquire())
        ---------------------------------------- -}

	      if (QMode == 2 && _cxq != NULL) {
	          // QMode == 2 : cxq has precedence over EntryList.
	          // Try to directly wake a successor from the cxq.
	          // If successful, the successor will need to unlink itself from cxq.
	          w = _cxq ;
	          assert (w != NULL, "invariant") ;
	          assert (w->TState == ObjectWaiter::TS_CXQ, "Invariant") ;
	          ExitEpilog (Self, w) ;
	          return ;
	      }
	
    {- -------------------------------------------
  (1.1) Knob_QMode が 3 の場合は, 
        cxq の要素を EntryList に移動させておく.
  
        (なお, (EntryList に移動するので) 
         TState は ObjectWaiter::TS_ENTER に変更されている.
         また, cxq では線形リストでつながっていたが, doubly linked list にしている.)
  
        (なおコメントによると, 
         EntryList の末尾に追加するので, 末尾を簡単に取得できるように 
         circular doubly linked list (CDLL) にしてはどうか, 
         とのこと)
        ---------------------------------------- -}

	      if (QMode == 3 && _cxq != NULL) {
	          // Aggressively drain cxq into EntryList at the first opportunity.
	          // This policy ensure that recently-run threads live at the head of EntryList.
	          // Drain _cxq into EntryList - bulk transfer.
	          // First, detach _cxq.
	          // The following loop is tantamount to: w = swap (&cxq, NULL)
	          w = _cxq ;
	          for (;;) {
	             assert (w != NULL, "Invariant") ;
	             ObjectWaiter * u = (ObjectWaiter *) Atomic::cmpxchg_ptr (NULL, &_cxq, w) ;
	             if (u == w) break ;
	             w = u ;
	          }
	          assert (w != NULL              , "invariant") ;
	
	          ObjectWaiter * q = NULL ;
	          ObjectWaiter * p ;
	          for (p = w ; p != NULL ; p = p->_next) {
	              guarantee (p->TState == ObjectWaiter::TS_CXQ, "Invariant") ;
	              p->TState = ObjectWaiter::TS_ENTER ;
	              p->_prev = q ;
	              q = p ;
	          }
	
	          // Append the RATs to the EntryList
	          // TODO: organize EntryList as a CDLL so we can locate the tail in constant-time.
	          ObjectWaiter * Tail ;
	          for (Tail = _EntryList ; Tail != NULL && Tail->_next != NULL ; Tail = Tail->_next) ;
	          if (Tail == NULL) {
	              _EntryList = w ;
	          } else {
	              Tail->_next = w ;
	              w->_prev = Tail ;
	          }
	
	          // Fall thru into code that tries to wake a successor from EntryList
	      }
	
    {- -------------------------------------------
  (1.1) Knob_QMode が 4 の場合は, 
        cxq の要素を EntryList に移動させておく.
  
        (この処理は Knob_QMode が 3 の場合の処理とよく似ているが,
         3 の場合は EntryList の末尾に追加したのに対し, 
         こちらでは EntryList の先頭に追加する.)
  
        (なお, (EntryList に移動するので) 
         TState は ObjectWaiter::TS_ENTER に変更されている.
         また, cxq では線形リストでつながっていたが, doubly linked list にしている.)
        ---------------------------------------- -}

	      if (QMode == 4 && _cxq != NULL) {
	          // Aggressively drain cxq into EntryList at the first opportunity.
	          // This policy ensure that recently-run threads live at the head of EntryList.
	
	          // Drain _cxq into EntryList - bulk transfer.
	          // First, detach _cxq.
	          // The following loop is tantamount to: w = swap (&cxq, NULL)
	          w = _cxq ;
	          for (;;) {
	             assert (w != NULL, "Invariant") ;
	             ObjectWaiter * u = (ObjectWaiter *) Atomic::cmpxchg_ptr (NULL, &_cxq, w) ;
	             if (u == w) break ;
	             w = u ;
	          }
	          assert (w != NULL              , "invariant") ;
	
	          ObjectWaiter * q = NULL ;
	          ObjectWaiter * p ;
	          for (p = w ; p != NULL ; p = p->_next) {
	              guarantee (p->TState == ObjectWaiter::TS_CXQ, "Invariant") ;
	              p->TState = ObjectWaiter::TS_ENTER ;
	              p->_prev = q ;
	              q = p ;
	          }
	
	          // Prepend the RATs to the EntryList
	          if (_EntryList != NULL) {
	              q->_next = _EntryList ;
	              _EntryList->_prev = q ;
	          }
	          _EntryList = w ;
	
	          // Fall thru into code that tries to wake a successor from EntryList
	      }
	
    {- -------------------------------------------
  (1.1) この時点で EntryList が空でない場合には, その先頭要素を ObjectMonitor::ExitEpilog() で起床させる.
        この場合, ここでリターン.
  
        (なお, この時点では起こされたスレッド自体は EntryList に含まれたままになっている.
         See: ObjectMonitor::UnlinkAfterAcquire())
        ---------------------------------------- -}

	      w = _EntryList  ;
	      if (w != NULL) {
	          // I'd like to write: guarantee (w->_thread != Self).
	          // But in practice an exiting thread may find itself on the EntryList.
	          // Lets say thread T1 calls O.wait().  Wait() enqueues T1 on O's waitset and
	          // then calls exit().  Exit release the lock by setting O._owner to NULL.
	          // Lets say T1 then stalls.  T2 acquires O and calls O.notify().  The
	          // notify() operation moves T1 from O's waitset to O's EntryList. T2 then
	          // release the lock "O".  T2 resumes immediately after the ST of null into
	          // _owner, above.  T2 notices that the EntryList is populated, so it
	          // reacquires the lock and then finds itself on the EntryList.
	          // Given all that, we have to tolerate the circumstance where "w" is
	          // associated with Self.
	          assert (w->TState == ObjectWaiter::TS_ENTER, "invariant") ;
	          ExitEpilog (Self, w) ;
	          return ;
	      }
	
    {- -------------------------------------------
  (1.1) (アンロック処理を行った時点では, 少なくともどちらか片方は空でなかったはずだが)
        現時点では EntryList も cxq も空になっている場合, 
        もう一度ループの先頭に戻ってやり直す.
        ---------------------------------------- -}

	      // If we find that both _cxq and EntryList are null then just
	      // re-run the exit protocol from the top.
	      w = _cxq ;
	      if (w == NULL) continue ;
	
    {- -------------------------------------------
  (1.1) (ここまで到達したケースでは, _EntryList は空だが cxq は空ではない, という状態のはず)
        ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) cxq から全要素を取り出す. cxq 自体は空(NULL)にする.
        ---------------------------------------- -}

	      // Drain _cxq into EntryList - bulk transfer.
	      // First, detach _cxq.
	      // The following loop is tantamount to: w = swap (&cxq, NULL)
	      for (;;) {
	          assert (w != NULL, "Invariant") ;
	          ObjectWaiter * u = (ObjectWaiter *) Atomic::cmpxchg_ptr (NULL, &_cxq, w) ;
	          if (u == w) break ;
	          w = u ;
	      }

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	      TEVENT (Inflated exit - drain cxq into EntryList) ;
	
    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	      assert (w != NULL              , "invariant") ;
	      assert (_EntryList  == NULL    , "invariant") ;
	
    {- -------------------------------------------
  (1.1) cxq に入っていた内容を EntryList に移し変える.
        Knob_QMode が 1 の場合には, cxq に入っていた順とは逆順で EntryList に格納する.
        それ以外の場合には, cxq の時と同じ順番で EntryList に格納する.
  
        (なお, Knob_QMode が 3 の場合と 4 の場合は, ここまで到達することはないはず.)
  
        (なお, (EntryList に移動するので) 
         TState は ObjectWaiter::TS_ENTER に変更されている.
         また, cxq では線形リストでつながっていたが, doubly linked list にしている.)
    
        (なおコメントによると, この処理は outer lock を保持した状態で行うので
         できる限り高速にやる必要がある, とのこと.
         そのために, EntryList を circular doubly linked list (CDLL) にしてはどうか, とも書かれている.)
        ---------------------------------------- -}

	      // Convert the LIFO SLL anchored by _cxq into a DLL.
	      // The list reorganization step operates in O(LENGTH(w)) time.
	      // It's critical that this step operate quickly as
	      // "Self" still holds the outer-lock, restricting parallelism
	      // and effectively lengthening the critical section.
	      // Invariant: s chases t chases u.
	      // TODO-FIXME: consider changing EntryList from a DLL to a CDLL so
	      // we have faster access to the tail.
	
	      if (QMode == 1) {
	         // QMode == 1 : drain cxq to EntryList, reversing order
	         // We also reverse the order of the list.
	         ObjectWaiter * s = NULL ;
	         ObjectWaiter * t = w ;
	         ObjectWaiter * u = NULL ;
	         while (t != NULL) {
	             guarantee (t->TState == ObjectWaiter::TS_CXQ, "invariant") ;
	             t->TState = ObjectWaiter::TS_ENTER ;
	             u = t->_next ;
	             t->_prev = u ;
	             t->_next = s ;
	             s = t;
	             t = u ;
	         }
	         _EntryList  = s ;
	         assert (s != NULL, "invariant") ;
	      } else {
	         // QMode == 0 or QMode == 2
	         _EntryList = w ;
	         ObjectWaiter * q = NULL ;
	         ObjectWaiter * p ;
	         for (p = w ; p != NULL ; p = p->_next) {
	             guarantee (p->TState == ObjectWaiter::TS_CXQ, "Invariant") ;
	             p->TState = ObjectWaiter::TS_ENTER ;
	             p->_prev = q ;
	             q = p ;
	         }
	      }
	
    {- -------------------------------------------
  (1.1) (1-0 mode においては, EntryList の変更とロックの解放(owner の変更)は
         以下の順で処理しなければいけない, とのこと.
  
           ST EntryList; MEMBAR #storestore; ST _owner = NULL
  
         ただし, MEMBAR については ExitEpilog() 内の release_store() で満たされる.)
        ---------------------------------------- -}

	      // In 1-0 mode we need: ST EntryList; MEMBAR #storestore; ST _owner = NULL
	      // The MEMBAR is satisfied by the release_store() operation in ExitEpilog().
	
    {- -------------------------------------------
  (1.1) もしこの時点でスピンしているスレッドがいれば(= _succ が NULL でなければ), 
        ループの先頭に戻ってやり直す.
        (これは, コンテキストスイッチの回数をできる限り減らすための措置)
        ---------------------------------------- -}

	      // See if we can abdicate to a spinner instead of waking a thread.
	      // A primary goal of the implementation is to reduce the
	      // context-switch rate.
	      if (_succ != NULL) continue;
	
    {- -------------------------------------------
  (1.1) この時点で EntryList が空でない場合には, その先頭要素を ObjectMonitor::ExitEpilog() で起床させる.
        この場合, ここでリターン.
        (というコードになってはいるが, この段階で EntryList が NULL というケースはあり得るのか??
         少し上にある assert で fail するケースなので, 普通はあり得ない気がする... #TODO)
  
        (なお, この時点では起こされたスレッド自体は EntryList に含まれたままになっている.
         See: ObjectMonitor::UnlinkAfterAcquire())
        ---------------------------------------- -}

	      w = _EntryList  ;
	      if (w != NULL) {
	          guarantee (w->TState == ObjectWaiter::TS_ENTER, "invariant") ;
	          ExitEpilog (Self, w) ;
	          return ;
	      }
	   }
	}
	
```


