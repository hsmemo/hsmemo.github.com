---
layout: default
title: Dict クラス関連のクラス (Dict, DictI, 及びそれらの補助クラス(bucket))
---
[Top](../index.html)

#### Dict クラス関連のクラス (Dict, DictI, 及びそれらの補助クラス(bucket))

これらは, ADLC 内で使用される写像クラス(Dictionary クラス).
key と value の対応を記録する (See: [here](nop0Yyr-jc.html) for details).


```
    ((cite: hotspot/src/share/vm/libadt/dict.hpp))
    // These dictionaries define a key-value mapping.  They can be inserted to,
    // searched or deleted from.  They grow and shrink as needed.  The key is a
    // pointer to something (or anything which can be stored in a pointer).  A
    // key comparison routine determines if two keys are equal or not.  A hash
    // function can be provided; if it's not provided the key itself is used
    // instead.  A nice string hash function is included.
```


### クラス一覧(class list)

  * [Dict](#noX4XqhcgR)
  * [DictI](#noztRERMuU)
  * [bucket](#no5ZkuryC5)


---
## <a name="noX4XqhcgR" id="noX4XqhcgR">Dict</a>

### 概要(Summary)
key と value の対応を記録する写像クラス(Dictionary クラス).


```
    ((cite: hotspot/src/share/vm/libadt/dict.hpp))
    class Dict : public ResourceObj { // Dictionary structure
```



### 詳細(Details)
See: [here](../doxygen/classDict.html) for details

---
## <a name="noztRERMuU" id="noztRERMuU">DictI</a>

### 概要(Summary)
Dict クラスの中身を処理するためのイテレータクラス.

以下のように使う.

  `for( DictI i(dict); i.test(); ++i ) { body = i.key; body = i.value;}`


```
    ((cite: hotspot/src/share/vm/libadt/dict.hpp))
    //------------------------------Iteration--------------------------------------
    // The class of dictionary iterators.  Fails in the presences of modifications
    // to the dictionary during iteration (including searches).
    // Usage:  for( DictI i(dict); i.test(); ++i ) { body = i.key; body = i.value;}
    class DictI {
```



### 詳細(Details)
See: [here](../doxygen/classDictI.html) for details

---
## <a name="no5ZkuryC5" id="no5ZkuryC5">bucket</a>

### 概要(Summary)
Dict クラス内で使用される補助クラス.
このクラスを用いてハッシュが実装されている.


```
    ((cite: hotspot/src/share/vm/libadt/dict.cpp))
    //------------------------------bucket---------------------------------------
    class bucket : public ResourceObj {
```




### 詳細(Details)
See: [here](../doxygen/classbucket.html) for details

---
