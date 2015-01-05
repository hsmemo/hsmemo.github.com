---
layout: default
title: 同期排他処理 ： java.lang.Object.wait(), java.lang.Object.notify(), java.lang.Object.notifyAll() の処理  
---
[Up](no2114NIs.html) [Top](../index.html)

#### 同期排他処理 ： java.lang.Object.wait(), java.lang.Object.notify(), java.lang.Object.notifyAll() の処理  

--- 
## 概要(Summary)
java.lang.Object.wait() メソッド, java.lang.Object.notify() メソッド, 及び java.lang.Object.notifyAll() メソッドは, 
java.lang.Object クラスのネイティブメソッドとして実装されている.
これらのメソッドを呼び出すとそれぞれ以下の CVMI 関数が呼び出される.

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table26814WPs -->
| Method | CVMI function |
|---|---|
| java.lang.Object.wait() | JVM_MonitorWait() |
| java.lang.Object.notify() | JVM_MonitorNotify() |
| java.lang.Object.notifyAll() | JVM_MonitorNotifyAll() |
<!-- END RECEIVE ORGTBL table26814WPs -->

<!-- 
#+ORGTBL: SEND table26814WPs orgtbl-to-gfm :no-escape t
| Method                       | CVMI function          |
|------------------------------+------------------------|
| java.lang.Object.wait()      | JVM_MonitorWait()      |
| java.lang.Object.notify()    | JVM_MonitorNotify()    |
| java.lang.Object.notifyAll() | JVM_MonitorNotifyAll() |
-->



```cpp
    ((cite: jdk/src/share/native/java/lang/Object.c))
    static JNINativeMethod methods[] = {
        {"hashCode",    "()I",                    (void *)&JVM_IHashCode},
        {"wait",        "(J)V",                   (void *)&JVM_MonitorWait},
        {"notify",      "()V",                    (void *)&JVM_MonitorNotify},
        {"notifyAll",   "()V",                    (void *)&JVM_MonitorNotifyAll},
        {"clone",       "()Ljava/lang/Object;",   (void *)&JVM_Clone},
    };
```

なお, wait() 処理が必要とする「待機しているスレッドの管理」用のデータ構造は inflated 状態にしか存在しない.
このため, wait()/notify()/notifyAll() が呼び出された場合, ロック状態は強制的に inflated 状態になる.

  * wait() が呼ばれると, stack-locked や inflated (locked) から inflated (unlocked) に遷移する.

    wait() はロックを取得しているスレッドからしか呼び出せないと規程されているため,
    neutral や inflated (unlocked) 状態で呼ばれると IllegalMonitorException が発生する.

  * notify()/notifyAll() は inflated (locked) で実行され, 状態遷移はしない.

より具体的にいうと, wait()/notify() の待ちキューは ObjectMonitor オブジェクトの _WaitSet フィールドで管理している.
_WaitSet は ObjectWaiter オブジェクトで構成された doubly linked list であり, 
以下のように使用される (See: ObjectMonitor).

  * wait() メソッドは, 呼び出したスレッドを _WaitSet に登録した後, 一時停止(park())させる.

  * notify()/notifyAll() は, _WaitSet 中のスレッドを _EntryList か _cxq に移動させる.

    (_EntryList と _cxq はロック解放を待っているスレッド用の待ち行列 (See: [here](noXF2ZIHEZ.html) for details).
     どちらに移動するかは Knob_MoveNotifyee の値に応じて決まる)

    移動されたスレッドは, 
    それ以降のロック解放処理時(monitorexit バイトコード実行時/synchronized メソッドからの脱出時)に起床(unpark())される.

なお, 現状では notify() と notifyAll() の実装は非常に似ている.
違いは, notify() は _WaitSet から 1つスレッドを選んで移動させるが,
notifyAll() は _WaitSet 中の全スレッドを移動させるという点だけ.

また, wait()/notify()/notifyAll() では,
timeout や interrupt による起床と, 実際に notify されたことによる起床を区別するために
ObjectWaiter オブジェクトの _notified というフィールドを使っている.
このフィールドは ObjectWaiter オブジェクトが生成された直後は 0 だが, ObjectMonitor::notify() で起こされると 1 になる
(つまり, 起床した際に _notified がゼロのままなら notify() 以外の原因による起床) (See: ObjectWaiter).

