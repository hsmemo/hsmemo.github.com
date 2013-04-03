---
layout: default
title: ciFlags クラス 
---
[Top](../index.html)

#### ciFlags クラス 



---
## <a name="noOVcJnMkg" id="noOVcJnMkg">ciFlags</a>

### 概要(Summary)
ciInstanceKlass クラス, ciMethod クラス, 及び ciField クラス用の補助クラス.

JIT Compiler から AccessFlags オブジェクトの情報にアクセスするための一時オブジェクト(ValueObjクラス).
1つの ciFlags オブジェクトが 1つの AccessFlags オブジェクトに対応する.
(See: AccessFlags)


```
    ((cite: hotspot/src/share/vm/ci/ciFlags.hpp))
    // ciFlags
    //
    // This class represents klass or method flags.
    class ciFlags VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 ciInstanceKlass オブジェクトの _flags フィールド
* 各 ciMethod オブジェクトの _flags フィールド
* 各 ciField オブジェクトの _flags フィールド

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ValueObj クラスなので「生成」というのは少し違和感があるが, 以下の箇所でのみ新しい値を持ったインスタンスが生成されている).

* (上記のフィールドは全て, ポインタ型ではなく実体なので,
  格納しているオブジェクトの生成時に一緒に生成される)
  (といってもコンストラクタ内で再初期化されるが...)

* ciInstanceKlass::ciInstanceKlass
  
  (ciInstanceKlass::_flags フィールドの初期化用)  

* ciMethod::ciMethod
  
  (ciMethod::_flags フィールドの初期化用)

* ciField::initialize_from()
  
  (ciField::_flags フィールドの初期化用)
  
### 内部構造(Internal structure)
定義されているフィールドはこれだけ.


```
    ((cite: hotspot/src/share/vm/ci/ciFlags.hpp))
      jint _flags;
```




### 詳細(Details)
See: [here](../doxygen/classciFlags.html) for details

---
