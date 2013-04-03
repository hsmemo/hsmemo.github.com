---
layout: default
title: Monitor クラスおよび Mutex クラス (Monitor, Mutex)
---
[Top](../index.html)

#### Monitor クラスおよび Mutex クラス (Monitor, Mutex)

これらは, HotSpot 内でのスレッド制御用のクラス.
より具体的に言うと, スレッド間で排他を取るためのユーティリティ・クラス
(See: [here](no2114cio.html) for details).

### 概要(Summary)
Monitor や Mutex クラスは HotSpot 内部で使用されるロック(Native Monitors)を表すクラス.
以下は使い方や注意点など.

* これらは HotSpot の内部的な処理のための排他を行うクラス.
  Java レベルの monitor とは全く関係ないので, 混同しないように注意.

  (slow-path 処理の実装はよく似た感じにはなっているけど役割としては無関係.
  Java レベルの monitor については ObjectMonitor クラスを参照)

  Native Monitors は, Java レベルの monitor と異なり, 同一スレッドが複数回取得してはいけない.
  それ以外の点では基本的に Hoare-flavor monitors.

* CAS で _LockWord フィールドの LockByte 目を non-zero に書き換えるとロックを取得したことになる.

  (_Owner というフィールドもあるが, これはヒント的なものに過ぎず, 
   unlock() 時の verify 処理でしか使用されないので間違えないように)

* contention が起きていた場合は, ロックを取ろうとしたスレッドは
  自分自身を contention queue (cxq と呼ぶ) に CAS で挿入し, spin/park で待機すること.

  (なお, _LockWord フィールドに LockByte と一緒に 
  contention queue の先頭アドレスも入っている.
  これらを一緒に入れておくことである種の race condition が防げる.)

* LockByte を "separately addressable"#TODO にしたことで,
  ロック/アンロック処理の実装として CAS:MEMBAR もしくは CAS:0 が使えるようになった.
  現状では CAS:MEMBAR (MEMBAR は CAS に比べればレイテンシが短いので).

  もし必要があれば CAS:0 にしてもいい.
  その場合, 結果として生じる race に対しては, Java Monitors と同じようにタイマーでの対処が考えられる.
 
  以下に書いてある議論も参照のこと.

  * http://blogs.sun.com/dave/entry/biased_locking_in_hotspot
  * http://blogs.sun.com/dave/resource/MustangSync.pdf
  * http://blogs.sun.com/dave/resource/synchronization-public2.pdf
  * synchronizer.cpp

* 全体的な目標 (以下の事項をぜひとも実現したい)

   * コンテキストスイッチの最小化
   * ... #TODO
   * ... 

* スレッドは, 以下の順で待ち行列を移動していく

      Contention queue --> EntryList --> OnDeck --> Owner --> !Owner
      [..resident on monitor list..]
      [...........contending..................]

  * contention queue (cxq) には最近到着したスレッド (recently-arrived threads, RATs) が格納されている.
    この中のスレッドは, 最終的には EntryList に移動させられる.
  * どんな場合にでも, スレッドは cxq, EntryList, WaitSet の複数に同時に存在することはない
  * どの Monitor も, 最大1つの "OnDeck" スレッドを持つ
    (ただし, もし必要があればこの条件は緩和してもいい).

* WaitSet と EntryList は, Linked List であり, ParkEvents オブジェクトで構成されている.
  これは ParkEvents オブジェクトが immortal かつ type-stable だから.
  (これは, 既に寿命が尽きているかもしれない要素に対して, 
  unpark() 処理中に安全に unpark() を呼び出せる, ということを意味する)

* "Succession policy" について

  unlock() を呼び出したスレッドが, 
  EntryList 中の1つを次にロック確保を試みる権利があるスレッド("heir presumptive")として選び, 
  そのスレッドを unpark() で起床させる.
  (これが "OnDeck" thread になる).
   
  なお, ロックを持っているスレッドが直接次のスレッドにロックを渡す("handoff" succession), ということはしない.
  選ばれたスレッドは, 起こしてもらった後, 自力でロックを取りに行く(Competitive handoff).
  
  Competitive handoff は, 全体的には素晴らしいスループットを提供する. 
  ただし, 短期的な公平性は犠牲になる.
  もし公平性が問題になるようであれば, 次のような解決策が考えられる.

  「各 monitor に AcquireCounter フィールドを持たせ, 
  スレッドがロックを取得する度に AcquireCounter フィールドの値をデクリメントしていく.
  値が 0 になったら, そのスレッドはロックを直接 EntryList 内のスレッドに渡し, 
  AcquireCounter フィールドは初期値に戻して, 自分は EntryList の終端に移動する.」

  しかし, 実用上のたいていの場合は, ロックの公平性は問題にならないはず.
  また, 高速なロックから公平なロックは作るのは, その逆を行うよりは簡単.

* cxq には, 同時に複数のスレッドが push されてくるかもしれないが, 一度に1つしか detach 出来ない.
  これは, CAS の ABA 問題に対する対策として働く.
  OnDeck を擬似的なロックとして用いることで, 最大で1つしか detach 出来ない状況を保証している.