## 備考(Notes)
正確には, java.lang.Object.wait() はオーバーロードされており 3つのバージョンが存在する.

ネイティブメソッドとして定義されているのは java.lang.Object.wait(long timeout).
その他の 2つ (java.lang.Object.wait(), java.lang.Object.wait(long timeout, int nanos)) はそのラッパーになっている.

## 処理の流れ (概要)(Execution Flows : Summary)
### java.lang.Object.wait(long timeout) の処理の流れ
<div class="flow-abst"><pre>
JVM_MonitorWait()
-&gt; ObjectSynchronizer::wait()
   -&gt; BiasedLocking::revoke_and_rebias()
   -&gt; ObjectSynchronizer::inflate()
   -&gt; ObjectMonitor::wait()
      -&gt; CHECK_OWNER()
      -&gt; ObjectMonitor::AddWaiter()
      -&gt; ObjectMonitor::exit()
         -&gt; (See: <a href="noS3vRzujM.html">here</a> for details)
      -&gt; os::PlatformEvent::park()  or  os::PlatformEvent::park(jlong millis)
         -&gt; (See: <a href="no2114COc.html">here</a> for details)
      -&gt; ObjectMonitor::DequeueSpecificWaiter()
      -&gt; ObjectMonitor::enter()  or  ObjectMonitor::ReenterI()
         -&gt; (See: <a href="no96623Ns.html">here</a> for details)
</pre></div>

### java.lang.Object.wait() 及び java.lang.Object.wait(long timeout, int nanos) の処理の流れ
<div class="flow-abst"><pre>
java.lang.Object.wait()
-&gt; java.lang.Object.wait(long timeout)
   -&gt; (上述)

java.lang.Object.wait(long timeout, int nanos)
-&gt; java.lang.Object.wait(long timeout)
   -&gt; (上述)
</pre></div>

### java.lang.Object.notify() の処理の流れ
<div class="flow-abst"><pre>
JVM_MonitorNotify()
-&gt; ObjectSynchronizer::notify()
   -&gt; ObjectMonitor::notify()
      -&gt; CHECK_OWNER()
      -&gt; ObjectMonitor::DequeueWaiter()
</pre></div>

### java.lang.Object.notifyAll() の処理の流れ
<div class="flow-abst"><pre>
JVM_MonitorNotifyAll()
-&gt; ObjectSynchronizer::notifyall()
   -&gt; ObjectMonitor::notifyAll()
      -&gt; CHECK_OWNER()
      -&gt; ObjectMonitor::DequeueWaiter()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### java.lang.Object.wait()
See: [here](no4230Q2z.html) for details
### java.lang.Object.wait(long timeout, int nanos)
See: [here](no319774ok.html) for details

### JVM_MonitorWait()
See: [here](no4230cUP.html) for details
### ObjectSynchronizer::wait()
See: [here](no4230peV.html) for details
### ObjectMonitor::wait()
See: [here](no42302ob.html) for details
### CHECK_OWNER()
See: [here](no4230Dzh.html) for details
### ObjectMonitor::AddWaiter()
See: [here](no3059owy.html) for details
### ObjectMonitor::DequeueSpecificWaiter()
See: [here](no3059nEI.html) for details
### ObjectMonitor::ReenterI()
See: [here](no3059Oja.html) for details
### ObjectWaiter::wait_reenter_end()
See: [here](no3059BZU.html) for details


### JVM_MonitorNotify()
See: [here](no3059Ocm.html) for details
### ObjectSynchronizer::notify()
See: [here](no4230dHu.html) for details
### ObjectMonitor::notify()
See: [here](no4230qR0.html) for details
### ObjectMonitor::DequeueWaiter()
See: [here](no3059a6B.html) for details
### ObjectWaiter::wait_reenter_begin()
See: [here](no30590OO.html) for details

### JVM_MonitorNotifyAll()
See: [here](no3059bms.html) for details
### ObjectSynchronizer::notifyall()
See: [here](no4230cbD.html) for details
### ObjectMonitor::notifyAll()
See: [here](no4230plJ.html) for details






