---
layout: default
title: StackValueCollection クラス 
---
[Top](../index.html)

#### StackValueCollection クラス 



---
## <a name="noHXIdpwCZ" id="noHXIdpwCZ">StackValueCollection</a>

### 概要(Summary)
vframe クラス関連のクラス (具体的には interpretedVFrame, compiledVFrame, vframeArrayElement) 用の補助クラス.

スタックフレーム上の値を参照する処理で使用される一時オブジェクト(ResourceObjクラス).
スタックフレーム上にある値の集合を表す.

1つの StackValueCollection オブジェクトは以下のどちらかに対応する.

* 1つのスタックフレーム上の局所変数領域にある値全体 (全ての引数および全ての局所変数)
* 1つのスタックフレーム上のオペランドスタック内にある値全体


```
    ((cite: hotspot/src/share/vm/runtime/stackValueCollection.hpp))
    class StackValueCollection : public ResourceObj {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ただし, ResourceObjクラスなので一時的なオブジェクト).

```
* 脱最適化処理 (Deoptimization 処理)

  * vframeArrayElement::fill_in()

* interpretedVFrame/compiledVFrame 関係の処理 (#TODO)

  * interpretedVFrame::locals()
  * interpretedVFrame::expressions()
  * compiledVFrame::locals()
  * compiledVFrame::expressions()
```

### 内部構造(Internal structure)
実際の値の情報は StackValue 内に格納されている.




### 詳細(Details)
See: [here](../doxygen/classStackValueCollection.html) for details

---
