---
layout: default
title: Dict クラス関連のクラス (Dict, DictI, 及びそれらの補助クラス(bucket))
---
[Top](../index.html)

#### Dict クラス関連のクラス (Dict, DictI, 及びそれらの補助クラス(bucket))

これらは, hotspot/src/share/vm/libadt/dict.hpp で定義されている同名のクラス群と(コメントまで含めて)ほとんど同じ.
(なお, libadt/ 下の方は ResourceObj のサブクラスだがこちらはスーパークラスを持たない, という違いはある)

(どう使い分けられている? #TODO)


### クラス一覧(class list)

  * [Dict](#noTWcPWKjO)
  * [DictI](#no0pjKwRKO)
  * [bucket](#nofinhlzH2)


---
## <a name="noTWcPWKjO" id="noTWcPWKjO">Dict</a>

### 概要(Summary)
(#Under Construction)


```cpp
    ((cite: hotspot/src/share/vm/adlc/dict2.hpp))
    class Dict { // Dictionary structure
```



### 詳細(Details)
See: [here](../doxygen/classDict.html) for details

---
## <a name="no0pjKwRKO" id="no0pjKwRKO">DictI</a>

### 概要(Summary)
(#Under Construction)


```cpp
    ((cite: hotspot/src/share/vm/adlc/dict2.hpp))
    //------------------------------Iteration--------------------------------------
    // The class of dictionary iterators.  Fails in the presences of modifications
    // to the dictionary during iteration (including searches).
    // Usage:  for( DictI i(dict); i.test(); ++i ) { body = i.key; body = i.value;}
    class DictI {
```



### 詳細(Details)
See: [here](../doxygen/classDictI.html) for details

---
## <a name="nofinhlzH2" id="nofinhlzH2">bucket</a>

### 概要(Summary)
(#Under Construction)


```cpp
    ((cite: hotspot/src/share/vm/adlc/dict2.cpp))
    class bucket {
```




### 詳細(Details)
See: [here](../doxygen/classbucket.html) for details

---
