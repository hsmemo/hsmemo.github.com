---
layout: default
title: constMethodKlass クラス 
---
[Top](../index.html)

#### constMethodKlass クラス 



---
## <a name="no4Wjjwcl9" id="no4Wjjwcl9">constMethodKlass</a>

### 概要(Summary)
constMethodOopDesc 用の Klass クラス.


```
    ((cite: hotspot/src/share/vm/oops/constMethodKlass.hpp))
    // A constMethodKlass is the klass of a constMethodOop
    
    class constMethodKlass : public Klass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _constMethodKlassObj フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
constMethodKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details



### 詳細(Details)
See: [here](../doxygen/classconstMethodKlass.html) for details

---
