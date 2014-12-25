---
layout: default
title: ciTypeArrayKlass クラス 
---
[Top](../index.html)

#### ciTypeArrayKlass クラス 



---
## <a name="no8u1d8Lj4" id="no8u1d8Lj4">ciTypeArrayKlass</a>

### 概要(Summary)
ciArrayKlass クラスの具象サブクラスの1つ. typeArrayKlass 用の ciKlass クラス.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciTypeArrayKlass.hpp))
    // ciTypeArrayKlass
    //
    // This class represents a klassOop in the HotSpot virtual machine
    // whose Klass part in a TypeArrayKlass.
    class ciTypeArrayKlass : public ciArrayKlass {
```

### 使われ方(Usage)
ciObjectFactory::create_new_object() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classciTypeArrayKlass.html) for details

---
