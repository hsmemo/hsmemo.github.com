---
layout: default
title: methodKlass クラス 
---
[Top](../index.html)

#### methodKlass クラス 



---
## <a name="nopHgvsLzX" id="nopHgvsLzX">methodKlass</a>

### 概要(Summary)
methodOopDesc 用の Klass クラス.


```
    ((cite: hotspot/src/share/vm/oops/methodKlass.hpp))
    // a methodKlass is the klass of a methodOop
    
    class methodKlass : public Klass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _methodKlassObj フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
methodKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details



### 詳細(Details)
See: [here](../doxygen/classmethodKlass.html) for details

---
