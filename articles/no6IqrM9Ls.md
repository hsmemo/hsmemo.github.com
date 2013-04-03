---
layout: default
title: G1MarkSweep クラス (G1MarkSweep, 及びその補助クラス(G1PrepareCompactClosure, FindFirstRegionClosure, G1AdjustPointersClosure, G1SpaceCompactClosure))
---
[Top](../index.html)

#### G1MarkSweep クラス (G1MarkSweep, 及びその補助クラス(G1PrepareCompactClosure, FindFirstRegionClosure, G1AdjustPointersClosure, G1SpaceCompactClosure))

これらは, G1CollectedHeap の MarkSweep 処理(Major GC 処理)で使用される補助クラス (See: [here](no2935ATn.html) and [here](no28916_jv.html) for details).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.hpp))
    // G1MarkSweep takes care of global mark-compact garbage collection for a
    // G1CollectedHeap using a four-phase pointer forwarding algorithm.  All
    // generations are assumed to support marking; those that can also support
    // compaction.
    //
    // Class unloading will only occur when a full gc is invoked.
```


### クラス一覧(class list)

  * [G1MarkSweep](#noxNo2K0O_)
  * [G1PrepareCompactClosure](#no2ztauFHv)
  * [FindFirstRegionClosure](#noyPXsF1k1)
  * [G1AdjustPointersClosure](#noMRya546U)
  * [G1SpaceCompactClosure](#nor3ZGfljJ)


---
## <a name="noxNo2K0O_" id="noxNo2K0O_">G1MarkSweep</a>

### 概要(Summary)
G1CollectedHeap 用の MarkSweep クラス
(つまり, Mark Sweep Compact 処理で使用される補助関数や Closure クラス等を納めた名前空間(AllStatic クラス). 
 ただし, 正確に言うとこのクラスは MarkSweep クラスのサブクラスではない (See: MarkSweep)).

シングルスレッドでの Mark-Sweep-Compact 処理を実装している (See: [here](no2935ATn.html) and [here](no28916_jv.html) for details).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.hpp))
    class G1MarkSweep : AllStatic {
```

### 使われ方(Usage)
Major GC 処理はこのクラスの G1MarkSweep::invoke_at_safepoint() メソッドが呼び出されることで実行される (See: [here](no2935ATn.html) for details).

### 内部構造(Internal structure)
内部には, Mark-Sweep-Compact 処理のための以下のようなメソッドが定義されている.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.hpp))
      // Mark live objects
      static void mark_sweep_phase1(bool& marked_for_deopt,
                                    bool clear_all_softrefs);
      // Calculate new addresses
      static void mark_sweep_phase2();
      // Update pointers
      static void mark_sweep_phase3();
      // Move objects to new positions
      static void mark_sweep_phase4();
    
      static void allocate_stacks();
```




### 詳細(Details)
See: [here](../doxygen/classG1MarkSweep.html) for details

---
## <a name="no2ztauFHv" id="no2ztauFHv">G1PrepareCompactClosure</a>

### 概要(Summary)
G1MarkSweep クラス内で使用される補助クラス(StackObjクラス).

各 live object に対して, コンパクション後の新しいアドレスを (forwarding pointer として) 埋め込む.

(また, HumongousRegionSet オブジェクトを内部に保持しており, 
埋め込み処理中に生じた変更を本体である MasterHumongousRegionSet に反映させる機能も備えている)


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.cpp))
    class G1PrepareCompactClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1MarkSweep::mark_sweep_phase2() 内で(のみ)使用されている (See: [here](no2935ATn.html) for details).




### 詳細(Details)
See: [here](../doxygen/classG1PrepareCompactClosure.html) for details

---
## <a name="noyPXsF1k1" id="noyPXsF1k1">FindFirstRegionClosure</a>

### 概要(Summary)
G1MarkSweep クラス内で使用される補助クラス(StackObjクラス).

一番最初の HeapRegion を取得するための Closure クラス.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.cpp))
    // Finds the first HeapRegion.
    class FindFirstRegionClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1MarkSweep::mark_sweep_phase2() 内で(のみ)使用されている (See: [here](no2935ATn.html) for details).




### 詳細(Details)
See: [here](../doxygen/classFindFirstRegionClosure.html) for details

---
## <a name="noMRya546U" id="noMRya546U">G1AdjustPointersClosure</a>

### 概要(Summary)
G1MarkSweep クラス内で使用される補助クラス(StackObjクラス).

各 live object 内のポインタを新しいアドレスに修正する.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.cpp))
    class G1AdjustPointersClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1MarkSweep::mark_sweep_phase3() 内で(のみ)使用されている (See: [here](no2935ATn.html) for details).




### 詳細(Details)
See: [here](../doxygen/classG1AdjustPointersClosure.html) for details

---
## <a name="nor3ZGfljJ" id="nor3ZGfljJ">G1SpaceCompactClosure</a>

### 概要(Summary)
G1MarkSweep クラス内で使用される補助クラス(StackObjクラス).

各 live object を新しいアドレスに移動させる.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.cpp))
    class G1SpaceCompactClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1MarkSweep::mark_sweep_phase4() 内で(のみ)使用されている (See: [here](no2935ATn.html) for details).




### 詳細(Details)
See: [here](../doxygen/classG1SpaceCompactClosure.html) for details

---
