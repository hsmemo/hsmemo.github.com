---
layout: default
title: ParMarkBitMap クラス 
---
[Top](../index.html)

#### ParMarkBitMap クラス 



---
## <a name="noUa4zL47u" id="noUa4zL47u">ParMarkBitMap</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス
(つまり, Parallel Compaction 処理中で使用される補助クラス).

Mark Sweep Compact 処理の phase 1 処理 (marking 処理) の結果を格納するビットマップ.
格納された情報は phase 3 や phase 4 の処理で使用される (See: [here](no28916Gft.html) for details).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parMarkBitMap.hpp))
    class ParMarkBitMap: public CHeapObj
    {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PSParallelCompact クラスの _mark_bitmap フィールドに(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class PSParallelCompact : AllStatic {
    ...
      static ParMarkBitMap        _mark_bitmap;
```

#### 生成箇所(where its instances are created)
(PSParallelCompact クラスの _mark_bitmap フィールドは, ポインタ型ではなく実体なので,
 初期段階で自動的に生成される)

### 内部構造(Internal structure)
内部には, ２本の BitMap を保持している.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parMarkBitMap.hpp))
      BitMap          _beg_bits;
      BitMap          _end_bits;
```

これらの BitMap は, デフォルトでは全部の bit が 0 になっている.
marking 処理で live オブジェクトが見つかると, 該当する箇所の bit が 1 に変更される.

(より正確に言うと, 各 live オブジェクトに対して,
 _beg_bits 中のそのオブジェクトの先頭に位置する箇所が 1 になり,
 _end_bits 中のそのオブジェクトの終端に位置する箇所が 1 になる.)

なお, これらの BitMap 中の bit は, メモリ領域中の各 MinObjAlignment バイトの領域に対応する
(具体的な MinObjAlignment の大きさについては, set_object_alignment() 参照).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parMarkBitMap.hpp))
      // Each bit in the bitmap represents one unit of 'object granularity.' Objects
      // are double-word aligned in 32-bit VMs, but not in 64-bit VMs, so the 32-bit
      // granularity is 2, 64-bit is 1.
      static inline size_t obj_granularity() { return size_t(MinObjAlignment); }
      static inline int obj_granularity_shift() { return LogMinObjAlignment; }
```




### 詳細(Details)
See: [here](../doxygen/classParMarkBitMap.html) for details

---
