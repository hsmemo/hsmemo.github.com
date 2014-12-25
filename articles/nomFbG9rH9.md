---
layout: default
title: DefNewGeneration クラス (DefNewGeneration, 及びその補助クラス(DefNewGeneration::IsAliveClosure, DefNewGeneration::KeepAliveClosure, DefNewGeneration::FastKeepAliveClosure, DefNewGeneration::EvacuateFollowersClosure, DefNewGeneration::FastEvacuateFollowersClosure,  RemoveForwardPointerClosure))
---
[Top](../index.html)

#### DefNewGeneration クラス (DefNewGeneration, 及びその補助クラス(DefNewGeneration::IsAliveClosure, DefNewGeneration::KeepAliveClosure, DefNewGeneration::FastKeepAliveClosure, DefNewGeneration::EvacuateFollowersClosure, DefNewGeneration::FastEvacuateFollowersClosure,  RemoveForwardPointerClosure))

これらは, Java ヒープ中の 「New 世代領域 (New Generation)」 を管理するためのクラス (See: [here](no3718kvd.html) for details).

New Generation を管理するクラスは使用する GC アルゴリズムによって異なるが,
これらのクラスは GC アルゴリズムが Serial GC の場合に使用される
(つまり, ParallelScavenge でも G1GC でも ParNew でもない場合に使用される
 (See: PSYoungGen, YoungList, ParNewGeneration)).


