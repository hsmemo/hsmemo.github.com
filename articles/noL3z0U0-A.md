---
layout: default
title: ObjectSynchronizer クラスおよび ObjectLocker クラス (ObjectSynchronizer, ObjectLocker, 及びそれらの補助クラス(ReleaseJavaMonitorsClosure))
---
[Top](../index.html)

#### ObjectSynchronizer クラスおよび ObjectLocker クラス (ObjectSynchronizer, ObjectLocker, 及びそれらの補助クラス(ReleaseJavaMonitorsClosure))

これらは, 同期排他処理 (monitorenter/monitorexit 命令, synchronized 修飾子, wait()/notify()/notifyAll() メソッド, 等) のためのクラス
(See: [here](no2114NIs.html) for details).


### クラス一覧(class list)

  * [ObjectSynchronizer](#no11oN4Wgq)
  * [ObjectLocker](#noto8qrUCN)
  * [ReleaseJavaMonitorsClosure](#noKdtQer36)


---
## <a name="no11oN4Wgq" id="no11oN4Wgq">ObjectSynchronizer</a>

### 概要(Summary)
Java の同期排他処理のための関数を納めた名前空間(AllStatic クラス).

同期排他関係の処理は基本的に全てこのクラスに実装されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/synchronizer.hpp))
    class ObjectSynchronizer : AllStatic {
```

ただし, いちいちこいつを呼んでると遅いので, 
インタープリタや JIT 生成コードの fast path では使われない.

(このクラスには fast path 用のメソッドも定義されているのだが, 
 実際の fast path の処理部にはそのメソッドの内容がコピーされ手動でアセンブリに直された状態で展開されている)

(そのため, このクラスの fast path 用のメソッドを変更する際には (その展開箇所も含めて) 全部変えないと不整合が起きてまずい, とのこと)


```cpp
    ((cite: hotspot/src/share/vm/runtime/synchronizer.cpp))
    // The "core" versions of monitor enter and exit reside in this file.
    // The interpreter and compilers contain specialized transliterated
    // variants of the enter-exit fast-path operations.  See i486.ad fast_lock(),
    // for instance.  If you make changes here, make sure to modify the
    // interpreter, and both C1 and C2 fast-path inline locking code emission.
    //
    //
    // -----------------------------------------------------------------------------
```


### 使われ方(Usage)
以下の処理で使用される. (他の使用箇所は?? #TODO)

* monitorenter/monitorexit 命令や synchronized 修飾子の処理
  
  (See: [here](noOroadKvi.html) and [here](noXF2ZIHEZ.html) for details)

* java.lang.Object クラスの wait()/notify()/notifyAll() メソッドの処理
  
  (See: [here](no3059BSg.html) for details)

* JNI による同期排他処理 (MonitorEnter(), MonitorExit()) の処理

  (See: [here](no5248b4E.html) for details)

### 内部構造(Internal structure)
内部には, 同期排他処理のための以下のようなメソッドが定義されている.

* monitorenter/monitorexit 命令(や synchronized 修飾子)の fast path 用の処理 (See: [here](noOroadKvi.html) and [here](noXF2ZIHEZ.html) for details)

  (わざわざ名前に "fast_" とつけて高速版だと分かるようにしたとのこと.
  また, これらはアセンブリ版がインタープリタや C1/C2 JIT 内部にコピーされているので, 
  変更する際には全部変えないと不整合になってまずい, とのこと)


```cpp
    ((cite: hotspot/src/share/vm/runtime/synchronizer.hpp))
      // This is full version of monitor enter and exit. I choose not
      // to use enter() and exit() in order to make sure user be ware
      // of the performance and semantics difference. They are normally
      // used by ObjectLocker etc. The interpreter and compiler use
      // assembly copies of these routines. Please keep them synchornized.
      //
      // attempt_rebias flag is used by UseBiasedLocking implementation
      static void fast_enter  (Handle obj, BasicLock* lock, bool attempt_rebias, TRAPS);
      static void fast_exit   (oop obj,    BasicLock* lock, Thread* THREAD);
```

* monitorenter/monitorexit 命令(や synchronized 修飾子)の slow path 用の処理 (See: [here](noOroadKvi.html) and [here](noXF2ZIHEZ.html) for details)


```cpp
    ((cite: hotspot/src/share/vm/runtime/synchronizer.hpp))
      // WARNING: They are ONLY used to handle the slow cases. They should
      // only be used when the fast cases failed. Use of these functions
      // without previous fast case check may cause fatal error.
      static void slow_enter  (Handle obj, BasicLock* lock, TRAPS);
      static void slow_exit   (oop obj,    BasicLock* lock, Thread* THREAD);
```

* java.lang.Object クラスの wait()/notify()/notifyall() 用の処理 (See: [here](no3059BSg.html) for details)
  
  (インタープリタ, JIT 生成コード, JNI を問わずこれが使われる)


```cpp
    ((cite: hotspot/src/share/vm/runtime/synchronizer.hpp))
      // Handle all interpreter, compiler and jni cases
      static void wait               (Handle obj, jlong millis, TRAPS);
      static void notify             (Handle obj,               TRAPS);
      static void notifyall          (Handle obj,               TRAPS);
```

* JNI による同期排他処理 (MonitorEnter(), MonitorExit()) の処理 (See: [here](no5248b4E.html) for details)


```cpp
    ((cite: hotspot/src/share/vm/runtime/synchronizer.hpp))
      // Used only to handle jni locks or other unmatched monitor enter/exit
      // Internally they will use heavy weight monitor.
      static void jni_enter   (Handle obj, TRAPS);
      static bool jni_try_enter(Handle obj, Thread* THREAD); // Implements Unsafe.tryMonitorEnter
      static void jni_exit    (oop obj,    Thread* THREAD);
```




### 詳細(Details)
See: [here](../doxygen/classObjectSynchronizer.html) for details

---
## <a name="noto8qrUCN" id="noto8qrUCN">ObjectLocker</a>

### 概要(Summary)
HotSpot 内で「Java オブジェクトのロックを取得して何らかの処理を行う」場合用のユーティリティ・クラス(StackObjクラス).

(例えば, クラスローディング処理中だけはロックしておきたい, 等).

ロックの確保／開放処理をソースコード上のスコープに合わせて行ってくれる
(ただし, pending exception がセットされているかもしれないので必要に応じてチェックするように, とのこと).


```cpp
    ((cite: hotspot/src/share/vm/runtime/synchronizer.hpp))
    // ObjectLocker enforced balanced locking and can never thrown an
    // IllegalMonitorStateException. However, a pending exception may
    // have to pass through, and we must also be able to deal with
    // asynchronous exceptions. The caller is responsible for checking
    // the threads pending exception if needed.
    // doLock was added to support classloading with UnsyncloadClass which
    // requires flag based choice of locking the classloader lock.
    class ObjectLocker : public StackObj {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
#### 定義されているフィールド
定義されているフィールドは以下の通り.

(BasicLock (displaced header) はオブジェクト内にフィールドとして保持している)


```cpp
    ((cite: hotspot/src/share/vm/runtime/synchronizer.hpp))
      Thread*   _thread;
      Handle    _obj;
      BasicLock _lock;
      bool      _dolock;   // default true
```

#### 内部の処理
コンストラクタで ObjectSynchronizer::fast_enter() を呼んでロックを確保し, 
デストラクタで ObjectSynchronizer::fast_exit() で解放している.

(ただし, doLock コンストラクタ引数が false の場合は, 何も処理を行わない)


```cpp
    ((cite: hotspot/src/share/vm/runtime/synchronizer.cpp))
    // -----------------------------------------------------------------------------
    // Internal VM locks on java objects
    // standard constructor, allows locking failures
    ObjectLocker::ObjectLocker(Handle obj, Thread* thread, bool doLock) {
      _dolock = doLock;
      _thread = thread;
      debug_only(if (StrictSafepointChecks) _thread->check_for_valid_safepoint_state(false);)
      _obj = obj;
    
      if (_dolock) {
        TEVENT (ObjectLocker) ;
    
        ObjectSynchronizer::fast_enter(_obj, &_lock, false, _thread);
      }
    }
    
    ObjectLocker::~ObjectLocker() {
      if (_dolock) {
        ObjectSynchronizer::fast_exit(_obj(), &_lock, _thread);
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classObjectLocker.html) for details

---
## <a name="noKdtQer36" id="noKdtQer36">ReleaseJavaMonitorsClosure</a>

### 概要(Summary)
ObjectSynchronizer クラス内で使用される補助クラス.

JNI の DetachCurrentThread() 関数の処理で使用される補助クラス.
カレントスレッドがロックを握っている ObjectMonitor のロックを開放する
(JNI の仕様で DetachCurrentThread() 時には対象のスレッドが保持しているロックを全て解放しなければいけないと規定されている).


```cpp
    ((cite: hotspot/src/share/vm/runtime/synchronizer.cpp))
    // Iterate through monitor cache and attempt to release thread's monitors
    // Gives up on a particular monitor if an exception occurs, but continues
    // the overall iteration, swallowing the exception.
    class ReleaseJavaMonitorsClosure: public MonitorClosure {
```

### 使われ方(Usage)
ObjectSynchronizer::release_monitors_owned_by_thread() 内で(のみ)使用されている (See: [here](no2935w3j.html) for details).




### 詳細(Details)
See: [here](../doxygen/classReleaseJavaMonitorsClosure.html) for details

---
