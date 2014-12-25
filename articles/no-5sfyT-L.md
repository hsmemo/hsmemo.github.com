---
layout: default
title: SpaceDecorator 及び SpaceMangler クラス関連のクラス (SpaceDecorator, SpaceMangler, GenSpaceMangler, MutableSpaceMangler)
---
[Top](../index.html)

#### SpaceDecorator 及び SpaceMangler クラス関連のクラス (SpaceDecorator, SpaceMangler, GenSpaceMangler, MutableSpaceMangler)

これらは, デバッグ用(開発時用)のクラス.

### 概要(Summary)
これらのクラスは, Space クラスや MutableSpace クラス内の未使用領域に対して, 明示的におかしな値で書き込む(mangle)処理を行う
(これにより, 未使用領域の誤使用に関するバグを検出しやすくする).

mangle 処理は以下のタイミングで行われる.

  * 初期化時 (mangle_region() で全領域を壊す)
  * 領域が伸びた(grow した)際 (mangle_region() で伸びた分を壊す)
  * GC が起こった際 (新たに未使用になった領域を壊す)
  * その他特殊な用途で Space が使用された後 (未使用になった分を壊す)
    (例: mark-sweep-compact 内で一時的な作業領域として To space が使われた後, 等)

SpaceMangler クラスが mangle 処理用のメソッドを提供する.
(内部では対象の領域の現在の使用量(top 位置)を記憶している模様)
(また, きちんと mangle されているかどうかのチェックを行うメソッドも提供している. 
DEBUG_MANGLING が #define されていないと何もしないが...).

また, SpaceMangler クラスにはサブクラスとして GenSpaceMangler と MutableSpaceMangler がある. 
それぞれ GenCollectedHeap と ParallelScavengeHeap 用
(これらは ContiguousSpace と MutableSpace の違いを吸収するために作られたサブクラス. 中身的にはどちらもほぼ同じ)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceDecorator.hpp))
    // Functionality for use with class Space and class MutableSpace.
    //   The approach taken with the mangling is to mangle all
    // the space initially and then to mangle areas that have
    // been allocated since the last collection.  Mangling is
    // done in the context of a generation and in the context
    // of a space.
    //   The space in a generation is mangled when it is first
    // initialized and when the generation grows.  The spaces
    // are not necessarily up-to-date when this mangling occurs
    // and the method mangle_region() is used.
    //   After allocations have been done in a space, the space generally
    // need to be remangled.  Remangling is only done on the
    // recently allocated regions in the space.  Typically, that is
    // the region between the new top and the top just before a
    // garbage collection.
    //   An exception to the usual mangling in a space is done when the
    // space is used for an extraordinary purpose.  Specifically, when
    // to-space is used as scratch space for a mark-sweep-compact
    // collection.
    //   Spaces are mangled after a collection.  If the generation
    // grows after a collection, the added space is mangled as part of
    // the growth of the generation.  No additional mangling is needed when the
    // spaces are resized after an expansion.
    //   The class SpaceMangler keeps a pointer to the top of the allocated
    // area and provides the methods for doing the piece meal mangling.
    // Methods for doing sparces and full checking of the mangling are
    // included.  The full checking is done if DEBUG_MANGLING is defined.
    //   GenSpaceMangler is used with the GenCollectedHeap collectors and
    // MutableSpaceMangler is used with the ParallelScavengeHeap collectors.
    // These subclasses abstract the differences in the types of spaces used
    // by each heap.
```

なおデバッグ用の機能なので, ほとんどのメソッドは #ifndef PRODUCT 時にしか定義されない.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceDecorator.cpp))
    #ifndef PRODUCT
```



### クラス一覧(class list)

  * [SpaceDecorator](#noDNNciQOH)
  * [SpaceMangler](#noO2_Q1ZRv)
  * [GenSpaceMangler](#no7wko3MuR)
  * [MutableSpaceMangler](#no32Lhk-0m)


---
## <a name="noDNNciQOH" id="noDNNciQOH">SpaceDecorator</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

メモリ空間の Mangle 処理を行うかどうかを示す定数値を納めた名前空間(AllStatic クラス)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceDecorator.hpp))
    class SpaceDecorator: public AllStatic {
```

### 内部構造(Internal structure)
以下の4つの定数値が定義されているのみ.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceDecorator.hpp))
      static const bool Clear               = true;
      static const bool DontClear           = false;
      static const bool Mangle              = true;
      static const bool DontMangle          = false;
```




### 詳細(Details)
See: [here](../doxygen/classSpaceDecorator.html) for details

---
## <a name="noO2_Q1ZRv" id="noO2_Q1ZRv">SpaceMangler</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

メモリ領域に対する mangle 処理を行うクラス (の基底クラス).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceDecorator.hpp))
    class SpaceMangler: public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classSpaceMangler.html) for details

---
## <a name="no7wko3MuR" id="no7wko3MuR">GenSpaceMangler</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

SpaceMangler クラスの具象サブクラスの1つ.
このクラスは GenCollectedHeap 用.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceDecorator.hpp))
    // For use with GenCollectedHeap's
    class GenSpaceMangler: public SpaceMangler {
```

### 内部構造(Internal structure)
親クラスの SpaceMangler と違うのは以下の３メソッドのみ.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceDecorator.hpp))
      ContiguousSpace* sp() { return _sp; }
    
      HeapWord* top() const { return _sp->top(); }
      HeapWord* end() const { return _sp->end(); }
```




### 詳細(Details)
See: [here](../doxygen/classGenSpaceMangler.html) for details

---
## <a name="no32Lhk-0m" id="no32Lhk-0m">MutableSpaceMangler</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

SpaceMangler クラスの具象サブクラスの1つ.
このクラスは ParallelScavengeHeap 用.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceDecorator.hpp))
    // For use with ParallelScavengeHeap's.
    class MutableSpaceMangler: public SpaceMangler {
```

### 内部構造(Internal structure)
親クラスの SpaceMangler と違うのは以下の３メソッドのみ.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceDecorator.hpp))
      MutableSpace* sp() { return _sp; }
    
      HeapWord* top() const { return _sp->top(); }
      HeapWord* end() const { return _sp->end(); }
```




### 詳細(Details)
See: [here](../doxygen/classMutableSpaceMangler.html) for details

---
