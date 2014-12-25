---
layout: default
title: ciInstance クラス 
---
[Top](../index.html)

#### ciInstance クラス 



---
## <a name="nonFL2oLac" id="nonFL2oLac">ciInstance</a>

### 概要(Summary)
ciObject クラスの具象サブクラスの1つ.
instanceOopDesc 用の ciObject クラス
(ただし java.lang.invoke.MethodHandle オブジェクトと java.lang.invoke.CallSite オブジェクトは除く. 備考参照).


```cpp
    ((cite: hotspot/src/share/vm/ci/ciInstance.hpp))
    // ciInstance
    //
    // This class represents an instanceOop in the HotSpot virtual
    // machine.  This is an oop which corresponds to a non-array
    // instance of java.lang.Object.
    class ciInstance : public ciObject {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の箇所).

* 各 ciObjectFactory オブジェクトの _unloaded_instances フィールド
  
  unresolved なクラスのインスタンスを表す ciInstance オブジェクトが格納されている.
  
  (正確には, このフィールドは ciInstance の GrowableArray を格納するフィールド.
  この中に, ciObjectFactory::get_unloaded_instance() で生成された全ての ciInstance オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciObjectFactory::create_new_object()

  (ファクトリメソッド)

* ciObjectFactory::get_unloaded_instance()
  
  (こちらは unresolved なクラスのインスタンス用.
  なお, 一度生成したオブジェクトは ciObjectFactory::_unloaded_instances フィールドにメモイズされる)

### 備考(Notes)
java.lang.invoke.MethodHandle オブジェクトと java.lang.invoke.CallSite オブジェクトについては, 
最適化する上で重要なので特別にサブクラスが用意されている
(See: ciMethodHandle, ciCallSite).




### 詳細(Details)
See: [here](../doxygen/classciInstance.html) for details

---
