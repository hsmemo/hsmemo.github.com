---
layout: default
title: ciNullObject クラス 
---
[Top](../index.html)

#### ciNullObject クラス 



---
## <a name="no7_Dfy_Ds" id="no7_Dfy_Ds">ciNullObject</a>

### 概要(Summary)
ciObject クラスの具象サブクラスの1つ.

NULL 用の ciObject クラス. 変換元の oop が NULL だった場合, このオブジェクトが返される.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciNullObject.hpp))
    // ciNullObject
    //
    // This class represents a null reference in the VM.
    class ciNullObject : public ciObject {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ciEnv クラスの _null_object_instance フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
ciObjectFactory::init_shared_objects() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
ciEnv::_null_object_instance フィールドは以下の箇所で(のみ)参照されている.

* ciEnv::get_object()
  
* ciNullObject::make()
  



### 詳細(Details)
See: [here](../doxygen/classciNullObject.html) for details

---
