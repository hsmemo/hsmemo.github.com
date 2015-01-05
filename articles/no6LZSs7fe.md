---
layout: default
title: HeapRegion クラス関連のクラス (HeapRegionDCTOC, G1OffsetTableContigSpace, HeapRegion, HeapRegionClosure, 及びそれらの補助クラス(VerifyLiveClosure, NextCompactionHeapRegionClosure))
---
[Top](../index.html)

#### HeapRegion クラス関連のクラス (HeapRegionDCTOC, G1OffsetTableContigSpace, HeapRegion, HeapRegionClosure, 及びそれらの補助クラス(VerifyLiveClosure, NextCompactionHeapRegionClosure))

これらは, G1GC で使用するメモリ領域を管理するためのクラス.
G1GC では, メモリ領域は HeapRegion という部分領域に分けて管理される.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegion.hpp))
    // A HeapRegion is the smallest piece of a G1CollectedHeap that
    // can be collected independently.
```

なお, HeapRegion は Space のサブクラスなのだが Space::initDirtyCardClosure() は呼び出すな, とのこと.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegion.hpp))
    // NOTE: Although a HeapRegion is a Space, its
    // Space::initDirtyCardClosure method must not be called.
    // The problem is that the existence of this method breaks
    // the independence of barrier sets from remembered sets.
    // The solution is to remove this method from the definition
    // of a Space.
```

なお, これらのクラスは以下のような継承関係を持つ.

  * ContiguousSpace  (<= これは別ファイルで定義されているクラス)
      * G1OffsetTableContigSpace
          * HeapRegion



