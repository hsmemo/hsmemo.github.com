---
layout: default
title: GCTaskManager クラス関連のクラス (GCTask, GCTask::Kind, GCTaskQueue, SynchronizedGCTaskQueue, NotifyDoneClosure, GCTaskManager, NoopGCTask, BarrierGCTask, ReleasingBarrierGCTask, NotifyingBarrierGCTask, WaitForBarrierGCTask, MonitorSupply)
---
[Top](../index.html)

#### GCTaskManager クラス関連のクラス (GCTask, GCTask::Kind, GCTaskQueue, SynchronizedGCTaskQueue, NotifyDoneClosure, GCTaskManager, NoopGCTask, BarrierGCTask, ReleasingBarrierGCTask, NotifyingBarrierGCTask, WaitForBarrierGCTask, MonitorSupply)

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, ParallelScavenge GC において, GC スレッド (GCTaskThread) に対する GC 処理要求を管理するためのクラス
(See: [here](no24805iK.html) for details).


### クラス一覧(class list)

  * [GCTask](#notduakcBI)
  * [GCTask::Kind](#noUaV_9M2n)
  * [GCTaskQueue](#noQXEteMXN)
  * [SynchronizedGCTaskQueue](#noHV-799E-)
  * [NotifyDoneClosure](#nolMRpyk74)
  * [GCTaskManager](#no5enQ87G5)
  * [NoopGCTask](#nofk7VGLgZ)
  * [BarrierGCTask](#noM4DXPY72)
  * [ReleasingBarrierGCTask](#no2IsnCUHb)
  * [NotifyingBarrierGCTask](#nosLyUWWnr)
  * [WaitForBarrierGCTask](#nowy5ZORuW)
  * [MonitorSupply](#nojoulp5cx)


---
## <a name="notduakcBI" id="notduakcBI">GCTask</a>

### 概要(Summary)
GCTaskThread に対する処理要求を表すクラスの基底クラス (See: [here](no24805iK.html) for details).

GCTaskThread に対する処理要求は GCTask クラス(のサブクラス)のオブジェクトとして表現される.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    // The abstract base GCTask.
    class GCTask : public ResourceObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classGCTask.html) for details

---
## <a name="noUaV_9M2n" id="noUaV_9M2n">GCTask::Kind</a>

### 概要(Summary)
GCTask 用の補助クラス (See: [here](no24805iK.html) for details).

GCTask の種別を表す定数値を納めた名前空間(AllStatic クラス)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
      // Known kinds of GCTasks, for predicates.
      class Kind : AllStatic {
```

### 内部構造(Internal structure)
現状では, 種別を示す以下の定数値が定義されているだけ.

(正確には, この定数値を文字列化する to_string() メソッドも定義されている)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
        enum kind {
          unknown_task,
          ordinary_task,
          barrier_task,
          noop_task
        };
```




### 詳細(Details)
See: [here](../doxygen/classGCTask_1_1Kind.html) for details

---
## <a name="noQXEteMXN" id="noQXEteMXN">GCTaskQueue</a>

### 概要(Summary)
GCTask を詰めるキュー (というか doubly-linked list).
GCTask はこのキューに詰められて GCTaskThread に渡され,  
GCTaskThread はここから GCTask を取り出して処理を行う
(See: [here](no24805iK.html) and [here](no28916egj.html) for details).

なお, キューの操作は MT-safe ではないので, 
排他的に処理したい場合は SynchronizedGCTaskQueue でラップして使うこと, とのこと.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    // A doubly-linked list of GCTasks.
    // The list is not synchronized, because sometimes we want to
    // build up a list and then make it available to other threads.
    // See also: SynchronizedGCTaskQueue.
    class GCTaskQueue : public ResourceObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
GCTaskQueue には現状では2つの使われ方がある.

  * GCTaskManager 内で使用されるキュー
    (なお, こちらの場合は SynchronizedGCTaskQueue でラップした状態で使われる (See: SynchronizedGCTaskQueue))
  * GCTaskManager に渡される GCTask を詰めるためのキュー

GCTask は正確には GCTaskManager クラス経由で GCTaskThread に渡される.

GCTaskManager は, それ自身が (内部的に GCTaskQueue を用いた) キューのようなクラスになっており,
GCTaskThread はここから自分が実行する GCTask を取得する.

GC 処理を要求するスレッドは, 新しい GCTaskQueue を作成し, 実行して欲しい GCTask をそこに詰めて GCTaskManager に渡す.
GCTaskManager は, 渡された GCTaskQueue の中身を自身の GCTaskQueue に移し替える.
そしてその中身が GCTaskThread へと伝えられる.

#### 生成箇所(where its instances are created)
GCTaskQueue::create_on_c_heap() と GCTaskQueue::create() という2つのファクトリメソッドが用意されており,
その中で(のみ)生成されている.

* GCTaskQueue::create_on_c_heap() は, GCTaskManager が保持する GCTaskQueue を生成するファクトリメソッド

  (GCTaskManager::initialize() からのみ呼び出されている)
  (GCTaskManager 用のキューは, 開放されるとまずいので, C ヒープ上に確保する).

* GCTaskQueue::create() は, GC 処理を要求するスレッドが使用するファクトリメソッド.

  GC 処理を要求するスレッドは, このメソッドが生成した GCTaskQueue に GCTask を詰めて GCTaskManager に引き渡す.

#### 参考(for your information): GCTaskQueue::create()
See: [here](no7882QoT.html) for details
#### 参考(for your information): GCTaskQueue::create_on_c_heap()
See: [here](no7882dyZ.html) for details



### 詳細(Details)
See: [here](../doxygen/classGCTaskQueue.html) for details

---
## <a name="noHV-799E-" id="noHV-799E-">SynchronizedGCTaskQueue</a>

### 概要(Summary)
排他的にキューへの挿入が行える GCTaskQueue.

(といっても, 現在は GCTaskManager クラス内でしか使われていないので, GCTaskManager 用の補助クラスといった感じ)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    // A GCTaskQueue that can be synchronized.
    // This "has-a" GCTaskQueue and a mutex to do the exclusion.
    class SynchronizedGCTaskQueue : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
GCTaskManager クラスの _queue フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    class GCTaskManager : public CHeapObj {
    ...
      SynchronizedGCTaskQueue*  _queue;             // Queue of tasks.
```

#### 生成箇所(where its instances are created)
SynchronizedGCTaskQueue::create() というファクトリメソッドが用意されており, その中で(のみ)生成されている.

(そして, このファクトリメソッドは GCTaskManager::initialize() 内で(のみ)呼び出されている)

#### 参考(for your information): SynchronizedGCTaskQueue::create()
See: [here](no7882Qhf.html) for details
#### 参考(for your information): GCTaskManager::initialize()
See: [here](no7882Qar.html) for details
### 内部構造(Internal structure)
なお, このクラス自体は GCTaskQueue のサブクラスにはなっていない.

その代わりに, private フィールドに GCTaskQueue オブジェクトを保持して明示的に delegate している.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    private:
      // Instance state.
      GCTaskQueue* _unsynchronized_queue;   // Has-a unsynchronized queue.
      Monitor *    _lock;                   // Lock to control access.
```




### 詳細(Details)
See: [here](../doxygen/classSynchronizedGCTaskQueue.html) for details

---
## <a name="nolMRpyk74" id="nolMRpyk74">NotifyDoneClosure</a>

### 概要(Summary)
GCTaskManager からのコールバックを表すクラスの基底クラス.

(GCTaskManager 内の GCTask が全て完了すると呼び出される模様)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    // This is an abstract base class for getting notifications
    // when a GCTaskManager is done.
    class NotifyDoneClosure : public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

### 使われ方(Usage)
肝心のサブクラスが見当たらないような... #TODO

それに, GCTaskManager にはコンストラクタの第2引数で渡せることになっているのに,
引数1個のコンストラクタしか使われていないような... #TODO

(WaitForBarrierGCTask があるからこのクラスは要らない?? #TODO)




### 詳細(Details)
See: [here](../doxygen/classNotifyDoneClosure.html) for details

---
## <a name="no5enQ87G5" id="no5enQ87G5">GCTaskManager</a>

### 概要(Summary)
ParallelScavenge において, GCTaskThread や GCTask の管理を行うクラス (See: [here](no24805iK.html) for details).

GCTaskThread を生成したり, GCTaskThread に GC 要求が渡される際のブローカーとして働いたりする

(GC 処理を要求したいスレッドは GCTaskManager 経由で GCTaskThread に要求を出す.
GCTaskManager は, それ自身が (内部的に GCTaskQueue を用いた) キューのようなクラスになっており,
GCTaskThread はここから自分が実行する GCTask を取得する)



```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    // The GCTaskManager is a queue of GCTasks, and accessors
    // to allow the queue to be accessed from many threads.
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    class GCTaskManager : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ParallelScavengeHeap オブジェクトの  _gc_task_manager フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.hpp))
    class ParallelScavengeHeap : public CollectedHeap {
    ...
      static GCTaskManager*          _gc_task_manager;      // The task manager.
```

#### 生成箇所(where its instances are created)
GCTaskManager::create(uint workers) と GCTaskManager::create(uint workers, NotifyDoneClosure* ndc) という2つのファクトリメソッドが用意されており,
その中で(のみ)生成されている.

* GCTaskManager::create(uint workers)

  ParallelScavengeHeap::initialize() 内で(のみ)呼び出されている.

* GCTaskManager::create(uint workers, NotifyDoneClosure* ndc)

  (こちらは使われていないような... #TODO)

#### 参考(for your information): GCTaskManager::create(uint workers)
See: [here](no78823Gm.html) for details
#### 参考(for your information): ParallelScavengeHeap::initialize()
See: [here](no344Yjc.html) for details
#### 参考(for your information): GCTaskManager::create(uint workers, NotifyDoneClosure* ndc)
See: [here](no7882ERs.html) for details



### 詳細(Details)
See: [here](../doxygen/classGCTaskManager.html) for details

---
## <a name="nofk7VGLgZ" id="nofk7VGLgZ">NoopGCTask</a>

### 概要(Summary)
GCTask クラスのサブクラスの1つ.

このクラスは「何も行わない処理(NOOP)」を表す.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    // A noop task that does nothing,
    // except take us around the GCTaskThread loop.
    class NoopGCTask : public GCTask {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
GCTaskThread が確保しているリソースを解放したいときなど, とりあえず GCTaskThread の処理ループを1回まわしたい, という時に使われる模様(?)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.cpp))
        // The queue is empty, but we were woken up.
        // Just hand back a Noop task,
        // in case someone wanted us to release resources, or whatever.
        result = noop_task();
```

#### インスタンスの格納場所(where its instances are stored)
GCTaskManager オブジェクトの _noop_task フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    class GCTaskManager : public CHeapObj {
    ...
      NoopGCTask*               _noop_task;         // The NoopGCTask instance.
```

#### 生成箇所(where its instances are created)
NoopGCTask::create() と NoopGCTask::create_on_c_heap() という2つのファクトリメソッドが用意されており,
その中で(のみ)生成されている.

* NoopGCTask::create()

  (こちらは使われていないような... #TODO)

* NoopGCTask::create_on_c_heap()

  GCTaskManager::initialize() 内で(のみ)呼び出されている.

#### 参考(for your information): NoopGCTask::create()
See: [here](no2480uMq.html) for details
#### 参考(for your information): NoopGCTask::create_on_c_heap()
See: [here](no7882QvH.html) for details
#### 参考(for your information): GCTaskManager::initialize()
See: [here](no7882Qar.html) for details



### 詳細(Details)
See: [here](../doxygen/classNoopGCTask.html) for details

---
## <a name="noM4DXPY72" id="noM4DXPY72">BarrierGCTask</a>

### 概要(Summary)
GCTask クラスのサブクラスの1つ.

このクラスはバリア同期として働く (先行するタスクが全部完了するまで後続のタスクを開始させない).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    // A BarrierGCTask blocks other tasks from starting,
    // and waits until it is the only task running.
    class BarrierGCTask : public GCTask {
```




### 詳細(Details)
See: [here](../doxygen/classBarrierGCTask.html) for details

---
## <a name="no2IsnCUHb" id="no2IsnCUHb">ReleasingBarrierGCTask</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらないような...)

BarrierGCTask クラスのサブクラスの1つ.

バリアを張るついでに, 全ての GCTaskThread に対して 
resource area を解放するよう要求を出す.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    // A ReleasingBarrierGCTask is a BarrierGCTask
    // that tells all the tasks to release their resource areas.
    class ReleasingBarrierGCTask : public BarrierGCTask {
```




### 詳細(Details)
See: [here](../doxygen/classReleasingBarrierGCTask.html) for details

---
## <a name="nosLyUWWnr" id="nosLyUWWnr">NotifyingBarrierGCTask</a>

### 概要(Summary)
BarrierGCTask クラスのサブクラスの1つ.

バリアを張るついでに, 先行するタスクが全て完了したら 
NotifyDoneClosure での通知も行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    // A NotifyingBarrierGCTask is a BarrierGCTask
    // that calls a notification method when it is the only task running.
    class NotifyingBarrierGCTask : public BarrierGCTask {
```

### 使われ方(Usage)
(このクラスの使用箇所は見当たらないが... #TODO)




### 詳細(Details)
See: [here](../doxygen/classNotifyingBarrierGCTask.html) for details

---
## <a name="nowy5ZORuW" id="nowy5ZORuW">WaitForBarrierGCTask</a>

### 概要(Summary)
BarrierGCTask クラスのサブクラスの1つ.

呼び出し元がバリア完了時までブロックされるメソッド(同期式のメソッド)を備えている.
(NotifyingBarrierGCTask の多くのユースケースはこれでカバーできる, とのこと)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    // A WaitForBarrierGCTask is a BarrierGCTask
    // with a method you can call to wait until
    // the BarrierGCTask is done.
    // This may cover many of the uses of NotifyingBarrierGCTasks.
    class WaitForBarrierGCTask : public BarrierGCTask {
```

### 内部構造(Internal structure)
WaitForBarrierGCTask::wait_for() メソッドで待機を行う.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
      void     wait_for();
```




### 詳細(Details)
See: [here](../doxygen/classWaitForBarrierGCTask.html) for details

---
## <a name="nojoulp5cx" id="nojoulp5cx">MonitorSupply</a>

### 概要(Summary)
WaitForBarrierGCTask クラス内で使用される補助クラス.

バリア同期を待つための Monitor (およびそれを操作するメソッド) を納めた名前空間(AllStatic クラス).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    class MonitorSupply : public AllStatic {
```

メソッドとしては以下の2種類が定義されている.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    public:
      // Reserve a Monitor*.
      static Monitor* reserve();
      // Release a Monitor*.
      static void release(Monitor* instance);
```

### 使われ方(Usage)
WaitForBarrierGCTask のコンストラクタで Monitor が取得される.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.cpp))
    WaitForBarrierGCTask::WaitForBarrierGCTask(bool on_c_heap) :
    ...
      _monitor = MonitorSupply::reserve();
```

そして WaitForBarrierGCTask::wait_for() 内で, その Monitor に対して wait() が呼び出される.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.cpp))
    void WaitForBarrierGCTask::wait_for() {
    ...
          monitor()->wait(Mutex::_no_safepoint_check_flag, 0);
```

そして WaitForBarrierGCTask::do_it() 内で notify_all() が呼び出される.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.cpp))
    void WaitForBarrierGCTask::do_it(GCTaskManager* manager, uint which) {
    ...
        monitor()->notify_all();
        // Release monitor().
```




### 詳細(Details)
See: [here](../doxygen/classMonitorSupply.html) for details

---
