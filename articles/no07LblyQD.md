---
layout: default
title: Thread クラス関連のクラス (Thread, NamedThread, WorkerThread, WatcherThread, JavaThread, CompilerThread, Threads, ThreadClosure, SignalHandlerMark, 及びそれらの補助クラス(TraceSuspendDebugBits, RememberProcessedThread))
---
[Top](../index.html)

#### Thread クラス関連のクラス (Thread, NamedThread, WorkerThread, WatcherThread, JavaThread, CompilerThread, Threads, ThreadClosure, SignalHandlerMark, 及びそれらの補助クラス(TraceSuspendDebugBits, RememberProcessedThread))

これらは, HotSpot 内でのスレッド管理用のクラス
(See: [here](no1BWFggBW.html) and [here](noJb9iXZL-.html) for details).

なお, これらのクラスは以下のような継承関係を持つ.


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    // Class hierarchy
    // - Thread
    //   - NamedThread
    //     - VMThread
    //     - ConcurrentGCThread
    //     - WorkerThread
    //       - GangWorker
    //       - GCTaskThread
    //   - JavaThread
    //   - WatcherThread
```


### クラス一覧(class list)

  * [Thread](#no6OKLH09q)
  * [NamedThread](#noHxzZQqSH)
  * [WorkerThread](#noPJFT8RVN)
  * [WatcherThread](#nowYOeoQTW)
  * [JavaThread](#no0sjGpj7a)
  * [CompilerThread](#noTyQ47dX8)
  * [Threads](#noUPbrTXoG)
  * [ThreadClosure](#nozTxng1HA)
  * [SignalHandlerMark](#noAniqPE4W)
  * [TraceSuspendDebugBits](#noJEkHf2pz)
  * [RememberProcessedThread](#noAFtgUg1Z)


---
## <a name="no6OKLH09q" id="no6OKLH09q">Thread</a>

### 概要(Summary)
HotSpot 内で使用されるスレッドを表すクラス (の基底クラス).
1つの Thread オブジェクトが 1つのスレッドに対応する.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    class Thread: public ThreadShadow {
```




### 詳細(Details)
See: [here](../doxygen/classThread.html) for details

---
## <a name="noHxzZQqSH" id="noHxzZQqSH">NamedThread</a>

### 概要(Summary)
HotSpot 内で特殊な用途に使用されるスレッドの基底クラス
(より具体的に言うと JavaThread でも WatcherThread でもない全てのスレッドの基底クラス).

