---
layout: default
title: constantPoolKlass クラス 
---
[Top](../index.html)

#### constantPoolKlass クラス 



---
## <a name="noJNaIfxCQ" id="noJNaIfxCQ">constantPoolKlass</a>

### 概要(Summary)
constantPoolOopDesc 用の Klass クラス.


```
    ((cite: hotspot/src/share/vm/oops/constantPoolKlass.hpp))
    // A constantPoolKlass is the klass of a constantPoolOop
    
    class constantPoolKlass : public Klass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _constantPoolKlassObj フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
constantPoolKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details



### 詳細(Details)
See: [here](../doxygen/classconstantPoolKlass.html) for details

---
