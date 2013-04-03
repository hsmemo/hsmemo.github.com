---
layout: default
title: GenerationSpec クラス関連のクラス (GenerationSpec, PermanentGenerationSpec)
---
[Top](../index.html)

#### GenerationSpec クラス関連のクラス (GenerationSpec, PermanentGenerationSpec)

これらは, SharedHeap 用 (及び GenCollectedHeap 用) の補助クラス (See: [here](no3718kvd.html), [here](no2114tfN.html) and [here](no2114gVH.html) for details).

より具体的に言うと, SharedHeap 内では GC アルゴリズムに応じて様々な Generation クラスが使用されるが,
この GenerationSpec クラスが具体的にどの Generation を使うかを管理している.


### クラス一覧(class list)

  * [GenerationSpec](#no9fxR31pW)
  * [PermanentGenerationSpec](#nokV8u043J)


---
## <a name="no9fxR31pW" id="no9fxR31pW">GenerationSpec</a>

### 概要(Summary)
GenCollectedHeap クラス内で使用される補助クラス.

GenCollectedHeap 内の New 領域および Old 領域について, それらを管理する Generation クラスを決定する.

また, Generation 毎に異なるいくつかの挙動をカプセル化する役割も果たしている
(コメントによると, Generation オブジェクトの生成前に使うので
Generation クラスの virtual method では実現できない, とのこと).


```
    ((cite: hotspot/src/share/vm/memory/generationSpec.hpp))
    // The specification of a generation.  This class also encapsulates
    // some generation-specific behavior.  This is done here rather than as a
    // virtual function of Generation because these methods are needed in
    // initialization of the Generations.
    class GenerationSpec : public CHeapObj {
```



### 詳細(Details)
See: [here](../doxygen/classGenerationSpec.html) for details

---
## <a name="nokV8u043J" id="nokV8u043J">PermanentGenerationSpec</a>

### 概要(Summary)
SharedHeap クラス内で使用される補助クラス.

SharedHeap 内の Perm 領域を管理する Generation クラスを決定する.

なお, 役割としては GenerationSpec によく似ているが
PermGen が Generation クラスになっていないので GenerationSpec とは別のクラスにしている, とのこと.


```
    ((cite: hotspot/src/share/vm/memory/generationSpec.hpp))
    // The specification of a permanent generation. This class is very
    // similar to GenerationSpec in use. Due to PermGen's not being a
    // true Generation, we cannot combine the spec classes either.
    class PermanentGenerationSpec : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classPermanentGenerationSpec.html) for details

---
