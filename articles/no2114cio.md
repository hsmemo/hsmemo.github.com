---
layout: default
title: Thread の待機処理の枠組み ： Mutex, Monitor による処理
---
[Up](noIpUCxk3g.html) [Top](../index.html)

#### Thread の待機処理の枠組み ： Mutex, Monitor による処理

--- 
## 概要(Summary)
これらは ParkEvent 及び SpinLock, Mux を用いて構築された Mutex 及びモニタ実装.
長い critical section にも利用できる
(See: Mutex, Monitor).

内部的には, 各 Thread オブジェクトが _MutexEvent というフィールドに ParkEvent オブジェクトを保持している.
これが Mutex や Monitor の処理 (lock/unlock, wait/notify/notifyAll) で使用される.

* Monitor の lock 処理では,
  各スレッドが自分の _MutexEvent を Monitor に登録してから, 
  その _MutexEvent に対して park() を呼んで待機する.

* 逆に unlock 処理では, 登録されている ParkEvent オブジェクトの中から
  1つを選んで unpark() させることで, 寝ているスレッドを起床させる.

* wait 処理では


* なお, wait/notify/notifyAll では,
  spurious wake-up を防ぐために
  ParkEvent オブジェクトの Notified というフィールドを使っている.
  
  Monitor::IWait() で寝る前に Notified をゼロにしておき,
  Monitor::notify() で起こした対象の Notified を 1 にする.
  起きたスレッドは, Notified がゼロのままなら再び眠りにつく.


## 処理の流れ (概要)(Execution Flows : Summary)
### Monitor::lock()  (Mutex::lock())
<div class="flow-abst"><pre>
Monitor::lock()
-&gt; Monitor::TryFast()           (成功すれば, ここでリターン)
-&gt; Monitor::TrySpin()           (成功すれば, ここでリターン)
   -&gt; Monitor::TryLock()        (成功すれば, ここでリターン)
   -&gt; 適当な回数だけ CAS によるロックを試みる.
      -&gt; CASPTR() マクロ
      -&gt; SpinPause() で時間をつぶす
      -&gt; MarsagliaXORV() 又は Stall() で時間をつぶす
-&gt; Monitor::ILock()
   -&gt; Monitor::TryFast()        (成功すれば, ここでリターン)
   -&gt; Monitor::TrySpin()        (成功すれば, ここでリターン)
   -&gt; Monitor::AcquireOrPush()  (成功すれば, ここでリターン) (失敗した場合は, 待ち行列に追加される)
   -&gt; ParkCommon() で待機する. この中では以下のどちらかを呼び出す
      * タイムアウト時間が指定されていない場合
        -&gt; os::PlatformEvent::park()
      * タイムアウト時間が指定されている場合
        -&gt; os::PlatformEvent::park(jlong millis)
   -&gt; Monitor::TrySpin()
</pre></div>

### Monitor::unlock()  (Mutex::unlock())
<div class="flow-abst"><pre>
Monitor::unlock()
-&gt; Monitor::IUnlock()
</pre></div>

### Monitor::wait()
<div class="flow-abst"><pre>
Monitor::wait()
-&gt; Monitor::IWait()
   -&gt; カレントスレッドを _WaitSet に登録する
   -&gt; Monitor::IUnlock()
   -&gt; ParkCommon()
   -&gt; Monitor::ILock()
</pre></div>

### Monitor::notify()
<div class="flow-abst"><pre>
Monitor::notify()
-&gt; _WaitSet の先頭のスレッドを cxq に移動させる
   (これで Monitor::notify() を呼んだスレッドが unlock した際 (あるいはそれ以降の unlock() 時) に unpark() されるようになる)
</pre></div>

### Monitor::notify_all()
<div class="flow-abst"><pre>
Monitor::notify_all()
-&gt; Monitor::notify()  (← wait しているスレッド数分だけ呼び出す)
   -&gt; (上述)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### Mutex::lock()
See: [here](no2114CVQ.html) for details
### Monitor::lock(Thread * Self)
See: [here](no2114PfW.html) for details
### Monitor::TryFast()
See: [here](no2114pzi.html) for details
### CASPTR() マクロ
See: [here](no2114cwQ.html) for details
### Monitor::set_owner()
See: [here](no211429o.html) for details
### Monitor::TrySpin()
See: [here](no2114DIv.html) for details
### Monitor::TryLock()
See: [here](no2114p6W.html) for details
### MarsagliaXORV()
See: [here](no9662cGB.html) for details
### Stall()
See: [here](no9662peE.html) for details
### Monitor::ILock()
See: [here](no2114QS1.html) for details
### UNS() マクロ
See: [here](no2114DPj.html) for details
### Monitor::AcquireOrPush()
See: [here](no2114CcE.html) for details
### ParkCommon()
See: [here](no2114QZp.html) for details

### Mutex::unlock()
See: [here](no2114djv.html) for details
### Monitor::IUnlock()
See: [here](no2114qt1.html) for details

### Monitor::wait()
See: [here](no2114220.html) for details
### Monitor::IWait()
See: [here](no24825p73.html) for details

### Monitor::notify()
See: [here](no21141KK.html) for details

### Monitor::notify_all()
See: [here](no2114oAE.html) for details






