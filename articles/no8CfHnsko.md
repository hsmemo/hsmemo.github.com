---
layout: default
title: ciInstanceKlassKlass クラス 
---
[Top](../index.html)

#### ciInstanceKlassKlass クラス 



---
## <a name="noHnhpZkx_" id="noHnhpZkx_">ciInstanceKlassKlass</a>

### 概要(Summary)
ciKlassKlass クラスの具象サブクラスの1つ. instanceKlassKlass 用の ciKlass クラス.


```
    ((cite: hotspot/src/share/vm/ci/ciInstanceKlassKlass.hpp))
    // ciInstanceKlassKlass
    //
    // This class represents a klassOop in the HotSpot virtual machine
    // whose Klass part is a instanceKlassKlass.
    class ciInstanceKlassKlass : public ciKlassKlass {
```

### 使われ方(Usage)
ciObjectFactory::create_new_object() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classciInstanceKlassKlass.html) for details

---
