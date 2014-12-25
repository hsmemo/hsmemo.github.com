---
layout: default
title: HeapRegionSet クラス関連のクラス (HeapRegionSetBase, hrs_ext_msg, HeapRegionSet, HeapRegionLinkedList, HeapRegionLinkedListIterator)
---
[Top](../index.html)

#### HeapRegionSet クラス関連のクラス (HeapRegionSetBase, hrs_ext_msg, HeapRegionSet, HeapRegionLinkedList, HeapRegionLinkedListIterator)

これらは, G1GC で使用するメモリ領域の統計情報を管理するためのクラス.

### 概要(Summary)
HeapRegionSet は, 複数の HeapRegion からなる集合 (の統計情報) を管理するためのクラス.
例えば, 集合の要素数(含まれる HeapRegion の個数), 合計のメモリ量, 等を記録している.

なお, サブクラスの HeapRegionLinkedList では, 集合に含まれる HeapRegion オブジェクト1つ1つまで管理している.
それ以外のクラスでは, 実際にどういう HeapRegion が含まれているかは管理していない.
`#ifdef ASSERT` 時にだけは HeapRegionLinkedList 以外でもどういう HeapRegion を含んでいるかを管理しているが, 
この場合も対応関係は HeapRegionSet ではなく HeapRegion オブジェクトの方に記録されている
(HeapRegion::_containing_set フィールド参照)

なお, これらのクラスは以下のような継承関係を持つ.

  * HeapRegionSetBase                    (abstract class)
      * HeapRegionSet                    (abstract class)
          * HumongousRegionSet             (<= これは別ファイルで定義されているクラス)
              * MasterHumongousRegionSet   (<= これは別ファイルで定義されているクラス)
      * HeapRegionLinkedList             (abstract class)
          * FreeRegionList                 (<= これは別ファイルで定義されているクラス)
              * MasterFreeRegionList       (<= これは別ファイルで定義されているクラス)
              * SecondaryFreeRegionList    (<= これは別ファイルで定義されているクラス)



### クラス一覧(class list)

  * [HeapRegionSetBase](#noEp6q11NH)
  * [HeapRegionSet](#noPbqhIsoZ)
  * [HeapRegionLinkedList](#noZeLlPh7H)
  * [hrs_ext_msg](#noSkICvGU4)
  * [HeapRegionLinkedListIterator](#noDIi0QkDw)


---
## <a name="noEp6q11NH" id="noEp6q11NH">HeapRegionSetBase</a>

### 概要(Summary)
全ての HeapRegionSet クラスの基底クラス.
サブクラスで共通して使われる基本的なフィールドやメソッドを提供している
(length, region num, used bytes sum, verification(), etc).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp))
    // Base class for all the classes that represent heap region sets. It
    // contains the basic attributes that each set needs to maintain
    // (e.g., length, region num, used bytes sum) plus any shared
    // functionality (e.g., verification).
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp))
    class HeapRegionSetBase VALUE_OBJ_CLASS_SPEC {
```




### 詳細(Details)
See: [here](../doxygen/classHeapRegionSetBase.html) for details

---
## <a name="noPbqhIsoZ" id="noPbqhIsoZ">HeapRegionSet</a>

### 概要(Summary)
HeapRegionSetBase クラスのサブクラスの1つ.
集合に含まれる HeapRegion オブジェクト1つ1つまで記録しない HeapRegionSet クラスの基底クラス
(記録する場合の基底クラスは HeapRegionLinkedList).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp))
    // This class represents heap region sets whose members are not
    // explicitly tracked. It's helpful to group regions using such sets
    // so that we can reason about all the region groups in the heap using
    // the same interface (namely, the HeapRegionSetBase API).
    
    class HeapRegionSet : public HeapRegionSetBase {
```

### 内部構造(Internal structure)
(スーパークラスである HeapRegionSetBase クラスのメソッドに加えて) 
集合に HeapRegion を追加/削除するための HeapRegionSet::add(), HeapRegionSet::remove() 等が定義されている
(それぞれ HeapRegionSetBase::add_internal(), HeapRegionSetBase::remove_internal() を呼んでいるだけだが...).

また, マルチスレッド時に remove が競合して遅くならないように,
thread local なコピーを作る HeapRegionSet::remove_with_proxy() と, 
コピーの結果を本体に反映させるための HeapRegionSet::update_from_proxy() も備えている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp))
      // It adds hr to the set. The region should not be a member of
      // another set.
      inline void add(HeapRegion* hr);
    
      // It removes hr from the set. The region should be a member of
      // this set.
      inline void remove(HeapRegion* hr);
    
      // It removes a region from the set. Instead of updating the fields
      // of the set to reflect this removal, it accumulates the updates
      // in proxy_set. The idea is that proxy_set is thread-local to
      // avoid multiple threads updating the fields of the set
      // concurrently and having to synchronize. The method
      // update_from_proxy() will update the fields of the set from the
      // proxy_set.
      inline void remove_with_proxy(HeapRegion* hr, HeapRegionSet* proxy_set);
    
      // After multiple calls to remove_with_proxy() the updates to the
      // fields of the set are accumulated in proxy_set. This call
      // updates the fields of the set from proxy_set.
      void update_from_proxy(HeapRegionSet* proxy_set);
