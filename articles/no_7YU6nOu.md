---
layout: default
title: BarrierSet クラス 
---
[Top](../index.html)

#### BarrierSet クラス 



---
## <a name="no7U8DlGaj" id="no7U8DlGaj">BarrierSet</a>

### 概要(Summary)
Garbage Collection 処理用の補助クラス (See: [here](no3718kvd.html) for details).

write barrier 処理で使われるクラスで, Java ヒープ中で変更された箇所を記録しておくためのもの (See: [here](no2114EV0.html) for details).


```
    ((cite: hotspot/src/share/vm/memory/barrierSet.hpp))
    // This class provides the interface between a barrier implementation and
    // the rest of the system.
    
    class BarrierSet: public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```
    ((cite: hotspot/src/share/vm/memory/barrierSet.hpp))
      // These operations indicate what kind of barriers the BarrierSet has.
      virtual bool has_read_ref_barrier() = 0;
      virtual bool has_read_prim_barrier() = 0;
      virtual bool has_write_ref_barrier() = 0;
      virtual bool has_write_ref_pre_barrier() = 0;
      virtual bool has_write_prim_barrier() = 0;
```




### 詳細(Details)
See: [here](../doxygen/classBarrierSet.html) for details

---
