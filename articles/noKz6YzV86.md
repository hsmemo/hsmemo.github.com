---
layout: default
title: objArrayKlassKlass クラス 
---
[Top](../index.html)

#### objArrayKlassKlass クラス 



---
## <a name="nowDGI8l3b" id="nowDGI8l3b">objArrayKlassKlass</a>

### 概要(Summary)
objArrayKlass 用の Klass クラス.


```
    ((cite: hotspot/src/share/vm/oops/objArrayKlassKlass.hpp))
    // The objArrayKlassKlass is klass for all objArrayKlass'
    
    class objArrayKlassKlass : public arrayKlassKlass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _objArrayKlassKlassObj フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
objArrayKlassKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details



### 詳細(Details)
See: [here](../doxygen/classobjArrayKlassKlass.html) for details

---