### クラス一覧(class list)

  * [DefNewGeneration](#no4UIXde-i)
  * [DefNewGeneration::IsAliveClosure](#noI-iKXWa1)
  * [DefNewGeneration::KeepAliveClosure](#noV-mcHhOi)
  * [DefNewGeneration::FastKeepAliveClosure](#noAI4MDxmS)
  * [DefNewGeneration::EvacuateFollowersClosure](#noGiCjrfXU)
  * [DefNewGeneration::FastEvacuateFollowersClosure](#noneWZlEbL)
  * [RemoveForwardPointerClosure](#noZsLb3FQp)


---
## <a name="no4UIXde-i" id="no4UIXde-i">DefNewGeneration</a>

### 概要(Summary)
GenCollectedHeap 使用時において, New Generation の管理を担当するクラスの 1つ (See: [here](no3718kvd.html) for details).

このクラスは, GC アルゴリズムが ParNewGC ではない場合用 (つまり Serial GC 用) (See: ParNewGeneration).


```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.hpp))
    // DefNewGeneration is a young generation containing eden, from- and
    // to-space.
    
    class DefNewGeneration: public Generation {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 GenCollectedHeap オブジェクトの _gens フィールドに(のみ)格納されている.

(正確には, このフィールドは Generation のポインタの配列を格納するフィールド.
この中に, その GenCollectedHeap 内で使用される全ての Generation オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
GenerationSpec::init() 内で(のみ)生成されている.

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
(略) (See: [here](no2114gVH.html) for details)
-> GenCollectedHeap::initialize()
   -> GenerationSpec::init()
```

#### 使用箇所(where its instances are used)
...

このクラスの DefNewGeneration::collect() メソッドが Serial GC 処理のエントリポイントになっている (See: [here](no2114uZg.html) for details).




### 詳細(Details)
See: [here](../doxygen/classDefNewGeneration.html) for details

---
## <a name="noI-iKXWa1" id="noI-iKXWa1">DefNewGeneration::IsAliveClosure</a>

### 概要(Summary)
DefNewGeneration の GC 処理 (= Serial GC 処理) で使用される Closure クラス (See: [here](no2114uZg.html) for details).

参照オブジェクト(java.lang.ref オブジェクト)の処理に用いられる Closure クラス.
DefNewGeneration::IsAliveClosure::do_object_b() メソッドが呼ばれると, 処理対象のオブジェクトが生きているかどうかを返す.


```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.hpp))
      class IsAliveClosure: public BoolObjectClosure {
```

### 使われ方(Usage)
DefNewGeneration::collect() 内で(のみ)使用されている (See: [here](no2114uZg.html) for details).




### 詳細(Details)
See: [here](../doxygen/classDefNewGeneration_1_1IsAliveClosure.html) for details

---
## <a name="noV-mcHhOi" id="noV-mcHhOi">DefNewGeneration::KeepAliveClosure</a>

### 概要(Summary)
DefNewGeneration の GC 処理 (= Serial GC 処理) で使用される Closure クラス (の基底クラス) (See: [here](no2114uZg.html) for details).

参照オブジェクト(java.lang.ref オブジェクト)の処理に用いられる Closure クラス.
まだコピーされていないオブジェクトに対して, コピー処理を行い, 
さらに元の場所にフォワーディングポインタを埋める処理を行う.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス (See: DefNewGeneration::FastKeepAliveClosure).


```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.hpp))
      class KeepAliveClosure: public OopClosure {
```

### 内部構造(Internal structure)
呼び出されると, 引数で与えられた ScanWeakRefClosure を各オブジェクトに対して適用する
(ついでに, 該当箇所の card の dirty 化処理も行っている) (See: ScanWeakRefClosure).

(なおこのクラスの場合, サブクラスである FastKeepAliveClosure とは異なり,
 処理対象となった全ての oop に対して
 CardTableRS::inline_write_ref_field_gc() を呼び出す.
 (See: FastKeepAliveClosure))




### 詳細(Details)
See: [here](../doxygen/classDefNewGeneration_1_1KeepAliveClosure.html) for details

---
## <a name="noAI4MDxmS" id="noAI4MDxmS">DefNewGeneration::FastKeepAliveClosure</a>

### 概要(Summary)
DefNewGeneration の GC 処理 (= Serial GC 処理) で使用される Closure クラス (See: [here](no2114uZg.html) for details).

DefNewGeneration::KeepAliveClosure クラスの具象サブクラス
(= GC 時に参照オブジェクト(java.lang.ref オブジェクト)の処理に用いられる Closure.
まだコピーされていないオブジェクトに対して, コピー処理を行い, 
さらに元の場所にフォワーディングポインタを埋める処理を行う).


```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.hpp))
      class FastKeepAliveClosure: public KeepAliveClosure {
```

### 使われ方(Usage)
DefNewGeneration::collect() 内で(のみ)使用されている (See: [here](no2114uZg.html) for details).

### 内部構造(Internal structure)
呼び出されると, 各オブジェクトに対して, 引数で与えられた ScanWeakRefClosure を適用する
(ついでに, 該当箇所の card の dirty 化処理も行っている). (See: ScanWeakRefClosure)

(KeepAliveClosure とほぼ同じだが,
 このクラスの場合は DefNewGeneration 内を指しているポインタでなければ
 CardTableRS::inline_write_ref_field_gc() を呼び出さない.
 このため, KeepAliveClosure よりも高速.)




### 詳細(Details)
See: [here](../doxygen/classDefNewGeneration_1_1FastKeepAliveClosure.html) for details

---
## <a name="noGiCjrfXU" id="noGiCjrfXU">DefNewGeneration::EvacuateFollowersClosure</a>

### 概要(Summary)
DefNewGeneration の GC 処理 (= Serial GC 処理) で使用される Closure クラス (See: [here](no2114uZg.html) for details).

処理したオブジェクトから辿れる範囲全てについて再帰的に処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.hpp))
      class EvacuateFollowersClosure: public VoidClosure {
```

### 使われ方(Usage)
このクラスのインスタンス自体は, 現状はどこからも使われていない模様
(?? DefNewGeneration::FastEvacuateFollowersClosure があるから要らない?? #TODO).




### 詳細(Details)
See: [here](../doxygen/classDefNewGeneration_1_1EvacuateFollowersClosure.html) for details

---
## <a name="noneWZlEbL" id="noneWZlEbL">DefNewGeneration::FastEvacuateFollowersClosure</a>

### 概要(Summary)
DefNewGeneration の GC 処理 (= Serial GC 処理) で使用される Closure クラス (See: [here](no2114uZg.html) for details).

処理したオブジェクトから辿れる範囲全てについて再帰的に処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.hpp))
      class FastEvacuateFollowersClosure: public VoidClosure {
```

### 使われ方(Usage)
DefNewGeneration::collect() 内で(のみ)使用されている (See: [here](no2114uZg.html) for details).

### 内部構造(Internal structure)
(EvacuateFollowersClosure と何が違う??
 処理は実質的には同じように見える... #TODO)




### 詳細(Details)
See: [here](../doxygen/classDefNewGeneration_1_1FastEvacuateFollowersClosure.html) for details

---
## <a name="noZsLb3FQp" id="noZsLb3FQp">RemoveForwardPointerClosure</a>

### 概要(Summary)
DefNewGeneration の GC 処理 (= Serial GC 処理) で使用される Closure クラス (See: [here](no2114uZg.html) for details).

GC 処理が失敗した際に, 各オブジェクトの mark フィールドを初期状態に戻すための Closure
(GC 処理後には mark フィールドには forwarding pointer が埋められているため, それをクリアする).


```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.cpp))
    class RemoveForwardPointerClosure: public ObjectClosure {
```

### 使われ方(Usage)
DefNewGeneration::remove_forwarding_pointers() 内で(のみ)使用されている (See: [here](no2114uZg.html) for details).




### 詳細(Details)
See: [here](../doxygen/classRemoveForwardPointerClosure.html) for details

---
