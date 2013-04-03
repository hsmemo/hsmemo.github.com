---
layout: default
title: ciMethodKlass クラス 
---
[Top](../index.html)

#### ciMethodKlass クラス 



---
## <a name="now0F5_7U_" id="now0F5_7U_">ciMethodKlass</a>

### 概要(Summary)
ciKlass クラスの具象サブクラスの1つ. methodKlass 用の ciKlass クラス.


```
    ((cite: hotspot/src/share/vm/ci/ciMethodKlass.hpp))
    // ciMethodKlass
    //
    // This class represents a klassOop in the HotSpot virtual machine
    // whose Klass part in a methodKlass.
    class ciMethodKlass : public ciKlass {
```

### 使われ方(Usage)
ciObjectFactory::create_new_object() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classciMethodKlass.html) for details

---
