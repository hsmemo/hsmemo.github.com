---
layout: default
title: ciObjArray クラス 
---
[Top](../index.html)

#### ciObjArray クラス 



---
## <a name="noHKyTN4_b" id="noHKyTN4_b">ciObjArray</a>

### 概要(Summary)
ciObject クラスの具象サブクラスの1つ. objArrayOopDesc 用の ciObject クラス.


```
    ((cite: hotspot/src/share/vm/ci/ciObjArray.hpp))
    // ciObjArray
    //
    // This class represents a ObjArrayOop in the HotSpot virtual
    // machine.
    class ciObjArray : public ciArray {
```

### 使われ方(Usage)
ciObjectFactory::create_new_object() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classciObjArray.html) for details

---
