---
layout: default
title: methodDataKlass クラス 
---
[Top](../index.html)

#### methodDataKlass クラス 



---
## <a name="nonswFSCZW" id="nonswFSCZW">methodDataKlass</a>

### 概要(Summary)
methodDataOopDesc 用の Klass クラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/methodDataKlass.hpp))
    // a methodDataKlass is the klass of a methodDataOop
    
    class methodDataKlass : public Klass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _methodDataKlassObj フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
methodDataKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details



### 詳細(Details)
See: [here](../doxygen/classmethodDataKlass.html) for details

---
