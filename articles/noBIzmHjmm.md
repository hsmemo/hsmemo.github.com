---
layout: default
title: ObjectMonitor クラス関連のクラス (ObjectWaiter, ObjectMonitor)
---
[Top](../index.html)

#### ObjectMonitor クラス関連のクラス (ObjectWaiter, ObjectMonitor)

これらは, 同期排他処理 (monitorenter/monitorexit 命令, synchronized 修飾子, wait()/notify()/notifyAll() メソッド, 等) のためのクラス
(See: [here](no2114NIs.html) for details).


### クラス一覧(class list)

  * [ObjectMonitor](#noIMtVGOOY)
  * [ObjectWaiter](#noCnbdZrda)


---
## <a name="noIMtVGOOY" id="noIMtVGOOY">ObjectMonitor</a>

### 概要(Summary)
各 Java オブジェクトのモニタの状態(ロックが取られているか否か,等)を管理するためのクラス.
1つの ObjectMonitor オブジェクトが 1つの Java オブジェクトに対応する.

なお, Java オブジェクトのモニタ状態は全てこのクラスだけで表現できるが, 
ナイーブに実装すると全ての oop に対して ObjectMonitor が1つ必要になりメモリ消費量が大きくなるので, 
インタープリタや JIT 生成コードの fast path で終わっている間はこのオブジェクトは使用されない
(というか, そもそもメモリ上に確保されない).
fast path で終わらなかったオブジェクト(= inflate したオブジェクト)についてだけこいつが確保される (See: [here](no2114NIs.html) for details).

なおコメントによると, 非常に繊細なクラスであり変更するときはよく考えてから行うように, とのこと.


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
    // WARNING:
    //   This is a very sensitive and fragile class. DO NOT make any
    // change unless you are fully aware of the underlying semantics.
    
    //   This class can not inherit from any other class, because I have
    // to let the displaced header be the very first word. Otherwise I
    // have to let markOop include this file, which would export the
    // monitor data structure to everywhere.
    //
    // The ObjectMonitor class is used to implement JavaMonitors which have
    // transformed from the lightweight structure of the thread stack to a
    // heavy weight lock due to contention
    
    // It is also used as RawMonitor by the JVMTI
```


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
    class ObjectMonitor {
```


### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各オブジェクト(oop)の _mark フィールド

  fast path で終わらなかったオブジェクト(= inflate したオブジェクト)については, 
  mark フィールドに ObjectMonitor へのポインタが埋め込まれている (See: markOopDesc).

* 各 Thread オブジェクトの omFreeList フィールド

  スレッドローカルなフリーリスト.

  (このフィールドは ObjectMonitor の線形リストを格納するフィールド(フリーリスト).
  ObjectMonitor オブジェクトは FreeNext フィールドで次の ObjectMonitor オブジェクトを指せる構造になっている.
  このフィールドの線形リストに未使用な ObjectMonitor オブジェクトがつながれている.)

* 各 Thread オブジェクトの omInUseList フィールド

  (ObjectSynchronizer::deflate_idle_monitors() を高速化するためのフィールド.
  MonitorInUseLists オプションが指定されている場合にのみ使用される)

  対応する Thread によってロック中の ObjectMonitor の一覧
  (格納している ObjectMonitor オブジェクト自体は, 各オブジェクト(oop)の _mark が指しているものと重複).

  (このフィールドは ObjectMonitor の線形リストを格納するフィールド.
  ObjectMonitor オブジェクトは FreeNext フィールドで次の ObjectMonitor オブジェクトを指せる構造になっている.
  その Thread オブジェクトがロックした ObjectMonitor オブジェクトは全てこの線形リストに格納されている.)

* 各 Thread オブジェクトの _current_pending_monitor フィールド

  対応する Thread がロック待ちしている ObjectMonitor
  (格納している ObjectMonitor オブジェクト自体は, 各オブジェクト(oop)の _mark が指しているものと重複).

* 各 Thread オブジェクトの _current_waiting_monitor フィールド

  対応する Thread が java.lang.Object.wait() で待っている ObjectMonitor
  (格納している ObjectMonitor オブジェクト自体は, 各オブジェクト(oop)の _mark が指しているものと重複).

* ObjectSynchronizer クラスの gBlockList フィールド (static フィールド)

  全ての ObjectMonitor を格納したリスト 
  (使用中／未使用に関わらず全ての ObjectMonitor を格納している.
   格納している ObjectMonitor オブジェクト自体は, 他の格納箇所と重複)

  (正確には, これらのフィールドは ObjectMonitor の固定長配列の線形リストを格納するフィールド.
  リストの各要素は ObjectSynchronizer::_BLOCKSIZE 個の要素からなる ObjectMonitor 配列で, 
  配列の先頭の ObjectMonitor の FreeNext フィールドが次の配列の先頭を指す)

* ObjectSynchronizer クラスの gFreeList フィールド (static フィールド)

  大域的なフリーリスト.

  (このフィールドは ObjectMonitor の線形リストを格納するフィールド(フリーリスト).
  ObjectMonitor オブジェクトは FreeNext フィールドで次の ObjectMonitor オブジェクトを指せる構造になっている.
  このフィールドの線形リストに未使用な ObjectMonitor オブジェクトがつながれている.)

* ObjectSynchronizer クラスの gOmInUseList フィールド (static フィールド)

  (ObjectSynchronizer::deflate_idle_monitors() を高速化するためのフィールド.
  MonitorInUseLists オプションが指定されている場合にのみ使用される)

  各 Thread オブジェクトが破棄される際に, 
  omInUseList フィールドにつながれていた ObjectMonitor がこのフィールドに移される.

  (このフィールドは ObjectMonitor の線形リストを格納するフィールド.
  ObjectMonitor オブジェクトは FreeNext フィールドで次の ObjectMonitor オブジェクトを指せる構造になっている.
  その Thread オブジェクトがロックした ObjectMonitor オブジェクトは全てこの線形リストに格納されている.)

#### 生成箇所(where its instances are created)
ObjectSynchronizer::omAlloc() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
ObjectSynchronizer::inflate()
-> ObjectSynchronizer::omAlloc()
```

#### 削除箇所(where its instances are deleted)
未使用のオブジェクトはフリーリスト管理するので削除(delete)されない?? #TODO

Safepoint で実行される ObjectSynchronizer::deflate_idle_monitors() により, 
使われていない ObjectMonitor オブジェクトは ObjectSynchronizer::gFreeList に戻される.

```
(略) (See: [here](noFCZ0Hp3S.html) for details)
-> SafepointSynchronize::begin()
   -> SafepointSynchronize::do_cleanup_tasks()
      -> ObjectSynchronizer::deflate_idle_monitors()
```


```
    ((cite: hotspot/src/share/vm/runtime/synchronizer.cpp))
    // -----------------------------------------------------------------------------
    // ObjectMonitor Lifecycle
    // -----------------------
    // Inflation unlinks monitors from the global gFreeList and
    // associates them with objects.  Deflation -- which occurs at
    // STW-time -- disassociates idle monitors from objects.  Such
    // scavenged monitors are returned to the gFreeList.
    //
    // The global list is protected by ListLock.  All the critical sections
    // are short and operate in constant-time.
    //
    // ObjectMonitors reside in type-stable memory (TSM) and are immortal.
    //
    // Lifecycle:
    // --   unassigned and on the global free list
    // --   unassigned and on a thread's private omFreeList
    // --   assigned to an object.  The object is inflated and the mark refers
    //      to the objectmonitor.
    //
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* volatile markOop 	_header
  
  ロック対象のオブジェクトの (ロック前の) mark フィールドの値.

* void *volatile 	_object
  
  ロック対象のオブジェクトへのポインタ

* double 	SharingPad [1]
  
  (?? 使用箇所が見当たらない)

* void *volatile 	_owner
  
  ロックを取得しているスレッドへのポインタ (NULL であればロックされていない状態).

  (ただし, stack-locked から inflated に遷移した直後は, 
  一時的に (スレッドのポインタの代わりに) そのスレッドが所有している BasicObjectLock が格納されていることもある.
  OwnerIsThread フィールドが 1 ならスレッド, 0 なら BasicObjectLock を指している.
  stack-locked から遷移したときは BasicObjectLock を指している.)

* volatile intptr_t 	_recursions
  
  重複してロックが取得された場合(再帰的にロックされた場合)用に, 重複回数を記録しておくフィールド.

* int 	OwnerIsThread
  
  (_owner フィールドを参照) _owner がスレッドへのポインタなら 1, BasicObjectLock なら 0.

* ObjectWaiter *volatile 	_cxq
  
  ロックが解放されるのを待っているスレッド用の待ち行列
  (初めは _cxq に登録され, その後で _EntryList に移される.
  2つに分かれているのはアルゴリズム上の都合)

* ObjectWaiter *volatile 	_EntryList
  
  ロックが解放されるのを待っているスレッド用の待ち行列.
  (_cxq フィールドも参照)

* Thread *volatile 	_succ
  
  最適化用のフィールド.
  何度も起床処理(_cxq や _EntryList で寝ているスレッドを起こす処理)が行われると無駄なので(futile wakeup),
  それを防ぐためのもの
  (Monitor クラスにおける _OnDeck フィールドと同じようなもの).

  既に起床済みのスレッドがいる場合に, このフィールドにそのスレッドを登録しておく.
  このフィールドが NULL でなければ unlock() 時に起床処理を行う必要はない.

  (なお, 既に起床済みのスレッド以外に, 
  現在スピンロックでロックを取りに行っているスレッドが登録されることもある.
  この場合も使われ方は同じで, そういうスレッドがいるのなら起床処理を行う必要は無いので
  unlock() 時に起床処理が省略される)
  
  (なお, Monitor::_OnDeck と同じく, _succ にいるスレッドでも必ずロックが取れるわけではない.
  ロックの確保は, 現在待ち行列におらず新しくロックを取りに来たスレッドと競争して行う)

  (なお, _succ を使わない方法として (余り魅力的な方法ではないが) 以下のようなアイデアも提案されている.
  「exit するスレッドがロックを落とした後で少し待ってみる.
   spin しているスレッドがロックを取れたのを確認したら起床処理を省略してリターンする.」)


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.cpp))
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
```
  
* Thread *volatile 	_Responsible
  
  1-0 model の inflated fast path 処理用のフィールド.

* int 	_PromptDrain
  
  (?? 使用箇所が見当たらない)

* volatile int 	_Spinner
  
  Adaptive Spinning 用のフィールド.

* volatile int 	_SpinFreq
  
  Adaptive Spinning 用のフィールド (現在は使われていない模様).

* volatile int 	_SpinClock
  
  Adaptive Spinning 用のフィールド (現在は使われていない模様).

* volatile int 	_SpinDuration
  
  Adaptive Spinning 用のフィールド.

* volatile intptr_t 	_SpinState

  Adaptive Spinning 用のフィールド (現在は使われていない模様).

* volatile intptr_t 	_count
  
  ObjectSynchronizer::deflate_idle_monitors() で回収されないようにするための reference counter.

  _count の値は, おおよそ |_WaitSet| + |_EntryList| という数になっている.

* volatile intptr_t 	_waiters
  
  _WaitSet 中にあるスレッドの数 (See: [here](no3059BSg.html) for details)

* ObjectWaiter *volatile 	_WaitSet
  
  wait() で停止している (= notify() を待っている) スレッドの待ち行列  (See: [here](no3059BSg.html) for details)

* volatile int 	_WaitSetLock
  
  _WaitSet 操作を保護するためのロック (Monitor クラスにおける _WaitLock のようなもの)
  (See: [here](no3059BSg.html) for details)

* int 	_QMix
  
  (?? 使用箇所が見当たらない)

* ObjectMonitor * 	FreeNext
  
  未使用時にフリーリスト管理するためのポインタ (次の ObjectMonitor を指す).

* intptr_t 	StatA
  
  (?? 使用箇所が見当たらない)

* intptr_t 	StatsB

  (?? 使用箇所が見当たらない)


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      // WARNING: this must be the very first word of ObjectMonitor
      // This means this class can't use any virtual member functions.
      // TODO-FIXME: assert that offsetof(_header) is 0 or get rid of the
      // implicit 0 offset in emitted code.
    
      volatile markOop   _header;       // displaced object header word - mark
      void*     volatile _object;       // backward object pointer - strong root
    
      double SharingPad [1] ;           // temp to reduce false sharing
    
      // All the following fields must be machine word aligned
      // The VM assumes write ordering wrt these fields, which can be
      // read from other threads.
```


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      void *  volatile _owner;          // pointer to owning thread OR BasicLock
      volatile intptr_t  _recursions;   // recursion count, 0 for first entry
