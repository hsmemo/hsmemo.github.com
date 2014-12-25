---
layout: default
title: arrayKlassKlass クラス 
---
[Top](../index.html)

#### arrayKlassKlass クラス 



---
## <a name="noMEuDwfwU" id="noMEuDwfwU">arrayKlassKlass</a>

### 概要(Summary)
arrayKlass 用の Klass クラスの基底クラス.

(なお, このクラスは abstract class ではない)


```cpp
    ((cite: hotspot/src/share/vm/oops/arrayKlassKlass.hpp))
    // arrayKlassKlass is the abstract baseclass for all array class classes
    
    class arrayKlassKlass : public klassKlass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _arrayKlassKlassObj フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
arrayKlassKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details
#### 使用箇所(where its instances are used)
(どこで使われている?? #TODO)




### 詳細(Details)
See: [here](../doxygen/classarrayKlassKlass.html) for details

---
