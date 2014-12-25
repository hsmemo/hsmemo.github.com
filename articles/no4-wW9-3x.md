---
layout: default
title: GC_locker クラス関連のクラス (GC_locker, No_GC_Verifier, Pause_No_GC_Verifier, No_Safepoint_Verifier, Pause_No_Safepoint_Verifier, SkipGCALot, JRT_Leaf_Verifier, No_Alloc_Verifier)
---
[Top](../index.html)

#### GC_locker クラス関連のクラス (GC_locker, No_GC_Verifier, Pause_No_GC_Verifier, No_Safepoint_Verifier, Pause_No_Safepoint_Verifier, SkipGCALot, JRT_Leaf_Verifier, No_Alloc_Verifier)

これらは, HotSpot を一時的に「GC が発生しない状態」にするためのクラス (See: [here](no2114SDG.html) for details).

### 概要(Summary)
HotSpot の実行状態の中には「GC を実行するとマズイ」状況がある. 
具体的には以下の状況だと GC はできない.

* JNI の Get*Critical() 関数が呼ばれているため, オブジェクトを移動できない.
* まだ HotSpot の初期化中なので GC 処理が行えない.
* etc (#TODO)

GC_locker は, こういった状況に対処するためのクラス.
HotSpot を一時的に「GC が発生しない状態」にする.

内部的な処理としては以下のようになっている.

1. どの GC アルゴリズムも GC 処理の開始時に GC_locker の状態を確認するようになっている.

   (なお, 確認は GC_locker::check_active_before_gc() で行える)

2. もし GC_locker がロックされていれば GC 処理をその時点で中止する



### クラス一覧(class list)

  * [GC_locker](#no0zFME_jp)
  * [No_GC_Verifier](#nogOuzPsbF)
  * [Pause_No_GC_Verifier](#norRtNj2zl)
  * [No_Safepoint_Verifier](#noBtxt5NU7)
  * [Pause_No_Safepoint_Verifier](#noVCcbrmBB)
  * [SkipGCALot](#no6tfD9cqD)
  * [JRT_Leaf_Verifier](#nosT_MRCH0)
  * [No_Alloc_Verifier](#no6SUZyL-T)


---
## <a name="no0zFME_jp" id="no0zFME_jp">GC_locker</a>

### 概要(Summary)
HotSpot を一時的に「GC が発生しない状態」にするためのクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)) (See: [here](no2114SDG.html) for details).

具体的には以下のようなケースで使用される.

* JNI の Get*Critical() 関数が呼ばれているため, オブジェクトを移動できない.
* まだ HotSpot の初期化中なので GC 処理が行えない.
* etc (#TODO)


```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    // The direct lock/unlock calls do not force a collection if an unlock
    // decrements the count to zero. Avoid calling these if at all possible.
    
    class GC_locker: public AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classGC__locker.html) for details

---
## <a name="nogOuzPsbF" id="nogOuzPsbF">No_GC_Verifier</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (`#ifdef ASSERT` 時以外には空のクラスとして定義される).

「あるコード範囲で GC が起きない」ということをコード上に明示したい場合に使用される.

```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    // A No_GC_Verifier object can be placed in methods where one assumes that
    // no garbage collection will occur. The destructor will verify this property
    // unless the constructor is called with argument false (not verifygc).
    //
    // The check will only be done in debug mode and if verifygc true.
    
    class No_GC_Verifier: public StackObj {
```

### 使われ方(Usage)
このクラスのインスタンス自体は, 現状はどこからも使われていない模様
(サブクラスである No_Safepoint_Verifier は使われている).

(デバッグ用のクラスなので, 自分でコードを書き直して入れるべきもの??)

### 内部構造(Internal structure)
`#ifdef ASSERT` でなければ, 中身のないクラスとして定義される.

```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    #ifdef ASSERT
    ...
    #else
      No_GC_Verifier(bool verifygc = true) {}
      ~No_GC_Verifier() {}
    #endif
```

`#ifdef ASSERT` 時には, 
コンストラクタ内で現在の GC 回数を保存し, デストラクタ内でその時点での GC 回数と比較している.
もし回数が 違っていれば, fatal() を呼んで HotSpot を異常終了させる.

(なお, `#ifdef ASSERT` 時であっても, コンストラクタ引数が false であれば何もしない)


```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.cpp))
    #ifdef ASSERT
    
    No_GC_Verifier::No_GC_Verifier(bool verifygc) {
      _verifygc = verifygc;
      if (_verifygc) {
        CollectedHeap* h = Universe::heap();
        assert(!h->is_gc_active(), "GC active during No_GC_Verifier");
        _old_invocations = h->total_collections();
      }
    }
    
    
    No_GC_Verifier::~No_GC_Verifier() {
      if (_verifygc) {
        CollectedHeap* h = Universe::heap();
        assert(!h->is_gc_active(), "GC active during No_GC_Verifier");
        if (_old_invocations != h->total_collections()) {
          fatal("collection in a No_GC_Verifier secured function");
        }
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classNo__GC__Verifier.html) for details

---
## <a name="norRtNj2zl" id="norRtNj2zl">Pause_No_GC_Verifier</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (`#ifdef ASSERT` 時以外には空のクラスとして定義される).

No_GC_Verifier の働きを一時的に無効化したい場合に使用する.

(当然ながら No_GC_Verifier が動いている場合でないとこのクラスの意味はない.
 このため, このクラスも `#ifdef ASSERT` 時でないと意味は無い.
 また, No_GC_Verifier オブジェクトのコンストラクタ引数が false である場合も意味は無い)

```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    // A Pause_No_GC_Verifier is used to temporarily pause the behavior
    // of a No_GC_Verifier object. If we are not in debug mode or if the
    // No_GC_Verifier object has a _verifygc value of false, then there
    // is nothing to do.
    
    class Pause_No_GC_Verifier: public StackObj {
```

### 使われ方(Usage)
このクラスのインスタンス自体は, 現状はどこからも使われていない模様
(サブクラスである Pause_No_Safepoint_Verifier は使われている).

(デバッグ用のクラスなので, 自分でコードを書き直して入れるべきもの??)




### 詳細(Details)
See: [here](../doxygen/classPause__No__GC__Verifier.html) for details

---
## <a name="noBtxt5NU7" id="noBtxt5NU7">No_Safepoint_Verifier</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (`#ifdef ASSERT` 時以外には空のクラスとして定義される).

「あるコード範囲で Safepoint が起きない」ということをコード上に明示したい場合に使用される.
なお, 「Safepoint が起きない」とは, 以下のどの操作も行われないということを意味する.

 * oop の確保
 * Mutex や JavaLock によるブロック
 * VM operation の実行

なお, このクラスの safepoint チェックは StrictSafepointChecks オプションが指定されている場合にしか行われない.
指定されていない場合は, このクラスは No_GC_Verifier と同じになる.
(See: Thread::check_for_valid_safepoint_state())


```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    // A No_Safepoint_Verifier object will throw an assertion failure if
    // the current thread passes a possible safepoint while this object is
    // instantiated. A safepoint, will either be: an oop allocation, blocking
    // on a Mutex or JavaLock, or executing a VM operation.
    //
    // If StrictSafepointChecks is turned off, it degrades into a No_GC_Verifier
    //
    class No_Safepoint_Verifier : public No_GC_Verifier {
```

### 使われ方(Usage)
明示したいコード箇所で No_Safepoint_Verifier 型の局所変数を宣言するだけ.


```cpp
    ((cite: hotspot/src/share/vm/classfile/classFileParser.cpp))
      {
        debug_only(No_Safepoint_Verifier nsv;)
    ...
      }
```

### 内部構造(Internal structure)
(なお, `#ifdef ASSERT` 時であっても, コンストラクタ引数が false であれば何もしない)

カレントスレッドの以下の2つのフィールドを, コンストラクタでインクリメントし, デストラクタでデクリメントしている.

* _allow_allocation_count フィールド : 
  このフィールドが 1 以上だと, Thread::allow_allocation() が false を返すようになる (See: Thread::allow_allocation())

* _allow_safepoint_count フィールド
  このフィールドが 1 以上だと, Thread::check_for_valid_safepoint_state() が false を返すようになる (See: Thread::check_for_valid_safepoint_state()))

(なお, No_GC_Verifier のサブクラスなので, 以上に加えて No_GC_Verifier としての処理も行われる)
 

```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    #ifdef ASSERT
      No_Safepoint_Verifier(bool activated = true, bool verifygc = true ) :
        No_GC_Verifier(verifygc),
        _activated(activated) {
        _thread = Thread::current();
        if (_activated) {
          _thread->_allow_allocation_count++;
          _thread->_allow_safepoint_count++;
        }
      }
    
      ~No_Safepoint_Verifier() {
        if (_activated) {
          _thread->_allow_allocation_count--;
          _thread->_allow_safepoint_count--;
        }
      }
```




### 詳細(Details)
See: [here](../doxygen/classNo__Safepoint__Verifier.html) for details

---
## <a name="noVCcbrmBB" id="noVCcbrmBB">Pause_No_Safepoint_Verifier</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (`#ifdef ASSERT` 時以外には空のクラスとして定義される).

No_Safepoint_Verifier の働きを一時的に無効化したい場合に使用する.

(当然ながら No_Safepoint_Verifier が動いている場合でないとこのクラスの意味はない.
 このため, このクラスも `#ifdef ASSERT` 時でないと意味は無い.
 また, No_Safepoint_Verifier オブジェクトのコンストラクタ引数が false である場合には, 
 safepoint や allocation の監視機能については意味は無い.
 ただし, このクラスは No_Safepoint_Verifier の No_GC_Verifier としての働きについては抑止しない.)


```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    // A Pause_No_Safepoint_Verifier is used to temporarily pause the
    // behavior of a No_Safepoint_Verifier object. If we are not in debug
    // mode then there is nothing to do. If the No_Safepoint_Verifier
    // object has an _activated value of false, then there is nothing to
    // do for safepoint and allocation checking, but there may still be
    // something to do for the underlying No_GC_Verifier object.
    
    class Pause_No_Safepoint_Verifier : public Pause_No_GC_Verifier {
```

### 使われ方(Usage)
コード中で Pause_No_Safepoint_Verifier 型の局所変数を宣言するだけ.

```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiRedefineClasses.cpp))
                {
                  Pause_No_Safepoint_Verifier pnsv(&nsv);
    
    ...
                }
```

### 内部構造(Internal structure)
カレントスレッドの以下の2つのフィールドを, コンストラクタでデクリメントし, デストラクタでインクリメントしている.

* _allow_allocation_count フィールド : 
* _allow_safepoint_count フィールド

(No_Safepoint_Verifier とちょうど反対の挙動. (See: No_Safepoint_Verifier))


```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    #ifdef ASSERT
      Pause_No_Safepoint_Verifier(No_Safepoint_Verifier * nsv)
        : Pause_No_GC_Verifier(nsv) {
    
        _nsv = nsv;
        if (_nsv->_activated) {
          _nsv->_thread->_allow_allocation_count--;
          _nsv->_thread->_allow_safepoint_count--;
        }
      }
    
      ~Pause_No_Safepoint_Verifier() {
        if (_nsv->_activated) {
          _nsv->_thread->_allow_allocation_count++;
          _nsv->_thread->_allow_safepoint_count++;
        }
      }
```




### 詳細(Details)
See: [here](../doxygen/classPause__No__Safepoint__Verifier.html) for details

---
## <a name="no6tfD9cqD" id="no6tfD9cqD">SkipGCALot</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (`#ifdef ASSERT` 時以外には空のクラスとして定義される).

デバッグ用の機能である GCALot 機能を一時的に停止させたい場合に使用する
(See: GCALotAtAllSafepoints, ScavengeALot, FullGCALot).

現行では, GC 処理中に (GCALot 機能によって) 再び GC 処理に再入してしまうのを防ぐために(のみ)使用されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    // A SkipGCALot object is used to elide the usual effect of gc-a-lot
    // over a section of execution by a thread. Currently, it's used only to
    // prevent re-entrant calls to GC.
    class SkipGCALot : public StackObj {
```

### 使われ方(Usage)
VMThread::execute() 内で, 
(GCALot により) VM operation 処理に再入するのを防ぐ為に使われている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/vmThread.cpp))
    void VMThread::execute(VM_Operation* op) {
    ...
      if (!t->is_VM_thread()) {
        SkipGCALot sgcalot(t);    // avoid re-entrant attempts to gc-a-lot
    ...
```

### 内部構造(Internal structure)
指定されたスレッドの skip_gcalot を, コンストラクタで true にし, デストラクタで元の値に戻している

(これが true だと Thread::skip_gcalot() が true を返すようになる. (See: Thread::skip_gcalot()))
 

```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    #ifdef ASSERT
        SkipGCALot(Thread* t) : _t(t) {
          _saved = _t->skip_gcalot();
          _t->set_skip_gcalot(true);
        }
    
        ~SkipGCALot() {
          assert(_t->skip_gcalot(), "Save-restore protocol invariant");
          _t->set_skip_gcalot(_saved);
        }
```




### 詳細(Details)
See: [here](../doxygen/classSkipGCALot.html) for details

---
## <a name="nosT_MRCH0" id="nosT_MRCH0">JRT_Leaf_Verifier</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (`#ifdef ASSERT` 時以外には空のクラスとして定義される).

JRT_LEAF() マクロを用いて定義された関数について, 
呼び出し時に守るべき条件を満たしているかどうかをチェックする (See: JRT_LEAF()).

(なお今のところ, JRT_LEAF 関数は, 
 スレッドが _thread_in_Java 状態か _thread_in_native 状態でないと呼び出せないことになっている.
 _thread_in_native 状態の場合は, 他のスレッドが GC を発生させるのは問題ない.)


```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    // JRT_LEAF currently can be called from either _thread_in_Java or
    // _thread_in_native mode. In _thread_in_native, it is ok
    // for another thread to trigger GC. The rest of the JRT_LEAF
    // rules apply.
    class JRT_Leaf_Verifier : public No_Safepoint_Verifier {
```

JRT_LEAF() が満たすべき Safepoint に関する規則は以下の通り.
このうち, JRT_Leaf_Verifier は 1~3 の規則をチェックする.

```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.cpp))
    // JRT_LEAF rules:
    // A JRT_LEAF method may not interfere with safepointing by
    //   1) acquiring or blocking on a Mutex or JavaLock - checked
    //   2) allocating heap memory - checked
    //   3) executing a VM operation - checked
    //   4) executing a system call (including malloc) that could block or grab a lock
    //   5) invoking GC
    //   6) reaching a safepoint
    //   7) running too long
    // Nor may any method it calls.
```

### 使われ方(Usage)
コード中で JRT_Leaf_Verifier 型の局所変数を宣言するだけ.

現在は, JRT_LEAF() マクロの中で(のみ)使われている
(JRT_LEAF() マクロの中で, JRT_Leaf_Verifier 型の局所変数が宣言されている).


```cpp
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    #define JRT_LEAF(result_type, header)                                \
      result_type header {                                               \
      __LEAF(result_type, header)                                        \
      debug_only(JRT_Leaf_Verifier __jlv;)
```

### 内部構造(Internal structure)
内部的には, 基底クラスである No_Safepoint_Verifier のコンストラクタ/デストラクタを呼び出すだけ.

(なお, No_Safepoint_Verifier の verifygc 機能を有効にするかどうかは
 JRT_Leaf_Verifier::should_verify_GC() で決定される.
 _thread_in_Java 状態なら true, _thread_in_native 状態なら false になる.)


```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.cpp))
    JRT_Leaf_Verifier::JRT_Leaf_Verifier()
      : No_Safepoint_Verifier(true, JRT_Leaf_Verifier::should_verify_GC())
    {
    }
    
    JRT_Leaf_Verifier::~JRT_Leaf_Verifier()
    {
    }
```


```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.cpp))
    bool JRT_Leaf_Verifier::should_verify_GC() {
      switch (JavaThread::current()->thread_state()) {
      case _thread_in_Java:
        // is in a leaf routine, there must be no safepoint.
        return true;
      case _thread_in_native:
        // A native thread is not subject to safepoints.
        // Even while it is in a leaf routine, GC is ok
        return false;
      default:
        // Leaf routines cannot be called from other contexts.
        ShouldNotReachHere();
        return false;
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classJRT__Leaf__Verifier.html) for details

---
## <a name="no6SUZyL-T" id="no6SUZyL-T">No_Alloc_Verifier</a>

デバッグ用(開発時用)のクラス (`#ifdef ASSERT` 時以外には空のクラスとして定義される).

「あるコード範囲でオブジェクトの確保処理が起きない」ということをコード上に明示したい場合に使用される.


```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    // A No_Alloc_Verifier object can be placed in methods where one assumes that
    // no allocation will occur. The destructor will verify this property
    // unless the constructor is called with argument false (not activated).
    //
    // The check will only be done in debug mode and if activated.
    // Note: this only makes sense at safepoints (otherwise, other threads may
    // allocate concurrently.)
    
    class No_Alloc_Verifier : public StackObj {
```

### 使われ方(Usage)
このクラスのインスタンス自体は, 現状はどこからも使われていない模様.

(デバッグ用のクラスなので, 自分でコードを書き直して入れるべきもの??)

### 内部構造(Internal structure)
(なお, `#ifdef ASSERT` 時であっても, コンストラクタ引数が false であれば何もしない)

コンストラクタでカレントスレッドの _allow_allocation_count フィールドをインクリメントし, デストラクタでデクリメントしている.

(このフィールドが 1 以上だと Thread::allow_allocation() が false を返すようになる. (See: Thread::allow_allocation())


```cpp
    ((cite: hotspot/src/share/vm/memory/gcLocker.hpp))
    #ifdef ASSERT
      No_Alloc_Verifier(bool activated = true) {
        _activated = activated;
        if (_activated) Thread::current()->_allow_allocation_count++;
      }
    
      ~No_Alloc_Verifier() {
        if (_activated) Thread::current()->_allow_allocation_count--;
      }
```




### 詳細(Details)
See: [here](../doxygen/classNo__Alloc__Verifier.html) for details

---
