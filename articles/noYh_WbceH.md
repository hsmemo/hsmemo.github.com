---
layout: default
title: Generation クラス関連のクラス (Generation, CardGeneration, OneContigSpaceCardGeneration, 及びそれらの補助クラス(GenerationIsInReservedClosure, GenerationIsInClosure, GenerationBlockStartClosure, GenerationBlockSizeClosure, GenerationBlockIsObjClosure, GenerationOopIterateClosure, GenerationObjIterateClosure, GenerationSafeObjIterateClosure, AdjustPointersClosure))
---
[Top](../index.html)

#### Generation クラス関連のクラス (Generation, CardGeneration, OneContigSpaceCardGeneration, 及びそれらの補助クラス(GenerationIsInReservedClosure, GenerationIsInClosure, GenerationBlockStartClosure, GenerationBlockSizeClosure, GenerationBlockIsObjClosure, GenerationOopIterateClosure, GenerationObjIterateClosure, GenerationSafeObjIterateClosure, AdjustPointersClosure))

これらは, 世代別 GC における各「世代領域 (Generation)」を管理するためのクラス.


```
    ((cite: hotspot/src/share/vm/memory/generation.hpp))
    // A Generation models a heap area for similarly-aged objects.
    // It will contain one ore more spaces holding the actual objects.
```

世代領域を管理するクラスは使用する GC アルゴリズムによって異なるが,
これらのクラスは GC アルゴリズムが ParallelScavengeHeap 以外の場合に使用される
(より正確には, GC アルゴリズムが Serial, ParNew, Serial Old, CMS の場合の New/Old/Perm 領域,
及び GC アルゴリズムが G1GC の場合の Perm 領域が関連する).

なお, これらのクラスは以下のような継承関係を持つ.


```
    ((cite: hotspot/src/share/vm/memory/generation.hpp))
    // The Generation class hierarchy:
    //
    // Generation                      - abstract base class
    // - DefNewGeneration              - allocation area (copy collected)
    //   - ParNewGeneration            - a DefNewGeneration that is collected by
    //                                   several threads
    // - CardGeneration                 - abstract class adding offset array behavior
    //   - OneContigSpaceCardGeneration - abstract class holding a single
    //                                    contiguous space with card marking
    //     - TenuredGeneration         - tenured (old object) space (markSweepCompact)
    //     - CompactingPermGenGen      - reflective object area (klasses, methods, symbols, ...)
    //   - ConcurrentMarkSweepGeneration - Mostly Concurrent Mark Sweep Generation
    //                                       (Detlefs-Printezis refinement of
    //                                       Boehm-Demers-Schenker)
```

### 備考(Notes)
以下のクラスの実装については最適化の余地があるとのこと.

* GenerationBlockStartClosure
* GenerationBlockSizeClosure
* GenerationBlockIsObjClosure
* GenerationOopIterateClosure
* GenerationObjIterateClosure
* GenerationSafeObjIterateClosure
* AdjustPointersClosure


```
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    // Some of these are mediocre general implementations.  Should be
    // overridden to get better performance.
```



