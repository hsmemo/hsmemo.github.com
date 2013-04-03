---
layout: default
title: Thread の待機処理の枠組み ： SpinLock, Mux による処理
---
[Up](noIpUCxk3g.html) [Top](../index.html)

#### Thread の待機処理の枠組み ： SpinLock, Mux による処理

--- 
## 概要(Summary)
これらは ParkEvent を用いて構築されたシンプルなスピンロック, 及び Mutex 実装.
ごく短い critical section に対してのみ使われることになっている.

具体的には, 以下の 2種類のロック/アンロック関数が用意されている.

  * Thread::SpinAcquire(), Thread::SpinRelease()

  * Thread::muxAcquire(), Thread::muxRelease()

(#Under Construction)


```
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
    // Internal SpinLock and Mutex
    // Based on ParkEvent
    
    // Ad-hoc mutual exclusion primitives: SpinLock and Mux
    //
    // We employ SpinLocks _only for low-contention, fixed-length
    // short-duration critical sections where we're concerned
    // about native mutex_t or HotSpot Mutex:: latency.
    // The mux construct provides a spin-then-block mutual exclusion
    // mechanism.
    //
    // Testing has shown that contention on the ListLock guarding gFreeList
    // is common.  If we implement ListLock as a simple SpinLock it's common
    // for the JVM to devolve to yielding with little progress.  This is true
    // despite the fact that the critical sections protected by ListLock are
    // extremely short.
    //
    // TODO-FIXME: ListLock should be of type SpinLock.
    // We should make this a 1st-class type, integrated into the lock
    // hierarchy as leaf-locks.  Critically, the SpinLock structure
    // should have sufficient padding to avoid false-sharing and excessive
    // cache-coherency traffic.
```

### Thread::SpinAcquire(), Thread::SpinRelease()
引数で指定された int 型のメモリ領域を mutex として使用する.

* 0 から 1 に書き換えるとロックを取得したことになる. 逆に 1 から 0 に戻すとロックを解放したことになる.

* ロックが取れなかった場合はスピンロックして待機する.
  ただし, あまり長期間取れないようなら ParkEvent による待機に切り替える.

### Thread::muxAcquire(), Thread::muxRelease()
引数で指定された intptr_t 型のメモリ領域を mutex として使用する.

* 値の最下位ビット(LSB)が立っていれば「ロックされている状態」を意味する.

  最下位ビット以外のビットは, ロック待ちしているスレッドのポインタを格納する.

* ロックが取れなかった場合は ParkEvent による待機を行う.
  
  (なお, 現在の実装では, Thread オブジェクトの _MuxEvent フィールドにある ParkEvent オブジェクトを使用している)

* これらの関数は leaf lock として使用しないといけない.

* これらの関数はスレッドの状態を変更しない (= thread state transitions を行わない) ので, 
  短時間しか保持されないロック以外には使ってはいけない.
  (でないと safepoint 処理に支障をきたす)

* spurious wake-up を防ぐために, 
  ParkEvent オブジェクトの OnList というフィールドを
  「ロック待ちに入ったスレッドが起床していいかどうか」の判断に使っている.
  
  Thread::muxAcquire() で寝る前に OnList を非ゼロにしておき,
  Thread::muxRelease() で起こした対象の OnList をゼロにクリアする.
  起きたスレッドは, OnList が非ゼロのままなら再び眠りにつく.

* 別の実装方針としては Taura-Oyama-Yonezawa locks (田浦-大山-米澤ロック) を使うのもよい (<= コメント中では "Yonenzawa" になっているが Yonezawa の typo).

* ... (#Under Construction)


```
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
    // muxAcquire and muxRelease:
    //
    // *  muxAcquire and muxRelease support a single-word lock-word construct.
    //    The LSB of the word is set IFF the lock is held.
    //    The remainder of the word points to the head of a singly-linked list
    //    of threads blocked on the lock.
    //
    // *  The current implementation of muxAcquire-muxRelease uses its own
    //    dedicated Thread._MuxEvent instance.  If we're interested in
    //    minimizing the peak number of extant ParkEvent instances then
    //    we could eliminate _MuxEvent and "borrow" _ParkEvent as long
    //    as certain invariants were satisfied.  Specifically, care would need
    //    to be taken with regards to consuming unpark() "permits".
    //    A safe rule of thumb is that a thread would never call muxAcquire()
    //    if it's enqueued (cxq, EntryList, WaitList, etc) and will subsequently
    //    park().  Otherwise the _ParkEvent park() operation in muxAcquire() could
    //    consume an unpark() permit intended for monitorenter, for instance.
    //    One way around this would be to widen the restricted-range semaphore
    //    implemented in park().  Another alternative would be to provide
    //    multiple instances of the PlatformEvent() for each thread.  One
    //    instance would be dedicated to muxAcquire-muxRelease, for instance.
    //
    // *  Usage:
    //    -- Only as leaf locks
    //    -- for short-term locking only as muxAcquire does not perform
    //       thread state transitions.
    //
    // Alternatives:
    // *  We could implement muxAcquire and muxRelease with MCS or CLH locks
    //    but with parking or spin-then-park instead of pure spinning.
    // *  Use Taura-Oyama-Yonenzawa locks.
    // *  It's possible to construct a 1-0 lock if we encode the lockword as
    //    (List,LockByte).  Acquire will CAS the full lockword while Release
    //    will STB 0 into the LockByte.  The 1-0 scheme admits stranding, so
    //    acquiring threads use timers (ParkTimed) to detect and recover from
    //    the stranding window.  Thread/Node structures must be aligned on 256-byte
    //    boundaries by using placement-new.
    // *  Augment MCS with advisory back-link fields maintained with CAS().
    //    Pictorially:  LockWord -> T1 <-> T2 <-> T3 <-> ... <-> Tn <-> Owner.
    //    The validity of the backlinks must be ratified before we trust the value.
    //    If the backlinks are invalid the exiting thread must back-track through the
    //    the forward links, which are always trustworthy.
    // *  Add a successor indication.  The LockWord is currently encoded as
    //    (List, LOCKBIT:1).  We could also add a SUCCBIT or an explicit _succ variable
    //    to provide the usual futile-wakeup optimization.
    //    See RTStt for details.
    // *  Consider schedctl.sc_nopreempt to cover the critical section.
    //
```


## 処理の流れ (概要)(Execution Flows : Summary)
### Thread::SpinAcquire()
```
Thread::SpinAcquire()
-> Atomic::cmpxchg() と待機を繰り返す.
   待機処理では以下のどれかが呼び出される.
   * os::NakedYield()
   * os::PlatformEvent::park()
   * SpinPause())
```

### Thread::SpinRelease()
```
Thread::SpinRelease()
-> OrderAccess::fence() でメモリバリアを張った後,
   引数(adr)で指定された箇所を 0 で上書きする(= ロックを解放する)だけ
```

### Thread::muxAcquire()
```
Thread::muxAcquire()
-> Atomic::cmpxchg_ptr() によるスピンロックと
   os::PlatformEvent::park() による待機が繰り返される
```

### Thread::muxRelease()
```
Thread::muxRelease()
-> Atomic::cmpxchg_ptr()
-> os::PlatformEvent::unpark()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### Thread::SpinAcquire()
See: [here](no24825Eux.html) for details
### os::NakedYield()  (Linux の場合)
See: [here](no24825QMN.html) for details
### os::NakedYield()  (Solaris の場合)
See: [here](no24825dWT.html) for details
### os::NakedYield()  (Windows の場合)
See: [here](no2114ZRr.html) for details
### SpinPause()  (Linux x86-32 の場合)
See: [here](no24825QTB.html) for details
### SpinPause()  (Linux x86-64 の場合)
See: [here](no24825R_r.html) for details
### SpinPause()  (Linux Sparc の場合)
(#Under Construction)

### SpinPause()  (Linux zero の場合)
See: [here](no24825ddH.html) for details
### SpinPause()  (Solaris Sparc の場合)
See: [here](no248253qf.html) for details
### SpinPause()  (Solaris x86 の場合)
(#Under Construction)

### SpinPause()  (Windows x86 の場合)
See: [here](no24825E1l.html) for details

### Thread::SpinRelease()
See: [here](no24825qnN.html) for details

### Thread::muxAcquire()
See: [here](no24825E8Z.html) for details

### Thread::muxAcquireW()
(#Under Construction)
(どこからも使われてなさそうだが...)


### Thread::muxRelease()
See: [here](no24825RGg.html) for details






