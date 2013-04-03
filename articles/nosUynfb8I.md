---
layout: default
title: ThreadService クラス関連のクラス (ThreadService, ThreadStatistics, ThreadSnapshot, ThreadStackTrace, StackFrameInfo, ThreadConcurrentLocks, ConcurrentLocksDump, ThreadDumpResult, DeadlockCycle, ThreadsListEnumerator, JavaThreadStatusChanger, JavaThreadInObjectWaitState, JavaThreadParkedState, JavaThreadBlockedOnMonitorEnterState, JavaThreadSleepState, 及びそれらの補助クラス(InflatedMonitorsClosure))
---
[Top](../index.html)

#### ThreadService クラス関連のクラス (ThreadService, ThreadStatistics, ThreadSnapshot, ThreadStackTrace, StackFrameInfo, ThreadConcurrentLocks, ConcurrentLocksDump, ThreadDumpResult, DeadlockCycle, ThreadsListEnumerator, JavaThreadStatusChanger, JavaThreadInObjectWaitState, JavaThreadParkedState, JavaThreadBlockedOnMonitorEnterState, JavaThreadSleepState, 及びそれらの補助クラス(InflatedMonitorsClosure))

これらは, Platform MXBean 機能のためのクラス.
より具体的に言うと, java.lang.management.ThreadMXBean クラス
(及びその補助クラスである java.lang.management.ThreadInfo クラス, java.lang.StackTraceElement クラス, java.lang.management.LockInfo クラス)
の実装を担当するクラス.
(See: [here](no2114twV.html) for details)
(See: [here](no21146np.html) for details)

### 備考(Notes)
(完全に Platform MXBean 用ではなく,
 java.lang.Thread.getStackTrace() と java.lang.Thread.getAllStackTraces() からも使われるクラスが一部混じっている)



