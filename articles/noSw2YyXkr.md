---
layout: default
title: ciKlassKlass クラス 
---
[Top](../index.html)

#### ciKlassKlass クラス 



---
## <a name="nowxxDMFnS" id="nowxxDMFnS">ciKlassKlass</a>

### 概要(Summary)
ciKlass クラスの具象サブクラスの1つ. klassKlass (またはそのサブクラス) 用の ciKlass クラス.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciKlassKlass.hpp))
    // ciKlassKlass
    //
    // This class represents a klassOop in the HotSpot virtual machine
    // whose Klass part is a klassKlass or one of its subclasses
    // (instanceKlassKlass, objArrayKlassKlass, typeArrayKlassKlass).
    class ciKlassKlass : public ciKlass {
```

### 使われ方(Usage)
ciObjectFactory::create_new_object() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classciKlassKlass.html) for details

---
