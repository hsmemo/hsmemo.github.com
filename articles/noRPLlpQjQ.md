---
layout: default
title: PhaseLive クラス 
---
[Top](../index.html)

#### PhaseLive クラス 



---
## <a name="no4brMRKfC" id="no4brMRKfC">PhaseLive</a>

### 概要(Summary)
PhaseChaitin クラス内で使用される補助クラス
(つまり, このクラスは現在はレジスタ割り当て処理でのみ使用されている).

Phase クラスの具象サブクラスの1つ.
生存区間解析(liveness analysis)を行う.


```cpp
    ((cite: hotspot/src/share/vm/opto/live.hpp))
    //------------------------------PhaseLive--------------------------------------
    // Compute live-in/live-out
    class PhaseLive : public Phase {
```


```cpp
    ((cite: hotspot/src/share/vm/opto/live.cpp))
    //=============================================================================
    //------------------------------PhaseLive--------------------------------------
    // Compute live-in/live-out.  We use a totally incremental algorithm.  The LIVE
    // problem is monotonic.  The steady-state solution looks like this: pull a
    // block from the worklist.  It has a set of delta's - values which are newly
    // live-in from the block.  Push these to the live-out sets of all predecessor
    // blocks.  At each predecessor, the new live-out values are ANDed with what is
    // already live-out (extra stuff is added to the live-out sets).  Then the
    // remaining new live-out values are ANDed with what is locally defined.
    // Leftover bits become the new live-in for the predecessor block, and the pred
    // block is put on the worklist.
    //   The locally live-in stuff is computed once and added to predecessor
    // live-out sets.  This separate compilation is done in the outer loop below.
```

(#TODO  MethodLiveness と PhaseLive と buildOopMap でやっている liveness analysis はそれぞれどう違う？)

### 使われ方(Usage)
PhaseChaitin::Register_Allocate() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPhaseLive.html) for details

---
