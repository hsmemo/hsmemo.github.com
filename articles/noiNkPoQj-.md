---
layout: default
title: ciConstant クラス 
---
[Top](../index.html)

#### ciConstant クラス 



---
## <a name="noIGC23ZZ9" id="noIGC23ZZ9">ciConstant</a>

### 概要(Summary)
JIT Compiler 内で定数を扱うためのユーティリティ・クラス(ValueObjクラス).
1つの ciConstant オブジェクトが 1つの定数値に対応する.


```
    ((cite: hotspot/src/share/vm/ci/ciConstant.hpp))
    // ciConstant
    //
    // This class represents a constant value.
    class ciConstant VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
(フィールド内に格納されているインスタンスも存在する)

* 各 ciField オブジェクトの _constant_value フィールド
  (アクセサは ciField::constant_value()).

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ValueObj クラスなので「生成」というのは少し違和感があるが, 以下の箇所でのみ新しい値を持ったインスタンスが生成されている. 
他の使用箇所はコピーコンストラクタ, あるいは既に生成済みの値へのポインタ).

* ciEnv::get_constant_by_index_impl()

* ciInstance::field_value()

* ciField::initialize_from()
    
  (ciField::_constant_value フィールドの初期化用)

### 内部構造(Internal structure)
定義されているフィールドは以下の通り
(定数値の型と値を格納しているだけ).


```
    ((cite: hotspot/src/share/vm/ci/ciConstant.hpp))
      BasicType _type;
      union {
        jint      _int;
        jlong     _long;
        jint      _long_half[2];
        jfloat    _float;
        jdouble   _double;
        ciObject* _object;
      } _value;
```





### 詳細(Details)
See: [here](../doxygen/classciConstant.html) for details

---
