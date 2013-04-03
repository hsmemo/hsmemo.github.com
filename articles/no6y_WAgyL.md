---
layout: default
title: Phase クラス 
---
[Top](../index.html)

#### Phase クラス 



---
## <a name="notbxvKiku" id="notbxvKiku">Phase</a>

### 概要(Summary)
C2 JIT コンパイラの作業中に使われる一時オブジェクト(StackObjクラス) (の基底クラス).

C2 JIT コンパイラの「処理フェーズ」を表すクラス.
C2 JIT コンパイラ内で行われる各処理は Phase クラスのサブクラスとして実装されている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/phase.hpp))
    //------------------------------Phase------------------------------------------
    // Most optimizations are done in Phases.  Creating a phase does any long
    // running analysis required, and caches the analysis in internal data
    // structures.  Later the analysis is queried using transform() calls to
    // guide transforming the program.  When the Phase is deleted, so is any
    // cached analysis info.  This basic Phase class mostly contains timing and
    // memory management code.
    class Phase : public StackObj {
```





### 詳細(Details)
See: [here](../doxygen/classPhase.html) for details

---
