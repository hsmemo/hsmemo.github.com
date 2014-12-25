---
layout: default
title: ciObjArrayKlassKlass クラス 
---
[Top](../index.html)

#### ciObjArrayKlassKlass クラス 



---
## <a name="nowe0P_1fY" id="nowe0P_1fY">ciObjArrayKlassKlass</a>

### 概要(Summary)
ciArrayKlassKlass クラスの具象サブクラスの1つ. objArrayKlassKlass 用の ciKlass クラス.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciObjArrayKlassKlass.hpp))
    // ciObjArrayKlassKlass
    //
    // This class represents a klassOop in the HotSpot virtual machine
    // whose Klass part is a objArrayKlassKlass.
    class ciObjArrayKlassKlass : public ciArrayKlassKlass {
```

### 使われ方(Usage)
ciObjectFactory::create_new_object() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classciObjArrayKlassKlass.html) for details

---