* なお, cxq と EntryList は, 2つ合わせて 1つの論理的な待ちキューを構成している.
  この待ちキューには, ロックを取れなかったため待機しているスレッドが並んでいる.

  2つに分けているのは, リスト終端部での競合を減らしたいため.
  lock() するスレッドは cxq に追加され, 一方 unlock() するスレッドは EntryList から取り出される.
  (Michael Scott の "2Q" algorithm)

  重要なのは, queue や monitor のメタデータの操作時間を最小化すること.
  なぜならこの処理は outer monitor lock を取得した状態で行わなければならず, 
  これが長いことは critical section の増加を意味する.

  EntryList はキューとして使用する. 
  実装方法は色々考えられる (doubly-linked list とか circular doubly-linked list とか).
  もし優先順位付きのキューが欲しければ Solaris の sleepq みたいなのがいいかも. 以下参照.

  * http://agg.eng/ws/on10_nightly/source/usr/src/uts/common/os/sleepq.c.
  * http://cvs.opensolaris.org/source/xref/onnv/onnv-gate/usr/src/uts/common/os/sleepq.c

  unlock() 時にキューであることを保証する (このときに cxq の中身を EntryList に追加し, 順序を適切に整える).

  ... #TODO
  ("Barring "lock barging", this mechanism provides fair cyclic ordering,
   somewhat similar to an elevator-scan.")

* OnDeck について

    * 全ての Monitor は, OnDeck スレッドを同時に最大 1つまで持つことが出来る.

      なお, OnDeck スレッドになっているスレッドは
      ロックを取ろうとしている途中のスレッドだが, cxq や EntryList からはもう外されている状態.
      (スレッドを cxq や EntryList から外して OnDeck スレッドに指定する処理は unlock() 時に行われる)
      なお, 一度 OnDeck スレッドになると, ロックを取得できるまではずっと OnDeck スレッドで有り続ける.

    * スレッドは, cxq や EntryList にいる間は, ロック確保を試みることが許されない.

    * OnDeck は "inner lock" としても働く. 

      unlock() 処理においては, 
      まず LockByte がクリアされ outer lock が解放される.
      その後, CAS 操作で OnDeck をロックする処理が行われる.

      これが成功すると, cxq の中身を EntryList に追加し, 
      OnDeck をロックしている間のみ, EntryList を操作したり, cxq の中身を EntryList に追加することが許される
      (これにより ABA 問題を防げる).

      その後, 次にロック確保を試みるスレッドが選ばれ, OnDeck にそのポインタが格納され, そしてそのスレッドが unpark() される.
      選ばれたスレッドは, 最終的にはロックを取得し, OnDeck をクリアする.

      (なお, OnDeck をロックとしてみると, unlock() 処理をしたスレッドがロックを確保し, 
      OnDeck スレッドとして選ばれたスレッドがロックを解放することになる. 
      少し対称性がないので注意.
      また, inner lock については, 決して競合は起こらない (inner lock 取得のためにロック待ちしたりすることはあり得ない) ことにも注意.)

    * OnDeck は futile wakeup を防止する役割もある. 

      (次の資料の 3.3 を参照.
      http://www.usenix.org/events/jvm01/full_papers/dice/dice.pdf)

      (あるいは, ObjectMonitor の _succ を参照)
      (または ObjectWaiter の TState を参照)

* wait() で待機しているスレッドは, WaitSet リスト中に記録されている.

  Notify() や notifyAll() の処理は, 単にスレッドを WaitSet リストから cxq や EntryList に移すだけ
  (その後の unlock() 操作で unpark() されるはず).
  notify() の中で unpark() されるのは効率が悪い.

* このロック実装は obstruction-free になっている.

  つまり, unlock() 処理中で OnDeck (inner lock) をロックしたまま descheduling されたとしても, 
  他のスレッドは問題なくロックを取得できる.

* なお, Monitor や Mutex を使う前に, 各スレッドの ThreadLocalStorage は初期化済みでないといけない.
  というのは, 内部処理で Thread::current() を使用するので.
  
* Monitor の実装内では, プラットフォーム依存な park/unpark の層以外では, ネイティブで用意されている同期機構を使わずに済ませている.
  os_solaris.cpp 内のコメントも参照.
  この Monitor 実装が RUNNING->BLOCKED 及び BLOCKED->READY の状態遷移を扱い, 
  OS の層には READY<->RUN の状態遷移を任せている.

* なお, memory ordering に対しては, 
  lock()/unlock() 操作は Java Memory Model (JSR-133) と同等かそれ以上に強い性質を保証する.

* 現状の Thread オブジェクトでは, 特定の用途に特化した ParkEvent オブジェクトを保持している
  (_MutexEvent, _ParkEvent, 等).
  よりよい実装としては, 
  ...#TODO

* 少し軽量にしたバージョンの ILock() や IUnlock() の実装については, その一部について
  T=1,2,3,4 の範囲で Murphi による(安全性と progress の)モデルチェックを行った (#TODO)
  TLA/TLC でのモデルチェッキングも面白いかもしれない.

* Mutex や Monitor は "leaf" subsystem でなければいけない.

  これはつまり, Mutex や Monitor の中から JVM の他の機能を呼び出してはいけない, という意味.
  なぜなら, その中で Mutex や Monitor を取得する可能性があるから.

  ただし, ThreadBlockInVM だけは例外とする.
  ThreadBlockInVM ではデストラクタで.... (という説明があるが, 現状の実装とあっているか??? #TODO)
  
  Mutex や Monitor とスレッドの状態遷移は別の物であって, 混同するのはよくない.
  現状の実装では, これらが一緒くたになってしまっている.

* TODO-FIXME:

  * DTrace のプローブを入れたい (と書いてあるが, これはもう入っているような..?? #TODO)
  * JVM のコード内には mutex のような働きをするものが多すぎる.

      * objectMonitors for Java-level synchronization (synchronizer.cpp)
      * low-level muxAcquire and muxRelease
      * low-level spinAcquire and spinRelease
      * native Mutex:: and Monitor::
      * jvm_raw_lock() and _unlock()
      * JVMTI raw monitors -- distinct from (5) despite having a confusingly similar name.

```
    ((cite: hotspot/src/share/vm/runtime/mutex.cpp))
    // o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o
    //
    // Native Monitor-Mutex locking - theory of operations
    //
    // * Native Monitors are completely unrelated to Java-level monitors,
    //   although the "back-end" slow-path implementations share a common lineage.
    //   See objectMonitor:: in synchronizer.cpp.
    //   Native Monitors do *not* support nesting or recursion but otherwise
    //   they're basically Hoare-flavor monitors.
    //
    // * A thread acquires ownership of a Monitor/Mutex by CASing the LockByte
    //   in the _LockWord from zero to non-zero.  Note that the _Owner field
    //   is advisory and is used only to verify that the thread calling unlock()
    //   is indeed the last thread to have acquired the lock.
    //
    // * Contending threads "push" themselves onto the front of the contention
    //   queue -- called the cxq -- with CAS and then spin/park.
    //   The _LockWord contains the LockByte as well as the pointer to the head
    //   of the cxq.  Colocating the LockByte with the cxq precludes certain races.
    //
    // * Using a separately addressable LockByte allows for CAS:MEMBAR or CAS:0
    //   idioms.  We currently use MEMBAR in the uncontended unlock() path, as
    //   MEMBAR often has less latency than CAS.  If warranted, we could switch to
    //   a CAS:0 mode, using timers to close the resultant race, as is done
    //   with Java Monitors in synchronizer.cpp.
    //
    //   See the following for a discussion of the relative cost of atomics (CAS)
    //   MEMBAR, and ways to eliminate such instructions from the common-case paths:
    //   -- http://blogs.sun.com/dave/entry/biased_locking_in_hotspot
    //   -- http://blogs.sun.com/dave/resource/MustangSync.pdf
    //   -- http://blogs.sun.com/dave/resource/synchronization-public2.pdf
    //   -- synchronizer.cpp
    //
    // * Overall goals - desiderata
    //   1. Minimize context switching
    //   2. Minimize lock migration
    //   3. Minimize CPI -- affinity and locality
    //   4. Minimize the execution of high-latency instructions such as CAS or MEMBAR
    //   5. Minimize outer lock hold times
    //   6. Behave gracefully on a loaded system
    //
    // * Thread flow and list residency:
    //
    //   Contention queue --> EntryList --> OnDeck --> Owner --> !Owner
    //   [..resident on monitor list..]
    //   [...........contending..................]
    //
    //   -- The contention queue (cxq) contains recently-arrived threads (RATs).
    //      Threads on the cxq eventually drain into the EntryList.
    //   -- Invariant: a thread appears on at most one list -- cxq, EntryList
    //      or WaitSet -- at any one time.
    //   -- For a given monitor there can be at most one "OnDeck" thread at any
    //      given time but if needbe this particular invariant could be relaxed.
    //
    // * The WaitSet and EntryList linked lists are composed of ParkEvents.
    //   I use ParkEvent instead of threads as ParkEvents are immortal and
    //   type-stable, meaning we can safely unpark() a possibly stale
    //   list element in the unlock()-path.  (That's benign).
    //
    // * Succession policy - providing for progress:
    //
    //   As necessary, the unlock()ing thread identifies, unlinks, and unparks
    //   an "heir presumptive" tentative successor thread from the EntryList.
    //   This becomes the so-called "OnDeck" thread, of which there can be only
    //   one at any given time for a given monitor.  The wakee will recontend
    //   for ownership of monitor.
    //
    //   Succession is provided for by a policy of competitive handoff.
    //   The exiting thread does _not_ grant or pass ownership to the
    //   successor thread.  (This is also referred to as "handoff" succession").
    //   Instead the exiting thread releases ownership and possibly wakes
    //   a successor, so the successor can (re)compete for ownership of the lock.
    //
    //   Competitive handoff provides excellent overall throughput at the expense
    //   of short-term fairness.  If fairness is a concern then one remedy might
    //   be to add an AcquireCounter field to the monitor.  After a thread acquires
    //   the lock it will decrement the AcquireCounter field.  When the count
    //   reaches 0 the thread would reset the AcquireCounter variable, abdicate
    //   the lock directly to some thread on the EntryList, and then move itself to the
    //   tail of the EntryList.
    //
    //   But in practice most threads engage or otherwise participate in resource
    //   bounded producer-consumer relationships, so lock domination is not usually
    //   a practical concern.  Recall too, that in general it's easier to construct
    //   a fair lock from a fast lock, but not vice-versa.
    //
    // * The cxq can have multiple concurrent "pushers" but only one concurrent
    //   detaching thread.  This mechanism is immune from the ABA corruption.
    //   More precisely, the CAS-based "push" onto cxq is ABA-oblivious.
    //   We use OnDeck as a pseudo-lock to enforce the at-most-one detaching
    //   thread constraint.
    //
    // * Taken together, the cxq and the EntryList constitute or form a
    //   single logical queue of threads stalled trying to acquire the lock.
    //   We use two distinct lists to reduce heat on the list ends.
    //   Threads in lock() enqueue onto cxq while threads in unlock() will
    //   dequeue from the EntryList.  (c.f. Michael Scott's "2Q" algorithm).
    //   A key desideratum is to minimize queue & monitor metadata manipulation
    //   that occurs while holding the "outer" monitor lock -- that is, we want to
    //   minimize monitor lock holds times.
    //
    //   The EntryList is ordered by the prevailing queue discipline and
    //   can be organized in any convenient fashion, such as a doubly-linked list or
    //   a circular doubly-linked list.  If we need a priority queue then something akin
    //   to Solaris' sleepq would work nicely.  Viz.,
    //   -- http://agg.eng/ws/on10_nightly/source/usr/src/uts/common/os/sleepq.c.
    //   -- http://cvs.opensolaris.org/source/xref/onnv/onnv-gate/usr/src/uts/common/os/sleepq.c
    //   Queue discipline is enforced at ::unlock() time, when the unlocking thread
    //   drains the cxq into the EntryList, and orders or reorders the threads on the
    //   EntryList accordingly.
    //
    //   Barring "lock barging", this mechanism provides fair cyclic ordering,
    //   somewhat similar to an elevator-scan.
    //
    // * OnDeck
    //   --  For a given monitor there can be at most one OnDeck thread at any given
    //       instant.  The OnDeck thread is contending for the lock, but has been
    //       unlinked from the EntryList and cxq by some previous unlock() operations.
    //       Once a thread has been designated the OnDeck thread it will remain so
    //       until it manages to acquire the lock -- being OnDeck is a stable property.
    //   --  Threads on the EntryList or cxq are _not allowed to attempt lock acquisition.
    //   --  OnDeck also serves as an "inner lock" as follows.  Threads in unlock() will, after
    //       having cleared the LockByte and dropped the outer lock,  attempt to "trylock"
    //       OnDeck by CASing the field from null to non-null.  If successful, that thread
    //       is then responsible for progress and succession and can use CAS to detach and
    //       drain the cxq into the EntryList.  By convention, only this thread, the holder of
    //       the OnDeck inner lock, can manipulate the EntryList or detach and drain the
    //       RATs on the cxq into the EntryList.  This avoids ABA corruption on the cxq as
    //       we allow multiple concurrent "push" operations but restrict detach concurrency
    //       to at most one thread.  Having selected and detached a successor, the thread then
    //       changes the OnDeck to refer to that successor, and then unparks the successor.
    //       That successor will eventually acquire the lock and clear OnDeck.  Beware
    //       that the OnDeck usage as a lock is asymmetric.  A thread in unlock() transiently
    //       "acquires" OnDeck, performs queue manipulations, passes OnDeck to some successor,
    //       and then the successor eventually "drops" OnDeck.  Note that there's never
    //       any sense of contention on the inner lock, however.  Threads never contend
    //       or wait for the inner lock.
    //   --  OnDeck provides for futile wakeup throttling a described in section 3.3 of
    //       See http://www.usenix.org/events/jvm01/full_papers/dice/dice.pdf
    //       In a sense, OnDeck subsumes the ObjectMonitor _Succ and ObjectWaiter
    //       TState fields found in Java-level objectMonitors.  (See synchronizer.cpp).
    //
    // * Waiting threads reside on the WaitSet list -- wait() puts
    //   the caller onto the WaitSet.  Notify() or notifyAll() simply
    //   transfers threads from the WaitSet to either the EntryList or cxq.
    //   Subsequent unlock() operations will eventually unpark the notifyee.
    //   Unparking a notifee in notify() proper is inefficient - if we were to do so
    //   it's likely the notifyee would simply impale itself on the lock held
    //   by the notifier.
    //
    // * The mechanism is obstruction-free in that if the holder of the transient
    //   OnDeck lock in unlock() is preempted or otherwise stalls, other threads
    //   can still acquire and release the outer lock and continue to make progress.
    //   At worst, waking of already blocked contending threads may be delayed,
    //   but nothing worse.  (We only use "trylock" operations on the inner OnDeck
    //   lock).
    //
    // * Note that thread-local storage must be initialized before a thread
    //   uses Native monitors or mutexes.  The native monitor-mutex subsystem
    //   depends on Thread::current().
    //
    // * The monitor synchronization subsystem avoids the use of native
    //   synchronization primitives except for the narrow platform-specific
    //   park-unpark abstraction.  See the comments in os_solaris.cpp regarding
    //   the semantics of park-unpark.  Put another way, this monitor implementation
    //   depends only on atomic operations and park-unpark.  The monitor subsystem
    //   manages all RUNNING->BLOCKED and BLOCKED->READY transitions while the
    //   underlying OS manages the READY<->RUN transitions.
    //
    // * The memory consistency model provide by lock()-unlock() is at least as
    //   strong or stronger than the Java Memory model defined by JSR-133.
    //   That is, we guarantee at least entry consistency, if not stronger.
    //   See http://g.oswego.edu/dl/jmm/cookbook.html.
    //
    // * Thread:: currently contains a set of purpose-specific ParkEvents:
    //   _MutexEvent, _ParkEvent, etc.  A better approach might be to do away with
    //   the purpose-specific ParkEvents and instead implement a general per-thread
    //   stack of available ParkEvents which we could provision on-demand.  The
    //   stack acts as a local cache to avoid excessive calls to ParkEvent::Allocate()
    //   and ::Release().  A thread would simply pop an element from the local stack before it
    //   enqueued or park()ed.  When the contention was over the thread would
    //   push the no-longer-needed ParkEvent back onto its stack.
    //
    // * A slightly reduced form of ILock() and IUnlock() have been partially
    //   model-checked (Murphi) for safety and progress at T=1,2,3 and 4.
    //   It'd be interesting to see if TLA/TLC could be useful as well.
    //
    // * Mutex-Monitor is a low-level "leaf" subsystem.  That is, the monitor
    //   code should never call other code in the JVM that might itself need to
    //   acquire monitors or mutexes.  That's true *except* in the case of the
    //   ThreadBlockInVM state transition wrappers.  The ThreadBlockInVM DTOR handles
    //   mutator reentry (ingress) by checking for a pending safepoint in which case it will
    //   call SafepointSynchronize::block(), which in turn may call Safepoint_lock->lock(), etc.
    //   In that particular case a call to lock() for a given Monitor can end up recursively
    //   calling lock() on another monitor.   While distasteful, this is largely benign
    //   as the calls come from jacket that wraps lock(), and not from deep within lock() itself.
    //
    //   It's unfortunate that native mutexes and thread state transitions were convolved.
    //   They're really separate concerns and should have remained that way.  Melding
    //   them together was facile -- a bit too facile.   The current implementation badly
    //   conflates the two concerns.
    //
    // * TODO-FIXME:
    //
    //   -- Add DTRACE probes for contended acquire, contended acquired, contended unlock
    //      We should also add DTRACE probes in the ParkEvent subsystem for
    //      Park-entry, Park-exit, and Unpark.
    //
    //   -- We have an excess of mutex-like constructs in the JVM, namely:
    //      1. objectMonitors for Java-level synchronization (synchronizer.cpp)
    //      2. low-level muxAcquire and muxRelease
    //      3. low-level spinAcquire and spinRelease
    //      4. native Mutex:: and Monitor::
    //      5. jvm_raw_lock() and _unlock()
    //      6. JVMTI raw monitors -- distinct from (5) despite having a confusingly
    //         similar name.
    //
    // o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o-o
```

なお, 普通に考えると Monitor が Mutex のサブクラスだが, 現在は逆になっている
(といっても Java SE 7 になるまでは今とは逆だったようだが...).

(Monitor は Mutex に wait/notify/notifyAll 等の機能を付けたもの, という位置づけ.
 現在の Mutex は, ShouldNotReachHere() で明示的に wait/notify/notifyAll を潰した Monitor になっている)

そもそも Monitor と Mutex が分かれているのも歴史的な事情で
Mutex を廃止して Monitor だけにしてもいい, 
とのこと.


```
    ((cite: hotspot/src/share/vm/runtime/mutex.hpp))
    // Normally we'd expect Monitor to extend Mutex in the sense that a monitor
    // constructed from pthreads primitives might extend a mutex by adding
    // a condvar and some extra metadata.  In fact this was the case until J2SE7.
    //
    // Currently, however, the base object is a monitor.  Monitor contains all the
    // logic for wait(), notify(), etc.   Mutex extends monitor and restricts the
    // visiblity of wait(), notify(), and notify_all().
    //
    // Another viable alternative would have been to have Monitor extend Mutex and
    // implement all the normal mutex and wait()-notify() logic in Mutex base class.
    // The wait()-notify() facility would be exposed via special protected member functions
    // (e.g., _Wait() and _Notify()) in Mutex.  Monitor would extend Mutex and expose wait()
    // as a call to _Wait().  That is, the public wait() would be a wrapper for the protected
    // _Wait().
    //
    // An even better alternative is to simply eliminate Mutex:: and use Monitor:: instead.
    // After all, monitors are sufficient for Java-level synchronization.   At one point in time
    // there may have been some benefit to having distinct mutexes and monitors, but that time
    // has past.
    //
    // The Mutex/Monitor design parallels that of Java-monitors, being based on
    // thread-specific park-unpark platform-specific primitives.
```



### クラス一覧(class list)

  * [Monitor](#nonw4C5eL5)
  * [Mutex](#noIGw8MJ_A)


---
## <a name="nonw4C5eL5" id="nonw4C5eL5">Monitor</a>

### 概要(Summary)
スレッド間の排他処理を行うためのユーティリティ・クラス.

Mutex クラスとの違いは, wait/notify/notifyAll() の機能が使用可能な点.

なお, スレッド間の排他処理を行うためのクラスは幾つか存在している (See: [here](noFBHxUsnN.html) for details).
Monitor や Mutex は長い critical section にも使用可能なクラス (See: [here](no2114cio.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/mutex.hpp))
    class Monitor : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 以下の大域変数
  
  * SystemDictionary_lock
  * JNICritical_lock
  * JvmtiPendingEvent_lock
  * Heap_lock
  * VMOperationQueue_lock
  * VMOperationRequest_lock
  * Safepoint_lock
  * SerializePage_lock
  * Threads_lock
  * CGC_lock
  * SLT_lock
  * iCMS_lock
  * FullGCCount_lock
  * CMark_lock
  * SATB_Q_CBL_mon
  * DirtyCardQ_CBL_mon
  * MethodCompileQueue_lock
  * CompileThread_lock
  * Terminator_lock
  * BeforeExit_lock
  * Notify_lock
  * Interrupt_lock
  * ProfileVM_lock
  * ObjAllocPost_lock
  * SecondaryFreeList_lock
  * GCTaskManager_lock
  * Service_lock

* 各 CompileTask オブジェクトの _lock フィールド

* 各 Thread オブジェクトの _SR_lock フィールド

* 各 OSThread オブジェクトの _startThread_lock フィールド

* VMThread クラスの _terminate_lock フィールド (static フィールド)

* 各 SurrogateLockerThread オブジェクトの _monitor フィールド

* 各 SuspendibleThreadSet オブジェクトの _m フィールド
  
* 各 GCTaskManager オブジェクトの _monitor フィールド
  
* 各 AbstractWorkGang オブジェクトの _monitor フィールド
  
* 各 ConcurrentG1RefineThread オブジェクトの _monitor フィールド
  
* 各 SharkCompiler オブジェクトの _execution_engine_lock フィールド

* 各 WorkGangBarrierSync オブジェクトの _monitor フィールド


(なお, mutexLocker.hpp で extern されている大域変数は mutexLocker.cpp で実際に定義されているものより少ない.
 具体的には次のものが含まれていない(SerializePage_lock, ObjAllocPost_lock, GCTaskManager_lock).
 といっても, これらは使われていないので特に問題は無いが.)


```
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
    // Mutexes used in the VM.
    
    ...
    extern Monitor* SystemDictionary_lock;           // a lock on the system dictonary
    ...
    extern Monitor* JNICritical_lock;                // a lock used while entering and exiting JNI critical regions, allows GC to sometimes get in
    ...
    extern Monitor* JvmtiPendingEvent_lock;          // a lock on the JVMTI pending events list
    extern Monitor* Heap_lock;                       // a lock on the heap
    ...
    extern Monitor* VMOperationQueue_lock;           // a lock on queue of vm_operations waiting to execute
    extern Monitor* VMOperationRequest_lock;         // a lock on Threads waiting for a vm_operation to terminate
    extern Monitor* Safepoint_lock;                  // a lock used by the safepoint abstraction
    extern Monitor* Threads_lock;                    // a lock on the Threads table of active Java threads
                                                     // (also used by Safepoints too to block threads creation/destruction)
    extern Monitor* CGC_lock;                        // used for coordination between
                                                     // fore- & background GC threads.
    ...
    extern Monitor* SLT_lock;                        // used in CMS GC for acquiring PLL
    extern Monitor* iCMS_lock;                       // CMS incremental mode start/stop notification
    extern Monitor* FullGCCount_lock;                // in support of "concurrent" full gc
    extern Monitor* CMark_lock;                      // used for concurrent mark thread coordination
    ...
    extern Monitor* SATB_Q_CBL_mon;                  // Protects SATB Q
                                                     // completed buffer queue.
    ...
    extern Monitor* DirtyCardQ_CBL_mon;              // Protects dirty card Q
                                                     // completed buffer queue.
    ...
    extern Monitor* MethodCompileQueue_lock;         // a lock held when method compilations are enqueued, dequeued
    extern Monitor* CompileThread_lock;              // a lock held by compile threads during compilation system initialization
    ...
    extern Monitor* Terminator_lock;                 // a lock used to guard termination of the vm
    extern Monitor* BeforeExit_lock;                 // a lock used to guard cleanups and shutdown hooks
    extern Monitor* Notify_lock;                     // a lock used to synchronize the start-up of the vm
    extern Monitor* Interrupt_lock;                  // a lock used for condition variable mediated interrupt processing
    extern Monitor* ProfileVM_lock;                  // a lock used for profiling the VMThread
    ...
    extern Monitor* SecondaryFreeList_lock;          // protects the secondary free region list
    ...
    extern Monitor* Service_lock;                    // a lock used for service thread operation
```


```
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.cpp))
    // Mutexes used in the VM (see comment in mutexLocker.hpp):
    //
    // Note that the following pointers are effectively final -- after having been
    // set at JVM startup-time, they should never be subsequently mutated.
    // Instead of using pointers to malloc()ed monitors and mutexes we should consider
    // eliminating the indirection and using instances instead.
    // Consider using GCC's __read_mostly.
    
    ...
    Monitor* SystemDictionary_lock        = NULL;
    ...
    Monitor* JNICritical_lock             = NULL;
    ...
    Monitor* JvmtiPendingEvent_lock       = NULL;
    Monitor* Heap_lock                    = NULL;
    ...
    Monitor* VMOperationQueue_lock        = NULL;
    Monitor* VMOperationRequest_lock      = NULL;
    Monitor* Safepoint_lock               = NULL;
    Monitor* SerializePage_lock           = NULL;
    Monitor* Threads_lock                 = NULL;
    Monitor* CGC_lock                     = NULL;
    ...
    Monitor* SLT_lock                     = NULL;
    Monitor* iCMS_lock                    = NULL;
    Monitor* FullGCCount_lock             = NULL;
    Monitor* CMark_lock                   = NULL;
    ...
    Monitor* SATB_Q_CBL_mon               = NULL;
    ...
    Monitor* DirtyCardQ_CBL_mon           = NULL;
    ...
    Monitor* MethodCompileQueue_lock      = NULL;
    Monitor* CompileThread_lock           = NULL;
    ...
    Monitor* Terminator_lock              = NULL;
    Monitor* BeforeExit_lock              = NULL;
    Monitor* Notify_lock                  = NULL;
    Monitor* Interrupt_lock               = NULL;
    Monitor* ProfileVM_lock               = NULL;
    ...
    Monitor* ObjAllocPost_lock            = NULL;
    ...
    Monitor* SecondaryFreeList_lock       = NULL;
    ...
    
    Monitor* GCTaskManager_lock           = NULL;
    
    ...
    Monitor* Service_lock               = NULL;
```

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* Monitor を格納する大域変数の初期化
  
  mutex_init()

* CompileTask::_lock の初期化
  
  CompileTask::CompileTask()

* Thread::_SR_lock の初期化
  
  Thread::Thread()

* OSThread::_startThread_lock の初期化

  OSThread::pd_initialize()

* VMThread::_terminate_lock の初期化
  
  VMThread::create()

* (SurrogateLockerThread クラスの _monitor フィールドは, ポインタ型ではなく実体なので,
  SurrogateLockerThread オブジェクトの生成時に一緒に生成される)

* SuspendibleThreadSet::_m の初期化
  
  SuspendibleThreadSet::initialize_work()

* GCTaskManager::_monitor の初期化
  
  GCTaskManager::initialize()

* WaitForBarrierGCTask が使用する Monitor の確保
  
  MonitorSupply::reserve()

* AbstractWorkGang::_monitor の初期化
  
  AbstractWorkGang::AbstractWorkGang()

* ConcurrentG1RefineThread::_monitor の初期化
  
  ConcurrentG1RefineThread::ConcurrentG1RefineThread()

* SharkCompiler::_execution_engine_lock の初期化
  
  SharkCompiler::SharkCompiler()

* Windows 用の Serviceability Agent のアタッチ処理中
  
  attachToProcess()

* (WorkGangBarrierSync クラスの _monitor フィールドは, ポインタ型ではなく実体なので,
  WorkGangBarrierSync オブジェクトの生成時に一緒に生成される)

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

  * SplitWord _LockWord

    この Monitor オブジェクトのロック状態, およびロック待ちで待機しているスレッドの情報を示す.
    "contention queue" (cxq) と呼ばれているのはこれのこと.

    最近到着したロック待ちスレッド(RATs)が格納されている.

    _LBIT というビットが立っていれば, 誰かがロック済み
    (さらにその場合, _LBIT ビット以外の箇所はロック待ちしているスレッドの ParkEvent オブジェクトのアドレス(ポインタ)を表す模様).
    
    なお, SplitWord は以下のような union 型 (#TODO)
    

```
    ((cite: hotspot/src/share/vm/runtime/mutex.hpp))
    // The SplitWord construct allows us to colocate the contention queue
    // (cxq) with the lock-byte.  The queue elements are ParkEvents, which are
    // always aligned on 256-byte addresses - the least significant byte of
    // a ParkEvent is always 0.  Colocating the lock-byte with the queue
    // allows us to easily avoid what would otherwise be a race in lock()
    // if we were to use two completely separate fields for the contention queue
    // and the lock indicator.  Specifically, colocation renders us immune
    // from the race where a thread might enqueue itself in the lock() slow-path
    // immediately after the lock holder drops the outer lock in the unlock()
    // fast-path.
    //
    // Colocation allows us to use a fast-path unlock() form that uses
    // A MEMBAR instead of a CAS.  MEMBAR has lower local latency than CAS
    // on many platforms.
    //
    // See:
    // +  http://blogs.sun.com/dave/entry/biased_locking_in_hotspot
    // +  http://blogs.sun.com/dave/resource/synchronization-public2.pdf
    //
    // Note that we're *not* using word-tearing the classic sense.
    // The lock() fast-path will CAS the lockword and the unlock()
    // fast-path will store into the lock-byte colocated within the lockword.
    // We depend on the fact that all our reference platforms have
    // coherent and atomic byte accesses.  More precisely, byte stores
    // interoperate in a safe, sane, and expected manner with respect to
    // CAS, ST and LDs to the full-word containing the byte.
    // If you're porting HotSpot to a platform where that isn't the case
    // then you'll want change the unlock() fast path from:
    //    STB;MEMBAR #storeload; LDN
    // to a full-word CAS of the lockword.
    
    
    union SplitWord {   // full-word with separately addressable LSB
      volatile intptr_t FullWord ;
      volatile void * Address ;
      volatile jbyte Bytes [sizeof(intptr_t)] ;
    } ;
```

  * Thread * volatile _owner
    
    この Monitor オブジェクトのロックを保持しているスレッドを示す.
    ロックされていない場合は NULL.

  * ParkEvent * volatile _EntryList

    その Monitor に対してロック待ちで待機しているスレッドの一覧.

    役割としては _LockWord(cxq) と同様.
    もう少し正確に言うと, cxq と EntryList は論理的にはつながっていて 1本の待ち行列を構成する
    (2つに分かれているのはアルゴリズム上の都合).

    待ち状態に入ったスレッドは, まずは cxq に追加され, その後 EntryList に移される.

    実際には, _EntryList は ParkEvent 型のフィールドで
    ParkEvent オブジェクトの ListNext フィールドを用いてリストを構成している.

  * ParkEvent * volatile _OnDeck

    「ロック確保を行う権利があるスレッド」を示す.

    _EntryList の中から一つが選ばれて _OnDeck に移される
    (このため一度に _OnDeck に存在するスレッドは 1つだけ).

    (なお, _OnDeck にいるスレッドでも必ずロックが取れるわけではない.
     ロックの確保は, 現在待ち行列におらず新しくロックを取りに来たスレッドと競争して行う)

  * volatile intptr_t _WaitLock

    _WaitSet を更新する処理を排他するためのロック.

    Monitor::IWait() 中のコメントによると,
    「理想的には Monitor の outer lock (_LockWord?) を握っているスレッドだけが 
    WaitSet に触れるということにしたいが,
    wait 待ちしていたスレッドがタイムアウト切れで wait から外れる場合には 
    ロックを持っていない状態で WaitSet を変更する必要があるので
    WaitLock という構造を用意している」 
    とのこと.

  * ParkEvent * volatile  _WaitSet

    Monitor::wait() で待機しているスレッドの一覧.

    実際には, _WaitSet は ParkEvent 型のフィールドで
    ParkEvent オブジェクトの ListNext フィールドを用いてリストを構成している.

    notify() が呼ばれると, スレッドはここから cxq (または EntryList) に移動される.

  * volatile bool     _snuck

    VMThread による sneaky locking のために使われるフィールド.
      volatile bool     _snuck;              // Used for sneaky locking (evil).

    sneaky locking の場合には true になる.
    (そして, true だと unlock 時に unlock 処理が省略される)

  * int NotifyCount
    
    トラブルシューティング用フィールド.
    Monitor::notify() された回数を示す.
    (?? このフィールドはどこからも参照されていないような気がするが...)

  * char _name[MONITOR_NAME_LEN]
    
    この Monitor の名前.

  * ... #TODO


```
    ((cite: hotspot/src/share/vm/runtime/mutex.hpp))
      SplitWord _LockWord ;                  // Contention queue (cxq) colocated with Lock-byte
      enum LockWordBits { _LBIT=1 } ;
      Thread * volatile _owner;              // The owner of the lock
                                             // Consider sequestering _owner on its own $line
                                             // to aid future synchronization mechanisms.
      ParkEvent * volatile _EntryList ;      // List of threads waiting for entry
      ParkEvent * volatile _OnDeck ;         // heir-presumptive
      volatile intptr_t _WaitLock [1] ;      // Protects _WaitSet
      ParkEvent * volatile  _WaitSet ;       // LL of ParkEvents
      volatile bool     _snuck;              // Used for sneaky locking (evil).
      int NotifyCount ;                      // diagnostic assist
      char _name[MONITOR_NAME_LEN];          // Name of mutex
    
      // Debugging fields for naming, deadlock detection, etc. (some only used in debug mode)
    #ifndef PRODUCT
      bool      _allow_vm_block;
      debug_only(int _rank;)                 // rank (to avoid/detect potential deadlocks)
      debug_only(Monitor * _next;)           // Used by a Thread to link up owned locks
      debug_only(Thread* _last_owner;)       // the last thread to own the lock
      debug_only(static bool contains(Monitor * locks, Monitor * lock);)
      debug_only(static Monitor * get_least_ranked_lock(Monitor * locks);)
      debug_only(Monitor * get_least_ranked_lock_besides_this(Monitor * locks);)
    #endif
```




### 詳細(Details)
See: [here](../doxygen/classMonitor.html) for details

---
## <a name="noIGw8MJ_A" id="noIGw8MJ_A">Mutex</a>

### 概要(Summary)
スレッド間の排他処理を行うためのユーティリティ・クラス.

Monitor クラスとの違いは, wait/notify/notifyAll() の機能が使用不能な点.

なお, スレッド間の排他処理を行うためのクラスは幾つか存在している (See: [here](noFBHxUsnN.html) for details).
Monitor や Mutex は長い critical section にも使用可能なクラス (See: [here](no2114cio.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/mutex.hpp))
    class Mutex : public Monitor {      // degenerate Monitor
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 以下の大域変数
  
  * Patching_lock
  * PackageTable_lock
  * CompiledIC_lock
  * InlineCacheBuffer_lock
  * VMStatistic_lock
  * JNIGlobalHandle_lock
  * JNIHandleBlockFreeList_lock
  * JNICachedItableIndex_lock
  * JmethodIdCreation_lock
  * JfieldIdCreation_lock
  * JvmtiThreadState_lock
  * ExpandHeap_lock
  * AdapterHandlerLibrary_lock
  * SignatureHandlerLibrary_lock
  * VtableStubs_lock
  * SymbolTable_lock
  * StringTable_lock
  * CodeCache_lock
  * MethodData_lock
  * RetData_lock
  * STS_init_lock
  * CMRegionStack_lock
  * SATB_Q_FL_lock
  * Shared_SATB_Q_lock
  * DirtyCardQ_FL_lock
  * Shared_DirtyCardQ_lock
  * ParGCRareEvent_lock
  * EvacFailureStack_lock
  * DerivedPointerTableGC_lock
  * Compile_lock
  * CompileTaskAlloc_lock
  * CompileStatistics_lock
  * MultiArray_lock
  * ProfilePrint_lock
  * ExceptionCache_lock
  * OsrList_lock
  * FullGCALot_lock
  * Debug1_lock
  * Debug2_lock
  * Debug3_lock
  * tty_lock
  * RawMonitor_lock
  * PerfDataMemAlloc_lock
  * PerfDataManager_lock
  * OopMapCacheAlloc_lock
  * FreeList_lock
  * OldSets_lock
  * MMUTracker_lock
  * HotCardCache_lock
  * Management_lock

* 各 GCMemoryManager オブジェクトの _last_gc_lock フィールド

* 各 OffsetTableContigSpace オブジェクトの _par_alloc_lock フィールド

* 各 OopMapCache オブジェクトの _mut フィールド

* 各 JvmtiTagMap オブジェクトの _lock フィールド

* 各 CMSBitMap オブジェクトの _lock フィールド

* 各 CompactibleFreeListSpace オブジェクトの _indexedFreeListParLocks フィールド
   
  (正確には, このフィールドは Mutex の配列を格納するフィールド.
  この中に, CompactibleFreeListSpace 内で使用される全ての Mutex オブジェクトが格納されている)

* 各 CompactibleFreeListSpace オブジェクトの _freelistLock フィールド

* 各 CompactibleFreeListSpace オブジェクトの _parDictionaryAllocLock フィールド

* 各 CMSMarkStack オブジェクトの _par_lock フィールド

* 各 G1ParTask オブジェクトの _stats_lock フィールド

* 各 G1OffsetTableContigSpace オブジェクトの _par_alloc_lock フィールド

* 各 OtherRegionsTable オブジェクトの _m フィールド

* 各 OSThread オブジェクトの _current_callback_lock フィールド (Solaris の場合)
  
* os::Linux クラスの _createThread_lock フィールド (static フィールド) (Linux の場合)


(なお, mutexLocker.hpp で extern されている大域変数は mutexLocker.cpp で実際に定義されているものと少しずれている.
 具体的には次のものが含まれていない(tty_lock). 逆に次のものが含まれている(ParkerFreeList_lock). 
 tty_lock については実際の使用箇所に extern 宣言が書かれている.)


```
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
    // Mutexes used in the VM.
    
    extern Mutex*   Patching_lock;                   // a lock used to guard code patching of compiled code
    ...
    extern Mutex*   PackageTable_lock;               // a lock on the class loader package table
    extern Mutex*   CompiledIC_lock;                 // a lock used to guard compiled IC patching and access
    extern Mutex*   InlineCacheBuffer_lock;          // a lock used to guard the InlineCacheBuffer
    extern Mutex*   VMStatistic_lock;                // a lock used to guard statistics count increment
    extern Mutex*   JNIGlobalHandle_lock;            // a lock on creating JNI global handles
    extern Mutex*   JNIHandleBlockFreeList_lock;     // a lock on the JNI handle block free list
    extern Mutex*   JNICachedItableIndex_lock;       // a lock on caching an itable index during JNI invoke
    extern Mutex*   JmethodIdCreation_lock;          // a lock on creating JNI method identifiers
    extern Mutex*   JfieldIdCreation_lock;           // a lock on creating JNI static field identifiers
    ...
    extern Mutex*   JvmtiThreadState_lock;           // a lock on modification of JVMTI thread data
    ...
    extern Mutex*   ExpandHeap_lock;                 // a lock on expanding the heap
    extern Mutex*   AdapterHandlerLibrary_lock;      // a lock on the AdapterHandlerLibrary
    extern Mutex*   SignatureHandlerLibrary_lock;    // a lock on the SignatureHandlerLibrary
    extern Mutex*   VtableStubs_lock;                // a lock on the VtableStubs
    extern Mutex*   SymbolTable_lock;                // a lock on the symbol table
    extern Mutex*   StringTable_lock;                // a lock on the interned string table
    extern Mutex*   CodeCache_lock;                  // a lock on the CodeCache, rank is special, use MutexLockerEx
    extern Mutex*   MethodData_lock;                 // a lock on installation of method data
    extern Mutex*   RetData_lock;                    // a lock on installation of RetData inside method data
    extern Mutex*   DerivedPointerTableGC_lock;      // a lock to protect the derived pointer table
    ...
    extern Mutex*   STS_init_lock;                   // coordinate initialization of SuspendibleThreadSets.
    ...
    extern Mutex*   CMRegionStack_lock;              // used for protecting accesses to the CM region stack
    extern Mutex*   SATB_Q_FL_lock;                  // Protects SATB Q
                                                     // buffer free list.
    ...
    extern Mutex*   Shared_SATB_Q_lock;              // Lock protecting SATB
                                                     // queue shared by
                                                     // non-Java threads.
    
    extern Mutex*   DirtyCardQ_FL_lock;              // Protects dirty card Q
                                                     // buffer free list.
    ...
    extern Mutex*   Shared_DirtyCardQ_lock;          // Lock protecting dirty card
                                                     // queue shared by
                                                     // non-Java threads.
                                                     // (see option ExplicitGCInvokesConcurrent)
    extern Mutex*   ParGCRareEvent_lock;             // Synchronizes various (rare) parallel GC ops.
    extern Mutex*   EvacFailureStack_lock;           // guards the evac failure scan stack
    extern Mutex*   Compile_lock;                    // a lock held when Compilation is updating code (used to block CodeCache traversal, CHA updates, etc)
    ...
    extern Mutex*   CompileTaskAlloc_lock;           // a lock held when CompileTasks are allocated
    extern Mutex*   CompileStatistics_lock;          // a lock held when updating compilation statistics
    extern Mutex*   MultiArray_lock;                 // a lock used to guard allocation of multi-dim arrays
    ...
    extern Mutex*   ProfilePrint_lock;               // a lock used to serialize the printing of profiles
    extern Mutex*   ExceptionCache_lock;             // a lock used to synchronize exception cache updates
    extern Mutex*   OsrList_lock;                    // a lock used to serialize access to OSR queues
    
    #ifndef PRODUCT
    extern Mutex*   FullGCALot_lock;                 // a lock to make FullGCALot MT safe
    #endif
    extern Mutex*   Debug1_lock;                     // A bunch of pre-allocated locks that can be used for tracing
    extern Mutex*   Debug2_lock;                     // down synchronization related bugs!
    extern Mutex*   Debug3_lock;
    
    extern Mutex*   RawMonitor_lock;
    extern Mutex*   PerfDataMemAlloc_lock;           // a lock on the allocator for PerfData memory for performance data
    extern Mutex*   PerfDataManager_lock;            // a long on access to PerfDataManager resources
    extern Mutex*   ParkerFreeList_lock;
    extern Mutex*   OopMapCacheAlloc_lock;           // protects allocation of oop_map caches
    
    extern Mutex*   FreeList_lock;                   // protects the free region list during safepoints
    ...
    extern Mutex*   OldSets_lock;                    // protects the old region sets
    extern Mutex*   MMUTracker_lock;                 // protects the MMU
                                                     // tracker data structures
    extern Mutex*   HotCardCache_lock;               // protects the hot card cache
    
    extern Mutex*   Management_lock;                 // a lock used to serialize JVM management
    ...
```


```
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.cpp))
    // Mutexes used in the VM (see comment in mutexLocker.hpp):
    //
    // Note that the following pointers are effectively final -- after having been
    // set at JVM startup-time, they should never be subsequently mutated.
    // Instead of using pointers to malloc()ed monitors and mutexes we should consider
    // eliminating the indirection and using instances instead.
    // Consider using GCC's __read_mostly.
    
    Mutex*   Patching_lock                = NULL;
    ...
    Mutex*   PackageTable_lock            = NULL;
    Mutex*   CompiledIC_lock              = NULL;
    Mutex*   InlineCacheBuffer_lock       = NULL;
    Mutex*   VMStatistic_lock             = NULL;
    Mutex*   JNIGlobalHandle_lock         = NULL;
    Mutex*   JNIHandleBlockFreeList_lock  = NULL;
    Mutex*   JNICachedItableIndex_lock    = NULL;
    Mutex*   JmethodIdCreation_lock       = NULL;
    Mutex*   JfieldIdCreation_lock        = NULL;
    ...
    Mutex*   JvmtiThreadState_lock        = NULL;
    ...
    Mutex*   ExpandHeap_lock              = NULL;
    Mutex*   AdapterHandlerLibrary_lock   = NULL;
    Mutex*   SignatureHandlerLibrary_lock = NULL;
    Mutex*   VtableStubs_lock             = NULL;
    Mutex*   SymbolTable_lock             = NULL;
    Mutex*   StringTable_lock             = NULL;
    Mutex*   CodeCache_lock               = NULL;
    Mutex*   MethodData_lock              = NULL;
    Mutex*   RetData_lock                 = NULL;
    ...
    Mutex*   STS_init_lock                = NULL;
    ...
    Mutex*   CMRegionStack_lock           = NULL;
    Mutex*   SATB_Q_FL_lock               = NULL;
    ...
    Mutex*   Shared_SATB_Q_lock           = NULL;
    Mutex*   DirtyCardQ_FL_lock           = NULL;
    ...
    Mutex*   Shared_DirtyCardQ_lock       = NULL;
    Mutex*   ParGCRareEvent_lock          = NULL;
    Mutex*   EvacFailureStack_lock        = NULL;
    Mutex*   DerivedPointerTableGC_lock   = NULL;
    Mutex*   Compile_lock                 = NULL;
    ...
    Mutex*   CompileTaskAlloc_lock        = NULL;
    Mutex*   CompileStatistics_lock       = NULL;
    Mutex*   MultiArray_lock              = NULL;
    ...
    Mutex*   ProfilePrint_lock            = NULL;
    Mutex*   ExceptionCache_lock          = NULL;
    ...
    Mutex*   OsrList_lock                 = NULL;
    #ifndef PRODUCT
    Mutex*   FullGCALot_lock              = NULL;
    #endif
    
    Mutex*   Debug1_lock                  = NULL;
    Mutex*   Debug2_lock                  = NULL;
    Mutex*   Debug3_lock                  = NULL;
    
    Mutex*   tty_lock                     = NULL;
    
    Mutex*   RawMonitor_lock              = NULL;
    Mutex*   PerfDataMemAlloc_lock        = NULL;
    Mutex*   PerfDataManager_lock         = NULL;
    Mutex*   OopMapCacheAlloc_lock        = NULL;
    
    Mutex*   FreeList_lock                = NULL;
    ...
    Mutex*   OldSets_lock                 = NULL;
    Mutex*   MMUTracker_lock              = NULL;
    Mutex*   HotCardCache_lock            = NULL;
    
    ...
    
    Mutex*   Management_lock              = NULL;
    ...
```

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* Mutex を格納する大域変数の初期化
  
  mutex_init()

* GCMemoryManager::_last_gc_lock の初期化
  
  GCMemoryManager::GCMemoryManager()

* (OffsetTableContigSpace クラスの _par_alloc_lock フィールドは, ポインタ型ではなく実体なので,
  OffsetTableContigSpace オブジェクトの生成時に一緒に生成される)

* (OopMapCache クラスの _mut フィールドは, ポインタ型ではなく実体なので,
  OopMapCache オブジェクトの生成時に一緒に生成される)
   
* (JvmtiTagMap クラスの _lock フィールドは, ポインタ型ではなく実体なので,
  JvmtiTagMap オブジェクトの生成時に一緒に生成される)

* 
  
  JVM_RawMonitorCreate()

* CMSBitMap::_lock の初期化
  
  CMSBitMap::CMSBitMap()

* CompactibleFreeListSpace::_indexedFreeListParLocks の初期化
  
  CompactibleFreeListSpace::CompactibleFreeListSpace()

* (CompactibleFreeListSpace クラスの _freelistLock フィールドは, ポインタ型ではなく実体なので,
  CompactibleFreeListSpace オブジェクトの生成時に一緒に生成される)

* (CompactibleFreeListSpace クラスの _freelistLock フィールドは, ポインタ型ではなく実体なので,
  CompactibleFreeListSpace オブジェクトの生成時に一緒に生成される)
  
  CompactibleFreeListSpace _parDictionaryAllocLock

* (CMSMarkStack クラスの _par_lock フィールドは, ポインタ型ではなく実体なので,
  CMSMarkStack オブジェクトの生成時に一緒に生成される)

* (G1ParTask クラスの _stats_lock フィールドは, ポインタ型ではなく実体なので,
  G1ParTask オブジェクトの生成時に一緒に生成される)

* (G1OffsetTableContigSpace クラスの _par_alloc_lock フィールドは, ポインタ型ではなく実体なので,
  G1OffsetTableContigSpace オブジェクトの生成時に一緒に生成される)

* (OtherRegionsTable クラスの _m フィールドは, ポインタ型ではなく実体なので,
  OtherRegionsTable オブジェクトの生成時に一緒に生成される)

* OSThread::_current_callback_lock の初期化 (Solaris の場合)
  
  OSThread::pd_initialize()

* os::Linux::_createThread_lock の初期化 (Linux の場合)
  
  os::init_2()

### 内部構造(Internal structure)
実際の処理はスーパークラスである Monitor に丸投げしている.

(このクラスで新しく定義されているメソッド／フィールドはない.
 変更点は wait/notify/notifyAll() がオーバーライドされて使用不能になっていることだけ.)




### 詳細(Details)
See: [here](../doxygen/classMutex.html) for details

---
