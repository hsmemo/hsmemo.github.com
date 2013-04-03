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
void ATTR ObjectMonitor::EnterI (TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    Thread * Self = THREAD ;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert (Self->is_Java_thread(), "invariant") ;
	    assert (((JavaThread *) Self)->thread_state() == _thread_blocked   , "invariant") ;
	
  {- -------------------------------------------
  (1) まずは ObjectMonitor::TryLock() でロック確保を試みる.
      ロックが取れたら, ここでリターン.
      ---------------------------------------- -}

	    // Try the lock - TATAS
	    if (TryLock (Self) > 0) {
	        assert (_succ != Self              , "invariant") ;
	        assert (_owner == Self             , "invariant") ;
	        assert (_Responsible != Self       , "invariant") ;
	        return ;
	    }
	
  {- -------------------------------------------
  (1) ObjectMonitor::DeferredInitialize() を呼んで, 
      (まだ初期化が終わっていなければ) Knob_* 変数の値の初期化を行う.
      ---------------------------------------- -}

	    DeferredInitialize () ;
	
  {- -------------------------------------------
  (1) TrySpin() でスピンロックしてみる. スピンロックが成功したらここでリターン.
  
      (なお, TrySpin とは ObjectMonitor::TrySpin_VaryDuration() のこと.
       See: Adaptive Spinning)
  
      (なおコメントによると, 
       ロックを保持しているスレッドが稼働中でなければ(OFFPROC であれば) 
       そのスレッドに yield するようにしてはどうか, とのこと)
      ---------------------------------------- -}

	    // We try one round of spinning *before* enqueueing Self.
	    //
	    // If the _owner is ready but OFFPROC we could use a YieldTo()
	    // operation to donate the remainder of this thread's quantum
	    // to the owner.  This has subtle but beneficial affinity
	    // effects.
	
	    if (TrySpin (Self) > 0) {
	        assert (_owner == Self        , "invariant") ;
	        assert (_succ != Self         , "invariant") ;
	        assert (_Responsible != Self  , "invariant") ;
	        return ;
	    }
	
  {- -------------------------------------------
  (1) (これ以降は, しょうがないのでロックが解放されるまで眠って待つパス)
      ---------------------------------------- -}

	    // The Spin failed -- Enqueue and park the thread ...

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert (_succ  != Self            , "invariant") ;
	    assert (_owner != Self            , "invariant") ;
	    assert (_Responsible != Self      , "invariant") ;
	
  {- -------------------------------------------
  (1) (寝る前に, ロック待ち対象の ObjectMonitor の待ちキュー(_cxq)にカレントスレッドを登録しておく必要がある.
       以下で, その登録処理を行う.
       なお, 実際に登録するのはカレントスレッド自体ではなく, カレントスレッドを指し示す ObjectWaiter.)
       
      (なお, コメントによると, 
       もし仮に Java でこのコードを書き直すことになったら
       WaitNodes, ObjectMonitors, そして Events は 1st-class の Java のオブジェクトにするだろう.
       そうすれば, ライフサイクルに関する問題や ABA 問題を回避できるだろう, 
       とのこと.)
       
      (また, コメントによると, 
       ObjectWaiter は廃止して Threads 自体か Events を登録するようにしたい, 
       とのこと.)
      ---------------------------------------- -}

	    // Enqueue "Self" on ObjectMonitor's _cxq.
	    //
	    // Node acts as a proxy for Self.
	    // As an aside, if were to ever rewrite the synchronization code mostly
	    // in Java, WaitNodes, ObjectMonitors, and Events would become 1st-class
	    // Java objects.  This would avoid awkward lifecycle and liveness issues,
	    // as well as eliminate a subset of ABA issues.
	    // TODO: eliminate ObjectWaiter and enqueue either Threads or Events.
	    //
	
    {- -------------------------------------------
  (1.1) (変数宣言など)
        (以下の node が待ちキューに追加される ObjectWaiter)
        ---------------------------------------- -}

	    ObjectWaiter node(Self) ;

    {- -------------------------------------------
  (1.1) os::PlatformEvent::reset() を呼んで, 
        カレントスレッドの _ParkEvent フィールドをリセットしておく.
        (何故ここでやる?? もっと後でもいいのでは??)
        ---------------------------------------- -}

	    Self->_ParkEvent->reset() ;

    {- -------------------------------------------
  (1.1) node のフィールドを初期化しておく.
        ---------------------------------------- -}

	    node._prev   = (ObjectWaiter *) 0xBAD ;
	    node.TState  = ObjectWaiter::TS_CXQ ;
	
    {- -------------------------------------------
  (1.1) CAS で, node を待ちキューに追加する.
        (CAS は失敗することもあるので, 成功するまで以下の for ループを繰り返す.)
  
        (なお最適化のため, 以下の for ループ内では
         ObjectMonitor::TryLock() によるロック確保も試みながらループしている.
         待ちキューに追加するよりも先に ObjectMonitor::TryLock() が成功したら, 
         その時点でリターンする.)
        ---------------------------------------- -}

	    // Push "Self" onto the front of the _cxq.
	    // Once on cxq/EntryList, Self stays on-queue until it acquires the lock.
	    // Note that spinning tends to reduce the rate at which threads
	    // enqueue and dequeue on EntryList|cxq.
	    ObjectWaiter * nxt ;
	    for (;;) {
	        node._next = nxt = _cxq ;
	        if (Atomic::cmpxchg_ptr (&node, &_cxq, nxt) == nxt) break ;
	
	        // Interference - the CAS failed because _cxq changed.  Just retry.
	        // As an optional optimization we retry the lock.
	        if (TryLock (Self) > 0) {
	            assert (_succ != Self         , "invariant") ;
	            assert (_owner == Self        , "invariant") ;
	            assert (_Responsible != Self  , "invariant") ;
	            return ;
	        }
	    }

  {- -------------------------------------------
  (1) (ここまでが, ロック待ち対象の ObjectMonitor の待ちキューにカレントスレッドを登録する処理.)
      ---------------------------------------- -}
	
  {- -------------------------------------------
  (1) (以下のコメントに 1-0 model の説明が書いてある.
  
       1-0 model を採用して, inflated を unlock にする際に atomic 等を用いない場合, 
       待ち行列に入ったスレッドが二度と起きてこない(stranding)かもしれない危険性が生じる.
       そのため, 待ち行列(cxqまたはEntryList)中にいるスレッドの誰かが
       時間無制限のpark()の代わりにタイムアウト付きのpark()を使って
       定期的にロック(_owner)を確認することにする.
       このスレッドを "Responsible" と呼ぶ.
       
       他の解決方としては, 専属の "stranding checker" スレッドを用意してもいい.
       これはタイマーによるスケーラビリティへの影響を抑える. 
       (なぜなら, この場合タイムアウト付きで待つのは "stranding checker" スレッドだけになるので))
      ---------------------------------------- -}

	    // Check for cxq|EntryList edge transition to non-null.  This indicates
	    // the onset of contention.  While contention persists exiting threads
	    // will use a ST:MEMBAR:LD 1-1 exit protocol.  When contention abates exit
	    // operations revert to the faster 1-0 mode.  This enter operation may interleave
	    // (race) a concurrent 1-0 exit operation, resulting in stranding, so we
	    // arrange for one of the contending thread to use a timed park() operations
	    // to detect and recover from the race.  (Stranding is form of progress failure
	    // where the monitor is unlocked but all the contending threads remain parked).
	    // That is, at least one of the contended threads will periodically poll _owner.
	    // One of the contending threads will become the designated "Responsible" thread.
	    // The Responsible thread uses a timed park instead of a normal indefinite park
	    // operation -- it periodically wakes and checks for and recovers from potential
	    // strandings admitted by 1-0 exit operations.   We need at most one Responsible
	    // thread per-monitor at any given moment.  Only threads on cxq|EntryList may
	    // be responsible for a monitor.
	    //
	    // Currently, one of the contended threads takes on the added role of "Responsible".
	    // A viable alternative would be to use a dedicated "stranding checker" thread
	    // that periodically iterated over all the threads (or active monitors) and unparked
	    // successors where there was risk of stranding.  This would help eliminate the
	    // timer scalability issues we see on some platforms as we'd only have one thread
	    // -- the checker -- parked on a timer.
	
  {- -------------------------------------------
  (1) もし待ち行列中(cxq|EntryList)に誰もいなければ, 自分を _Responsible に登録しておく.
      (See: 1-0 model)
  
      (なお, この最適化は SyncFlags の 5bit目(16)が立っている場合にのみ行われる)
      ---------------------------------------- -}

	    if ((SyncFlags & 16) == 0 && nxt == NULL && _EntryList == NULL) {
	        // Try to assume the role of responsible thread for the monitor.
	        // CONSIDER:  ST vs CAS vs { if (Responsible==null) Responsible=Self }
	        Atomic::cmpxchg_ptr (Self, &_Responsible, NULL) ;
	    }
	
  {- -------------------------------------------
  (1) (待ちキューにカレントスレッドを登録している間に, ロックが解放されてしまったかもしれない.
       待ち行列に入ったスレッドが二度と起きてこない事態(stranding)を防ぐには, 
       眠りにつく前にロック(_owner)を再確認しておく必要がある.
       
       なお, デッカーのアルゴリズムによる確認になるので, 
       「cxq の変更, メモリバリア, _owner の確認」の順になる.
       ただし, 「cxq の変更, メモリバリア」は, cxq を変更する際に CAS を使ったことで実現できているため, 
       後は _owner の確認(= ロック確保処理) を行えばいい.)
    
      (なお, コメントによると, 
       スレッドの状態を変更する処理は重いので, できる限り遅延させたい, 
       とのこと.)
      ---------------------------------------- -}

	    // The lock have been released while this thread was occupied queueing
	    // itself onto _cxq.  To close the race and avoid "stranding" and
	    // progress-liveness failure we must resample-retry _owner before parking.
	    // Note the Dekker/Lamport duality: ST cxq; MEMBAR; LD Owner.
	    // In this case the ST-MEMBAR is accomplished with CAS().
	    //
	    // TODO: Defer all thread state transitions until park-time.
	    // Since state transitions are heavy and inefficient we'd like
	    // to defer the state transitions until absolutely necessary,
	    // and in doing so avoid some transitions ...
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	    TEVENT (Inflated enter - Contention) ;

  {- -------------------------------------------
  (1) (変数宣言など)
      (nWakeups の方は, 意味のある使われ方をしていない気もするが... #TODO)
      ---------------------------------------- -}

	    int nWakeups = 0 ;
	    int RecheckInterval = 1 ;
	
  {- -------------------------------------------
  (1) (以下の for ループ内で, 実際にロックを確保する処理を行う.
       ロックが取れるまで, このループからは出ない.)
      ---------------------------------------- -}

	    for (;;) {
	
    {- -------------------------------------------
  (1.1) まずは ObjectMonitor::TryLock() でロック確保を試みる.
        ロックが取れたら, ここでループから抜ける.
        ---------------------------------------- -}

	        if (TryLock (Self) > 0) break ;

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	        assert (_owner != Self, "invariant") ;
	
    {- -------------------------------------------
  (1.1) もし _Responsible に誰も登録されていなければ (= _Responsible が NULL であれば), 自分のアドレスをいれておく.
        (See: 1-0 model)
  
        (なお, この最適化は SyncFlags の 2bit目(2)が立っている場合にのみ行われる)
        ---------------------------------------- -}

	        if ((SyncFlags & 2) && _Responsible == NULL) {
	           Atomic::cmpxchg_ptr (Self, &_Responsible, NULL) ;
	        }
	
    {- -------------------------------------------
  (1.1) _ParkEvent フィールドに対して os::PlatformEvent::park() を呼ぶことで眠りにつく.
  
        なお, 条件によって呼び出す park() が少し変わる (See: 1-0 model).
        * 自分が _Responsible である場合 (または SyncFlags の 1bit目(1)が立っている場合  <= これは何だ?? #TODO)
          (誰かが起こしてくれなくても最悪自力で起きてこないといけないので)
          タイムアウト付きの os::PlatformEvent::park() を呼び出す.
          タイムアウト時間は RecheckInterval で指定しており, 
          最初は 1 だが park() する度に 8 倍ずつ増やしている (ただし 1000 以上には増やさない).
        * 上記以外の場合
          タイムアウト無しの os::PlatformEvent::park() を呼び出す.
    
        (なおどちらの場合も, ついでに(トレース出力)も出している)
        ---------------------------------------- -}

	        // park self
	        if (_Responsible == Self || (SyncFlags & 1)) {
	            TEVENT (Inflated enter - park TIMED) ;
	            Self->_ParkEvent->park ((jlong) RecheckInterval) ;
	            // Increase the RecheckInterval, but clamp the value.
	            RecheckInterval *= 8 ;
	            if (RecheckInterval > 1000) RecheckInterval = 1000 ;
	        } else {
	            TEVENT (Inflated enter - park UNTIMED) ;
	            Self->_ParkEvent->park() ;
	        }
	
    {- -------------------------------------------
  (1.1) 起きてきたら, ObjectMonitor::TryLock() でロック確保を試みる.
        ロックが取れたら, ここでループから抜ける.
        ---------------------------------------- -}

	        if (TryLock(Self) > 0) break ;
	
    {- -------------------------------------------
  (1.1) (ここに到達するのは, 一度寝てみたけどまだロックが競合しているケース.
         とりあえず, futile wakeup 回数の統計情報を更新しておく.
         なお, カウントアップする際に排他やアトミック操作を使っていないが, 
         これはオーバーヘッドを減らすための意図的な挙動.
         カウンタ値自体は多少狂っていてもいい.)
        ---------------------------------------- -}

	        // The lock is still contested.
	        // Keep a tally of the # of futile wakeups.
	        // Note that the counter is not protected by a lock or updated by atomics.
	        // That is by design - we trade "lossy" counters which are exposed to
	        // races during updates for a lower probe effect.

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	        TEVENT (Inflated enter - Futile wakeup) ;

    {- -------------------------------------------
  (1.1) (プロファイル情報の記録) (See: UsePerfData) (See: sun.rt._sync_FutileWakeups)
        ---------------------------------------- -}

	        if (ObjectMonitor::_sync_FutileWakeups != NULL) {
	           ObjectMonitor::_sync_FutileWakeups->inc() ;
	        }

    {- -------------------------------------------
  (1.1) ?? (nWakeups は, この後どこからも参照されていないが...)
        ---------------------------------------- -}

	        ++ nWakeups ;
	
    {- -------------------------------------------
  (1.1) もう一度 TrySpin() でスピンロックしてみる. 
        このスピンロックが成功したら, ここでループから抜ける
        (ただし, この最適化は Knob_SpinAfterFutile がセットされている場合にのみ行われる)
  
        (なお, TrySpin とは ObjectMonitor::TrySpin_VaryDuration() のこと.
         See: Adaptive Spinning)
  
        (なお, コメントによると, 
         spurious wakeup でなければ, この時点で _succ がカレントスレッドを指しているはず.
         そして, _succ をリセットするのはスピンロックしてからでも問題ないはず. 
         とのこと)
        ---------------------------------------- -}

	        // Assuming this is not a spurious wakeup we'll normally find _succ == Self.
	        // We can defer clearing _succ until after the spin completes
	        // TrySpin() must tolerate being called with _succ == Self.
	        // Try yet another round of adaptive spinning.
	        if ((Knob_SpinAfterFutile & 1) && TrySpin (Self) > 0) break ;
	
    {- -------------------------------------------
  (1.1) 上のスピンを行っている間に, カレントスレッドに対して unpark() が呼ばれたかもしれない.
         Knob_ResetEvent がセットされている場合は, この unpark() についてはノーカウントとする.
         (os::PlatformEvent::reset() により, クリアしておく)
  
         (なお, クリアしなくてもアルゴリズム上の支障は無い.
          ループの次の周で park() が即座に終了し, すぐにまたスピンが始まるだけ.
          失敗したスピンの直後のスピンということで, ロックが取れる可能性は低いけど.)
  
         (<= 逆に, ノーカンにして大丈夫なのか(stranding しないか), とも思ったが
          スピンが失敗したと言うことは誰かがロックを持っているわけだから
          そいつが unlock() するタイミングで誰かを起こすから問題ない(?) #TODO)
        ---------------------------------------- -}

	        // We can find that we were unpark()ed and redesignated _succ while
	        // we were spinning.  That's harmless.  If we iterate and call park(),
	        // park() will consume the event and return immediately and we'll
	        // just spin again.  This pattern can repeat, leaving _succ to simply
	        // spin on a CPU.  Enable Knob_ResetEvent to clear pending unparks().
	        // Alternately, we can sample fired() here, and if set, forgo spinning
	        // in the next iteration.
	
	        if ((Knob_ResetEvent & 1) && Self->_ParkEvent->fired()) {
	           Self->_ParkEvent->reset() ;
	           OrderAccess::fence() ;
	        }

    {- -------------------------------------------
  (1.1) _succ が自分を指していたら, NULL に戻しておく.
        (なお, _succ を NULL に戻した場合には, 
         眠りにつく前に _owner の確保を試みておかないといけない, という内部規則があることに注意.
         順番も大事なので, OrderAccess::fence() によりメモリバリアも張っている.)
        ---------------------------------------- -}

	        if (_succ == Self) _succ = NULL ;
	
	        // Invariant: after clearing _succ a thread *must* retry _owner before parking.
	        OrderAccess::fence() ;
	    }
	
  {- -------------------------------------------
  (1) (これ以降は, 無事にロックが取れた後の処理.
       後片付け的な処理を行う(cxq や EntryList からカレントスレッドを外す, 等).
  
       なお, 通常のケースではカレントスレッドは EntryList 内にいるはず.
       また, (ロックを握っている)カレントスレッドから見ると, 
       EntryList は不変であり, cxq も最後に要素が追加されるだけで途中の部分は不変.
       また, Self.TState も不変.)
      ---------------------------------------- -}

	    // Egress :
	    // Self has acquired the lock -- Unlink Self from the cxq or EntryList.
	    // Normally we'll find Self on the EntryList .
	    // From the perspective of the lock owner (this thread), the
	    // EntryList is stable and cxq is prepend-only.
	    // The head of cxq is volatile but the interior is stable.
	    // In addition, Self.TState is stable.
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert (_owner == Self      , "invariant") ;
	    assert (object() != NULL    , "invariant") ;
	    // I'd like to write:
	    //   guarantee (((oop)(object()))->mark() == markOopDesc::encode(this), "invariant") ;
	    // but as we're at a safepoint that's not safe.
	
  {- -------------------------------------------
  (1) ロックは取れたが, カレントスレッドに対応する ObjectWaiter が待ちキュー内に残ったままなので, 
      ObjectMonitor::UnlinkAfterAcquire() を呼んで除去しておく.
      ---------------------------------------- -}

	    UnlinkAfterAcquire (Self, &node) ;

  {- -------------------------------------------
  (1) _succ が自分を指していたら, NULL に戻しておく.
      ---------------------------------------- -}

	    if (_succ == Self) _succ = NULL ;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert (_succ != Self, "invariant") ;

  {- -------------------------------------------
  (1) もし _Responsible が自分を指していたら, NULL に戻しておく.
  
      (なお, コメントではこの後ろに OrderAccess::storeload() を入れた方がいいかも, と書かれている.
  
       _Responsible への書き込みと cxq|EntryList の読み込みは
       デッカーのアルゴリズム式の排他に使うため順序づけられてないといけない.
       * このスレッドは以下の順で処理を実行する.
          ST Responsible=null; MEMBAR             (in enter epilog - here)
          LD cxq|EntryList                        (in subsequent exit)
       * 一方, 新たに待ちキューに追加されるスレッドは, 以下の順で処理を実行する.
          ST cxq=nonnull; MEMBAR; LD Responsible  (in enter prolog)
          (なお, この (ST cxq; MEMBAR) は CAS で実行される)
  
       Responsible の変更後にメモリバリアを入れることで, 
       exit 時の load が Responsible の変更を追い抜くことを防ぐ必要がある.
  
       ただし, 現実的にはこの後の処理 (ObjectMonitor::enter() 内の処理とか) でアトミック操作を使うので, 
       そこで勝手にメモリバリアが張られる.
       このため, ここで明示的にバリアを入れておく必要性は薄い.)
      ---------------------------------------- -}

	    if (_Responsible == Self) {
	        _Responsible = NULL ;
	        // Dekker pivot-point.
	        // Consider OrderAccess::storeload() here
	
	        // We may leave threads on cxq|EntryList without a designated
	        // "Responsible" thread.  This is benign.  When this thread subsequently
	        // exits the monitor it can "see" such preexisting "old" threads --
	        // threads that arrived on the cxq|EntryList before the fence, above --
	        // by LDing cxq|EntryList.  Newly arrived threads -- that is, threads
	        // that arrive on cxq after the ST:MEMBAR, above -- will set Responsible
	        // non-null and elect a new "Responsible" timer thread.
	        //
	        // This thread executes:
	        //    ST Responsible=null; MEMBAR    (in enter epilog - here)
	        //    LD cxq|EntryList               (in subsequent exit)
	        //
	        // Entering threads in the slow/contended path execute:
	        //    ST cxq=nonnull; MEMBAR; LD Responsible (in enter prolog)
	        //    The (ST cxq; MEMBAR) is accomplished with CAS().
	        //
	        // The MEMBAR, above, prevents the LD of cxq|EntryList in the subsequent
	        // exit operation from floating above the ST Responsible=null.
	        //
	        // In *practice* however, EnterI() is always followed by some atomic
	        // operation such as the decrement of _count in ::enter().  Those atomics
	        // obviate the need for the explicit MEMBAR, above.
	    }
	
  {- -------------------------------------------
  (1) OrderAccess::fence() で, ここにメモリバリアを張っておく.
      (なお, この最適化は SyncFlags の 4bit目(8)が立っている場合にのみ行われる)
      
      (なおコメントによると, ここでメモリバリアを張るのは以下のような理由から, とのこと.
  
       ロック確保に使用した CAS はメモリバリアの効果も併せ持つが, 
       それ以降に行われた (_succ とか EntryList とか cxq とか Responsible といったメタデータへの) 書き込みも 
       ロックを解放する前に他のスレッドに visible にならないとまずい.
       もしそうならない場合, 次のスレッドがロックを確保に来た際に古いメタデータが見えてしまう恐れがあるので.
  
       これを防ぐには, exit() でロックを解放する際に STST|LDST のメモリバリアを張っておいてもいいのだが, 
       1-0 model においては exit 時にシリアライズ命令を出さないことがある.)
      ---------------------------------------- -}

	    // We've acquired ownership with CAS().
	    // CAS is serializing -- it has MEMBAR/FENCE-equivalent semantics.
	    // But since the CAS() this thread may have also stored into _succ,
	    // EntryList, cxq or Responsible.  These meta-data updates must be
	    // visible __before this thread subsequently drops the lock.
	    // Consider what could occur if we didn't enforce this constraint --
	    // STs to monitor meta-data and user-data could reorder with (become
	    // visible after) the ST in exit that drops ownership of the lock.
	    // Some other thread could then acquire the lock, but observe inconsistent
	    // or old monitor meta-data and heap data.  That violates the JMM.
	    // To that end, the 1-0 exit() operation must have at least STST|LDST
	    // "release" barrier semantics.  Specifically, there must be at least a
	    // STST|LDST barrier in exit() before the ST of null into _owner that drops
	    // the lock.   The barrier ensures that changes to monitor meta-data and data
	    // protected by the lock will be visible before we release the lock, and
	    // therefore before some other thread (CPU) has a chance to acquire the lock.
	    // See also: http://gee.cs.oswego.edu/dl/jmm/cookbook.html.
	    //
	    // Critically, any prior STs to _succ or EntryList must be visible before
	    // the ST of null into _owner in the *subsequent* (following) corresponding
	    // monitorexit.  Recall too, that in 1-0 mode monitorexit does not necessarily
	    // execute a serializing instruction.
	
	    if (SyncFlags & 8) {
	       OrderAccess::fence() ;
	    }

  {- -------------------------------------------
  (1) リターンする
      ---------------------------------------- -}

	    return ;
	}
	
```


