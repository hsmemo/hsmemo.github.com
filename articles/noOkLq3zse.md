---
layout: default
title: typeArrayOopDesc クラス 
---
[Top](../index.html)

#### typeArrayOopDesc クラス 



---
## <a name="noaWV6rjHl" id="noaWV6rjHl">typeArrayOopDesc</a>

### 概要(Summary)
arrayOopDesc クラスの具象サブクラスの1つ.

このクラスは「primitive 型の配列」を表す
(e.g. boolean[], byte[], char[], short[], int[], long[], float[], double[]).


```
    ((cite: hotspot/src/share/vm/oops/typeArrayOop.hpp))
    // A typeArrayOop is an array containing basic types (non oop elements).
    // It is used for arrays of {characters, singles, doubles, bytes, shorts, integers, longs}
    ...
    class typeArrayOopDesc : public arrayOopDesc {
```

### 備考(Notes)
なお, 実際の使用箇所では typeArrayOop という別名(もしくはラッパークラス)で使われることが多い (See: typeArrayOop).




### 詳細(Details)
See: [here](../doxygen/classtypeArrayOopDesc.html) for details

---