### クラス一覧(class list)

  * [ThreadService](#nolZFvy9dX)
  * [ThreadSnapshot](#no8jH0WyNB)
  * [ThreadStatistics](#noAC_Q5wbF)
  * [ThreadStackTrace](#noIp8HvvYR)
  * [StackFrameInfo](#noEFfvqGJv)
  * [ThreadConcurrentLocks](#nox_MyTn-7)
  * [ConcurrentLocksDump](#noao3VcBWk)
  * [ThreadDumpResult](#nofnwdigz6)
  * [DeadlockCycle](#noMKQRx82Z)
  * [ThreadsListEnumerator](#noZtm9U5hQ)
  * [JavaThreadStatusChanger](#noHwjGnB-b)
  * [JavaThreadInObjectWaitState](#noSWJOk5ZV)
  * [JavaThreadParkedState](#noEMPtq0QX)
  * [JavaThreadBlockedOnMonitorEnterState](#noC72rVII5)
  * [JavaThreadSleepState](#noh4dL80VK)
  * [InflatedMonitorsClosure](#noakyWScBQ)


---
## <a name="nolZFvy9dX" id="nolZFvy9dX">ThreadService</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する JMM 用の Java クラスからのみ使用される)
(See: java.lang.management.ThreadMXBean).
(See: [here](no2114Ooj.html) and [here](no2114pLf.html) for details)

スレッド関係の Platform MXBean に関する機能を納めた名前空間(AllStatic クラス).

メモリ関係の Platform MXBean 機能へのアクセスを提供するブローカー的なクラス
(このクラスを経由して, 実際の機能を提供する補助クラスが呼び出される).
また, スレッドに関する PerfData もこの名前空間に納められている.

なお, (デフォルトではoffになっているが) スレッドの contension 時間を計測する機能も備えている
(See: ThreadService::is_thread_monitoring_contention()).


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    // VM monitoring and management support for the thread and
    // synchronization subsystem
    //
    // Thread contention monitoring is disabled by default.
    // When enabled, the VM will begin measuring the accumulated
    // elapsed time a thread blocked on synchronization.
    //
    class ThreadService : public AllStatic {
```

### 内部構造(Internal structure)
内部的には, このクラスから様々な補助クラスが呼び出されることで
java.lang.management.ThreadMXBean の機能が実現される.

また, 内部には以下のような Perf データを格納している. 

(ただしコメントによると, PerfData については Thread クラスに移してもいい, とも書かれているが...)
 

```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
      // These counters could be moved to Threads class
      static PerfCounter*  _total_threads_count;
      static PerfVariable* _live_threads_count;
      static PerfVariable* _peak_threads_count;
      static PerfVariable* _daemon_threads_count;
    
      // These 2 counters are atomically incremented once the thread is exiting.
      // They will be atomically decremented when ThreadService::remove_thread is called.
      static volatile int  _exiting_threads_count;
      static volatile int  _exiting_daemon_threads_count;
    
      static bool          _thread_monitoring_contention_enabled;
      static bool          _thread_cpu_time_enabled;
      static bool          _thread_allocated_memory_enabled;
```

Perf で公開されたデータには, それぞれ以下の名前でアクセス可能.

  * java.threads.started
  * java.threads.live
  * java.threads.livePeak
  * java.threads.daemon

```
    ((cite: hotspot/src/share/vm/services/threadService.cpp))
      // These counters are for java.lang.management API support.
      // They are created even if -XX:-UsePerfData is set and in
      // that case, they will be allocated on C heap.
    
      _total_threads_count =
                    PerfDataManager::create_counter(JAVA_THREADS, "started",
                                                    PerfData::U_Events, CHECK);
    
      _live_threads_count =
                    PerfDataManager::create_variable(JAVA_THREADS, "live",
                                                     PerfData::U_None, CHECK);
    
      _peak_threads_count =
                    PerfDataManager::create_variable(JAVA_THREADS, "livePeak",
                                                     PerfData::U_None, CHECK);
    
      _daemon_threads_count =
                    PerfDataManager::create_variable(JAVA_THREADS, "daemon",
                                                     PerfData::U_None, CHECK);
```




### 詳細(Details)
See: [here](../doxygen/classThreadService.html) for details

---
## <a name="no8jH0WyNB" id="no8jH0WyNB">ThreadSnapshot</a>

### 概要(Summary)
Platform MXBean 機能のためのクラス.
より具体的に言うと, java.lang.management.ThreadInfo クラスの実装を担当するクラス.
(See: [here](no2114twV.html) and [here](no2114sqE.html) for details)

(なお正確には, java.lang.Thread.getStackTrace() と java.lang.Thread.getAllStackTraces() からも使われることがある.
 これらのメソッドは, java.lang.management.ThreadInfo 内に含まれる java.lang.StackTraceElement[] の部分だけを使用する.)


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    // Thread snapshot to represent the thread state and statistics
    class ThreadSnapshot : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classThreadSnapshot.html) for details

---
## <a name="noAC_Q5wbF" id="noAC_Q5wbF">ThreadStatistics</a>

### 概要(Summary)
ThreadSnapshot クラス内で使用される補助クラス (See: ThreadSnapshot).

(ThreadSnapshot と同じく) java.lang.management.ThreadInfo クラスを実現するためのクラス.
特にこのクラスは以下のメソッドを実現するために使われる.

* java.lang.management.ThreadInfo.getBlockedCount()
* java.lang.management.ThreadInfo.getBlockedTime()
* java.lang.management.ThreadInfo.getWaitedCount()
* java.lang.management.ThreadInfo.getWaitedTime()

```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    // Per-thread Statistics for synchronization
    class ThreadStatistics : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JavaThread オブジェクトの _thread_stat フィールドに(のみ)格納されている.

(アクセサメソッドは JavaThread::get_thread_stat())

```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    class JavaThread: public Thread {
    ...
     private:
      ThreadStatistics *_thread_stat;
    
     public:
      ThreadStatistics* get_thread_stat() const    { return _thread_stat; }
```

#### 使用箇所(where its instances are used)
java.lang.management.ThreadInfo 用の情報を蓄えるために使用されている
(この情報は ThreadSnapshot オブジェクトへとコピーされ,
 最終的に java.lang.management.ThreadInfo オブジェクトにコピーされる).
(See: [here](no21146np.html) for details)

(なお, JMM からの用途の他に, PerfClassTraceTime で使われることもある.)

### 内部構造(Internal structure)
内部には JavaThread の実行時間に関する統計情報を記録している. 
より具体的に言うと, 以下の３種類の情報を保持している (それぞれの情報毎に 1個の jlong 値および 1個の elapsedTimer を保持している).

* ロックを取得しようとした際に, 既に誰かが取得済みだったためブロックされた回数(およびブロックされていた合計時間)
* java.lang.Object.wait() や sun.misc.Unsafe.park() で待機した回数(及び待機していた合計時間)
* java.lang.Thread.sleep() で寝た回数(および寝ていた合計時間)


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
      jlong        _contended_enter_count;
      elapsedTimer _contended_enter_timer;
      jlong        _monitor_wait_count;
      elapsedTimer _monitor_wait_timer;
      jlong        _sleep_count;
      elapsedTimer _sleep_timer;
```

その他に以下のようなフィールドも持っているが,
これは PerfClassTraceTime クラスから(のみ)使用されている模様.
(See: PerfClassTraceTime)

```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
      // Keep accurate times for potentially recursive class operations
      int           _perf_recursion_counts[6];
      elapsedTimer  _perf_timers[6];
```

### 備考(Notes)
値をリセットする関数も用意されている.
以下の関数が呼び出されるとカウンタの値が初期値に戻る.

```
Java_sun_management_MemoryPoolImpl_resetPeakUsage0()
-> jmm_ResetStatistic()
   -> ThreadService::reset_contention_count_stat()

Java_sun_management_ThreadImpl_resetPeakThreadCount0()
-> jmm_ResetStatistic()
   -> (同上)

Java_sun_management_ThreadImpl_resetContentionTimes0()
-> jmm_ResetStatistic()
   -> (同上)
```




### 詳細(Details)
See: [here](../doxygen/classThreadStatistics.html) for details

---
## <a name="noIp8HvvYR" id="noIp8HvvYR">ThreadStackTrace</a>

### 概要(Summary)
java.lang.StackTraceElement 配列の実装を担当するクラス
(つまり, スレッドのスタックトレース情報を表すためのクラス).
(See: [here](no2114twV.html) for details)
(See: [here](no2114sqE.html) for details)


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    class ThreadStackTrace : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classThreadStackTrace.html) for details

---
## <a name="noEFfvqGJv" id="noEFfvqGJv">StackFrameInfo</a>

### 概要(Summary)
ThreadStackTrace クラス内で使用される補助クラス.

1つの StackFrameInfo オブジェクトが 1つの java.lang.StackTraceElement オブジェクトに対応する
(そして ThreadStackTrace オブジェクトは複数の StackFrameInfo を束ねたようなクラスになっており,
 java.lang.StackTraceElement の配列に対応する).
(See: [here](no2114twV.html) for details)
(See: [here](no2114sqE.html) for details)


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    // StackFrameInfo for keeping methodOop and bci during
    // stack walking for later construction of StackTraceElement[]
    // Java instances
    class StackFrameInfo : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classStackFrameInfo.html) for details

---
## <a name="nox_MyTn-7" id="nox_MyTn-7">ThreadConcurrentLocks</a>

### 概要(Summary)
Platform MXBean 機能のためのクラス.
より具体的に言うと, java.lang.management.LockInfo クラスの実装を担当するクラス
(つまり, スレッドによって現在ロックされているシンクロナイザの一覧を示すためのクラス.
 なおシンクロナイザとは, java.util.concurrent.locks.AbstractOwnableSynchronizer (またはそのサブクラス)
 のインスタンスのこと).
(See: [here](no2114twV.html) and [here](no2114sqE.html) for details)


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    class ThreadConcurrentLocks : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classThreadConcurrentLocks.html) for details

---
## <a name="noao3VcBWk" id="noao3VcBWk">ConcurrentLocksDump</a>

### 概要(Summary)
VM_ThreadDump クラス及び VM_PrintThreads クラス内で使用される補助クラス(StackObjクラス).

現在ロックされているシンクロナイザの情報を収集する
(収集された情報は ThreadConcurrentLocks オブジェクトとして参照される)
(ついでに, その情報を出力する機能も備えている).
(See: [here](no2114sqE.html) for details)


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    class ConcurrentLocksDump : public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
以下の 3つの public メソッドを備えている.
まず dump_at_safepoint() で情報を収集した後, 
thread_concurrent_locks() で取得する or print_locks_on() で出力する, という使い方になる.

* dump_at_safepoint() : 
  ロックされているシンクロナイザの情報を収集する

* thread_concurrent_locks() : 
  収集した情報を基に, 指定されたスレッドがロックしているシンクロナイザの情報を ThreadConcurrentLocks オブジェクトとしてリターンする

* print_locks_on() : 
  収集した情報を出力する.

```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
      void                        dump_at_safepoint();
      ThreadConcurrentLocks*      thread_concurrent_locks(JavaThread* thread);
      void                        print_locks_on(JavaThread* t, outputStream* st);
```

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* VM_PrintThreads::doit() 内 (正確にはその中で呼び出される Threads::print_on() 内)

* VM_ThreadDump::doit() 内

### 内部構造(Internal structure)
なお, ロックされているシンクロナイザの一覧情報を取得する処理は, 実際には HeapInspection クラスにほぼ丸投げしている
(See: HeapInspection).




### 詳細(Details)
See: [here](../doxygen/classConcurrentLocksDump.html) for details

---
## <a name="nofnwdigz6" id="nofnwdigz6">ThreadDumpResult</a>

### 概要(Summary)
ThreadSnapshot クラス用の補助クラス(StackObjクラス).

作業中に生成した ThreadSnapshot オブジェクトを溜めておくためのコンテナクラス
(See: [here](no2114sqE.html) for details).


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    class ThreadDumpResult : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* jmm_GetThreadInfo()

  これは java.lang.management.ThreadInfo オブジェクトを取得する関数
  (See: [here](no2114sqE.html) for details).

* jmm_DumpThreads()

  これは java.lang.management.ThreadInfo オブジェクトを取得する関数
  (See: [here](no2114sqE.html) for details).

* ThreadService::dump_stack_traces()

  これは java.lang.Thread.getStackTrace() および java.lang.Thread.getAllStackTraces() から呼び出される関数.
  (See: [here](no2114ieJ.html) for details)

### 備考(Notes)
ThreadDumpResult オブジェクトは GC 時の strong root になる.
これを記録しておくため, ThreadService::_threaddump_list という static フィールドが用意されている
(See: ThreadService::oops_do()).

新しい ThreadDumpResult オブジェクトが生成されると,
コンストラクタ内で ThreadService::_threaddump_list に追加される.
デストラクタでは逆に外す処理が行われる.




### 詳細(Details)
See: [here](../doxygen/classThreadDumpResult.html) for details

---
## <a name="noMKQRx82Z" id="noMKQRx82Z">DeadlockCycle</a>

### 概要(Summary)
Platform MXBean 機能のためのクラス.
より具体的に言うと, java.lang.management.ThreadMXBean の以下のメソッドの実装を担当するクラス (See: [here](no2114hUk.html) for details).

* java.lang.management.ThreadMXBean.findDeadlockedThreads()
* java.lang.management.ThreadMXBean.findMonitorDeadlockedThreads()

デッドロック検出処理の結果を格納するためのクラス.


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    class DeadlockCycle : public CHeapObj {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ThreadService::find_deadlocks_at_safepoint() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classDeadlockCycle.html) for details

---
## <a name="noZtm9U5hQ" id="noZtm9U5hQ">ThreadsListEnumerator</a>

### 概要(Summary)
java.lang.Thread.getThreads() 及び JVMTI の GetAllThreads() 用の補助クラス(StackObj).

全 JavaThread の一覧を取得する.


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    // Utility class to get list of java threads.
    class ThreadsListEnumerator : public StackObj {
```

### 使われ方(Usage)
#### 使用例(usage examples)
使用方法は次のような感じ.


```
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
      ThreadsListEnumerator tle(THREAD, false, false);
    ...
      int num_threads = tle.num_threads();
    ...
      for (int i = 0; i < num_threads; i++) {
        Handle h = tle.get_threadObj(i);
    ...
      }
```

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* JVM_GetAllThreads()

  (See: [here](no2114A3x.html) for details)

* JvmtiEnv::GetAllThreads()




### 詳細(Details)
See: [here](../doxygen/classThreadsListEnumerator.html) for details

---
## <a name="noHwjGnB-b" id="noHwjGnB-b">JavaThreadStatusChanger</a>

### 概要(Summary)
JavaThread の状態(java_lang_Thread::ThreadStatus)を操作するためのユーティリティ・クラス群(StackObjクラス)の基底クラス (See: [here](no21146np.html) for details).

JavaThread の状態(java_lang_Thread::ThreadStatus)を作業途中のあるスコープの中でだけ変更しておきたい, という場合に使われる
(ついでに, ThreadStatistics の統計情報を収集するという役割もある. (See: [here](no21146np.html) for details)).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    // abstract utility class to set new thread states, and restore previous after the block exits
    class JavaThreadStatusChanger : public StackObj {
```

### 備考(Notes)
JavaThread の状態(java_lang_Thread::ThreadStatus) は, (主に) JVMTI 機能と JMM 機能から使用される情報 (java.lang.Thread の処理でも一部使用されている).

java.lang.Thread オブジェクトの threadStatus フィールドに格納されている
(アクセサメソッドは java_lang_Thread::get_thread_status()/java_lang_Thread::set_thread_status()).

この情報は以下の箇所で(のみ)使用されている.

* java.lang.Thread クラスの処理で内部的に使用されている (See: java.lang.Thread.start(), java.lang.Thread.stop())
* java.lang.Thread.getState() メソッドがリターンする
* JVMTI の GetThreadState() 関数がリターンする
* JVMTI の GetAllStackTraces() 関数がリターンする jvmtiStackInfo 構造体内に含まれている
* JVMTI の GetThreadListStackTraces() がリターンする jvmtiStackInfo 構造体内に含まれている
* JMM で取得できる java.lang.management.ThreadInfo オブジェクト内に含まれている (アクセサメソッドは java.lang.management.ThreadInfo.getThreadState())

### 内部構造(Internal structure)
コンストラクタでは, JavaThreadStatusChanger::save_old_state() を呼んで JavaThread の状態を待避した後
JavaThreadStatusChanger::set_thread_status() で状態を変更する.

逆にデストラクタでは, JavaThread::set_thread_status() で待避していた状態に復帰させる.

```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
      JavaThreadStatusChanger(JavaThread* java_thread,
                              java_lang_Thread::ThreadStatus state) {
        save_old_state(java_thread);
        set_thread_status(state);
      }
    
      JavaThreadStatusChanger(JavaThread* java_thread) {
        save_old_state(java_thread);
      }
    
      ~JavaThreadStatusChanger() {
        set_thread_status(_old_state);
      }
```




### 詳細(Details)
See: [here](../doxygen/classJavaThreadStatusChanger.html) for details

---
## <a name="noSWJOk5ZV" id="noSWJOk5ZV">JavaThreadInObjectWaitState</a>

### 概要(Summary)
JavaThreadStatusChanger クラスの具象サブクラスの1つ.

JavaThread の状態を一時的に IN_OBJECT_WAIT (もしくは IN_OBJECT_WAIT_TIMED) に変更する.


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    // Change status to waiting on an object  (timed or indefinite)
    class JavaThreadInObjectWaitState : public JavaThreadStatusChanger {
```
(See: [here](no21146np.html) for details)

### 使われ方(Usage)
JVM_MonitorWait() 内で(のみ)使用されている (See: [here](no3059BSg.html) for details).

### 内部構造(Internal structure)
コンストラクタで, JavaThread の状態を 
java_lang_Thread::IN_OBJECT_WAIT_TIMED もしくは
java_lang_Thread::IN_OBJECT_WAIT に変更する (どちらに変更するかは, timed 引数の値で決まる).
(ついでに, ThreadStatistics::monitor_wait() 及び 
ThreadStatistics::monitor_wait_begin() を呼び出して統計情報の取得も行っている. (See: [here](no21146np.html) for details))

デストラクタでは (基底クラスである JavaThreadStatusChanger のデストラクタを用いて) 状態を元に戻している.
(ついでに, ThreadStatistics::monitor_wait_end() を呼び出して統計情報の取得も行っている. (See: [here](no21146np.html) for details)).


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
      JavaThreadInObjectWaitState(JavaThread *java_thread, bool timed) :
        JavaThreadStatusChanger(java_thread,
                                timed ? java_lang_Thread::IN_OBJECT_WAIT_TIMED : java_lang_Thread::IN_OBJECT_WAIT) {
        if (is_alive()) {
          _stat = java_thread->get_thread_stat();
          _active = ThreadService::is_thread_monitoring_contention();
          _stat->monitor_wait();
          if (_active) {
            _stat->monitor_wait_begin();
          }
        } else {
          _active = false;
        }
      }
    
      ~JavaThreadInObjectWaitState() {
        if (_active) {
          _stat->monitor_wait_end();
        }
      }
```




### 詳細(Details)
See: [here](../doxygen/classJavaThreadInObjectWaitState.html) for details

---
## <a name="noEMPtq0QX" id="noEMPtq0QX">JavaThreadParkedState</a>

### 概要(Summary)
JavaThreadStatusChanger クラスの具象サブクラスの1つ.

JavaThread の状態を一時的に PARKED (もしくは PARKED_TIMED) に変更する.


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    // Change status to parked (timed or indefinite)
    class JavaThreadParkedState : public JavaThreadStatusChanger {
```
(See: [here](no21146np.html) for details)

### 使われ方(Usage)
Unsafe_Park() 内で(のみ)使用されている (See: [here](no2114PYi.html) for details).

### 内部構造(Internal structure)
コンストラクタで, JavaThread の状態を 
java_lang_Thread::PARKED_TIMED もしくは
java_lang_Thread::PARKED に変更する (どちらに変更するかは, timed 引数の値で決まる).
(ついでに, ThreadStatistics::monitor_wait() 及び 
ThreadStatistics::monitor_wait_begin() を呼び出して統計情報の取得も行っている. (See: [here](no21146np.html) for details)).

デストラクタでは (基底クラスである JavaThreadStatusChanger のデストラクタを用いて) 状態を元に戻している.
(ついでに, ThreadStatistics::monitor_wait_end() を呼び出して統計情報の取得も行っている. (See: [here](no21146np.html) for details)).


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
     public:
      JavaThreadParkedState(JavaThread *java_thread, bool timed) :
        JavaThreadStatusChanger(java_thread,
                                timed ? java_lang_Thread::PARKED_TIMED : java_lang_Thread::PARKED) {
        if (is_alive()) {
          _stat = java_thread->get_thread_stat();
          _active = ThreadService::is_thread_monitoring_contention();
          _stat->monitor_wait();
          if (_active) {
            _stat->monitor_wait_begin();
          }
        } else {
          _active = false;
        }
      }
    
      ~JavaThreadParkedState() {
        if (_active) {
          _stat->monitor_wait_end();
        }
      }
```




### 詳細(Details)
See: [here](../doxygen/classJavaThreadParkedState.html) for details

---
## <a name="noC72rVII5" id="noC72rVII5">JavaThreadBlockedOnMonitorEnterState</a>

### 概要(Summary)
JavaThreadStatusChanger クラスの具象サブクラスの1つ.

JavaThread の状態を一時的に BLOCKED_ON_MONITOR_ENTER に変更する.


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    // Change status to blocked on (re-)entering a synchronization block
    class JavaThreadBlockedOnMonitorEnterState : public JavaThreadStatusChanger {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* ObjectMonitor::enter()

  これはロック確保処理の slow-path や JNI によるロック確保処理(MonitorEnter())で使用される関数
  (See: [here](no96623Ns.html) and [here](no5248b4E.html) for details).

* ObjectWaiter::wait_reenter_begin()

  これは java.lang.Object.notify() 及び java.lang.Object.notifyAll() の処理で使用される関数
  (See: [here](no3059BSg.html) for details).

* ObjectWaiter::wait_reenter_end()

  これは java.lang.Object.wait() の処理で使用される関数
  (See: [here](no3059BSg.html) for details).

なお, このクラスは(コンストラクタとデストラクタ以外に)以下の2つの static メソッドを持っている.
ObjectWaiter::wait_reenter_begin() と ObjectWaiter::wait_reenter_end() の処理ではこちらが使われている.

  * JavaThreadBlockedOnMonitorEnterState::wait_reenter_begin()
  * JavaThreadBlockedOnMonitorEnterState::wait_reenter_end()

### 内部構造(Internal structure)
コンストラクタで, JavaThreadBlockedOnMonitorEnterState::contended_enter_begin() を呼び出して
JavaThread の状態を java_lang_Thread::BLOCKED_ON_MONITOR_ENTER に変更する.
(ついでに, ThreadStatistics::contended_enter() を呼び出して統計情報の取得も行っている. (See: [here](no21146np.html) for details)).

デストラクタでは (基底クラスである JavaThreadStatusChanger のデストラクタを用いて) 状態を元に戻している.
(ついでに, ThreadStatistics::contended_enter_end() を呼び出して統計情報の取得も行っている. (See: [here](no21146np.html) for details)).


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
      JavaThreadBlockedOnMonitorEnterState(JavaThread *java_thread, ObjectMonitor *obj_m) :
        JavaThreadStatusChanger(java_thread) {
        assert((java_thread != NULL), "Java thread should not be null here");
        // Change thread status and collect contended enter stats for monitor contended
        // enter done for external java world objects and it is contended. All other cases
        // like for vm internal objects and for external objects which are not contended
        // thread status is not changed and contended enter stat is not collected.
        _active = false;
        if (is_alive() && ServiceUtil::visible_oop((oop)obj_m->object()) && obj_m->contentions() > 0) {
          _stat = java_thread->get_thread_stat();
          _active = contended_enter_begin(java_thread);
        }
      }
    
      ~JavaThreadBlockedOnMonitorEnterState() {
        if (_active) {
          _stat->contended_enter_end();
        }
      }
```




### 詳細(Details)
See: [here](../doxygen/classJavaThreadBlockedOnMonitorEnterState.html) for details

---
## <a name="noh4dL80VK" id="noh4dL80VK">JavaThreadSleepState</a>

### 概要(Summary)
JavaThreadStatusChanger クラスの具象サブクラスの1つ.

JavaThread の状態を一時的に SLEEPING に変更する.


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
    // Change status to sleeping
    class JavaThreadSleepState : public JavaThreadStatusChanger {
```

### 使われ方(Usage)
JVM_Sleep() 内で(のみ)使用されている (See: [here](no2114aSy.html) for details).

### 内部構造(Internal structure)
コンストラクタで, JavaThread の状態を 
java_lang_Thread::SLEEPING に変更する.
(ついでに, ThreadStatistics::thread_sleep() 及び 
ThreadStatistics::thread_sleep_begin() を呼び出して統計情報の取得も行っている. (See: [here](no21146np.html) for details)).

デストラクタでは (基底クラスである JavaThreadStatusChanger のデストラクタを用いて) 状態を元に戻している.
(ついでに, ThreadStatistics::thread_sleep_end() を呼び出して統計情報の取得も行っている. (See: [here](no21146np.html) for details)).


```
    ((cite: hotspot/src/share/vm/services/threadService.hpp))
      JavaThreadSleepState(JavaThread *java_thread) :
        JavaThreadStatusChanger(java_thread, java_lang_Thread::SLEEPING) {
        if (is_alive()) {
          _stat = java_thread->get_thread_stat();
          _active = ThreadService::is_thread_monitoring_contention();
          _stat->thread_sleep();
          if (_active) {
            _stat->thread_sleep_begin();
          }
        } else {
          _active = false;
        }
      }
    
      ~JavaThreadSleepState() {
        if (_active) {
          _stat->thread_sleep_end();
        }
      }
```




### 詳細(Details)
See: [here](../doxygen/classJavaThreadSleepState.html) for details

---
## <a name="noakyWScBQ" id="noakyWScBQ">InflatedMonitorsClosure</a>

### 概要(Summary)
ThreadStackTrace クラス内で使用される補助クラス.

JNI 関数内で取得されているロックの一覧を作成する.

```
    ((cite: hotspot/src/share/vm/services/threadService.cpp))
    // Iterate through monitor cache to find JNI locked monitors
    class InflatedMonitorsClosure: public MonitorClosure {
```

### 使われ方(Usage)
ThreadStackTrace::dump_stack_at_safepoint() 内で(のみ)使用されている (See: [here](no2114sqE.html) for details).

### 内部構造(Internal structure)
#### 参考(for your information): InflatedMonitorsClosure::do_monitor()
See: [here](no2114HAY.html) for details



### 詳細(Details)
See: [here](../doxygen/classInflatedMonitorsClosure.html) for details

---
