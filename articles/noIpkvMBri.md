---
layout: default
title: Parker クラスおよび ParkEvent クラス (Parker, ParkEvent)
---
[Top](../index.html)

#### Parker クラスおよび ParkEvent クラス (Parker, ParkEvent)

これらは, HotSpot 内でのスレッド制御用のクラス.
より具体的に言うと, スレッドを待機状態にさせる(ブロックさせる)ためのユーティリティ・クラス
(See: [here](no2114PYi.html) and [here](no2114COc.html) for details).


### クラス一覧(class list)

  * [Parker](#no1RnFuey4)
  * [ParkEvent](#noEC9k4fuj)


---
## <a name="no1RnFuey4" id="no1RnFuey4">Parker</a>

### 概要(Summary)
java.util.concurrent (JSR 166) の機能を実現するためのクラス.

より具体的に言うと, 
java.util.concurrent.locks.LockSupport.park(), 
および java.util.concurrent.locks.LockSupport.unpark() のためのクラス.
実際にスレッドを待機状態にさせる(ブロックさせる)処理, および待機状態から復帰させる処理を行う (See: [here](no2114PYi.html) for details).

なお似たような機能を提供しているクラス(スレッドを待機状態にさせるクラス)として ParkEvent クラスが存在する (See: ParkEvent).
コメントによると, 
将来的には Parker クラスは廃止して ParkEvent に統一したい (重複箇所が多いので), 
とのこと.


```cpp
    ((cite: hotspot/src/share/vm/runtime/park.hpp))
    /*
     * Per-thread blocking support for JSR166. See the Java-level
     * Documentation for rationale. Basically, park acts like wait, unpark
     * like notify.
     *
     * 6271289 --
     * To avoid errors where an os thread expires but the JavaThread still
     * exists, Parkers are immortal (type-stable) and are recycled across
     * new threads.  This parallels the ParkEvent implementation.
     * Because park-unpark allow spurious wakeups it is harmless if an
     * unpark call unparks a new thread using the old Parker reference.
     *
     * In the future we'll want to think about eliminating Parker and using
     * ParkEvent instead.  There's considerable duplication between the two
     * services.
     *
     */
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/park.hpp))
    class Parker : public os::PlatformParker {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 適当な Parker オブジェクトを用意して Parker::park() を呼ぶと, park() を呼んだスレッドがブロックされる.
2. 別のスレッドがその Parker オブジェクトに対して Parker::unpark() を呼ぶと, ブロックしていたスレッドが起床される.

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 JavaThread オブジェクトの _parker フィールド

* Parker クラスの FreeList フィールド (static フィールド)
  
  (正確には, このフィールドは Parker の線形リストを格納するフィールド(フリーリスト).
  Parker オブジェクトは FreeNext フィールドで次の Parker オブジェクトを指せる構造になっている.
  このフィールドの線形リストに全ての未使用な Parker オブジェクトがつながれている.)

#### 参考(for your information): Parker::Release()
See: [here](no2114okz.html) for details
#### 生成箇所(where its instances are created)
Parker::Allocate() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
JavaThread::JavaThread()
-&gt; JavaThread::initialize()
   -&gt; Parker::Allocate()
</pre></div>

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

<div class="flow-abst"><pre>
* java.util.concurrent の処理 (java.util.concurrent.locks.LockSupport.park() および java.util.concurrent.locks.LockSupport.unpark() の処理)

  (略) (See: <a href="no2114PYi.html">here</a> for details)
  -&gt; Unsafe_Park()
     -&gt; Parker::park()

  (略) (See: <a href="no2114PYi.html">here</a> for details)
  -&gt; Unsafe_Unpark()
     -&gt; Parker::unpark()

* java.lang.Thread.interrupt() の処理 (java.util.concurrent.locks.LockSupport.park() で待機しているスレッドに割り込む処理)

  (略) (See: <a href="no2114A-l.html">here</a> for details)
  -&gt; os::interrupt()
     -&gt; * Linux の場合
          -&gt; Parker::unpark()
        * Solaris の場合
          -&gt; Parker::unpark()
        * Windows の場合
          -&gt; Parker::unpark()
</pre></div>

### 内部構造(Internal structure)
実際の処理のほとんどは os::PlatformParker クラスに丸投げしている.




### 詳細(Details)
See: [here](../doxygen/classParker.html) for details

---
## <a name="noEC9k4fuj" id="noEC9k4fuj">ParkEvent</a>

### 概要(Summary)
スレッドの待機処理用のユーティリティ・クラス.

スレッドを待機状態にさせる機能(ブロックさせる機能), および待機状態から復帰させる機能を提供している (See: [here](no2114COc.html) for details).

(<= 要するに, pthread_cond_wait() や WaitForSingleObject() のラッパークラス)

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 適当な ParkEvent オブジェクトを用意して ParkEvent::park() を呼ぶと, 
   park() を呼んだスレッドがブロックされる.
2. 別のスレッドがその ParkEvent オブジェクトに対して ParkEvent::unpark() を呼ぶと, ブロックしていたスレッドが起床される.
3. なお, ブロックにタイムアウトを付けたい場合は, 
   ParkEvent::park() の代わりに ParkEvent::park(jlong millis) を使えばいい.

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 Thread オブジェクトの _ParkEvent フィールド

* 各 Thread オブジェクトの _SleepEvent フィールド

* 各 Thread オブジェクトの _MutexEvent フィールド

* 各 Thread オブジェクトの _MuxEvent フィールド

* ParkEvent クラスの FreeList フィールド (static フィールド)
  
  (正確には, このフィールドは ParkEvent の線形リストを格納するフィールド(フリーリスト).
  ParkEvent オブジェクトは FreeNext フィールドで次の ParkEvent オブジェクトを指せる構造になっている.
  このフィールドの線形リストに全ての未使用な ParkEvent オブジェクトがつながれている.)

#### 参考(for your information): ParkEvent::Release()
See: [here](no2114auC.html) for details
#### 生成箇所(where its instances are created)
ParkEvent::Allocate() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

* Thread オブジェクトの初期化処理

  Thread::Thread()

* ?? (この関数は使用箇所が見当たらない #TODO)

  Thread::muxAcquireW()

* 

  Monitor::jvm_raw_lock()
  (ただし, ここで生成された ParkEvent はこの関数内でしか使わておらず, この関数内でフリーリストに戻される)

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (ロック待ちの処理, sleep 処理, wait 処理, 等)
(#TODO).

### 内部構造(Internal structure)
実際の処理のほとんどはスーパークラスの os::PlatformEvent クラスに丸投げしている.




### 詳細(Details)
See: [here](../doxygen/classParkEvent.html) for details

---
