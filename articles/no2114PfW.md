---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/mutex.cpp
### 説明(description)
// ON THE VMTHREAD SNEAKING PAST HELD LOCKS:
// In particular, there are certain types of global lock that may be held
// by a Java thread while it is blocked at a safepoint but before it has
// written the _owner field. These locks may be sneakily acquired by the
// VM thread during a safepoint to avoid deadlocks. Alternatively, one should
// identify all such locks, and ensure that Java threads never block at
// safepoints while holding them (_no_safepoint_check_flag). While it
// seems as though this could increase the time to reach a safepoint
// (or at least increase the mean, if not the variance), the latter
// approach might make for a cleaner, more maintainable JVM design.
//
// Sneaking is vile and reprehensible and should be excised at the 1st
// opportunity.  It's possible that the need for sneaking could be obviated
// as follows.  Currently, a thread might (a) while TBIVM, call pthread_mutex_lock
// or ILock() thus acquiring the "physical" lock underlying Monitor/Mutex.
// (b) stall at the TBIVM exit point as a safepoint is in effect.  Critically,
// it'll stall at the TBIVM reentry state transition after having acquired the
// underlying lock, but before having set _owner and having entered the actual
// critical section.  The lock-sneaking facility leverages that fact and allowed the
// VM thread to logically acquire locks that had already be physically locked by mutators
// but where mutators were known blocked by the reentry thread state transition.
//
// If we were to modify the Monitor-Mutex so that TBIVM state transitions tightly
// wrapped calls to park(), then we could likely do away with sneaking.  We'd
// decouple lock acquisition and parking.  The critical invariant  to eliminating
// sneaking is to ensure that we never "physically" acquire the lock while TBIVM.
// An easy way to accomplish this is to wrap the park calls in a narrow TBIVM jacket.
// One difficulty with this approach is that the TBIVM wrapper could recurse and
// call lock() deep from within a lock() call, while the MutexEvent was already enqueued.
// Using a stack (N=2 at minimum) of ParkEvents would take care of that problem.
//
// But of course the proper ultimate approach is to avoid schemes that require explicit
// sneaking or dependence on any any clever invariants or subtle implementation properties
// of Mutex-Monitor and instead directly address the underlying design flaw.



### 名前(function name)
```
void Monitor::lock (Thread * Self) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef CHECK_UNHANDLED_OOPS 時にのみ実行) (See: UnhandledOops)
      ---------------------------------------- -}

	#ifdef CHECK_UNHANDLED_OOPS
	  // Clear unhandled oops so we get a crash right away.  Only clear for non-vm
	  // or GC threads.
	  if (Self->is_Java_thread()) {
	    Self->clear_unhandled_oops();
	  }
	#endif // CHECK_UNHANDLED_OOPS
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  debug_only(check_prelock_state(Self));
	  assert (_owner != Self              , "invariant") ;
	  assert (_OnDeck != Self->_MutexEvent, "invariant") ;
	
  {- -------------------------------------------
  (1) まずは Monitor::TryFast() で取得を試みる.
      成功すれば, ここでリターン.
  
      (ついでに, リターンする直前に
       Monitor::set_owner() でカレントスレッドを owner に設定している)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (なお, ここのリターン処理には Exeunt というラベルが付いている.
       これ以降でも, ロック処理を終了してリターンしたくなった箇所には
       "goto Exeunt" と書かれていることが多い.)
      ---------------------------------------- -}

	  if (TryFast()) {
	 Exeunt:
	    assert (ILocked(), "invariant") ;
	    assert (owner() == NULL, "invariant");
	    set_owner (Self);
	    return ;
	  }
	
  {- -------------------------------------------
  (1) (以降は, 本当に contention している場合の処理)
      ---------------------------------------- -}

	  // The lock is contended ...
	
  {- -------------------------------------------
  (1) もし以下の条件が全て満たされるのであれば, ロックは取れたことにする (上記のコメント参照).
      そのまま Exeunt ラベルに飛んでリターン.
      * ロックを取ろうとしているのが VMThread
      * 現在, Safepoint の最中
      * 対象のロックを確保している JavaThread は, ちょうどこれから critical section に入るところ (_owner が NULL)
  
      (なお, きちんとロックを取得したわけではないので, 
       _snuck フィールドを true にすることでそのことを覚えておく.
      (See: Monitor::unlock())
      ---------------------------------------- -}

	  bool can_sneak = Self->is_VM_thread() && SafepointSynchronize::is_at_safepoint();
	  if (can_sneak && _owner == NULL) {
	    // a java thread has locked the lock but has not entered the
	    // critical region -- let's just pretend we've locked the lock
	    // and go on.  we note this with _snuck so we can also
	    // pretend to unlock when the time comes.
	    _snuck = true;
	    goto Exeunt ;
	  }
	
  {- -------------------------------------------
  (1) Monitor::TrySpin() でスピンロックしてみて確保を試みる.
      もしこれでロックが取れたら, Exeunt ラベルに飛んでリターンする.
      ---------------------------------------- -}

	  // Try a brief spin to avoid passing thru thread state transition ...
	  if (TrySpin (Self)) goto Exeunt ;
	
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: Monitor::check_block_state())
      ---------------------------------------- -}

	  check_block_state(Self);

  {- -------------------------------------------
  (1) (ここまでの処理でロックが取れないなら) Monitor::ILock() で本格的なロック取得を行う.
      ロックが取れたら Exeunt ラベルに飛んでリターンする.
      
      (なお, ロックを取ろうとしているのが JavaThread の場合は
       Monitor::ILock() を呼ぶ前に
       ThreadBlockInVM で JavaThread の状態を変更しておく.
  
       コメントによると, ここまで来てしまうと状態遷移処理が必要で遅いので "Horribile dictu"(悲しむべきことに(?)), らしい.
       逆に, JavaThread でなければ "Mirabile dictu"(素晴らしいことに(?)) とのこと.)
      ---------------------------------------- -}

	  if (Self->is_Java_thread()) {
	    // Horribile dictu - we suffer through a state transition
	    assert(rank() > Mutex::special, "Potential deadlock with special or lesser rank mutex");
	    ThreadBlockInVM tbivm ((JavaThread *) Self) ;
	    ILock (Self) ;
	  } else {
	    // Mirabile dictu
	    ILock (Self) ;
	  }
	  goto Exeunt ;
	}
	
```


