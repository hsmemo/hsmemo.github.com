---
layout: default
title: compiledICHolderKlass クラス 
---
[Top](../index.html)

#### compiledICHolderKlass クラス 



---
## <a name="noS56vx9Xj" id="noS56vx9Xj">compiledICHolderKlass</a>

### 概要(Summary)
compiledICHolderOopDesc 用の Klass クラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/compiledICHolderKlass.hpp))
    // a compiledICHolderKlass is the klass of a compiledICHolderOop
    
    class compiledICHolderKlass : public Klass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _compiledICHolderKlassObj フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
compiledICHolderKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details



### 詳細(Details)
See: [here](../doxygen/classcompiledICHolderKlass.html) for details

---
