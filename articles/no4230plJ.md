---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/objectMonitor.cpp

### 名前(function name)
```
void ObjectMonitor::notifyAll(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ロックを持っているスレッドしか notifyAll() を呼んではいけないので)
      CHECK_OWNER() でロックを持っているかどうかをチェックしておく.
      持っていなければ IllegalMonitorStateException.
  
      (なお, もしカレントスレッドがロックを持っているにも関わらず
       ObjectMonitor 内の _owner フィールドがカレントスレッドを指していない場合, 
      _owner をカレントスレッドに変更してもいる)
      ---------------------------------------- -}

	  CHECK_OWNER();

  {- -------------------------------------------
  (1) もし誰も待ちキューにいなければ, ここでリターン.
      ---------------------------------------- -}

	  ObjectWaiter* iterator;
	  if (_WaitSet == NULL) {
	      TEVENT (Empty-NotifyAll) ;
	      return ;
	  }

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_MONITOR_PROBE(notifyAll, this, object(), THREAD);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (Knob_MoveNotifyee は起床処理の内容を決めるパラメータ. 後述)
      ---------------------------------------- -}

	  int Policy = Knob_MoveNotifyee ;

  {- -------------------------------------------
  (1) (変数宣言など)
      (Tally は, 実際にスレッドの起床処理が行われたかどうかを記録するための変数. 
       0 でなければ行われたことを意味する.
       後で, _sync_Notifications をカウントアップするかどうかの判断に使う)
      ---------------------------------------- -}

	  int Tally = 0 ;

  {- -------------------------------------------
  (1) (ここから先は _WaitSetLock で守られた critical section)
      ---------------------------------------- -}

	  Thread::SpinAcquire (&_WaitSetLock, "WaitSet - notifyall") ;
	
  {- -------------------------------------------
  (1) 以下の for ループ内で, 待ちキュー(WaitSet)内の全ての要素に対して起床処理を行う.
      
      なお, ここで行われる起床処理は Knob_MoveNotifyee の値に応じて 5 通りが存在.
      * 0 の場合: 
        EntryList の先頭に追加する.
      * 1 の場合: 
        EntryList の末尾に追加する.
      * 2 の場合:                 (<= このケースのみ, ObjectMonitor::notify() と若干挙動が違うが...#TODO)
        cxq の先頭に追加する.
      * 3 の場合: 
        cxq の末尾に追加する.
      * それ以外の場合: (実質的には 4 の場合??)
        EntryList や cxq には追加せず, この場で os::PlatformEvent::unpark() を呼んで起床させる.
      ---------------------------------------- -}

	  for (;;) {

    {- -------------------------------------------
  (1.1) ObjectMonitor::DequeueWaiter() で, 待ちキューから ObjectWaiter を取り出す.
        (取り出された ObjectWaiter は待ちキュー内には存在しなくなる).
        待ちキューが空であれば, このループは終了.
        ---------------------------------------- -}

	     iterator = DequeueWaiter () ;
	     if (iterator == NULL) break ;

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	     TEVENT (NotifyAll - Transfer1) ;

    {- -------------------------------------------
  (1.1) Tally をインクリメントしておく.
        ---------------------------------------- -}

	     ++Tally ;
	
    {- -------------------------------------------
  (1.1) (コメントには以下のように書かれている. 
         が, 今はそれを Knob_MoveNotifyee で調整しているのでは??
  
         「どう処理すればいいか?
           a. EntryList に追加する (末尾か先頭かに).
           b. cxq の先頭に追加する.
           現時点では a. の方式を採用」)
        ---------------------------------------- -}

	     // Disposition - what might we do with iterator ?
	     // a.  add it directly to the EntryList - either tail or head.
	     // b.  push it onto the front of the _cxq.
	     // For now we use (a).
	
    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	     guarantee (iterator->TState == ObjectWaiter::TS_WAIT, "invariant") ;
	     guarantee (iterator->_notified == 0, "invariant") ;

    {- -------------------------------------------
  (1.1) _notified フィールドを 1 にセットしておく.
        (_notified フィールドは, wait() 待ちに入ったスレッドが起床していいかどうかの判断に使っているフィールド.
         See: ObjectMonitor::wait())
        ---------------------------------------- -}

	     iterator->_notified = 1 ;

    {- -------------------------------------------
  (1.1) TState フィールドを変更しておく.
        (ただし, ここでは ObjectWaiter::TS_ENTER (= EntryList 内にいる状態) に変更する.
         Knob_MoveNotifyee の値によっては EntryList に追加しないケースもあるが, 
         その場合については後で改めて変更し直している.
  
         (<= EntryList の場合についても後でやればいい気がするが...?? #TODO))
        ---------------------------------------- -}

	     if (Policy != 4) {
	        iterator->TState = ObjectWaiter::TS_ENTER ;
	     }
	
    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	     ObjectWaiter * List = _EntryList ;

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	     if (List != NULL) {
	        assert (List->_prev == NULL, "invariant") ;
	        assert (List->TState == ObjectWaiter::TS_ENTER, "invariant") ;
	        assert (List != iterator, "invariant") ;
	     }
	
    {- -------------------------------------------
  (1.1) 待ちキューから取り出した要素を起床させる.
        Knob_MoveNotifyee の値に応じて, 上述のように 5 通りの起床方法がある.
        (EntryList の先頭に追加/末尾に追加, cxq の先頭に追加/末尾に追加, 追加せずこの場で起こす)
  
        (なおコメントによると, 
         EntryList の末尾を簡単に取得できるように, 
         EntryList を circular doubly linked list (CDLL) にしてはどうか, 
         とのこと)
        ---------------------------------------- -}

	     if (Policy == 0) {       // prepend to EntryList
	         if (List == NULL) {
	             iterator->_next = iterator->_prev = NULL ;
	             _EntryList = iterator ;
	         } else {
	             List->_prev = iterator ;
	             iterator->_next = List ;
	             iterator->_prev = NULL ;
	             _EntryList = iterator ;
	        }
	     } else
	     if (Policy == 1) {      // append to EntryList
	         if (List == NULL) {
	             iterator->_next = iterator->_prev = NULL ;
	             _EntryList = iterator ;
	         } else {
	            // CONSIDER:  finding the tail currently requires a linear-time walk of
	            // the EntryList.  We can make tail access constant-time by converting to
	            // a CDLL instead of using our current DLL.
	            ObjectWaiter * Tail ;
	            for (Tail = List ; Tail->_next != NULL ; Tail = Tail->_next) ;
	            assert (Tail != NULL && Tail->_next == NULL, "invariant") ;
	            Tail->_next = iterator ;
	            iterator->_prev = Tail ;
	            iterator->_next = NULL ;
	        }
	     } else
	     if (Policy == 2) {      // prepend to cxq
	         // prepend to cxq
	         iterator->TState = ObjectWaiter::TS_CXQ ;
	         for (;;) {
	             ObjectWaiter * Front = _cxq ;
	             iterator->_next = Front ;
	             if (Atomic::cmpxchg_ptr (iterator, &_cxq, Front) == Front) {
	                 break ;
	             }
	         }
	     } else
	     if (Policy == 3) {      // append to cxq
	        iterator->TState = ObjectWaiter::TS_CXQ ;
	        for (;;) {
	            ObjectWaiter * Tail ;
	            Tail = _cxq ;
	            if (Tail == NULL) {
	                iterator->_next = NULL ;
	                if (Atomic::cmpxchg_ptr (iterator, &_cxq, NULL) == NULL) {
	                   break ;
	                }
	            } else {
	                while (Tail->_next != NULL) Tail = Tail->_next ;
	                Tail->_next = iterator ;
	                iterator->_prev = Tail ;
	                iterator->_next = NULL ;
	                break ;
	            }
	        }
	     } else {
	        ParkEvent * ev = iterator->_event ;
	        iterator->TState = ObjectWaiter::TS_RUN ;
	        OrderAccess::fence() ;
	        ev->unpark() ;
	     }
	
    {- -------------------------------------------
  (1.1) (プロファイル情報の記録)
        (See: ObjectWaiter::wait_reenter_begin()) (See: [here](no21146np.html) for details)
        ---------------------------------------- -}

	     if (Policy < 4) {
	       iterator->wait_reenter_begin(this);
	     }
	
    {- -------------------------------------------
  (1.1) (_WaitSetLock は, EntryList ではなく _WaitSet を保護するためのロックなので, 
         EntryList に要素を追加する処理については _WaitSet で保護された critical section の外で行うようにしてもいい.
         ただし, _WaitSetLock はそもそもほとんど競合しない (wait() がタイムアウトするときか 
         interrupt されたときのみ競合する) ので, critical section を短くすることにさほどの意味は無い.)
        ---------------------------------------- -}

	     // _WaitSetLock protects the wait queue, not the EntryList.  We could
	     // move the add-to-EntryList operation, above, outside the critical section
	     // protected by _WaitSetLock.  In practice that's not useful.  With the
	     // exception of  wait() timeouts and interrupts the monitor owner
	     // is the only thread that grabs _WaitSetLock.  There's almost no contention
	     // on _WaitSetLock so it's not profitable to reduce the length of the
	     // critical section.
	  }
	
  {- -------------------------------------------
  (1) (ここまでが _WaitSetLock による critical section)
      ---------------------------------------- -}

	  Thread::SpinRelease (&_WaitSetLock) ;
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録) (See: UsePerfData) (See: sun.rt._sync_Notifications)
      (なお, 実際にスレッドの起床処理を行った場合にのみカウンタ値を増加させている)
      ---------------------------------------- -}

	  if (Tally != 0 && ObjectMonitor::_sync_Notifications != NULL) {
	     ObjectMonitor::_sync_Notifications->inc(Tally) ;
	  }
	}
	
```


