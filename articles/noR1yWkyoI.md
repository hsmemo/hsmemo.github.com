---
layout: default
title: constantPoolCacheKlass クラス 
---
[Top](../index.html)

#### constantPoolCacheKlass クラス 



---
## <a name="nobJhSJzOG" id="nobJhSJzOG">constantPoolCacheKlass</a>

### 概要(Summary)
constantPoolCacheOopDesc 用の Klass クラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/cpCacheKlass.hpp))
    class constantPoolCacheKlass: public Klass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _constantPoolCacheKlassObj フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
constantPoolCacheKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details



### 詳細(Details)
See: [here](../doxygen/classconstantPoolCacheKlass.html) for details

---
