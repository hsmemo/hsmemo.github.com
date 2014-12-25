---
layout: default
title: objArrayKlass クラス 
---
[Top](../index.html)

#### objArrayKlass クラス 



---
## <a name="noOlSd554n" id="noOlSd554n">objArrayKlass</a>

### 概要(Summary)
objArrayOopDesc 用の Klass クラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/objArrayKlass.hpp))
    // objArrayKlass is the klass for objArrays
    
    class objArrayKlass : public arrayKlass {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
methodKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

(See: [here](no6897xAZ.html) for details)




### 詳細(Details)
See: [here](../doxygen/classobjArrayKlass.html) for details

---
