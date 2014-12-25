---
layout: default
title: ciTypeArray クラス 
---
[Top](../index.html)

#### ciTypeArray クラス 



---
## <a name="no5CyJWhtY" id="no5CyJWhtY">ciTypeArray</a>

### 概要(Summary)
ciObject クラスの具象サブクラスの1つ. typeArrayOopDesc 用の ciObject クラス.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciTypeArray.hpp))
    // ciTypeArray
    //
    // This class represents a typeArrayOop in the HotSpot virtual
    // machine.
    class ciTypeArray : public ciArray {
```

### 使われ方(Usage)
ciObjectFactory::create_new_object() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classciTypeArray.html) for details

---