```


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      int OwnerIsThread ;               // _owner is (Thread *) vs SP/BasicLock
      ObjectWaiter * volatile _cxq ;    // LL of recently-arrived threads blocked on entry.
                                        // The list is actually composed of WaitNodes, acting
                                        // as proxies for Threads.
```


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      ObjectWaiter * volatile _EntryList ;     // Threads blocked on entry or reentry.
```


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      Thread * volatile _succ ;          // Heir presumptive thread - used for futile wakeup throttling
      Thread * volatile _Responsible ;
      int _PromptDrain ;                // rqst to drain cxq into EntryList ASAP
    
      volatile int _Spinner ;           // for exit->spinner handoff optimization
      volatile int _SpinFreq ;          // Spin 1-out-of-N attempts: success rate
      volatile int _SpinClock ;
      volatile int _SpinDuration ;
      volatile intptr_t _SpinState ;    // MCS/CLH list of spinners
    
      // TODO-FIXME: _count, _waiters and _recursions should be of
      // type int, or int32_t but not intptr_t.  There's no reason
      // to use 64-bit fields for these variables on a 64-bit JVM.
    
      volatile intptr_t  _count;        // reference count to prevent reclaimation/deflation
                                        // at stop-the-world time.  See deflate_idle_monitors().
                                        // _count is approximately |_WaitSet| + |_EntryList|
```


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      volatile intptr_t  _waiters;      // number of waiting threads
```


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      ObjectWaiter * volatile _WaitSet; // LL of threads wait()ing on the monitor
```


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      volatile int _WaitSetLock;        // protects Wait Queue - simple spinlock
```


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      int _QMix ;                       // Mixed prepend queue discipline
      ObjectMonitor * FreeNext ;        // Free list linkage
      intptr_t StatA, StatsB ;
```

