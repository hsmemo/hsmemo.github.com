---
layout: default
title: ciMethodHandle クラス 
---
[Top](../index.html)

#### ciMethodHandle クラス 



---
## <a name="noOJ7bw3Tu" id="noOJ7bw3Tu">ciMethodHandle</a>

### 概要(Summary)
ciInstance クラスのサブクラスの1つ. java.lang.invoke.MethodHandle オブジェクトを表す instanceOopDesc 用の ciObject クラス.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodHandle.hpp))
    // ciMethodHandle
    //
    // The class represents a java.lang.invoke.MethodHandle object.
    class ciMethodHandle : public ciInstance {
```

### 使われ方(Usage)
ciObjectFactory::create_new_object() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classciMethodHandle.html) for details

---
