---
layout: default
title: ConcurrentGCThread クラス関連のクラス (SuspendibleThreadSet, ConcurrentGCThread, SurrogateLockerThread)
---
[Top](../index.html)

#### ConcurrentGCThread クラス関連のクラス (SuspendibleThreadSet, ConcurrentGCThread, SurrogateLockerThread)

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, JavaThread の処理と並行して(concurrentに) GC 処理を行うためのクラス (See: [here](no7882ZnJ.html) for details).


### クラス一覧(class list)

  * [ConcurrentGCThread](#nofT9wf5wn)
  * [SuspendibleThreadSet](#noTlnyy8dl)
  * [SurrogateLockerThread](#no9C6v9Q0o)


---
## <a name="nofT9wf5wn" id="nofT9wf5wn">ConcurrentGCThread</a>

### 概要(Summary)
JavaThread (Mutator) と並行して GC 処理を行うための並行スレッドクラス (の基底クラス) (See: [here](no7882ZnJ.html) for details).

concurrent な GC 処理を行うスレッドは ConcurrentGCThread のサブクラスとして実装される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.hpp))
    class ConcurrentGCThread: public NamedThread {
```

なお, このクラス自体は abstract class であり, 実際に使われるのは以下のサブクラス.

* ConcurrentMarkSweepThread

  CMS 用の concurrent marking 処理を行うスレッド

* ConcurrentMarkThread

  G1GC 用の concurrent marking 処理を行うスレッド

* ConcurrentG1RefineThread

  G1GC 用の refine 処理 (Remembered Set を適切に保つ処理) を行うスレッド




### 詳細(Details)
See: [here](../doxygen/classConcurrentGCThread.html) for details

---
## <a name="noTlnyy8dl" id="noTlnyy8dl">SuspendibleThreadSet</a>

### 概要(Summary)
複数の ConcurrentGCThread 間で同期を取るためのクラス.


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.hpp))
    // A SuspendibleThreadSet is (obviously) a set of threads that can be
    // suspended.  A thread can join and later leave the set, and periodically
    // yield.  If some thread (not in the set) requests, via suspend_all, that
    // the threads be suspended, then the requesting thread is blocked until
    // all the threads in the set have yielded or left the set.  (Threads may
    // not enter the set when an attempted suspension is in progress.)  The
    // suspending thread later calls resume_all, allowing the suspended threads
    // to continue.
    
    class SuspendibleThreadSet {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. suspend 対象になりたいスレッドは, 適当な SuspendibleThreadSet オブジェクトに対して SuspendibleThreadSet::join() を呼び出す.

   これにより, 呼び出したスレッドがその SuspendibleThreadSet オブジェクトに登録される.

   (なお登録後にsuspend 対象から外れたくなったら, SuspendibleThreadSet::leave() を呼び出せばよい)

2. あるスレッドが, ある SuspendibleThreadSet 内の全スレッドを停止させたくなった場合は,
   その SuspendibleThreadSet に対して SuspendibleThreadSet::suspend_all() を呼び出す.

   (呼び出したスレッドは, その SuspendibleThreadSet オブジェクト内の全部のスレッドが実際に停止する
    (あるいは SuspendibleThreadSet::leave() で登録を解除する)までブロックされる)

3. SuspendibleThreadSet オブジェクトに自分を登録したスレッドは, 適当なタイミングで SuspendibleThreadSet::yield() を呼び出す.

   (SuspendibleThreadSet::yield() は, その SuspendibleThreadSet オブジェクトに対して
   SuspendibleThreadSet::suspend_all() が呼ばれていれば, 呼び出したスレッドを停止させる)

4. SuspendibleThreadSet::suspend_all() を呼び出したスレッドは,
   処理が終わったら SuspendibleThreadSet::resume_all() を呼び出す.

   (SuspendibleThreadSet::resume_all() は, yield() でブロックしているスレッドを Monitor::notify_all() でたたき起こす)


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.hpp))
      // Add the current thread to the set.  May block if a suspension
      // is in progress.
      void join();
      // Removes the current thread from the set.
      void leave();
      // Returns "true" iff an suspension is in progress.
      bool should_yield() { return _async_stop; }
      // Suspends the current thread if a suspension is in progress (for
      // the duration of the suspension.)
      void yield(const char* id);
      // Return when all threads in the set are suspended.
      void suspend_all();
      // Allow suspended threads to resume.
      void resume_all();
      // Redundant initializations okay.
```

#### インスタンスの格納場所(where its instances are stored)
ConcurrentGCThread クラスの _sts フィールド (static フィールド) に(のみ)格納されている.

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.hpp))
      // All instances share this one set.
      static SuspendibleThreadSet _sts;
```




### 詳細(Details)
See: [here](../doxygen/classSuspendibleThreadSet.html) for details

---
## <a name="no9C6v9Q0o" id="no9C6v9Q0o">SurrogateLockerThread</a>

### 概要(Summary)
ConcurrentGCThread から Java のモニタを操作するための Thread クラス.

なお, 現在は ConcurrentGCThread と ReferenceHandler スレッド (= java.lang.ref.Reference オブジェクトを処理するスレッド) 
が協調する用途で(のみ)使用されている
(SurrogateLockerThread スレッドは, ConcurrentGCThread による GC が始まる前に
java.lang.ref の pending_list_lock を取得して ReferenceHandler の処理と排他する役割を担っている) (See: [here](no7882OK1.html) for details).


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.hpp))
    // The SurrogateLockerThread is used by concurrent GC threads for
    // manipulating Java monitors, in particular, currently for
    // manipulating the pending_list_lock. XXX
    class SurrogateLockerThread: public JavaThread {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
SurrogateLockerThread::make() 内で(のみ)生成されている (See: [here](no7882OK1.html) for details).

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.
ただし, このクラスのインスタンス自体は1つしか存在しない (どちらも同じインスタンスを指している).

* ConcurrentMarkSweepThread クラスの _slt フィールド (static フィールド), または ConcurrentMarkThread クラスの _slt フィールド (static フィールド), 

  どちらに格納されるかは, 使用する GC アルゴリズムによって決まる (CMS なら ConcurrentMarkSweepThread, G1GC なら ConcurrentMarkThread).


```
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/concurrentMarkSweepThread.hpp))
    class ConcurrentMarkSweepThread: public ConcurrentGCThread {
    ...
      static SurrogateLockerThread*         _slt;
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMarkThread.hpp))
    class ConcurrentMarkThread: public ConcurrentGCThread {
    ...
      static SurrogateLockerThread*         _slt;
```

* Threads クラスの _thread_list フィールド (static フィールド)

  (正確には, このフィールドは JavaThread の線形リストを格納するフィールド.
  JavaThread オブジェクトは _next フィールドで次の JavaThread オブジェクトを指せる構造になっている.
  生成した JavaThread オブジェクトは全てこの線形リスト内に格納されている)

  (なお, この線形リストには Threads::add() で登録される)




### 詳細(Details)
See: [here](../doxygen/classSurrogateLockerThread.html) for details

---
