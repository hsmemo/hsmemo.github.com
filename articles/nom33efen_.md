---
layout: default
title: ciObjArrayKlass クラス 
---
[Top](../index.html)

#### ciObjArrayKlass クラス 



---
## <a name="no-BTZukQr" id="no-BTZukQr">ciObjArrayKlass</a>

### 概要(Summary)
ciArrayKlass クラスの具象サブクラスの1つ. objArrayKlass 用の ciKlass クラス.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciObjArrayKlass.hpp))
    // ciObjArrayKlass
    //
    // This class represents a klassOop in the HotSpot virtual machine
    // whose Klass part is an objArrayKlass.
    class ciObjArrayKlass : public ciArrayKlass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の箇所).

* 各 ciObjectFactory オブジェクトの _unloaded_klasses フィールド
  
  unloaded な配列クラスを表す ciObjArrayKlassciInstanceKlass オブジェクトが格納されている.
  
  (正確には, このフィールドは ciKlass の GrowableArray を格納するフィールド.
  この中に, ciObjectFactory::get_unloaded_klass() で生成された全ての ciInstanceKlass/ciObjArrayKlass オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciObjectFactory::create_new_object()
  
  (ファクトリメソッド)

* ciObjectFactory::get_unloaded_klass()

  (こちらは unloaded な配列クラス用.
  なお, 一度生成したオブジェクトは ciObjectFactory::_unloaded_klasses フィールドにメモイズされる)




### 詳細(Details)
See: [here](../doxygen/classciObjArrayKlass.html) for details

---
