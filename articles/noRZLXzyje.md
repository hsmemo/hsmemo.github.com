---
layout: default
title: Stack クラス関連のクラス (StackBase, Stack, ResourceStack, StackIterator)
---
[Top](../index.html)

#### Stack クラス関連のクラス (StackBase, Stack, ResourceStack, StackIterator)

これらは, 「スタック」として働くユーティリティ・クラス.

(<= ちなみに, 現状では GC 関連の処理からしか使われていないように見えるが... #TODO)

内部的には, 必要に応じてメモリ領域を追加し, リスト状につないで管理している.

以下は利用にあたっての注意点等:

  * スタックに入れる要素の大きさは, 「ポインタサイズの整数分の1」か「ポインタサイズの倍数」でないといけない.
  * スタックから出すときにデストラクタは呼ばれないので, デストラクタに頼ったオブジェクトは使用できない
  * Stack クラスでは C Heap からメモリを確保するが, この挙動は alloc()/free() を上書きすれば変えられる.


```cpp
    ((cite: hotspot/src/share/vm/utilities/stack.hpp))
    // Class Stack (below) grows and shrinks by linking together "segments" which
    // are allocated on demand.  Segments are arrays of the element type (E) plus an
    // extra pointer-sized field to store the segment link.  Recently emptied
    // segments are kept in a cache and reused.
    //
    // Notes/caveats:
    //
    // The size of an element must either evenly divide the size of a pointer or be
    // a multiple of the size of a pointer.
    //
    // Destructors are not called for elements popped off the stack, so element
    // types which rely on destructors for things like reference counting will not
    // work properly.
    //
    // Class Stack allocates segments from the C heap.  However, two protected
    // virtual methods are used to alloc/free memory which subclasses can override:
    //
    //      virtual void* alloc(size_t bytes);
    //      virtual void  free(void* addr, size_t bytes);
    //
    // The alloc() method must return storage aligned for any use.  The
    // implementation in class Stack assumes that alloc() will terminate the process
    // if the allocation fails.
```



### クラス一覧(class list)

  * [StackBase](#novj_X3Yn2)
  * [Stack](#no8gMRWDDB)
  * [ResourceStack](#nodBj6Qnlv)
  * [StackIterator](#nojJZj5-nm)


---
## <a name="novj_X3Yn2" id="novj_X3Yn2">StackBase</a>

### 概要(Summary)
スタックとして働くクラスの基底クラス.


```cpp
    ((cite: hotspot/src/share/vm/utilities/stack.hpp))
    // StackBase holds common data/methods that don't depend on the element type,
    // factored out to reduce template code duplication.
    class StackBase
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classStackBase.html) for details

---
## <a name="no8gMRWDDB" id="no8gMRWDDB">Stack</a>

### 概要(Summary)
StackBase クラスの具象サブクラス.

いわゆる「スタック」を表すクラス.
なお要素の型は template でパラメタライズされている.


```cpp
    ((cite: hotspot/src/share/vm/utilities/stack.hpp))
    template <class E>
    class Stack:  public StackBase
```

### 使われ方(Usage)
各種 GC 処理中で(のみ)使用されている. (#TODO)




### 詳細(Details)
See: [here](../doxygen/classStack.html) for details

---
## <a name="nodBj6Qnlv" id="nodBj6Qnlv">ResourceStack</a>

### 概要(Summary)
特殊な Stack クラス.

ResourceObj クラスとの多重継承になっている.

(ResourceObj なので一時的に使いたい場合用??)


```cpp
    ((cite: hotspot/src/share/vm/utilities/stack.hpp))
    template <class E> class ResourceStack:  public Stack<E>, public ResourceObj
```

### 使われ方(Usage)
?? (このクラスは使用箇所が見当たらない...)




### 詳細(Details)
See: [here](../doxygen/classResourceStack.html) for details

---
## <a name="nojJZj5-nm" id="nojJZj5-nm">StackIterator</a>

### 概要(Summary)
Stack オブジェクト内の要素をたどるためのイテレータクラス(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/utilities/stack.hpp))
    template <class E>
    class StackIterator: public StackObj
```

### 使われ方(Usage)
MarkSweep::adjust_marks() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classStackIterator.html) for details

---
