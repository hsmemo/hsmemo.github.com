---
layout: default
title: ciMethod クラス 
---
[Top](../index.html)

#### ciMethod クラス 



---
## <a name="noYa4w3Yr4" id="noYa4w3Yr4">ciMethod</a>

### 概要(Summary)
ciObject クラスの具象サブクラスの1つ. methodOopDesc 用の ciObject クラス.


```
    ((cite: hotspot/src/share/vm/ci/ciMethod.hpp))
    // ciMethod
    //
    // This class represents a methodOop in the HotSpot virtual
    // machine.
    class ciMethod : public ciObject {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の箇所).

* 各 ciObjectFactory オブジェクトの _unloaded_methods フィールド
  
  unloaded/unfound なメソッドを表す ciMethod オブジェクトが格納されている.
  
  (正確には, このフィールドは ciMethod の GrowableArray を格納するフィールド.
  この中に, ciObjectFactory::get_unloaded_method() で生成された全ての ciMethod オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciObjectFactory::create_new_object()
  
  (ファクトリメソッド)

* ciObjectFactory::get_unloaded_method()

  (こちらは unloaded/unfound なメソッド用.
  なお, 一度生成したオブジェクトは ciObjectFactory::_unloaded_methods フィールドにメモイズされる)




### 詳細(Details)
See: [here](../doxygen/classciMethod.html) for details

---