```




### 詳細(Details)
See: [here](../doxygen/classHeapRegionSet.html) for details

---
## <a name="noZeLlPh7H" id="noZeLlPh7H">HeapRegionLinkedList</a>

### 概要(Summary)
HeapRegionSetBase クラスのサブクラスの1つ.
集合に含まれる HeapRegion オブジェクト1つ1つまで記録する HeapRegionSet クラスの基底クラス
(記録しない場合の基底クラスは HeapRegionSet).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

名前の通り, 内部では HeapRegion を linked list 状につないで管理している.
性能上クリティカルなところで HeapRegionLinkedList 中の HeapRegion を辿るのはおすすめしない, とのこと
(大抵の場合は HeapRegion を 1つ add/remove したり
 2つの HeapRegionLinkedList を append するという使い方になるはずで, 
 これらであれば(linked list なので)定数時間だから問題ない, とのこと).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp))
    //////////////////// HeapRegionLinkedList ////////////////////
    
    // A set that links all the regions added to it in a singly-linked
    // list. We should try to avoid doing operations that iterate over
    // such lists in performance critical paths. Typically we should
    // add / remove one region at a time or concatenate two lists. All
    // those operations are done in constant time.
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp))
    class HeapRegionLinkedList : public HeapRegionSetBase {
```

### 内部構造(Internal structure)
内部では, head/tail というフィールドに HeapRegion を格納している
(リスト自体は HeapRegion オブジェクトの _next フィールドを用いて構築).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp))
      HeapRegion* _head;
      HeapRegion* _tail;
```




### 詳細(Details)
See: [here](../doxygen/classHeapRegionLinkedList.html) for details

---
## <a name="noSkICvGU4" id="noSkICvGU4">hrs_ext_msg</a>

### 概要(Summary)
HeapRegionSetBase クラス(及びそのサブクラス)内で使用される補助クラス.

HeapRegionSetBase 用の FormatBuffer クラス
(hrs_err_msg は FormatBuffer の別名 (See: FormatBuffer)).

コメントによると, 
HeapRegionSetBase の friend class になっているので(?), 
HeapRegionSet 内のフィールドの値にもアクセスでき, より詳細なエラー情報が出せる, 
とのこと
(See: HeapRegionSetBase::fill_in_ext_msg()).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp))
    // Customized err_msg for heap region sets. Apart from a
    // assert/guarantee-specific message it also prints out the values of
    // the fields of the associated set. This can be very helpful in
    // diagnosing failures.
    
    class hrs_ext_msg : public hrs_err_msg {
```

### 使われ方(Usage)
HeapRegionSet 中の様々な箇所で (主に assert/guarantee 用の文字列を構築する用途で) 使用されている.




### 詳細(Details)
See: [here](../doxygen/classhrs__ext__msg.html) for details

---
## <a name="noDIi0QkDw" id="noDIi0QkDw">HeapRegionLinkedListIterator</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス(??) (#ifdef ASSERT 時にしか使用されない? #TODO).

HeapRegionLinkedList 内の要素をたどるためのイテレータクラス(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp))
    //////////////////// HeapRegionLinkedListIterator ////////////////////
    
    // Iterator class that provides a convenient way to iterate over the
    // regions of a HeapRegionLinkedList instance.
    
    class HeapRegionLinkedListIterator : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* HeapRegionLinkedList::add_as_head()  (ただし, #ifdef ASSERT 時にしか使用されない)
* HeapRegionLinkedList::add_as_tail()  (ただし, #ifdef ASSERT 時にしか使用されない)
* HeapRegionLinkedList::print_on() 
  (この関数を呼び出しているのは HeapRegionSetBase::verify_region() だけ?? #TODO)




### 詳細(Details)
See: [here](../doxygen/classHeapRegionLinkedListIterator.html) for details

---
