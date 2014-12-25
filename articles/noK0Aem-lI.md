---
layout: default
title: arrayOopDesc クラス 
---
[Top](../index.html)

#### arrayOopDesc クラス 



---
## <a name="nopCk41oB6" id="nopCk41oB6">arrayOopDesc</a>

### 概要(Summary)
「Java の配列」を表すクラスの基底クラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/arrayOop.hpp))
    // arrayOopDesc is the abstract baseclass for all arrays.  It doesn't
    // declare pure virtual to enforce this because that would allocate a vtbl
    // in each instance, which we don't want.
```


```cpp
    ((cite: hotspot/src/share/vm/oops/arrayOop.hpp))
    class arrayOopDesc : public oopDesc {
```

なお, このクラス自体は abstract class であり, 実際に使われるのは以下のサブクラス.

* typeArrayOopDesc
* objArrayOopDesc

### 内部構造(Internal structure)
スーパークラスである oopDesc クラスの mark フィールド(markOopDesc)と klass フィールドに加え, 配列長を示す length フィールドを持つ.


```cpp
    ((cite: hotspot/src/share/vm/oops/arrayOop.hpp))
    // The layout of array Oops is:
    //
    //  markOop
    //  klassOop  // 32 bits if compressed but declared 64 in LP64.
    //  length    // shares klass memory or allocated after declared fields.
```

### 備考(Notes)
なお, 実際の使用箇所では arrayOop という別名(もしくはラッパークラス)で使われることが多い (See: arrayOop).




### 詳細(Details)
See: [here](../doxygen/classarrayOopDesc.html) for details

---
