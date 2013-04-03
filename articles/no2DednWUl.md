---
layout: default
title: Set クラス関連のクラス (Set, SetI_, SetI)
---
[Top](../index.html)

#### Set クラス関連のクラス (Set, SetI_, SetI)

これらは, ADLC 内で使用される集合クラスの基底クラス (See: [here](nop0Yyr-jc.html) for details).


### クラス一覧(class list)

  * [Set](#nomVTaDDht)
  * [SetI](#nokepdGWT8)
  * [SetI_](#nof50Jxzv1)


---
## <a name="nomVTaDDht" id="nomVTaDDht">Set</a>

### 概要(Summary)
ADLC 内で使用される集合クラスの基底クラス.


```
    ((cite: hotspot/src/share/vm/libadt/set.hpp))
    // Sets - An Abstract Data Type
```


```
    ((cite: hotspot/src/share/vm/libadt/set.hpp))
    //------------------------------Set--------------------------------------------
    class Set : public ResourceObj {
```

### 備考(Notes)
なお, 「Set クラス自体は abstract な基底クラスで, 
実際にはサブクラスである VectorSet, SparseSet, ListSet のどれかが使われる」と書かれているが, 
現状では実際に実装されているのは VectorSet だけのように見える(他のクラスは定義が見当たらない).


```
    ((cite: hotspot/src/share/vm/libadt/set.hpp))
    // These sets can grow or shrink, based on the initial size and the largest
    // element currently in them.  Basically, they allow a bunch of bits to be
    // grouped together, tested, set & cleared, intersected, etc.  The basic
    // Set class is an abstract class, and cannot be constructed.  Instead,
    // one of VectorSet, SparseSet, or ListSet is created.  Each variation has
    // different asymptotic running times for different operations, and different
    // constants of proportionality as well.
    // {n = number of elements, N = largest element}
    
    //              VectorSet       SparseSet       ListSet
    // Create       O(N)            O(1)            O(1)
    // Clear        O(N)            O(1)            O(1)
    // Insert       O(1)            O(1)            O(log n)
    // Delete       O(1)            O(1)            O(log n)
    // Member       O(1)            O(1)            O(log n)
    // Size         O(N)            O(1)            O(1)
    // Copy         O(N)            O(n)            O(n)
    // Union        O(N)            O(n)            O(n log n)
    // Intersect    O(N)            O(n)            O(n log n)
    // Difference   O(N)            O(n)            O(n log n)
    // Equal        O(N)            O(n)            O(n log n)
    // ChooseMember O(N)            O(1)            O(1)
    // Sort         O(1)            O(n log n)      O(1)
    // Forall       O(N)            O(n)            O(n)
    // Complement   O(1)            O(1)            O(1)
    
    // TIME:        N/32            n               8*n     Accesses
    // SPACE:       N/8             4*N+4*n         8*n     Bytes
    
    // Create:      Make an empty set
    // Clear:       Remove all the elements of a Set
    // Insert:      Insert an element into a Set; duplicates are ignored
    // Delete:      Removes an element from a Set
    // Member:      Tests for membership in a Set
    // Size:        Returns the number of members of a Set
    // Copy:        Copy or assign one Set to another
    // Union:       Union 2 sets together
    // Intersect:   Intersect 2 sets together
    // Difference:  Compute A & !B; remove from set A those elements in set B
    // Equal:       Test for equality between 2 sets
    // ChooseMember Pick a random member
    // Sort:        If no other operation changes the set membership, a following
    //              Forall will iterate the members in ascending order.
    // Forall:      Iterate over the elements of a Set.  Operations that modify
    //              the set membership during iteration work, but the iterator may
    //              skip any member or duplicate any member.
    // Complement:  Only supported in the Co-Set variations.  It adds a small
    //              constant-time test to every Set operation.
    //
    // PERFORMANCE ISSUES:
    // If you "cast away" the specific set variation you are using, and then do
    // operations on the basic "Set" object you will pay a virtual function call
    // to get back the specific set variation.  On the other hand, using the
    // generic Set means you can change underlying implementations by just
    // changing the initial declaration.  Examples:
    //      void foo(VectorSet vs1, VectorSet vs2) { vs1 |= vs2; }
    // "foo" must be called with a VectorSet.  The vector set union operation
    // is called directly.
    //      void foo(Set vs1, Set vs2) { vs1 |= vs2; }
    // "foo" may be called with *any* kind of sets; suppose it is called with
    // VectorSets.  Two virtual function calls are used to figure out the that vs1
    // and vs2 are VectorSets.  In addition, if vs2 is not a VectorSet then a
    // temporary VectorSet copy of vs2 will be made before the union proceeds.
    //
    // VectorSets have a small constant.  Time and space are proportional to the
    //   largest element.  Fine for dense sets and largest element < 10,000.
    // SparseSets have a medium constant.  Time is proportional to the number of
    //   elements, space is proportional to the largest element.
    //   Fine (but big) with the largest element < 100,000.
    // ListSets have a big constant.  Time *and space* are proportional to the
    //   number of elements.  They work well for a few elements of *any* size
    //   (i.e. sets of pointers)!
```



### 詳細(Details)
See: [here](../doxygen/classSet.html) for details

---
## <a name="nokepdGWT8" id="nokepdGWT8">SetI</a>

### 概要(Summary)
Set の要素を処理するためのイテレータクラス.

以下のように使用する.

  `for( SetI  i(s); i.test(); i++ ) { body = i.elem; }` または `for( i.reset(s); i.test(); i++ ) { body = i.elem; }`


```
    ((cite: hotspot/src/share/vm/libadt/set.hpp))
    //------------------------------Iteration--------------------------------------
    // Loop thru all elements of the set, setting "elem" to the element numbers
    // in random order.  Inserted or deleted elements during this operation may
    // or may not be iterated over; untouched elements will be affected once.
    
    // Usage:  for( SetI  i(s); i.test(); i++ ) { body = i.elem; }   ...OR...
    //         for( i.reset(s); i.test(); i++ ) { body = i.elem; }
```

```
    ((cite: hotspot/src/share/vm/libadt/set.hpp))
    class SetI {
```



### 詳細(Details)
See: [here](../doxygen/classSetI.html) for details

---
## <a name="nof50Jxzv1" id="nof50Jxzv1">SetI_</a>

### 概要(Summary)
SetI 内部で使用される補助クラス

```
    ((cite: hotspot/src/share/vm/libadt/set.hpp))
    class SetI_ : public ResourceObj {
```




### 詳細(Details)
See: [here](../doxygen/classSetI__.html) for details

---
