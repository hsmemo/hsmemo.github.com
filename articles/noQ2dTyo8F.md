---
layout: default
title: Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理
---
[Up](no30267vB.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理

--- 
## 概要(Summary)
GC 処理は CollectedHeap::mem_allocate() メソッドで行われる (See: [here](no28916Q0G.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/gc_interface/collectedHeap.hpp))
      // Raw memory allocation facilities
      // The obj and array allocate methods are covers for these methods.
      // The permanent allocation method should default to mem_allocate if
      // permanent memory isn't supported.
      virtual HeapWord* mem_allocate(size_t size,
                                     bool is_noref,
                                     bool is_tlab,
                                     bool* gc_overhead_limit_was_exceeded) = 0;
```

ただし, 実際にはこのメソッドは各サブクラスでオーバーライドされているため, それぞれのサブクラス毎 (= GC アルゴリズム毎) の処理が呼び出される.

* ParallelScavengeHeap の場合: 

  (See: [here](no3718vrX.html) for details)

* G1collectedheap の場合:
  
  (See: [here](no28916fAb.html) for details)

* GenCollectedHeap の場合:
  
  (See: [here](no28916sKh.html) for details)

## 備考(Notes)
なお, CollectedHeap::mem_allocate() の引数の意味は以下の通り.

  * size
    
    確保したいサイズ

  * is_noref (宣言箇所によっては is_large_noref という引数名になっている所も)
    
    ?? (そもそも何に使われている?? 有効に利用されている箇所が見当たらないが... #TODO)

  * is_tlab

    確保するのが TLAB かどうかを示す (true なら TLAB の確保).
    現状では, GenCollectedHeap::allocate_new_tlab() からの呼び出し時にのみ true になる模様.

  * gc_overhead_limit_was_exceeded

    呼び出し元から渡される bool 変数へのポインタ.
    CollectedHeap::mem_allocate() 内での確保処理が adaptive size policy で指定された時間制約が満たせなかった場合に true にセットされる.


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.hpp))
      // This method controls how a collector satisfies a request
      // for a block of memory.  "gc_time_limit_was_exceeded" will
      // be set to true if the adaptive size policy determine that
      // an excessive amount of time is being spent doing collections
      // and caused a NULL to be returned.  If a NULL is not returned,
      // "gc_time_limit_was_exceeded" has an undefined meaning.
```




## Subcategories
* [Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： ParallelScavengeHeap の場合](no3718vrX.html)
* [Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： G1CollectedHeap の場合 ](no28916fAb.html)
* [Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： GenCollectedHeap の場合 ](no28916sKh.html)



