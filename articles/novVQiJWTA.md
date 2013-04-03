---
layout: default
title: ciSignature クラス 
---
[Top](../index.html)

#### ciSignature クラス 



---
## <a name="nohO7h-65k" id="nohO7h-65k">ciSignature</a>

### 概要(Summary)
ciMethod クラス用の補助クラス.

メソッドの型を表す文字列(Signature String)情報を管理するクラス.


```
    ((cite: hotspot/src/share/vm/ci/ciSignature.hpp))
    // ciSignature
    //
    // This class represents the signature of a method.
    class ciSignature : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ciMethod オブジェクトの _signature フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciMethod::ciMethod(methodHandle h_m)
* ciMethod::ciMethod(ciInstanceKlass* holder, ciSymbol* name, ciSymbol* signature)




### 詳細(Details)
See: [here](../doxygen/classciSignature.html) for details

---
