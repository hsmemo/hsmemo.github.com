---
layout: default
title: VectorSet クラス関連のクラス (VectorSet, VectorSetI)
---
[Top](../index.html)

#### VectorSet クラス関連のクラス (VectorSet, VectorSetI)

これらは, ADLC 内で使用される集合クラス (See: [here](nop0Yyr-jc.html) for details).


### クラス一覧(class list)

  * [VectorSet](#nok5bOtxvb)
  * [VectorSetI](#noZYv9BauV)


---
## <a name="nok5bOtxvb" id="nok5bOtxvb">VectorSet</a>

### 概要(Summary)
Set クラスのサブクラス. 配列で実装されている.

最初に max_element 分の領域を確保してしまう実装になっているため, 
Insert, Delete, Member, Sort といった処理が O(1) でできる.

メモリ量としては  (max_element)/8 byte  を消費する.


```
    ((cite: hotspot/src/share/vm/libadt/vectset.hpp))
    // Vector Sets - An Abstract Data Type
    //INTERFACE
    
    // These sets can grow or shrink, based on the initial size and the largest
    // element currently in them.  Slow and bulky for sparse sets, these sets
    // are super for dense sets.  They are fast and compact when dense.
    
    // TIME:
    // O(1) - Insert, Delete, Member, Sort
    // O(max_element) - Create, Clear, Size, Copy, Union, Intersect, Difference,
    //                  Equal, ChooseMember, Forall
    
    // SPACE: (max_element)/(8*sizeof(int))
    
    
    //------------------------------VectorSet--------------------------------------
    class VectorSet : public Set {
```



### 詳細(Details)
See: [here](../doxygen/classVectorSet.html) for details

---
## <a name="noZYv9BauV" id="noZYv9BauV">VectorSetI</a>

### 概要(Summary)
VectorSet 内の要素を処理するためのイテレータクラス.

以下のように使用する.

  `for( VectorSetI i(s); i.test(); i++ ) { body = i.elem; }`


```
    ((cite: hotspot/src/share/vm/libadt/vectset.hpp))
    //------------------------------Iteration--------------------------------------
    // Loop thru all elements of the set, setting "elem" to the element numbers
    // in random order.  Inserted or deleted elements during this operation may
    // or may not be iterated over; untouched elements will be affected once.
    // Usage:  for( VectorSetI i(s); i.test(); i++ ) { body = i.elem; }
    
    class VectorSetI : public StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classVectorSetI.html) for details

---
