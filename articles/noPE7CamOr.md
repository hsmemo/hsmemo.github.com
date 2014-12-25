---
layout: default
title: objArrayOopDesc クラス 
---
[Top](../index.html)

#### objArrayOopDesc クラス 



---
## <a name="no-VTbzue-" id="no-VTbzue-">objArrayOopDesc</a>

### 概要(Summary)
arrayOopDesc クラスの具象サブクラスの1つ.

このクラスは「ポインタ型(オブジェクト or 配列)の配列」を表す.


```cpp
    ((cite: hotspot/src/share/vm/oops/objArrayOop.hpp))
    // An objArrayOop is an array containing oops.
    // Evaluating "String arg[10]" will create an objArrayOop.
    
    class objArrayOopDesc : public arrayOopDesc {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で生成されている.

* arrayKlass::allocate_arrayArray()
* objArrayKlass::allocate()
* objArrayKlass::multi_allocate()
* jni_NewObjectArray()
* JVM_AllocateNewArray()
* ...

### 備考(Notes)
なお, 実際の使用箇所では objArrayOop という別名(もしくはラッパークラス)で使われることが多い (See: objArrayOop).




### 詳細(Details)
See: [here](../doxygen/classobjArrayOopDesc.html) for details

---
