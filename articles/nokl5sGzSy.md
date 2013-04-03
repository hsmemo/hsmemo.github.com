---
layout: default
title: typeArrayKlassKlass クラス 
---
[Top](../index.html)

#### typeArrayKlassKlass クラス 



---
## <a name="noXRMEy-rn" id="noXRMEy-rn">typeArrayKlassKlass</a>

### 概要(Summary)
typeArrayKlass 用の Klass クラス.


```
    ((cite: hotspot/src/share/vm/oops/typeArrayKlassKlass.hpp))
    // A typeArrayKlassKlass is the klass of a typeArrayKlass
    
    class typeArrayKlassKlass : public arrayKlassKlass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _typeArrayKlassKlassObj フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
typeArrayKlassKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details



### 詳細(Details)
See: [here](../doxygen/classtypeArrayKlassKlass.html) for details

---
