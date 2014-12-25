---
layout: default
title: ciInstanceKlass クラス (ciInstanceKlass, 及びその補助クラス(NonStaticFieldFiller))
---
[Top](../index.html)

#### ciInstanceKlass クラス (ciInstanceKlass, 及びその補助クラス(NonStaticFieldFiller))



### クラス一覧(class list)

  * [ciInstanceKlass](#nosyIn4TwL)
  * [NonStaticFieldFiller](#nouvlNA0Li)


---
## <a name="nosyIn4TwL" id="nosyIn4TwL">ciInstanceKlass</a>

### 概要(Summary)
ciKlass クラスの具象サブクラスの1つ. instanceKlass 用の ciKlass クラス.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciInstanceKlass.hpp))
    // ciInstanceKlass
    //
    // This class represents a klassOop in the HotSpot virtual machine
    // whose Klass part is an instanceKlass.  It may or may not
    // be loaded.
    class ciInstanceKlass : public ciKlass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の箇所).

* ciEnv クラスの _unloaded_ciinstance_klass フィールド (static フィールド)
  
  (ダミーの ciInstanceKlass オブジェクト)

* 各 ciObjectFactory オブジェクトの _unloaded_klasses フィールド
  
  unloaded なクラスを表す ciInstanceKlass オブジェクトが格納されている.
  
  (正確には, このフィールドは ciKlass の GrowableArray を格納するフィールド.
  この中に, ciObjectFactory::get_unloaded_klass() で生成された全ての ciInstanceKlass/ciObjArrayKlass オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciObjectFactory::create_new_object()

  (ファクトリメソッド)

* ciObjectFactory::init_shared_objects()
  
  (ciEnv::_unloaded_ciinstance_klass フィールドの初期化用)

* ciObjectFactory::get_unloaded_klass()
  
  (こちらは unloaded なクラス用.
  なお, 一度生成したオブジェクトは ciObjectFactory::_unloaded_klasses フィールドにメモイズされる)




### 詳細(Details)
See: [here](../doxygen/classciInstanceKlass.html) for details

---
## <a name="nouvlNA0Li" id="nouvlNA0Li">NonStaticFieldFiller</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらないような...)

ciInstanceKlass クラス内で使用される補助クラス(Closure クラス).

ciInstanceKlass オブジェクトの 
_non_static_fields フィールドの遅延初期化を行うための Closure クラス.

(ただし, 肝心の ciInstanceKlass::_non_static_fields フィールド自体が使われていないような... #TODO)


```cpp
    ((cite: hotspot/src/share/vm/ci/ciInstanceKlass.cpp))
    class NonStaticFieldFiller: public FieldClosure {
```

### 使われ方(Usage)
ciInstanceKlass::non_static_fields() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classNonStaticFieldFiller.html) for details

---
