---
layout: default
title: PhaseCoalesce 及びそのサブクラス (PhaseCoalesce, PhaseAggressiveCoalesce, PhaseConservativeCoalesce)
---
[Top](../index.html)

#### PhaseCoalesce 及びそのサブクラス (PhaseCoalesce, PhaseAggressiveCoalesce, PhaseConservativeCoalesce)

これらは, C2 JIT Compiler 内の処理フェーズを表すクラス.
より具体的に言うと, レジスタ割り当て処理における合併処理を表すクラス.


### クラス一覧(class list)

  * [PhaseCoalesce](#noyI51lHE2)
  * [PhaseAggressiveCoalesce](#noSTkOwAkp)
  * [PhaseConservativeCoalesce](#noEbFA-kAV)


---
## <a name="noyI51lHE2" id="noyI51lHE2">PhaseCoalesce</a>

### 概要(Summary)
PhaseChaitin クラス内で使用される補助クラス(の基底クラス).

Phase クラスのサブクラスの1つ.
レジスタ割り当てでの合併処理(coalesce)を行う.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/coalesce.hpp))
    //------------------------------PhaseCoalesce----------------------------------
    class PhaseCoalesce : public Phase {
```





### 詳細(Details)
See: [here](../doxygen/classPhaseCoalesce.html) for details

---
## <a name="noSTkOwAkp" id="noSTkOwAkp">PhaseAggressiveCoalesce</a>

### 概要(Summary)
PhaseChaitin クラス内で使用される補助クラス.

PhaseCoalesce クラスの具象サブクラスの1つ.
このクラスは, Chaitin の aggressive coalescing による合併を行う.


```
    ((cite: hotspot/src/share/vm/opto/coalesce.hpp))
    //------------------------------PhaseAggressiveCoalesce------------------------
    // Aggressively, pessimistic coalesce copies.  Aggressive means ignore graph
    // colorability; perhaps coalescing to the point of forcing a spill.
    // Pessimistic means we cannot coalesce if 2 live ranges interfere.  This
    // implies we do not hit a fixed point right away.
    class PhaseAggressiveCoalesce : public PhaseCoalesce {
```

### 使われ方(Usage)
PhaseChaitin::Register_Allocate() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPhaseAggressiveCoalesce.html) for details

---
## <a name="noEbFA-kAV" id="noEbFA-kAV">PhaseConservativeCoalesce</a>

### 概要(Summary)
PhaseChaitin クラス内で使用される補助クラス.

PhaseCoalesce クラスの具象サブクラスの1つ.
このクラスは, Briggs らの conservative coalescing による合併を行う.


```
    ((cite: hotspot/src/share/vm/opto/coalesce.hpp))
    //------------------------------PhaseConservativeCoalesce----------------------
    // Conservatively, pessimistic coalesce copies.  Conservative means do not
    // coalesce if the resultant live range will be uncolorable.  Pessimistic
    // means we cannot coalesce if 2 live ranges interfere.  This implies we do
    // not hit a fixed point right away.
    class PhaseConservativeCoalesce : public PhaseCoalesce {
```

### 使われ方(Usage)
PhaseChaitin::Register_Allocate() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPhaseConservativeCoalesce.html) for details

---
