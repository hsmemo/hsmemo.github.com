---
layout: default
title: PSMarkSweepDecorator クラス 
---
[Top](../index.html)

#### PSMarkSweepDecorator クラス 



---
## <a name="noZ7x2lC0v" id="noZ7x2lC0v">PSMarkSweepDecorator</a>

### 概要(Summary)
ParallelScavengeHeap の MarkSweep 処理
(UseParallelOldGC オプションが指定されていない場合の Major GC 処理)で使用される補助クラス(See: [here](no2114YqK.html) for details).

MutableSpace 上での (シングルスレッドでの) Mark-Sweep-Compact 処理を実装したクラス
(MarkSweep 処理のエントリポイント自体は PSMarkSweep クラスだが, 処理の肝心な部分を担当しているのは PSMarkSweepDecorator クラス)


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweepDecorator.hpp))
    //
    // A PSMarkSweepDecorator is used to add "ParallelScavenge" style mark sweep operations
    // to a MutableSpace.
    //
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweepDecorator.hpp))
    class PSMarkSweepDecorator: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
MutableSpace オブジェクトを保持しているクラス内に一緒に保持されている.

* PSYoundGen

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psYoungGen.hpp))
    class PSYoungGen : public CHeapObj {
    ...
      // Spaces
      MutableSpace* _eden_space;
      MutableSpace* _from_space;
      MutableSpace* _to_space;
    ...
      // MarkSweep Decorators
      PSMarkSweepDecorator* _eden_mark_sweep;
      PSMarkSweepDecorator* _from_mark_sweep;
      PSMarkSweepDecorator* _to_mark_sweep;
```

* PSOldGen

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psOldGen.hpp))
    class PSOldGen : public CHeapObj {
    ...
      MutableSpace*            _object_space;      // Where all the objects live
      PSMarkSweepDecorator*    _object_mark_sweep; // The mark sweep view of _object_space
```

### 内部構造(Internal structure)
実際の Mark-Sweep-Compact 処理を行う以下の3つのメソッドを備えている.

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweepDecorator.hpp))
      // Work methods
      void adjust_pointers();
      void precompact();
      void compact(bool mangle_free_space);
```

### 備考(Notes)
なお, PSMarkSweepDecorator クラスは
PSMarkSweepDecorator 型の _destination_decorator という static フィールドに持っている.

これは, コンパクション先の領域を示すもの.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweepDecorator.hpp))
      static PSMarkSweepDecorator* _destination_decorator;
```

(以下のようなアクセサメソッドも用意されている)

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweepDecorator.hpp))
      // During a compacting collection, we need to collapse objects into
      // spaces in a given order. We want to fill space A, space B, and so
      // on. The code that controls that order is in the following methods.
      static void set_destination_decorator_tenured();
      static void set_destination_decorator_perm_gen();
      static void advance_destination_decorator();
      static PSMarkSweepDecorator* destination_decorator();
```

具体的な使い方としては, 
コンパクション処理を開始する前にここにコンパクション先の領域を設定しておき, 
処理中ではこのフィールドを参照してコンパクション先のアドレスを決定する, という感じになる.

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.cpp))
      // Begin compacting into the old gen
      PSMarkSweepDecorator::set_destination_decorator_tenured();
    
      // This will also compact the young gen spaces.
      old_gen->precompact();
```




### 詳細(Details)
See: [here](../doxygen/classPSMarkSweepDecorator.html) for details

---
