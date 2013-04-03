---
layout: default
title: HeapRegionSeq クラス (HeapRegionSeq, 及びその補助クラス(PrintHeapRegionClosure))
---
[Top](../index.html)

#### HeapRegionSeq クラス (HeapRegionSeq, 及びその補助クラス(PrintHeapRegionClosure))

これらは, G1GC で使用するメモリ領域を管理するためのクラス.


### クラス一覧(class list)

  * [HeapRegionSeq](#noQVhr1zKi)
  * [PrintHeapRegionClosure](#noBkv9wFwN)


---
## <a name="noQVhr1zKi" id="noQVhr1zKi">HeapRegionSeq</a>

### 概要(Summary)
G1CollectedHeap クラス内で使用される補助クラス.

全ての HeapRegion オブジェクトを束ねておくためのコンテナクラス (See: HeapRegion).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSeq.hpp))
    class HeapRegionSeq: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _hrs フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
G1CollectedHeap::initialize() 内で(のみ)生成されている.

### 内部構造(Internal structure)
内部的には GrowableArray<HeapRegion*> を用いて HeapRegion を管理している.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSeq.hpp))
      // _regions is kept sorted by start address order, and no two regions are
      // overlapping.
      GrowableArray<HeapRegion*> _regions;
```




### 詳細(Details)
See: [here](../doxygen/classHeapRegionSeq.html) for details

---
## <a name="noBkv9wFwN" id="noBkv9wFwN">PrintHeapRegionClosure</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらないような...)

デバッグ用(開発時用)のクラス(だと思われる).

HeapRegion の情報を出力する.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSeq.cpp))
    class PrintHeapRegionClosure : public  HeapRegionClosure {
```

### 使われ方(Usage)
HeapRegionSeq::print() 内で(のみ)使用されている
(が, この関数自体が使われていないような...?? 一応 _hrs の使用箇所を全部見てみたが... #TODO)




### 詳細(Details)
See: [here](../doxygen/classPrintHeapRegionClosure.html) for details

---
