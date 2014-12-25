---
layout: default
title: typeArrayKlass クラス 
---
[Top](../index.html)

#### typeArrayKlass クラス 



---
## <a name="nonm-MjQri" id="nonm-MjQri">typeArrayKlass</a>

### 概要(Summary)
typeArrayOopDesc 用の Klass クラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/typeArrayKlass.hpp))
    // A typeArrayKlass is the klass of a typeArray
    // It contains the type and size of the elements
    
    class typeArrayKlass : public arrayKlass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの以下のフィールドに(のみ)格納されている.

* Universe::_boolArrayKlassObj
* Universe::_byteArrayKlassObj
* Universe::_charArrayKlassObj
* Universe::_intArrayKlassObj
* Universe::_shortArrayKlassObj
* Universe::_longArrayKlassObj
* Universe::_singleArrayKlassObj
* Universe::_doubleArrayKlassObj

#### 生成箇所(where its instances are created)
typeArrayKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details



### 詳細(Details)
See: [here](../doxygen/classtypeArrayKlass.html) for details

---
