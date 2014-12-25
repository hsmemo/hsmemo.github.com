---
layout: default
title: GenMarkSweep クラス (GenMarkSweep, 及びその補助クラス(GenAdjustPointersClosure, GenCompactClosure))
---
[Top](../index.html)

#### GenMarkSweep クラス (GenMarkSweep, 及びその補助クラス(GenAdjustPointersClosure, GenCompactClosure))

これらは, Java ヒープに対する Major GC 処理を実装したクラス (See: [here](no2114hPa.html) for details).

Major GC 処理を担当するクラスは使用する GC アルゴリズムによって異なるが,
これらのクラスは GC アルゴリズムが Serial Old の場合に使用される.



### クラス一覧(class list)

  * [GenMarkSweep](#no5M8M7nr2)
  * [GenAdjustPointersClosure](#noe0nlkyyC)
  * [GenCompactClosure](#noyqbBoeIY)


---
## <a name="no5M8M7nr2" id="no5M8M7nr2">GenMarkSweep</a>

### 概要(Summary)
GenCollectedHeap ヒープに対する Major GC 処理を行うクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)) (See: [here](no2114hPa.html) for details).

このクラスは Serial Old 用 (「CMS ではない場合用」とも言う)


```cpp
    ((cite: hotspot/src/share/vm/memory/genMarkSweep.hpp))
    class GenMarkSweep : public MarkSweep {
```

なお現状では, このクラスは以下のクラスとセットで使用される.

* CollectorPolicy としては, MarkSweepPolicy クラス
* Generation としては, TenuredGeneration クラス




### 詳細(Details)
See: [here](../doxygen/classGenMarkSweep.html) for details

---
## <a name="noe0nlkyyC" id="noe0nlkyyC">GenAdjustPointersClosure</a>

### 概要(Summary)
GenMarkSweep クラス内で使用される補助クラス.

コンパクション処理時に, Java ヒープ内にある live object 内のポインタを新しいアドレスに修正するための Closure.


```cpp
    ((cite: hotspot/src/share/vm/memory/genMarkSweep.cpp))
    class GenAdjustPointersClosure: public GenCollectedHeap::GenClosure {
```

### 使われ方(Usage)
GenMarkSweep::mark_sweep_phase3() 内で(のみ)使用されている (See: [here](no2114hPa.html) for details).




### 詳細(Details)
See: [here](../doxygen/classGenAdjustPointersClosure.html) for details

---
## <a name="noyqbBoeIY" id="noyqbBoeIY">GenCompactClosure</a>

### 概要(Summary)
GenMarkSweep クラス内で使用される補助クラス.

コンパクション処理時に, Java ヒープ内の live object を新しいアドレスに移動させるための Closure.


```cpp
    ((cite: hotspot/src/share/vm/memory/genMarkSweep.cpp))
    class GenCompactClosure: public GenCollectedHeap::GenClosure {
```

### 使われ方(Usage)
GenMarkSweep::mark_sweep_phase4() 内で(のみ)使用されている (See: [here](no2114hPa.html) for details).




### 詳細(Details)
See: [here](../doxygen/classGenCompactClosure.html) for details

---
