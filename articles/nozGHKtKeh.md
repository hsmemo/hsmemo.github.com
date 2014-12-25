---
layout: default
title: instanceKlassKlass クラス (instanceKlassKlass, 及びその補助クラス(VerifyFieldClosure))
---
[Top](../index.html)

#### instanceKlassKlass クラス (instanceKlassKlass, 及びその補助クラス(VerifyFieldClosure))



### クラス一覧(class list)

  * [instanceKlassKlass](#nohdRFdA_G)
  * [VerifyFieldClosure](#noPVF89GHb)


---
## <a name="nohdRFdA_G" id="nohdRFdA_G">instanceKlassKlass</a>

### 概要(Summary)
instanceKlass (及びそのサブクラス) 用の Klass クラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/instanceKlassKlass.hpp))
    // An instanceKlassKlass is the klass of an instanceKlass
    
    class instanceKlassKlass : public klassKlass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _instanceKlassKlassObj フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
instanceKlassKlass::create_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは Universe::genesis() 内で(のみ)呼び出されている.

#### 参考(for your information): Universe::genesis()
See: [here](no4230JvC.html) for details



### 詳細(Details)
See: [here](../doxygen/classinstanceKlassKlass.html) for details

---
## <a name="noPVF89GHb" id="noPVF89GHb">VerifyFieldClosure</a>

### 概要(Summary)
instanceKlassKlass 内で使用される補助クラス.

フィールドに入っているポインタ値が Java ヒープ内を指しており,
かつ差し先が妥当な oop または NULL であることをチェックする.


```cpp
    ((cite: hotspot/src/share/vm/oops/instanceKlassKlass.cpp))
    // Verification
    
    class VerifyFieldClosure: public OopClosure {
```

### 使われ方(Usage)
?? (このクラスは使用箇所が見当たらない...)

(instanceKlassKlass::oop_verify_on() 内に局所変数として宣言されている箇所はあるが,
その局所変数が使用されている箇所はない... #TODO)

### 備考(Notes)
同名のクラスが hotspot/src/share/vm/oops/instanceKlass.cpp にいたりするが特に関係は無い模様.




### 詳細(Details)
See: [here](../doxygen/classVerifyFieldClosure.html) for details

---
