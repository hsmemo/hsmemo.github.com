---
layout: default
title: 同期排他処理 ： 初期化処理 ： biased locking 用の初期化処理
---
[Up](noPiTkAf85.html) [Top](../index.html)

#### 同期排他処理 ： 初期化処理 ： biased locking 用の初期化処理

--- 
## 概要(Summary)
Biased Locking 機能は HotSpot の起動時に初期化される.
この初期化処理では以下の 2つの処理が行われる.

* 既にロード済みのクラスオブジェクトについては, prototype header の値を biased locking pattern に置き換える.
* これからロードするクラスオブジェクトのためには, BiasedLocking::enabled() が true を返すように設定する.

これらの処理は VM_EnableBiasedLocking クラスで行われる.

ただし, BiasedLocking を起動直後から有効にしておくと, 
起動時に大量の revoke が発生して遅くなる恐れがある.
このため, ある程度時間が経過してから有効化できるようになっている.
この時間は BiasedLockingStartupDelay オプションで調整できる.

## 処理の流れ (概要)(Execution Flows : Summary)
### HotSpot 起動時の初期化処理
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; BiasedLocking::init()
      (BiasedLockingStartupDelay オプションの値に応じて, 以下のどちらかが行われる)
      * BiasedLockingStartupDelay が 0 よりも大きい値の場合:
        (EnableBiasedLockingTask が生成され, 指定時間後に VM_EnableBiasedLocking が呼び出される)
        -&gt; EnableBiasedLockingTask::EnableBiasedLockingTask()
        -&gt; PeriodicTask::enroll()
      * そうでない場合: 
        (この関数内で VM_EnableBiasedLocking により biased locking が有効化される)
        -&gt; VMThread::execute()
           -&gt; (See: <a href="no2935qaz.html">here</a> for details)
              -&gt; VM_EnableBiasedLocking::doit()
</pre></div>

### 上記の初期化処理で登録された EnableBiasedLockingTask の処理
<div class="flow-abst"><pre>
WatcherThread::run()
-&gt; (See: <a href="nohcAO37b3.html">here</a> for details)
   -&gt; EnableBiasedLockingTask::task()
      -&gt; VMThread::execute()
         -&gt; (See: <a href="no2935qaz.html">here</a> for details)
            -&gt; VM_EnableBiasedLocking::doit()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### BiasedLocking::init()
See: [here](no2480c2M.html) for details
### EnableBiasedLockingTask::EnableBiasedLockingTask()
See: [here](no319773Z1.html) for details
### PeriodicTask::enroll()
See: [here](no31977pjE.html) for details
### EnableBiasedLockingTask::task()
See: [here](no2480pAT.html) for details
### VM_EnableBiasedLocking::doit()
See: [here](no24802KZ.html) for details
### enable_biased_locking()
See: [here](no2480Qfl.html) for details
### BiasedLocking::enabled()
See: [here](no2480dpr.html) for details






