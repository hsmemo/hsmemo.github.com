---
layout: default
title: MutexLocker クラス関連のクラス (MutexLocker, MutexLockerEx, MonitorLockerEx, GCMutexLocker, MutexUnlocker, MutexUnlockerEx, VerifyMutexLocker)
---
[Top](../index.html)

#### MutexLocker クラス関連のクラス (MutexLocker, MutexLockerEx, MonitorLockerEx, GCMutexLocker, MutexUnlocker, MutexUnlockerEx, VerifyMutexLocker)

これらは, HotSpot 内でのスレッド制御用のクラス.
より具体的に言うと, スレッド間で排他を取るためのユーティリティ・クラス
(See: [here](no2114cio.html) for details).

### 概要(Summary)
MutexLocker は, 
Native Monitor(Mutex/Monitor) によるロック/アンロック処理をソースコード上のスコープに合わせて行うための補助クラス.
用途に応じて様々な変種が存在している.


```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
    // A MutexLocker provides mutual exclusion with respect to a given mutex
    // for the scope which contains the locker.  The lock is an OS lock, not
    // an object lock, and the two do not interoperate.  Do not use Mutex-based
    // locks to lock on Java objects, because they will not be respected if a
    // that object is locked using the Java locking mechanism.
```

なお HotSpot 内では, MutexLocker は以下のように処理を行うと想定している.
これを破るような変更を行うと HotSpot 内に大量の修正が必要なので注意.

* コンストラクタ(ロック時)には 「Load-Acquire」
* デストラクタ(アンロック時)には 「Store-Release」


```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
    //                NOTE WELL!!
    //
    // See orderAccess.hpp.  We assume throughout the VM that MutexLocker's
    // and friends constructors do a fence, a lock and an acquire *in that
    // order*.  And that their destructors do a release and unlock, in *that*
    // order.  If their implementations change such that these assumptions
    // are violated, a whole lot of code will break.
```


