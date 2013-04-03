---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/objectMonitor.cpp
### 説明(description)
コメントによると, 
「ReenterI() は EnterI() の contended slow-path の後半部分をコピーしたもの.
  現状では, この関数は wait() 処理でロックを再取得する際にのみ使用されている.
  そのうち EnterI() と一緒にリファクタリングする予定.
  その際には, Knob_Reset や Knob_SpinAfterFutile 用のコードも入れるようにしたい.」
とのこと.

```
// ReenterI() is a specialized inline form of the latter half of the
// contended slow-path from EnterI().  We use ReenterI() only for
// monitor reentry in wait().
//
// In the future we should reconcile EnterI() and ReenterI(), adding
// Knob_Reset and Knob_SpinAfterFutile support and restructuring the
// loop accordingly.

```

### 名前(function name)
```
void ATTR ObjectMonitor::ReenterI (Thread * Self, ObjectWaiter * SelfNode) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert (Self != NULL                , "invariant") ;
	    assert (SelfNode != NULL            , "invariant") ;
	    assert (SelfNode->_thread == Self   , "invariant") ;
	    assert (_waiters > 0                , "invariant") ;
	    assert (((oop)(object()))->mark() == markOopDesc::encode(this) , "invariant") ;
	    assert (((JavaThread *)Self)->thread_state() != _thread_blocked, "invariant") ;

  {- -------------------------------------------
  (1) (変数宣言など)
      (nWakeups の方は, 意味のある使われ方をしていない気もするが... #TODO)
      ---------------------------------------- -}

	    JavaThread * jt = (JavaThread *) Self ;
	
	    int nWakeups = 0 ;

  {- -------------------------------------------
  (1) (以下の for ループ内で, 実際にロックを確保する処理を行う.
       ロックが取れるまで, このループからは出ない.)
      ---------------------------------------- -}

	    for (;;) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	        ObjectWaiter::TStates v = SelfNode->TState ;

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	        guarantee (v == ObjectWaiter::TS_ENTER || v == ObjectWaiter::TS_CXQ, "invariant") ;
	        assert    (_owner != Self, "invariant") ;
	
    {- -------------------------------------------
  (1.1) まずは ObjectMonitor::TryLock() でロック確保を試みる.
        ロックが取れたら, ここでループから抜ける.
        ---------------------------------------- -}

	        if (TryLock (Self) > 0) break ;

    {- -------------------------------------------
  (1.1) 次に, TrySpin() でスピンロックしてみる. 
        スピンロックが成功したら, ここでループから抜ける.
  
        (なお, TrySpin とは ObjectMonitor::TrySpin_VaryDuration() のこと.
         See: Adaptive Spinning)
        ---------------------------------------- -}

	        if (TrySpin (Self) > 0) break ;
	
    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	        TEVENT (Wait Reentry - parking) ;
	
    {- -------------------------------------------
  (1.1) ThreadBlockInVM 及び OSThreadContendState で, JavaThread/OSThread の状態を変更した後, 
        _ParkEvent フィールドに対して os::PlatformEvent::park() を呼ぶことで眠りにつく.
  
        なお, 条件によって呼び出す park() が少し変わる (See: 1-0 model).
        *  SyncFlags の 1bit目(1)が立っている場合         (<= これはどういうこと?? #TODO)
          (誰かが起こしてくれなくても最悪自力で起きてこないといけないので(??))
          タイムアウト付きの os::PlatformEvent::park() を呼び出す.
          タイムアウト時間は 1000 で固定.
        * 上記以外の場合
          タイムアウト無しの os::PlatformEvent::park() を呼び出す.
  
        (なお, 眠っている間に suspend された場合のために, 
         os::PlatformEvent::park() から返ってきた後で 
         ObjectMonitor::ExitSuspendEquivalent() による確認を行っている.
  
         suspend されていた場合は, JavaThread::java_suspend_self() を呼んで
         サスペンドが解除されるまで再び眠りにつく.)
  
  
        (なおコメントによると, 
         ReenterI() ではぎりぎりまでスレッドの状態変更を遅らせることが出来ている, 
         とのこと.
         スレッドの状態変更処理はそれなりに重いので, これは ReenterI() の効率が良いと言うこと.)
  
  
        #TODO  JavaThread::set_suspend_equivalent() はどういう意味がある??
        ---------------------------------------- -}

	        // State transition wrappers around park() ...
	        // ReenterI() wisely defers state transitions until
	        // it's clear we must park the thread.
	        {
	           OSThreadContendState osts(Self->osthread());
	           ThreadBlockInVM tbivm(jt);
	
	           // cleared by handle_special_suspend_equivalent_condition()
	           // or java_suspend_self()
	           jt->set_suspend_equivalent();
	           if (SyncFlags & 1) {
	              Self->_ParkEvent->park ((jlong)1000) ;
	           } else {
	              Self->_ParkEvent->park () ;
	           }
	
	           // were we externally suspended while we were waiting?
	           for (;;) {
	              if (!ExitSuspendEquivalent (jt)) break ;
	              if (_succ == Self) { _succ = NULL; OrderAccess::fence(); }
	              jt->java_suspend_self();
	              jt->set_suspend_equivalent();
	           }
	        }
	
    {- -------------------------------------------
  (1.1) 起きてきたら, ObjectMonitor::TryLock() でロック確保を試みる.
        ロックが取れたら, ここでループから抜ける.
  
        (なおコメントによると, 
         別にこのまま次のループに突入してそこでロック確保してもいいんだが, 
         一応ここでロック確保を試みておくことで
         futile wakeup に関する統計情報をより正確に保てる, 
         とのこと.)
        ---------------------------------------- -}

	        // Try again, but just so we distinguish between futile wakeups and
	        // successful wakeups.  The following test isn't algorithmically
	        // necessary, but it helps us maintain sensible statistics.
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

	        TEVENT (Wait Reentry - futile wakeup) ;

    {- -------------------------------------------
  (1.1) ?? (nWakeups は, この後どこからも参照されていないが...)
        ---------------------------------------- -}

	        ++ nWakeups ;
	
    {- -------------------------------------------
  (1.1) _succ が自分を指していたら, NULL に戻しておく.
        (なお, コメントによると, 
         spurious wakeup でなければ, この時点で _succ がカレントスレッドを指しているはず.
         とのこと)
  
        (なお, _succ を NULL に戻した場合には, 
         眠りにつく前に _owner の確保を試みておかないといけない, という内部規則があることに注意.
         順番も大事なので, OrderAccess::fence() によりメモリバリアも張っている.)
        ---------------------------------------- -}

	        // Assuming this is not a spurious wakeup we'll normally
	        // find that _succ == Self.
	        if (_succ == Self) _succ = NULL ;
	
	        // Invariant: after clearing _succ a contending thread
	        // *must* retry  _owner before parking.
	        OrderAccess::fence() ;
	
    {- -------------------------------------------
  (1.1) (プロファイル情報の記録) (See: UsePerfData) (See: sun.rt._sync_FutileWakeups)
        ---------------------------------------- -}

	        if (ObjectMonitor::_sync_FutileWakeups != NULL) {
	          ObjectMonitor::_sync_FutileWakeups->inc() ;
	        }
	    }
	
  {- -------------------------------------------
  (1) (これ以降は, 無事にロックが取れた後の処理.
       後片付け的な処理を行う(cxq や EntryList からカレントスレッドを外す, 等).
  
       なお, 通常のケースではカレントスレッドは EntryList 内にいるはず.
       また, (ロックを握っている)カレントスレッドから見ると, 
       EntryList は不変であり, cxq も最後に要素が追加されるだけで途中の部分は不変.
       また, Self.TState も不変.)
      ---------------------------------------- -}

	    // Self has acquired the lock -- Unlink Self from the cxq or EntryList .
	    // Normally we'll find Self on the EntryList.
	    // Unlinking from the EntryList is constant-time and atomic-free.
	    // From the perspective of the lock owner (this thread), the
	    // EntryList is stable and cxq is prepend-only.
	    // The head of cxq is volatile but the interior is stable.
	    // In addition, Self.TState is stable.
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert (_owner == Self, "invariant") ;
	    assert (((oop)(object()))->mark() == markOopDesc::encode(this), "invariant") ;

  {- -------------------------------------------
  (1) ロックは取れたが, カレントスレッドに対応する ObjectWaiter が待ちキュー内に残ったままなので, 
      ObjectMonitor::UnlinkAfterAcquire() を呼んで除去しておく.
      ---------------------------------------- -}

	    UnlinkAfterAcquire (Self, SelfNode) ;

  {- -------------------------------------------
  (1) _succ が自分を指していたら, NULL に戻しておく.
      ---------------------------------------- -}

	    if (_succ == Self) _succ = NULL ;
	    assert (_succ != Self, "invariant") ;

  {- -------------------------------------------
  (1) TState を ObjectWaiter::TS_RUN に戻しておく.
      (この処理は, ObjectMonitor::UnlinkAfterAcquire() 内でもやってるので, 要らない気もするが...)
      ---------------------------------------- -}

	    SelfNode->TState = ObjectWaiter::TS_RUN ;

  {- -------------------------------------------
  (1) OrderAccess::fence() で, ここにメモリバリアを張っておく.
      (この理由については, ObjectMonitor::EnterI() の最後のコメントを参照)
      ---------------------------------------- -}

	    OrderAccess::fence() ;      // see comments at the end of EnterI()
	}
	
```


