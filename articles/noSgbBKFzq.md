---
layout: default
title: G1MMUTracker クラス関連のクラス (G1MMUTracker, G1MMUTrackerQueueElem, G1MMUTrackerQueue)
---
[Top](../index.html)

#### G1MMUTracker クラス関連のクラス (G1MMUTracker, G1MMUTrackerQueueElem, G1MMUTrackerQueue)

これらは, G1CollectorPolicy クラスの補助クラス.
ユーザーから指定されたソフトリアルタイム制約を満たすために, GC 処理の時間を追跡して「いつからなら GC 処理を行っていいか」を判断する (See: G1CollectorPolicy).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MMUTracker.hpp))
    // Keeps track of the GC work and decides when it is OK to do GC work
    // and for how long so that the MMU invariants are maintained.
```

なお, 数値の単位は全部「秒」, とのこと.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MMUTracker.hpp))
    /***** ALL TIMES ARE IN SECS!!!!!!! *****/
```


### クラス一覧(class list)

  * [G1MMUTracker](#no0UqDwp1X)
  * [G1MMUTrackerQueueElem](#noiw3iFQ5D)
  * [G1MMUTrackerQueue](#nonpMDBOY8)


---
## <a name="no0UqDwp1X" id="no0UqDwp1X">G1MMUTracker</a>

### 概要(Summary)
G1CollectorPolicy クラス用の補助クラス(の基底クラス).

ユーザーから指定されたソフトリアルタイム制約を満たすために, 
GC 処理の時間を追跡して「いつからなら GC 処理を行っていいか」を判断する.

このために, MaxGCPauseMillis や GCPauseIntervalMillis といったコマンドラインオプションの値を管理し,
また実際に Concurrent Mark 等で停止していた時間を記録している.
これらに基づいて, (上記のオプションで指定された) 制約を守るために必要となる時間を計算する.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MMUTracker.hpp))
    // this is the "interface"
    class G1MMUTracker: public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classG1MMUTracker.html) for details

---
## <a name="noiw3iFQ5D" id="noiw3iFQ5D">G1MMUTrackerQueueElem</a>

### 概要(Summary)
G1MMUTrackerQueue クラス内で使用される補助クラス(ValueObjクラス).

GC 処理による停止時間の情報を記録しておくためのクラス.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MMUTracker.hpp))
    class G1MMUTrackerQueueElem VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1MMUTrackerQueue オブジェクトの _array フィールドに(のみ)格納されている

(正確には, このフィールドは G1MMUTrackerQueueElem の配列を格納するフィールド.
この中に, その G1MMUTrackerQueue 内で使用される全ての G1MMUTrackerQueueElem オブジェクトが格納されている).

#### 生成箇所(where its instances are created)
(G1MMUTrackerQueue クラスの _array フィールドは, ポインタ型ではなく実体なので,
 G1MMUTrackerQueue オブジェクトの生成時に一緒に生成される)

#### 情報の記録箇所(where information is recorded)
値の更新処理は以下の箇所で行われている模様. (#TODO 他にもあるか??)

* G1MMUTrackerQueue::add_pause()

#### 使用箇所(where its instances are used)
記録された情報は以下の箇所で参照されている. (#TODO 他にもあるか??)

* G1MMUTrackerQueue::calculate_gc_time()
* G1MMUTrackerQueue::when_internal() 

### 内部構造(Internal structure)
定義されているフィールドはこれだけ
(メソッドもこれらのフィールドへのアクセサメソッドのみ).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MMUTracker.hpp))
      double _start_time;
      double _end_time;
```




### 詳細(Details)
See: [here](../doxygen/classG1MMUTrackerQueueElem.html) for details

---
## <a name="nonpMDBOY8" id="nonpMDBOY8">G1MMUTrackerQueue</a>

### 概要(Summary)
G1MMUTracker クラスの具象サブクラス.

固定長のキューに直近の GC 処理による停止時間を記録しておくことで処理を実現している.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MMUTracker.hpp))
    // this is an implementation of the MMUTracker using a (fixed-size) queue
    // that keeps track of all the recent pause times
    class G1MMUTrackerQueue: public G1MMUTracker {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectorPolicy オブジェクトの _mmu_tracker フィールドに(のみ)格納されている

#### 生成箇所(where its instances are created)
G1CollectorPolicy::G1CollectorPolicy() 内で(のみ)生成されている.

### 内部構造(Internal structure)
内部は, G1MMUTrackerQueueElem オブジェクトの配列として実現されている
(この配列をリングバッファとして使用している).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MMUTracker.hpp))
      // The array keeps track of all the pauses that fall within a time
      // slice (the last time slice during which pauses took place).
      // The data structure implemented is a circular queue.
      // Head "points" to the most recent addition, tail to the oldest one.
      // The array is of fixed size and I don't think we'll need more than
      // two or three entries with the current behaviour of G1 pauses.
      // If the array is full, an easy fix is to look for the pauses with
      // the shortest gap between them and consolidate them.
      // For now, we have taken the expedient alternative of forgetting
      // the oldest entry in the event that +G1UseFixedWindowMMUTracker, thus
      // potentially violating MMU specs for some time thereafter.
    
      G1MMUTrackerQueueElem _array[QueueLength];
      int                   _head_index;
      int                   _tail_index;
```

なお現状の実装では, 配列の長さは 64 で固定.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MMUTracker.hpp))
      enum PrivateConstants {
        QueueLength = 64
      };
```





### 詳細(Details)
See: [here](../doxygen/classG1MMUTrackerQueue.html) for details

---
