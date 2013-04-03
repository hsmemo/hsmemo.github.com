---
layout: default
title: ciTypeArrayKlassKlass クラス 
---
[Top](../index.html)

#### ciTypeArrayKlassKlass クラス 



---
## <a name="no5Z2x_Vmw" id="no5Z2x_Vmw">ciTypeArrayKlassKlass</a>

### 概要(Summary)
ciArrayKlassKlass クラスの具象サブクラスの1つ. typeArrayKlassKlass 用の ciKlass クラス.


```
    ((cite: hotspot/src/share/vm/ci/ciTypeArrayKlassKlass.hpp))
    // ciTypeArrayKlassKlass
    //
    // This class represents a klassOop in the HotSpot virtual machine
    // whose Klass part is a typeArrayKlassKlass.
    class ciTypeArrayKlassKlass : public ciArrayKlassKlass {
```

### 使われ方(Usage)
ciObjectFactory::create_new_object() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classciTypeArrayKlassKlass.html) for details

---
