---
layout: default
title: ServiceThread クラス 
---
[Top](../index.html)

#### ServiceThread クラス 



---
## <a name="noi5B8BaqQ" id="noi5B8BaqQ">ServiceThread</a>

### 概要(Summary)
保守運用機能のためのクラス (JVMTI 機能及び JMM 機能用のクラス).

JVMTI/JMM のいくつかの種類のイベントについて, 対応するコールバック関数を呼び出すためのスレッドクラス.
より具体的に言うと, 以下のイベント通知で使用される.

  * JVMTI のイベントの遅延通知 (See: [here](no3718UPQ.html) for details)

  * JMM のメモリ使用量の閾値超過通知処理 (See: [here](no2114x0x.html) for details)

  * JMM の sun.management.GarbageCollectorImpl における通知機能 (See: [here](no2114KPr.html) for details)


```cpp
    ((cite: hotspot/src/share/vm/runtime/serviceThread.hpp))
    // A JavaThread for low memory detection support and JVMTI
    // compiled-method-load events.
    class ServiceThread : public JavaThread {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.
ただし, このクラスのインスタンス自体は1つしか存在しない (どちらも同じインスタンスを指している).

* ServiceThread クラスの _instance フィールド (static フィールド)

* Threads クラスの _thread_list フィールド (static フィールド)

  (正確には, このフィールドは JavaThread の線形リストを格納するフィールド.
  JavaThread オブジェクトは _next フィールドで次の JavaThread オブジェクトを指せる構造になっている.
  生成した JavaThread オブジェクトは全てこの線形リスト内に格納されている)

  (なお, この線形リストには Threads::add() で登録される)

#### 生成箇所(where its instances are created)
ServiceThread::initialize() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; Management::initialize()
      -&gt; ServiceThread::initialize()
</pre></div>

### 内部構造(Internal structure)
生成された ServiceThread スレッドの処理のエントリポイントは ServiceThread::service_thread_entry().
この中では以下の処理を無限ループで繰り返すだけ.

  1. Service_lock ロックに対して wait() して眠りに入る. 
  2. 対応するイベントが発生したら (その発生源が Service_lock に notify かけてくれるので) 起き出して処理を行う.

なお 2. における各イベントの処理内容は以下の通り.

  * LowMemoryDetector::has_pending_requests() の場合: (See: [here](no2114x0x.html) for details)

    LowMemoryDetector::process_sensor_changes(jt) を呼ぶ.

  * JvmtiDeferredEventQueue::has_events() の場合: (See: [here](no3718UPQ.html) for details)

    JvmtiDeferredEventQueue::dequeue() で event を取り出し,
    その event の post() メソッドを呼ぶ.

  * GCNotifier::has_event() の場合: (See: [here](no2114KPr.html) for details)

    GCNotifier::sendNotification(CHECK) を呼ぶ.

#### 参考(for your information): ServiceThread::service_thread_entry()
See: [here](no2935HXD.html) for details



### 詳細(Details)
See: [here](../doxygen/classServiceThread.html) for details

---