### クラス一覧(class list)

  * [Generation](#nosV-_xlIC)
  * [CardGeneration](#noszTT_bKq)
  * [OneContigSpaceCardGeneration](#no5aLC-Y8E)
  * [GenerationIsInReservedClosure](#notTuRFBRZ)
  * [GenerationIsInClosure](#nocn2hPV31)
  * [GenerationBlockStartClosure](#noasNJpInu)
  * [GenerationBlockSizeClosure](#nohgyjGlOP)
  * [GenerationBlockIsObjClosure](#noHKpwXXcy)
  * [GenerationOopIterateClosure](#no5gUXMxOE)
  * [GenerationObjIterateClosure](#no6U6MeS1k)
  * [GenerationSafeObjIterateClosure](#noTcXZQALZ)
  * [AdjustPointersClosure](#novADSnWx2)


---
## <a name="nosV-_xlIC" id="nosV-_xlIC">Generation</a>

### 概要(Summary)
世代別 GC における各「世代領域 (Generation)」を管理するためのクラス (の基底クラス)

世代領域を管理するクラスは使用する GC アルゴリズムによって異なるが,
このクラスは GC アルゴリズムが ParallelScavengeHeap 以外の場合に使用される
(より正確には, GC アルゴリズムが Serial, ParNew, Serial Old, CMS の場合の New/Old/Perm 領域,
及び GC アルゴリズムが G1GC の場合の Perm 領域が関連する).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/memory/generation.hpp))
    class Generation: public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classGeneration.html) for details

---
## <a name="noszTT_bKq" id="noszTT_bKq">CardGeneration</a>

### 概要(Summary)
内部的に card table を使用する Generational クラス (の基底クラス).

card table を使用しているため効率的な write barrier 処理が行える.

(なお, block_start() メソッドを実装するために BlockOffsetArray も使用している)

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/memory/generation.hpp))
    // Class CardGeneration is a generation that is covered by a card table,
    // and uses a card-size block-offset array to implement block_start.
    ...
    
    class CardGeneration: public Generation {
```




### 詳細(Details)
See: [here](../doxygen/classCardGeneration.html) for details

---
## <a name="no5aLC-Y8E" id="no5aLC-Y8E">OneContigSpaceCardGeneration</a>

### 概要(Summary)
以下の条件を満たす CardGenerational クラス (の基底クラス).

* 管理するメモリ領域は, メモリ空間上の1つの連続領域.
* メモリ領域内には, (世代別 GC 用語で言うところの) old なオブジェクトのみを格納する .

Garbage Collection 処理は mark-sweep-compact 方式で行う.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/memory/generation.hpp))
    // OneContigSpaceCardGeneration models a heap of old objects contained in a single
    // contiguous space.
    //
    // Garbage collection is performed using mark-compact.
    
    class OneContigSpaceCardGeneration: public CardGeneration {
```




### 詳細(Details)
See: [here](../doxygen/classOneContigSpaceCardGeneration.html) for details

---
## <a name="notTuRFBRZ" id="notTuRFBRZ">GenerationIsInReservedClosure</a>

### 概要(Summary)
Generation クラス内で使用される補助クラス (Closure クラス).

指定された Space オブジェクトが「コンストラクタ引数で渡されたアドレスを含むかどうか」を調べる.


```
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    class GenerationIsInReservedClosure : public SpaceClosure {
```

### 使われ方(Usage)
Generation::space_containing() 内で(のみ)使用されている

(なおこの関数は, 指定の Generation オブジェクト内の全ての Space オブジェクトに対して
Space::is_in_reserved() を呼び出し, 処理対象のアドレスを含む Space があるかどうかを調べるためのもの).

### 内部構造(Internal structure)
指定された Space オブジェクトに対して Space::is_in_reserved() を呼び出すだけ.




### 詳細(Details)
See: [here](../doxygen/classGenerationIsInReservedClosure.html) for details

---
## <a name="nocn2hPV31" id="nocn2hPV31">GenerationIsInClosure</a>

### 概要(Summary)
Generation クラス内で使用される補助クラス (Closure クラス).

指定された Space オブジェクトが「コンストラクタ引数で渡されたアドレスを含むかどうか」を調べる.


```
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    class GenerationIsInClosure : public SpaceClosure {
```

### 使われ方(Usage)
Generation::is_in() 内で(のみ)使用されている

(なおこの関数は, 指定の Generation オブジェクト内の全ての Space オブジェクトに対して
Space::is_in() を呼び出し, 処理対象のアドレスを含む Space があるかどうかを調べるためのもの).

### 内部構造(Internal structure)
指定された Space オブジェクトに対して Space::is_in() を呼び出すだけ.




### 詳細(Details)
See: [here](../doxygen/classGenerationIsInClosure.html) for details

---
## <a name="noasNJpInu" id="noasNJpInu">GenerationBlockStartClosure</a>

### 概要(Summary)
Generation クラス内で使用される補助クラス (Closure クラス).

指定された Space オブジェクトが「コンストラクタ引数で渡されたアドレスを含むかどうか」を調べる
(かつ, もし含む場合はそのアドレスに対応する block_start アドレスを記録する).


```
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    class GenerationBlockStartClosure : public SpaceClosure {
```

### 使われ方(Usage)
Generation::block_start() 内で(のみ)使用されている

(なおこの関数は, 指定の Generation オブジェクト内の全ての Space オブジェクトに対して,
処理対象のアドレスを含む Space があるかどうかを調べ,
もしあれば対応する block_start アドレスを取得するためのもの).

### 内部構造(Internal structure)
指定された Space オブジェクトに対して Space::is_in_reserved() を呼び出す.
true が返された場合は, さらに Space::block_start() も呼び出してその結果を記録する.




### 詳細(Details)
See: [here](../doxygen/classGenerationBlockStartClosure.html) for details

---
## <a name="nohgyjGlOP" id="nohgyjGlOP">GenerationBlockSizeClosure</a>

### 概要(Summary)
Generation クラス内で使用される補助クラス (Closure クラス).

指定された Space オブジェクトが「コンストラクタ引数で渡されたアドレスを含むかどうか」を調べる
(かつ, もし含む場合はそのアドレスに対応する block_size を記録する).


```
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    class GenerationBlockSizeClosure : public SpaceClosure {
```

### 使われ方(Usage)
Generation::block_size() 内で(のみ)使用されている

(なおこの関数は, 指定の Generation オブジェクト内の全ての Space オブジェクトに対して,
処理対象のアドレスを含む Space があるかどうかを調べ,
もしあれば対応する block_size の大きさを取得するためのもの).

### 内部構造(Internal structure)
指定された Space オブジェクトに対して Space::is_in_reserved() を呼び出す.
true が返された場合は, さらに Space::block_size() も呼び出してその結果を記録する.




### 詳細(Details)
See: [here](../doxygen/classGenerationBlockSizeClosure.html) for details

---
## <a name="noHKpwXXcy" id="noHKpwXXcy">GenerationBlockIsObjClosure</a>

### 概要(Summary)
Generation クラス内で使用される補助クラス (Closure クラス).

指定された Space オブジェクトが「コンストラクタ引数で渡されたアドレスを含むかどうか」を調べる
(かつ, もし含む場合はそのアドレス位置にオブジェクトがあるかどうかを記録する).


```
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    class GenerationBlockIsObjClosure : public SpaceClosure {
```

### 使われ方(Usage)
Generation::block_is_obj() 内で(のみ)使用されている

(なおこの関数は, 指定の Generation オブジェクト内の全ての Space オブジェクトに対して,
処理対象のアドレスを含む Space があるかどうかを調べ,
さらにもしあった場合には, そのアドレス位置にオブジェクトがあるかどうかを調べるためのもの).

### 内部構造(Internal structure)
指定された Space オブジェクトに対して Space::is_in_reserved() を呼び出す.
true が返された場合は, さらに Space::block_is_obj() も呼び出してその結果を記録する.




### 詳細(Details)
See: [here](../doxygen/classGenerationBlockIsObjClosure.html) for details

---
## <a name="no5gUXMxOE" id="no5gUXMxOE">GenerationOopIterateClosure</a>

### 概要(Summary)
Generation クラス内で使用される補助クラス (Closure クラス).

他の OopClosure と組み合わせて使用される Closure クラス.
指定された Space オブジェクトの Space::oop_iterate() メソッドを呼び出し, 
コンストラクタ引数で渡された OopClosure を実行する.


```
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    class GenerationOopIterateClosure : public SpaceClosure {
```

### 使われ方(Usage)
Generation::oop_iterate(OopClosure* cl), および
Generation::oop_iterate(MemRegion mr, OopClosure* cl) の中で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classGenerationOopIterateClosure.html) for details

---
## <a name="no6U6MeS1k" id="no6U6MeS1k">GenerationObjIterateClosure</a>

### 概要(Summary)
Generation クラス内で使用される補助クラス (Closure クラス).

他の ObjectClosure と組み合わせて使用される Closure クラス.
指定された Space オブジェクトの Space::object_iterate() メソッドを呼び出し, 
コンストラクタ引数で渡された ObjectClosure を実行する.


```
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    class GenerationObjIterateClosure : public SpaceClosure {
```

### 使われ方(Usage)
Generation::object_iterate() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classGenerationObjIterateClosure.html) for details

---
## <a name="noTcXZQALZ" id="noTcXZQALZ">GenerationSafeObjIterateClosure</a>

### 概要(Summary)
Generation クラス内で使用される補助クラス (Closure クラス).

他の ObjectClosure と組み合わせて使用される Closure クラス.
指定された Space オブジェクトの Space::safe_object_iterate() メソッドを呼び出し, 
コンストラクタ引数で渡された ObjectClosure を実行する.


```
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    class GenerationSafeObjIterateClosure : public SpaceClosure {
```

### 使われ方(Usage)
Generation::safe_object_iterate() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classGenerationSafeObjIterateClosure.html) for details

---
## <a name="novADSnWx2" id="novADSnWx2">AdjustPointersClosure</a>

### 概要(Summary)
Generation クラス内で使用される補助クラス (Closure クラス).

Garbage Collection 処理で使用される Closure クラス.
指定された Space オブジェクトに対して Space::adjust_pointers() を呼び出し, 
その Space 内にある live object 中のポインタを新しいアドレスに修正する.


```
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    class AdjustPointersClosure: public SpaceClosure {
```

### 使われ方(Usage)
Generation::adjust_pointers() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classAdjustPointersClosure.html) for details

---
