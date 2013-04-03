---
layout: default
title: GrowableArray クラス関連のクラス (GenericGrowableArray, GrowableArray)
---
[Top](../index.html)

#### GrowableArray クラス関連のクラス (GenericGrowableArray, GrowableArray)

これらは, 「可変長の配列」として働くユーティリティ・クラス.


```
    ((cite: hotspot/src/share/vm/utilities/growableArray.hpp))
    // A growable array.
```



### クラス一覧(class list)

  * [GenericGrowableArray](#nogCTxfaXo)
  * [GrowableArray](#nogMWf40iA)


---
## <a name="nogCTxfaXo" id="nogCTxfaXo">GenericGrowableArray</a>

### 概要(Summary)
可変長の配列として働くユーティリティ・クラス (の基底クラス).


```
    ((cite: hotspot/src/share/vm/utilities/growableArray.hpp))
    class GenericGrowableArray : public ResourceObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classGenericGrowableArray.html) for details

---
## <a name="nogMWf40iA" id="nogMWf40iA">GrowableArray</a>

### 概要(Summary)
GenericGrowableArray クラスの具象サブクラス.

なお要素の型は template でパラメタライズされている.


```
    ((cite: hotspot/src/share/vm/utilities/growableArray.hpp))
    template<class E> class GrowableArray : public GenericGrowableArray {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 備考(Notes)
GrowableArray の中に Handle を入れる場合は Handle の有効範囲に注意するように, とのこと.

(Handle が GrowableArray の中に入ったまま
HandleMark のスコープ外に出てしまうとダングリングポインタになる).


```
    ((cite: hotspot/src/share/vm/utilities/growableArray.hpp))
    /*************************************************************************/
    /*                                                                       */
    /*     WARNING WARNING WARNING WARNING WARNING WARNING WARNING WARNING   */
    /*                                                                       */
    /* Should you use GrowableArrays to contain handles you must be certain  */
    /* the the GrowableArray does not outlive the HandleMark that contains   */
    /* the handles. Since GrowableArrays are typically resource allocated    */
    /* the following is an example of INCORRECT CODE,                        */
    /*                                                                       */
    /* ResourceMark rm;                                                      */
    /* GrowableArray<Handle>* arr = new GrowableArray<Handle>(size);         */
    /* if (blah) {                                                           */
    /*    while (...) {                                                      */
    /*      HandleMark hm;                                                   */
    /*      ...                                                              */
    /*      Handle h(THREAD, some_oop);                                      */
    /*      arr->append(h);                                                  */
    /*    }                                                                  */
    /* }                                                                     */
    /* if (arr->length() != 0 ) {                                            */
    /*    oop bad_oop = arr->at(0)(); // Handle is BAD HERE.                 */
    /*    ...                                                                */
    /* }                                                                     */
    /*                                                                       */
    /* If the GrowableArrays you are creating is C_Heap allocated then it    */
    /* hould not old handles since the handles could trivially try and       */
    /* outlive their HandleMark. In some situations you might need to do     */
    /* this and it would be legal but be very careful and see if you can do  */
    /* the code in some other manner.                                        */
    /*                                                                       */
    /*************************************************************************/
```




### 詳細(Details)
See: [here](../doxygen/classGrowableArray.html) for details

---
