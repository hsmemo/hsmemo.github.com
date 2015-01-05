---
layout: default
title: OSThread クラス関連のクラス (OSThread, OSThreadWaitState, OSThreadContendState)
---
[Top](../index.html)

#### OSThread クラス関連のクラス (OSThread, OSThreadWaitState, OSThreadContendState)

これらは, HotSpot 内でのスレッド管理用のクラス
(See: [here](no1BWFggBW.html) and [here](noJb9iXZL-.html) for details).


### クラス一覧(class list)

  * [OSThread](#noY1XPJYk1)
  * [OSThreadWaitState](#nor-ZzYrAj)
  * [OSThreadContendState](#noFaKT48dE)


---
## <a name="noY1XPJYk1" id="noY1XPJYk1">OSThread</a>

### 概要(Summary)
スレッドに関する OS 依存な情報を管理するためのクラス.
1つの OSThread オブジェクトが 1つのスレッドに対応する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/osThread.hpp))
    // The OSThread class holds OS-specific thread information.  It is equivalent
    // to the sys_thread_t structure of the classic JVM implementation.
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/osThread.hpp))
    // I'd make OSThread a ValueObj embedded in Thread to avoid an indirection, but
    // the assembler test in java.cpp expects that it can install the OSThread of
    // the main thread into its own Thread at will.
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/osThread.hpp))
    class OSThread: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Thread オブジェクトの _osthread フィールドに(のみ)格納されている.

なお正確に言うと, Solaris や Windows の場合は, 
「メインスレッド」を表す OSThread オブジェクトが以下のフィールドにも格納されている. 
しかし, これらのフィールドは使われていない.

* os クラスの _starting_thread フィールド (static フィールド) (Solaris の場合)
* os クラスの _starting_thread フィールド (static フィールド) (Windows の場合)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

<div class="flow-abst"><pre>
* os::create_thread()          (Linux の場合) (Solaris の場合) (Windows の場合)
* os::create_attached_thread() (Linux の場合)
* create_os_thread()           (Solaris の場合) (Windows の場合)
</pre></div>

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
* メインスレッドの初期化処理

  (HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
  -&gt; Threads::create_vm()
     -&gt; Thread::set_as_starting_thread()
        -&gt; os::create_main_thread()
           -&gt; * Linux の場合:
                -&gt; os::create_attached_thread()
              * Solaris の場合:
                -&gt; create_os_thread()
              * Windows の場合:
                -&gt; create_os_thread()

* 各種スレッドの作成処理
                
  JavaThread::JavaThread()
  -&gt; os::create_thread()
  
  Threads::create_vm()    (&lt;= VMThread の作成処理)
  -&gt; os::create_thread()
  
  WatcherThread::WatcherThread()
  -&gt; os::create_thread()
  
  ConcurrentMarkSweepThread::ConcurrentMarkSweepThread()
  -&gt; os::create_thread()
  
  GCTaskThread::GCTaskThread()
  -&gt; os::create_thread()
  
  ConcurrentGCThread::create_and_start()
  -&gt; os::create_thread()
  
  WorkGang::initialize_workers()
  -&gt; os::create_thread()

* JNI の AttachCurrentThread() 及び AttachCurrentThreadAsDaemon() の処理

  (略) (See: <a href="noxegGjntv.html">here</a> for details)
  -&gt; attach_current_thread()
     -&gt; os::create_attached_thread()
        -&gt; * Solaris の場合:
             -&gt; create_os_thread()
           * Windows の場合:
             -&gt; create_os_thread()
</pre></div>

### 備考(Notes)
OSThread::_state フィールドの型である ThreadState (enum 型) は 
legacy code なので JavaThread の state で書き換えたい, 
とのこと.


```cpp
    ((cite: hotspot/src/share/vm/runtime/osThread.hpp))
    // The thread states represented by the ThreadState values are platform-specific
    // and are likely to be only approximate, because most OSes don't give you access
    // to precise thread state information.
    
    // Note: the ThreadState is legacy code and is not correctly implemented.
    // Uses of ThreadState need to be replaced by the state in the JavaThread.
    
    enum ThreadState {
      ALLOCATED,                    // Memory has been allocated but not initialized
      INITIALIZED,                  // The thread has been initialized but yet started
      RUNNABLE,                     // Has been started and is runnable, but not necessarily running
      MONITOR_WAIT,                 // Waiting on a contended monitor lock
      CONDVAR_WAIT,                 // Waiting on a condition variable
      OBJECT_WAIT,                  // Waiting on an Object.wait() call
      BREAKPOINTED,                 // Suspended at breakpoint
      SLEEPING,                     // Thread.sleep()
      ZOMBIE                        // All done, but not reclaimed yet
    };
```




### 詳細(Details)
See: [here](../doxygen/classOSThread.html) for details

---
## <a name="nor-ZzYrAj" id="nor-ZzYrAj">OSThreadWaitState</a>

### 概要(Summary)
OSThread クラス用の補助クラス.

OSThread の状態(OSThread::_state)を作業途中のあるスコープの中でだけ
OBJECT_WAIT や CONDVAR_WAIT にしておきたい, という場合に使われる補助クラス(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/osThread.hpp))
    // Utility class for use with condition variables:
    class OSThreadWaitState : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* Monitor::wait()
* ObjectMonitor::wait()
* WatcherThread::run()

* os::sleep() (Linux の場合)
* Parker::park() (Linux の場合)

* os::sleep() (Solaris の場合)
* Parker::park() (Solaris の場合)

* os::sleep() (Windows の場合)
* Parker::park() (Windows の場合)

### 内部構造(Internal structure)
コンストラクタで OSThread::set_state() を呼んで状態を変更し, 
デストラクタで (同じく OSThread::set_state() を呼んで) 元に戻している.


```cpp
    ((cite: hotspot/src/share/vm/runtime/osThread.hpp))
      OSThreadWaitState(OSThread* osthread, bool is_object_wait) {
        _osthread  = osthread;
        _old_state = osthread->get_state();
        if (is_object_wait) {
          osthread->set_state(OBJECT_WAIT);
        } else {
          osthread->set_state(CONDVAR_WAIT);
        }
      }
      ~OSThreadWaitState() {
        _osthread->set_state(_old_state);
      }
```




### 詳細(Details)
See: [here](../doxygen/classOSThreadWaitState.html) for details

---
## <a name="noFaKT48dE" id="noFaKT48dE">OSThreadContendState</a>

### 概要(Summary)
OSThread クラス用の補助クラス.

OSThread の状態(OSThread::_state)を作業途中のあるスコープの中でだけ
MONITOR_WAIT にしておきたい, という場合に使われる補助クラス(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/osThread.hpp))
    // Utility class for use with contended monitors:
    class OSThreadContendState : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* ObjectMonitor::enter()
* ObjectMonitor::ReenterI()

### 内部構造(Internal structure)
コンストラクタで OSThread::set_state() を呼んで状態を変更し, 
デストラクタで (同じく OSThread::set_state() を呼んで) 元に戻している.


```cpp
    ((cite: hotspot/src/share/vm/runtime/osThread.hpp))
      OSThreadContendState(OSThread* osthread) {
        _osthread  = osthread;
        _old_state = osthread->get_state();
        osthread->set_state(MONITOR_WAIT);
      }
      ~OSThreadContendState() {
        _osthread->set_state(_old_state);
      }
```




### 詳細(Details)
See: [here](../doxygen/classOSThreadContendState.html) for details

---