### 備考(Notes)
static フィールドとして, 以下のパフォーマンスカウンタ(PerfData)を備えている.

  * PerfCounter * _sync_ContendedLockAttempts ;
  * PerfCounter * _sync_FutileWakeups ;
  * PerfCounter * _sync_Parks ;
  * PerfCounter * _sync_EmptyNotifications ;
  * PerfCounter * _sync_Notifications ;
    
    notify(), notifyAll() でスレッドが起床された回数を記録する 
    (これらが呼ばれたときに実際にスレッドの起床処理が発生したら, 1増加する)

  * PerfCounter * _sync_SlowEnter ;
  * PerfCounter * _sync_SlowExit ;
  * PerfCounter * _sync_SlowNotify ;
  * PerfCounter * _sync_SlowNotifyAll ;
  * PerfCounter * _sync_FailedSpins ;
  * PerfCounter * _sync_SuccessfulSpins ;
  * PerfCounter * _sync_PrivateA ;
  * PerfCounter * _sync_PrivateB ;
  * PerfCounter * _sync_MonInCirculation ;
  * PerfCounter * _sync_MonScavenged ;
  * PerfCounter * _sync_Inflations ;
  * PerfCounter * _sync_Deflations ;
  * PerfLongVariable * _sync_MonExtant ;


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      static PerfCounter * _sync_ContendedLockAttempts ;
      static PerfCounter * _sync_FutileWakeups ;
      static PerfCounter * _sync_Parks ;
      static PerfCounter * _sync_EmptyNotifications ;
      static PerfCounter * _sync_Notifications ;
      static PerfCounter * _sync_SlowEnter ;
      static PerfCounter * _sync_SlowExit ;
      static PerfCounter * _sync_SlowNotify ;
      static PerfCounter * _sync_SlowNotifyAll ;
      static PerfCounter * _sync_FailedSpins ;
      static PerfCounter * _sync_SuccessfulSpins ;
      static PerfCounter * _sync_PrivateA ;
      static PerfCounter * _sync_PrivateB ;
      static PerfCounter * _sync_MonInCirculation ;
      static PerfCounter * _sync_MonScavenged ;
      static PerfCounter * _sync_Inflations ;
      static PerfCounter * _sync_Deflations ;
      static PerfLongVariable * _sync_MonExtant ;
