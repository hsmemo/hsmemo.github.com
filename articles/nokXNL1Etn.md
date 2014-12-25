---
layout: default
title: Safepoint クラス関連のクラス (SafepointSynchronize, ThreadSafepointState)
---
[Top](../index.html)

#### Safepoint クラス関連のクラス (SafepointSynchronize, ThreadSafepointState)

これらは, Safepoint 停止処理のためのクラス (See: [here](no7882dWU.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/runtime/safepoint.hpp))
    //
    // Safepoint synchronization
    ////
    // The VMThread or CMS_thread uses the SafepointSynchronize::begin/end
    // methods to enter/exit a safepoint region. The begin method will roll
    // all JavaThreads forward to a safepoint.
    //
    // JavaThreads must use the ThreadSafepointState abstraction (defined in
    // thread.hpp) to indicate that that they are at a safepoint.
    //
    // The Mutex/Condition variable and ObjectLocker classes calls the enter/
    // exit safepoint methods, when a thread is blocked/restarted. Hence, all mutex exter/
    // exit points *must* be at a safepoint.
```


### クラス一覧(class list)

  * [SafepointSynchronize](#nopKQpxDmg)
  * [ThreadSafepointState](#noFycv8QiE)


---
## <a name="nopKQpxDmg" id="nopKQpxDmg">SafepointSynchronize</a>

### 概要(Summary)
Safepoint 停止処理に関する機能を納めた名前空間(AllStatic クラス).

Safepoint 処理を開始／終了するメソッドや, その他の Safepoint 停止に関する雑多な処理が納められている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/safepoint.hpp))
    //
    // Implements roll-forward to safepoint (safepoint synchronization)
    //
    class SafepointSynchronize : AllStatic {
```

### 使われ方(Usage)
Safepoint 停止処理は SafepointSynchronize::begin() で開始され SafepointSynchronize::end() で終了する 
(See: [here](no7882dWU.html) for details).




### 詳細(Details)
See: [here](../doxygen/classSafepointSynchronize.html) for details

---
## <a name="noFycv8QiE" id="noFycv8QiE">ThreadSafepointState</a>

### 概要(Summary)
SafepointSynchronize クラス用の補助クラス.

各 JavaThread の Safepoint 状態 (e.g. 活動中, Safepoint 停止中, etc) を記録しているクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/safepoint.hpp))
    // State class for a thread suspended at a safepoint
    class ThreadSafepointState: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JavaThread オブジェクトの _safepoint_state フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    class JavaThread: public Thread {
    ...
     private:
      ThreadSafepointState *_safepoint_state;        // Holds information about a thread during a safepoint
```

#### 生成箇所(where its instances are created)
ThreadSafepointState::create() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
JavaThread::JavaThread()
-> JavaThread::initialize()
   -> ThreadSafepointState::create()
```




### 詳細(Details)
See: [here](../doxygen/classThreadSafepointState.html) for details

---