### クラス一覧(class list)

  * [MutexLocker](#noVX2GNPAC)
  * [MutexLockerEx](#now5IQja62)
  * [MonitorLockerEx](#notTi5rDru)
  * [GCMutexLocker](#nosEAhExo4)
  * [MutexUnlocker](#no6092vN1k)
  * [MutexUnlockerEx](#no3M7YEkH6)
  * [VerifyMutexLocker](#noS7_yV_Xn)


---
## <a name="noVX2GNPAC" id="noVX2GNPAC">MutexLocker</a>

### 概要(Summary)
Mutex クラス用のユーティリティ・クラス.

Mutex を用いた排他処理(ロックの確保処理/解放処理)を簡単に記述するための補助クラス(StackObjクラス).
ロックの確保／開放処理をソースコード上のスコープに合わせて行ってくれる
(See: [here](no2114cio.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
    class MutexLocker: StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で MutexLocker 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
コンストラクタで指定された Mutex をロックし, デストラクタでアンロックする.


```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
      MutexLocker(Monitor * mutex) {
        assert(mutex->rank() != Mutex::special,
          "Special ranked mutex should only use MutexLockerEx");
        _mutex = mutex;
        _mutex->lock();
      }
    
      // Overloaded constructor passing current thread
      MutexLocker(Monitor * mutex, Thread *thread) {
        assert(mutex->rank() != Mutex::special,
          "Special ranked mutex should only use MutexLockerEx");
        _mutex = mutex;
        _mutex->lock(thread);
      }
    
      ~MutexLocker() {
        _mutex->unlock();
      }
```

### 備考(Notes)
なお, 名前は "MutexLocker" だが Mutex が Monitor のサブクラスになっている関係で
Mutex でも Monitor でも受け取れるようになって(しまって?)いる.


```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
      MutexLocker(Monitor * mutex) {
```




### 詳細(Details)
See: [here](../doxygen/classMutexLocker.html) for details

---
## <a name="now5IQja62" id="now5IQja62">MutexLockerEx</a>

### 概要(Summary)
特殊な MutexLocker クラス.

コンストラクタに NULL が渡されたときには何もしない.

(コメントによると, 
MutexLocker の処理を可能な限り高速にしたいので別クラスを作った, 
とのこと).

(しかしこの Win32 API みたいな名前は...)


```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
    // A MutexLockerEx behaves like a MutexLocker when its constructor is
    // called with a Mutex.  Unlike a MutexLocker, its constructor can also be
    // called with NULL, in which case the MutexLockerEx is a no-op.  There
    // is also a corresponding MutexUnlockerEx.  We want to keep the
    // basic MutexLocker as fast as possible.  MutexLockerEx can also lock
    // without safepoint check.
    
    class MutexLockerEx: public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で MutexLockerEx 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classMutexLockerEx.html) for details

---
## <a name="notTi5rDru" id="notTi5rDru">MonitorLockerEx</a>

### 概要(Summary)
MutexLockerEx のサブクラス.

コンストラクタに Mutex だけでなく Monitor も受け取れるようになっている
(というのが違いのはずだが, 現在では Mutex が Monitor のサブクラスなので, 
これだけだと MutexLockerEx と違いがない...).

また, MutexLockerEx と異なり wait() や notify() といったメソッドも提供している
(といっても, 単にコンストラクタに渡された Monitor オブジェクトの同名のメソッドを呼ぶだけ.
   なお, コンストラクタに渡されたのが Mutex だった場合には, 
 これらのメソッドは何もしないのかと思ったが, 
 MutexLockerEx とコンストラクタの型もかぶっているので, 
 Mutex だった場合も MonitorLockerEx のコンストラクタが動く.
   Mutex を渡された場合に wait/notify/notifyAll を呼び出してしまうと ShouldNotReachHere だが, 
 今のところ Mutex を渡されるコードがないから問題ない?? (まぁ普通そんな使い方しないだろうが))

(どうでもいいけど名前が MutexLockerExEx じゃなくてよかった...)


```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
    // A MonitorLockerEx is like a MutexLockerEx above, except it takes
    // a possibly null Monitor, and allows wait/notify as well which are
    // delegated to the underlying Monitor.
    
    class MonitorLockerEx: public MutexLockerEx {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で MonitorLockerEx 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* G1CollectedHeap::increment_full_collections_completed()
* GenCollectedHeap::update_full_collections_completed()
* GenCollectedHeap::update_full_collections_completed(unsigned int count)




### 詳細(Details)
See: [here](../doxygen/classMonitorLockerEx.html) for details

---
## <a name="nosEAhExo4" id="nosEAhExo4">GCMutexLocker</a>

### 概要(Summary)
特殊な MutexLocker クラス.

コンストラクタが呼ばれた時点で SafepointSynchronize::is_at_safepoint() であれば何もしない.

(GC 処理関係で必要なロックの確保に使用される.
既に GC 処理に入っておりロックを取得済みであれば native monitor の再帰取得になってしまうので何もしない.)


```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
    // A GCMutexLocker is usually initialized with a mutex that is
    // automatically acquired in order to do GC.  The function that
    // synchronizes using a GCMutexLocker may be called both during and between
    // GC's.  Thus, it must acquire the mutex if GC is not in progress, but not
    // if GC is in progress (since the mutex is already held on its behalf.)
    
    class GCMutexLocker: public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で GCMutexLocker 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* OneContigSpaceCardGeneration::expand()
* SystemDictionary::print()
* SystemDictionary::verify()
* SystemDictionary::verify_obj_klass_present()




### 詳細(Details)
See: [here](../doxygen/classGCMutexLocker.html) for details

---
## <a name="no6092vN1k" id="no6092vN1k">MutexUnlocker</a>

### 概要(Summary)
Mutex クラス用の補助クラス (See: [here](no2114cio.html) for details).

ソースコード中のあるスコープの間だけ, 指定した Mutex をアンロックしておきたい, 
という場面で使用される補助クラス(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
    // A MutexUnlocker temporarily exits a previously
    // entered mutex for the scope which contains the unlocker.
    
    class MutexUnlocker: StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で MutexUnlocker 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
コンストラクタで unlock() し, デストラクタで lock() し直すだけ.

```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
      MutexUnlocker(Monitor * mutex) {
        _mutex = mutex;
        _mutex->unlock();
      }
    
      ~MutexUnlocker() {
        _mutex->lock();
      }
```




### 詳細(Details)
See: [here](../doxygen/classMutexUnlocker.html) for details

---
## <a name="no3M7YEkH6" id="no3M7YEkH6">MutexUnlockerEx</a>

### 概要(Summary)
特殊な MutexUnlocker クラス.

no_safepoint_check コンストラクタ引数が true の場合は, 
デストラクタ内で Monitor::lock() の代わりに Monitor::lock_without_safepoint_check() を呼び出す.


```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
    // A MutexUnlockerEx temporarily exits a previously
    // entered mutex for the scope which contains the unlocker.
    
    class MutexUnlockerEx: StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で MutexUnlockerEx 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* 




### 詳細(Details)
See: [here](../doxygen/classMutexUnlockerEx.html) for details

---
## <a name="noS7_yV_Xn" id="noS7_yV_Xn">VerifyMutexLocker</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか使用されない).

特殊な MutexLocker クラス.

verify 時に使うためにデッドロック検出処理をゆるくしている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/mutexLocker.hpp))
    #ifndef PRODUCT
    //
    // A special MutexLocker that allows:
    //   - reentrant locking
    //   - locking out of order
    //
    // Only too be used for verify code, where we can relaxe out dead-lock
    // dection code a bit (unsafe, but probably ok). This code is NEVER to
    // be included in a product version.
    //
    class VerifyMutexLocker: StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で VerifyMutexLocker 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
nmethod::print_calls() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classVerifyMutexLocker.html) for details

---
