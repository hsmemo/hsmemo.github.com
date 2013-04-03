---
layout: default
title: ciCPCache クラス 
---
[Top](../index.html)

#### ciCPCache クラス 



---
## <a name="noTj5HAViD" id="noTj5HAViD">ciCPCache</a>

### 概要(Summary)
ciObject クラスの具象サブクラスの1つ. constantPoolCacheOopDesc 用の ciObject クラス.

なおコメントによると, 
「ciConstantPoolCache というクラス名は別の用途で使っているので ciCPCache という名前になっている」
とのこと.


```
    ((cite: hotspot/src/share/vm/ci/ciCPCache.hpp))
    // ciCPCache
    //
    // This class represents a constant pool cache.
    //
    // Note: This class is called ciCPCache as ciConstantPoolCache is used
    // for something different.
    class ciCPCache : public ciObject {
```

### 使われ方(Usage)
ciObjectFactory::create_new_object() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classciCPCache.html) for details

---
