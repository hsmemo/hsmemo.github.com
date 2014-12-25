---
layout: default
title: Verifier クラス関連のクラス (Verifier, ClassVerifier)
---
[Top](../index.html)

#### Verifier クラス関連のクラス (Verifier, ClassVerifier)

これらは, クラスファイルの verify 処理用のクラス (See: [here](no7882amm.html) for details).


### クラス一覧(class list)

  * [Verifier](#noBQgrounT)
  * [ClassVerifier](#nouz0dFUPJ)


---
## <a name="noBQgrounT" id="noBQgrounT">Verifier</a>

### 概要(Summary)
クラスファイルの verify 処理に関する機能を納めた名前空間(AllStatic クラス). (See: [here](no7882amm.html) for details)


```cpp
    ((cite: hotspot/src/share/vm/classfile/verifier.hpp))
    // The verifier class
    class Verifier : AllStatic {
```

### 内部構造(Internal structure)
このクラスの Verifier::verify() メソッドが verify 処理のエントリポイントになっている.




### 詳細(Details)
See: [here](../doxygen/classVerifier.html) for details

---
## <a name="nouz0dFUPJ" id="nouz0dFUPJ">ClassVerifier</a>

### 概要(Summary)
Verifier クラス内で使用される補助クラス.

type checking verifier (= Java SE 6 以降の新しい verifier. StackMapTable attribute を利用する) 
の処理で使用される一時オブジェクト(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/classfile/verifier.hpp))
    // A new instance of this class is created for each class being verified
    class ClassVerifier : public StackObj {
```

### 使われ方(Usage)
Verifier::verify() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classClassVerifier.html) for details

---