```




### 詳細(Details)
See: [here](../doxygen/classObjectMonitor.html) for details

---
## <a name="noCnbdZrda" id="noCnbdZrda">ObjectWaiter</a>

### 概要(Summary)
ObjectMonitor 用の補助クラス.

スレッドの待ち行列を表すための一時オブジェクト(StackObjクラス)
(= ロックの解放待ちや (wait 時の) notify() を待っているスレッドを管理するためのクラス).

1つの ObjectWaiter オブジェクトが 待機中のスレッド 1個に対応する
(ObjectWaiter は doubly-linked list を構成することができ, それによって待ち行列を表現する).

なおコメントによると, ObjectWaiter は廃止して ParkEvent を使うようにしたい, とのこと.


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
    // ObjectWaiter serves as a "proxy" or surrogate thread.
    // TODO-FIXME: Eliminate ObjectWaiter and use the thread-specific
    // ParkEvent instead.  Beware, however, that the JVMTI code
    // knows about ObjectWaiters, so we'll have to reconcile that code.
    // See next_waiter(), first_waiter(), etc.
    
    class ObjectWaiter : public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. コード中で ObjectWaiter 型の局所変数を宣言する (コンストラクタ引数で待機対象のスレッドを登録する).
2. 待機する ObjectMonitor の _cxq フィールド, _EntryList フィールド, または _WaitSet フィールドに, 
   新しく作った ObjectWaiter を追加する.
3. 待機が解けたら, 2. で追加したリストから ObjectWaiter を外す.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* ObjectMonitor::EnterI()
* ObjectMonitor::wait()

* JvmtiRawMonitor::SimpleEnter()
* JvmtiRawMonitor::SimpleWait()

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* ObjectWaiter *volatile 	_next
  
  doubly-linked list を構成するためのポインタ. 次の ObjectWaiter を指す.

* ObjectWaiter *volatile 	_prev

  doubly-linked list を構成するためのポインタ. 前の ObjectWaiter を指す.

* Thread * 	_thread
  
  待機対象のスレッド.

* ParkEvent * 	_event
  
  待機対象のスレッドの Thread::_ParkEvent を指す.

* volatile int 	_notified
  
  wait/notify/notifyAll の処理で, 
  timeout や interrupt による起床と, 実際の notify による起床を区別するためのフィールド.
  
  ObjectWaiter オブジェクトが生成された直後は 0 だが, ObjectMonitor::notify() で起こされると 1 になる
  (つまり, 起床した際に _notified がゼロのままなら notify() 以外の原因による起床).

* volatile TStates 	TState
  
  futile wakeup 対策用のフィールド(?).
  この ObjectMonitor の状態を示す (= ObjectMonitor 中のどこにいるのかを示す).

  なお, TStates 型は以下のように定義された enum 値

  * TS_RUN なら実行中
  * TS_ENTER なら EntryList 内で待機中
  * TS_CXQ なら cxq で待機中
  * TS_WAIT なら WaitSet で待機中
  * それ以外の値は使われていない


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      enum TStates { TS_UNDEF, TS_READY, TS_RUN, TS_WAIT, TS_ENTER, TS_CXQ } ;
```

* Sorted 	_Sorted
  
  (?? 使用箇所が見当たらない)
  
  なお, Sorted 型は以下のように定義された enum 値.


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      enum Sorted  { PREPEND, APPEND, SORTED } ;
```

* bool 	_active
  
  保守運用機能のためのフィールド (JMM 機能用のフィールド).

  スレッド毎のコンテンション時間／待機時間の取得処理 (java.lang.management.ThreadInfo.get*{Count|Time}() の処理) 
  が有効になっているかどうかを記録している (See: [here](no21146np.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/objectMonitor.hpp))
      ObjectWaiter * volatile _next;
      ObjectWaiter * volatile _prev;
      Thread*       _thread;
      ParkEvent *   _event;
      volatile int  _notified ;
      volatile TStates TState ;
      Sorted        _Sorted ;           // List placement disposition
      bool          _active ;           // Contention monitoring is enabled
```




### 詳細(Details)
See: [here](../doxygen/classObjectWaiter.html) for details

---
