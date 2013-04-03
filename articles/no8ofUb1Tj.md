---
layout: default
title: ObjectStartArray クラス 
---
[Top](../index.html)

#### ObjectStartArray クラス 



---
## <a name="nofdlVimyD" id="nofdlVimyD">ObjectStartArray</a>

### 概要(Summary)
ParallelScavenge 用の BlockOffsetTable クラス (See: BlockOffsetTable) (See: [here](no3718kvd.html) for details)

(継承関係としては BlockOffsetTable クラスと何の関係もないが, 果たしている役割は類似)


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/objectStartArray.hpp))
    // This class can be used to locate the beginning of an object in the
    // covered region.
    //
    
    class ObjectStartArray : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PSOldGen クラスの _start_array フィールドに(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psOldGen.hpp))
    class PSOldGen : public CHeapObj {
    ...
      ObjectStartArray         _start_array;       // Keeps track of where objects start in a 512b block
```

#### 生成箇所(where its instances are created)
(PSOldGen クラスの _start_array フィールドは, ポインタ型ではなく実体なので,
 PSOldGen オブジェクトの生成時に一緒に生成される)

### 内部構造(Internal structure)
なお現状の block size は, 2^9 = 512 byte.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/objectStartArray.hpp))
      enum BlockSizeConstants {
        block_shift                  = 9,
        block_size                   = 1 << block_shift,
        block_size_in_words          = block_size / sizeof(HeapWord)
      };
```




### 詳細(Details)
See: [here](../doxygen/classObjectStartArray.html) for details

---
