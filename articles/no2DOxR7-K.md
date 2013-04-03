---
layout: default
title: klassKlass クラス 
---
[Top](../index.html)

#### klassKlass クラス 



---
## <a name="noNq91bnBC" id="noNq91bnBC">klassKlass</a>

### 概要(Summary)
klassOopDesc 用の Klass クラス.

klass フィールドによるポインタの連鎖の不動点として働く Klass であり,
klassKlass の klass は klassKlass 自身になっている.


```
    ((cite: hotspot/src/share/vm/oops/klassKlass.hpp))
    // A klassKlass serves as the fix point of the klass chain.
    // The klass of klassKlass is itself.
    
    class klassKlass: public Klass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _klassKlassObj フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
klassKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details



### 詳細(Details)
See: [here](../doxygen/classklassKlass.html) for details

---