### クラス一覧(class list)

  * [HeapRegion](#nojpdERqVs)
  * [HeapRegionDCTOC](#noT6i--03W)
  * [G1OffsetTableContigSpace](#nokB7-Bnhc)
  * [HeapRegionClosure](#noEzTorR_Q)
  * [VerifyLiveClosure](#no6DmPWkYp)
  * [NextCompactionHeapRegionClosure](#notSrkSD8u)


---
## <a name="nojpdERqVs" id="nojpdERqVs">HeapRegion</a>

### 概要(Summary)
G1CollectedHeap が管理するメモリ領域中の部分領域を表すクラス.

G1GC では, メモリ領域は HeapRegion という部分領域に分けて管理される.
G1GC ではこの部分領域単位で Garbage Collection 処理 (evacuation 処理) が行われる.

1つの HeapRegion オブジェクトが 1つの部分領域に対応する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegion.hpp))
    class HeapRegion: public G1OffsetTableContigSpace {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 G1CollectedHeap オブジェクトの _hrs フィールド
  
  (正確には, このフィールドは HeapRegionSeq オブジェクトを格納するフィールド.
  この中に, 使用される全ての HeapRegion オブジェクトが格納されている)

* G1AllocRegion クラスの _dummy_region フィールド (static フィールド)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* G1CollectedHeap::expand() 内
  
  G1CollectedHeap オブジェクトの _hrs フィールドに格納する HeapRegion を生成.

* G1CollectedHeap::initialize() 内
  
  G1AllocRegion クラスの _dummy_region フィールド用の HeapRegion を生成.




### 詳細(Details)
See: [here](../doxygen/classHeapRegion.html) for details

---
## <a name="noT6i--03W" id="noT6i--03W">HeapRegionDCTOC</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

HeapRegion 用の DirtyCardToOopClosure クラス (See: DirtyCardToOopClosure).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegion.hpp))
    // A dirty card to oop closure for heap regions. It
    // knows how to get the G1 heap and how to use the bitmap
    // in the concurrent marker used by G1 to filter remembered
    // sets.
    
    class HeapRegionDCTOC : public ContiguousSpaceDCTOC {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
HeapRegion::new_dcto_closure() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
(略) (See: <a href="no2935YzN.html">here</a> for details)
-&gt; ScanRSClosure::doHeapRegion()
   -&gt; ScanRSClosure::scanCard()
      -&gt; HeapRegion::new_dcto_closure()
</pre></div>

#### 使用箇所(where its instances are used)
ScanRSClosure::scanCard() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classHeapRegionDCTOC.html) for details

---
## <a name="nokB7-Bnhc" id="nokB7-Bnhc">G1OffsetTableContigSpace</a>

### 概要(Summary)
G1GC 用の OffsetTableContigSpace クラス (See: OffsetTableContigSpace).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

コメントによると, 
BlockOffsetTable がうまく流用できなかったので G1GC 版 (G1BlockOffsetTable) を作ったら,
ついでに OffsetTableContigSpace も G1GC 版を作る必要ができてしまった, 
とのこと
(なので, 
将来的に G1BlockOffsetTable と BlockOffsetTable が統合されれば G1OffsetTableContigSpace も統合されるかも, 
とも書かれている).

なお, 
OffsetTableContigSpace とは違って time stamp 情報も持っているが (_gc_time_stamp フィールド),
これは GC の度に全ての region に対して save_marks() しないですむように最適化するためのもの, 
とのこと (詳細は以下のコメント参照).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegion.hpp))
    // The complicating factor is that BlockOffsetTable diverged
    // significantly, and we need functionality that is only in the G1 version.
    // So I copied that code, which led to an alternate G1 version of
    // OffsetTableContigSpace.  If the two versions of BlockOffsetTable could
    // be reconciled, then G1OffsetTableContigSpace could go away.
    
    // The idea behind time stamps is the following. Doing a save_marks on
    // all regions at every GC pause is time consuming (if I remember
    // well, 10ms or so). So, we would like to do that only for regions
    // that are GC alloc regions. To achieve this, we use time
    // stamps. For every evacuation pause, G1CollectedHeap generates a
    // unique time stamp (essentially a counter that gets
    // incremented). Every time we want to call save_marks on a region,
    // we set the saved_mark_word to top and also copy the current GC
    // time stamp to the time stamp field of the space. Reading the
    // saved_mark_word involves checking the time stamp of the
    // region. If it is the same as the current GC time stamp, then we
    // can safely read the saved_mark_word field, as it is valid. If the
    // time stamp of the region is not the same as the current GC time
    // stamp, then we instead read top, as the saved_mark_word field is
    // invalid. Time stamps (on the regions and also on the
    // G1CollectedHeap) are reset at every cleanup (we iterate over
    // the regions anyway) and at the end of a Full GC. The current scheme
    // that uses sequential unsigned ints will fail only if we have 4b
    // evacuation pauses between two cleanups, which is _highly_ unlikely.
    
    class G1OffsetTableContigSpace: public ContiguousSpace {
```




### 詳細(Details)
See: [here](../doxygen/classG1OffsetTableContigSpace.html) for details

---
## <a name="noEzTorR_Q" id="noEzTorR_Q">HeapRegionClosure</a>

### 概要(Summary)
HeapRegion に対して何らかの処理を行う Closure クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス
(そして, サブクラスは大量にある...).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegion.hpp))
    // HeapRegionClosure is used for iterating over regions.
    // Terminates the iteration when the "doHeapRegion" method returns "true".
    class HeapRegionClosure : public StackObj {
```

### 使われ方(Usage)
HeapRegion を処理する doHeapRegion() メソッドを備えている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegion.hpp))
      // Typically called on each region until it returns true.
      virtual bool doHeapRegion(HeapRegion* r) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classHeapRegionClosure.html) for details

---
## <a name="no6DmPWkYp" id="no6DmPWkYp">VerifyLiveClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegion.cpp))
    class VerifyLiveClosure: public OopClosure {
```

### 使われ方(Usage)
HeapRegion::verify() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classVerifyLiveClosure.html) for details

---
## <a name="notSrkSD8u" id="notSrkSD8u">NextCompactionHeapRegionClosure</a>

### 概要(Summary)
G1CollectedHeap の Major GC 処理で使用される Closure クラス (See: [here](no2935ATn.html) for details).

コンストラクタ引数で指定された HeapRegion 以外で, かつ Humongous ではない最初の HeapRegion を取得する
(コンパクション先の HeapRegion を選ぶ際に, 
初めに選んだ HeapRegion が Humongous でしかも生きていた場合に, HeapRegion を選び直す処理で使用される).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegion.cpp))
    class NextCompactionHeapRegionClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
HeapRegion::next_compaction_space() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
(略) (See: <a href="no2935ATn.html">here</a> for details)
-&gt; G1MarkSweep::mark_sweep_phase2()
   -&gt; HeapRegion::next_compaction_space()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classNextCompactionHeapRegionClosure.html) for details

---