このクラス(のサブクラス)のスレッドオブジェクトはそれぞれが固有の名前(name)を持つ (これが "NamedThread" というクラス名の由来).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    // Name support for threads.  non-JavaThread subclasses with multiple
    // uniquely named instances should derive from this.
    class NamedThread: public Thread {
```




### 詳細(Details)
See: [here](../doxygen/classNamedThread.html) for details

---
## <a name="noPJFT8RVN" id="noPJFT8RVN">WorkerThread</a>

### 概要(Summary)
HotSpot 内の何らかの処理をマルチスレッド化するためのスレッドクラス (の基底クラス).

(なお, このクラス(のサブクラス)は, 現在は GC 処理を並列化するためにのみ使用されている)

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    // Worker threads are named and have an id of an assigned work.
    class WorkerThread: public NamedThread {
```




### 詳細(Details)
See: [here](../doxygen/classWorkerThread.html) for details

---
## <a name="nowYOeoQTW" id="nowYOeoQTW">WatcherThread</a>

### 概要(Summary)
HotSpot 内で定期的に実行しなければならない処理を実現するためのスレッドクラス (See: [here](nohcAO37b3.html) for details).

なお, WatcherThread によって実行される処理は PeriodicTask クラス(のサブクラス)として表現されている 
(See: PeriodicTask).


```
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
    // The watcher thread exists to simulate timer interrupts.  It should
    // be replaced by an abstraction over whatever native support for
    // timer interrupts exists on the platform.
```


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    // A single WatcherThread is used for simulating timer interrupts.
    class WatcherThread: public Thread {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
WatcherThread クラスの _watcher_thread フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
WatcherThread::start() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

(ただし, この時点で PeriodicTask オブジェクトが 1つも存在していなければ生成されない)

```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> WatcherThread::start()
```




### 詳細(Details)
See: [here](../doxygen/classWatcherThread.html) for details

---
## <a name="no0sjGpj7a" id="no0sjGpj7a">JavaThread</a>

### 概要(Summary)
実際に Java プログラムを実行するスレッドを表すスレッドクラス
(つまり java.lang.Thread を表すスレッドクラス).
1つの JavaThread オブジェクトが 1つの java.lang.Thread オブジェクトに対応する.


```
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
    // A JavaThread is a normal Java thread
```


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    class JavaThread: public Thread {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Threads クラスの _thread_list フィールド (static フィールド) に(のみ)格納されている.

(正確には, このフィールドは JavaThread の線形リストを格納するフィールド.
JavaThread オブジェクトは _next フィールドで次の JavaThread オブジェクトを指せる構造になっている.
生成した JavaThread オブジェクトは全てこの線形リスト内に格納されている)

(なお, この線形リストには Threads::add() で登録される)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

```
* メインスレッド用の JavaThread オブジェクトの生成処理

  (HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
  -> Threads::create_vm()

* シグナル処理を行うためのスレッド("Signal Dispatcher" スレッド)の生成処理
  
  (略) (See: [here](noNmlmYDJk.html) for details)
  -> os::signal_init()

* JavaThread の開始処理 (java.lang.Thread.start() の処理)

  (略) (See: [here](no2935KMw.html) for details)
  -> JVM_StartThread()

* JNI の AttachCurrentThread() 及び AttachCurrentThreadAsDaemon() の処理

  (略) (See: [here](noxegGjntv.html) for details)
  -> jni_AttachCurrentThread()
     -> attach_current_thread()
        -> JvmtiExport::post_thread_start()
           -> (同上)

  (略) (See: [here](noxegGjntv.html) for details)
  -> jni_AttachCurrentThreadAsDaemon()
     -> attach_current_thread()
        -> JvmtiExport::post_thread_start()
           -> (同上)

* AttachListener の処理を行うスレッドの生成処理
  
  (略) (See: [here](no3026gMG.html) for details)
  -> AttachListener::init()
```




### 詳細(Details)
See: [here](../doxygen/classJavaThread.html) for details

---
## <a name="noTyQ47dX8" id="noTyQ47dX8">CompilerThread</a>

### 概要(Summary)
JIT Compiler 用のクラス.
実際に JIT コンパイル処理を行うスレッドを表すスレッドクラス.


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    // A thread used for Compilation.
    class CompilerThread : public JavaThread {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Threads クラスの _thread_list フィールド (static フィールド) に(のみ)格納されている.

(正確には, このフィールドは JavaThread の線形リストを格納するフィールド.
JavaThread オブジェクトは _next フィールドで次の JavaThread オブジェクトを指せる構造になっている.
生成した JavaThread オブジェクトは全てこの線形リスト内に格納されている)

(なお, このフィールドへの登録は Threads::add() で行われる)

#### 生成箇所(where its instances are created)
CompileBroker::make_compiler_thread() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> CompileBroker::compilation_init()
      -> CompileBroker::init_compiler_threads()
         -> CompileBroker::make_compiler_thread()
```




### 詳細(Details)
See: [here](../doxygen/classCompilerThread.html) for details

---
## <a name="noUPbrTXoG" id="noUPbrTXoG">Threads</a>

### 概要(Summary)
HotSpot 内でのスレッド管理用の機能を納めた名前空間(AllStatic クラス).

スレッド管理機構を初期化／終了するためのメソッドや, 
生成された JavaThread の一覧を管理するためのメソッドを提供している.


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    // The active thread queue. It also keeps track of the current used
    // thread priorities.
    class Threads: AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classThreads.html) for details

---
## <a name="nozTxng1HA" id="nozTxng1HA">ThreadClosure</a>

### 概要(Summary)
Thread に対して何らかの処理を行う Closure クラスの基底クラス.


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    // Thread iterator
    class ThreadClosure: public StackObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

### 使われ方(Usage)
Thread* を処理する do_thread() メソッドを備えている.


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
      virtual void do_thread(Thread* thread) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classThreadClosure.html) for details

---
## <a name="noAniqPE4W" id="noAniqPE4W">SignalHandlerMark</a>

### 概要(Summary)
"Signal Dispatcher" スレッドによるシグナル処理で使用される補助クラス(StackObjクラス).

ソースコード中のあるスコープの間だけ, 
指定された Thread オブジェクトの _num_nested_signal フィールドの値を増加させておくためのクラス.

(なお, Thread::_num_nested_signal フィールドは os::fork_and_exec() 及び Thread::current() で(のみ)参照されている.
 See: Thread::is_inside_signal_handler())


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    class SignalHandlerMark: public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (Windows 版では使用されていない).

* JVM_handle_linux_signal()
* JVM_handle_solaris_signal()

### 内部構造(Internal structure)
コンストラクタで Thread::enter_signal_handler() を呼んで Thread::_num_nested_signal フィールドの値を増加させる.
デストラクタでは Thread::leave_signal_handler() を呼んで元に戻している.


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
      SignalHandlerMark(Thread* t) {
        _thread = t;
        if (_thread) _thread->enter_signal_handler();
      }
      ~SignalHandlerMark() {
        if (_thread) _thread->leave_signal_handler();
        _thread = NULL;
      }
```




### 詳細(Details)
See: [here](../doxygen/classSignalHandlerMark.html) for details

---
## <a name="noJEkHf2pz" id="noJEkHf2pz">TraceSuspendDebugBits</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する product オプションが指定されている場合にのみ使用される)
(See: AssertOnSuspendWaitFailure, TraceSuspendWaitFailures).

JavaThread の suspend 処理が失敗していないことを確認する.


```
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
    // Helper class for tracing suspend wait debug bits.
    //
    // 0x00000100 indicates that the target thread exited before it could
    // self-suspend which is not a wait failure. 0x00000200, 0x00020000 and
    // 0x00080000 each indicate a cancelled suspend request so they don't
    // count as wait failures either.
```


```
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
    class TraceSuspendDebugBits : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* JavaThread::is_ext_suspend_completed()
  
  (ただしこちらの使用箇所では _is_wait 引数が false なので, 実質的には使われていない)

* JavaThread::wait_for_ext_suspend_completion()

### 内部構造(Internal structure)
デストラクタ内で確認処理を行っている.

確認する内容は,
「コンストラクタで指定されたポインタ内の値(*bits)について, DEBUG_FALSE_BITS に対応するビットが立っていないかどうか」.


```
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
      TraceSuspendDebugBits(JavaThread *_jt, bool _is_wait, bool _called_by_wait,
                            uint32_t *_bits) {
        jt             = _jt;
        is_wait        = _is_wait;
        called_by_wait = _called_by_wait;
        bits           = _bits;
      }
    
      ~TraceSuspendDebugBits() {
        if (!is_wait) {
    #if 1
          // By default, don't trace bits for is_ext_suspend_completed() calls.
          // That trace is very chatty.
          return;
    #else
          if (!called_by_wait) {
            // If tracing for is_ext_suspend_completed() is enabled, then only
            // trace calls to it from wait_for_ext_suspend_completion()
            return;
          }
    #endif
        }
    
        if (AssertOnSuspendWaitFailure || TraceSuspendWaitFailures) {
          if (bits != NULL && (*bits & DEBUG_FALSE_BITS) != 0) {
            MutexLocker ml(Threads_lock);  // needed for get_thread_name()
            ResourceMark rm;
    
            tty->print_cr(
                "Failed wait_for_ext_suspend_completion(thread=%s, debug_bits=%x)",
                jt->get_thread_name(), *bits);
    
            guarantee(!AssertOnSuspendWaitFailure, "external suspend wait failed");
          }
        }
      }
```

なお, DEBUG_FALSE_BITS は以下の定数値.

0x00000010 は, JavaThread::is_ext_suspend_completed() のエラー時の返値.
0x00200000 は, JavaThread::wait_for_ext_suspend_completion() のエラー時の返値.


```
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
    #define DEBUG_FALSE_BITS (0x00000010 | 0x00200000)
```




### 詳細(Details)
See: [here](../doxygen/classTraceSuspendDebugBits.html) for details

---
## <a name="noAFtgUg1Z" id="noAFtgUg1Z">RememberProcessedThread</a>

### 概要(Summary)
JavaThread クラス内で使用される補助クラス.

ソースコード中のあるスコープの間だけ, 
カレントスレッドに対応する NamedThread の _processed_thread フィールドに, 
指定された JavaThread オブジェクトをセットしておくためのクラス(StackObjクラス).

(なお, NamedThread::_processed_thread フィールドは VMError::report() 内で(のみ)参照されている. (See: VMError))


```
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
    // If the caller is a NamedThread, then remember, in the current scope,
    // the given JavaThread in its _processed_thread field.
    class RememberProcessedThread: public StackObj {
```

### 使われ方(Usage)
JavaThread::oops_do() 内で(のみ)使用されている.

### 内部構造(Internal structure)
コンストラクタで NamedThread::set_processed_thread() を呼んで _processed_thread フィールドをセットし, 
デストラクタで (同じく NamedThread::set_processed_thread() を呼んで) NULL に戻している

(なお, カレントスレッドが NamedThread (のサブクラス) でなければ何もしない).


```
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
      RememberProcessedThread(JavaThread* jthr) {
        Thread* thread = Thread::current();
        if (thread->is_Named_thread()) {
          _cur_thr = (NamedThread *)thread;
          _cur_thr->set_processed_thread(jthr);
        } else {
          _cur_thr = NULL;
        }
      }
    
      ~RememberProcessedThread() {
        if (_cur_thr) {
          _cur_thr->set_processed_thread(NULL);
        }
      }
```




### 詳細(Details)
See: [here](../doxygen/classRememberProcessedThread.html) for details

---
