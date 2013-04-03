---
layout: default
title: ciCallSite クラス 
---
[Top](../index.html)

#### ciCallSite クラス 



---
## <a name="no9Jil2ny9" id="no9Jil2ny9">ciCallSite</a>

### 概要(Summary)
ciInstance クラスのサブクラスの1つ. java.lang.invoke.CallSite オブジェクトを表す instanceOopDesc 用の ciObject クラス.


```
    ((cite: hotspot/src/share/vm/ci/ciCallSite.hpp))
    // ciCallSite
    //
    // The class represents a java.lang.invoke.CallSite object.
    class ciCallSite : public ciInstance {
```

### 使われ方(Usage)
ciObjectFactory::create_new_object() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classciCallSite.html) for details

---
