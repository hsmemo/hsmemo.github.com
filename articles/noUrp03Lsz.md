---
layout: default
title: ConcurrentG1RefineThread クラス 
---
[Top](../index.html)

#### ConcurrentG1RefineThread クラス 



---
## <a name="node_R8ArZ" id="node_R8ArZ">ConcurrentG1RefineThread</a>

### 概要(Summary)
G1GC アルゴリズムを補佐するスレッドクラス(ConcurrentGCThreadクラス).
このクラスは, Remembered Set を適切に保つ処理を行う
(論文中では "concurrent remembered set thread"). (See: [here](no2935dGZ.html) for details)


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentG1RefineThread.hpp))
    // The G1 Concurrent Refinement Thread (could be several in the future).
    
    class ConcurrentG1RefineThread: public ConcurrentGCThread {
```

### 使われ方(Usage)
各 ConcurrentG1Refine オブジェクトの _threads フィールドに(のみ)格納されている
(「各」と言っても1つしかいないが...)

(正確には, このフィールドは ConcurrentG1RefineThread の配列を格納するフィールド.
この中に, 使用される全ての ConcurrentG1RefineThread オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
ConcurrentG1Refine::ConcurrentG1Refine() 内で(のみ)生成されている
(配列のメモリ領域も ConcurrentG1Refine::ConcurrentG1Refine() 内で(のみ)確保されている).

### 内部構造(Internal structure)
ConcurrentG1RefineThread は, 最初に最大数分だけ生成された後, 仕事の量に応じて動的に稼働数を変える
(仕事のないスレッドはブロックする).

これを実現するため,
各 ConcurrentG1RefineThread はそれぞれ個別の Monitor オブジェクトを保持している
(0 版目の ConcurrentG1RefineThread だけは, DirtyCardQ_CBL_mon を使用する).

各 ConcurrentG1RefineThread は, worker id が一つ小さい ConcurrentG1RefineThread によって起床される
(0 版目の ConcurrentG1RefineThread だけは, write barrier 処理から起床される). (See: [here](no2935dGZ.html) for details)


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentG1RefineThread.cpp))
      // Each thread has its own monitor. The i-th thread is responsible for signalling
      // to thread i+1 if the number of buffers in the queue exceeds a threashold for this
      // thread. Monitors are also used to wake up the threads during termination.
      // The 0th worker in notified by mutator threads and has a special monitor.
      // The last worker is used for young gen rset size sampling.
```


定義されているフィールドは以下の通り.

* _active : 現在稼働しているかどうかを示す (正確にはこの値を見てブロックするので, 予定を示す)
* _next : 1つ次の ConcurrentG1RefineThread オブジェクト (処理が増えてきた場合はこの ConcurrentG1RefineThread オブジェクトを起床させる. なおこの値はコンストラクタ引数で指定される)
* _thread_threshold_step, _deactivation_threshold : 稼働数を増減させる契機を決定するための閾値


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentG1RefineThread.hpp))
      double _vtime_start;  // Initial virtual time.
      double _vtime_accum;  // Initial virtual time.
      int _worker_id;
      int _worker_id_offset;
    
      // The refinement threads collection is linked list. A predecessor can activate a successor
      // when the number of the rset update buffer crosses a certain threshold. A successor
      // would self-deactivate when the number of the buffers falls below the threshold.
      bool _active;
      ConcurrentG1RefineThread* _next;
      Monitor* _monitor;
      ConcurrentG1Refine* _cg1r;
    
      int _thread_threshold_step;
      // This thread activation threshold
      int _threshold;
      // This thread deactivation threshold
      int _deactivation_threshold;
```




### 詳細(Details)
See: [here](../doxygen/classConcurrentG1RefineThread.html) for details

---
