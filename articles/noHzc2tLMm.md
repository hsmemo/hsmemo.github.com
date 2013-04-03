---
layout: default
title: ciMethodBlocks クラス関連のクラス (ciMethodBlocks, ciBlock)
---
[Top](../index.html)

#### ciMethodBlocks クラス関連のクラス (ciMethodBlocks, ciBlock)



### クラス一覧(class list)

  * [ciMethodBlocks](#no58FP6jNg)
  * [ciBlock](#noou1PH4U4)


---
## <a name="no58FP6jNg" id="no58FP6jNg">ciMethodBlocks</a>

### 概要(Summary)
ciMethod クラス用の補助クラス.

そのメソッド内の基本ブロック(Basic Block)情報を表す.


```
    ((cite: hotspot/src/share/vm/ci/ciMethodBlocks.hpp))
    class ciMethodBlocks : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ciMethod オブジェクトの _method_blocks フィールドに(のみ)格納されている.

(ただし, ciMethodBlocks オブジェクトの生成自体は実際に必要になるまで遅延されている)

#### 生成箇所(where its instances are created)
ciMethod::get_method_blocks() 内で(のみ)生成されている (= 初めて使用される時まで生成が遅延されている).

### 内部構造(Internal structure)
基本ブロック情報の構築は, コンストラクタ内で行われている.

実際の基本ブロック情報は ciBlock オブジェクト内に格納されている.




### 詳細(Details)
See: [here](../doxygen/classciMethodBlocks.html) for details

---
## <a name="noou1PH4U4" id="noou1PH4U4">ciBlock</a>

### 概要(Summary)
ciMethodBlocks クラス用の補助クラス.

実際の基本ブロック情報を表すクラス. 
1つの ciBlock オブジェクトが 1つの基本ブロックに対応する.


```
    ((cite: hotspot/src/share/vm/ci/ciMethodBlocks.hpp))
    class ciBlock : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ciMethodBlocks オブジェクトの _blocks フィールドに(のみ)格納されている.

(正確には, このフィールドは ciBlock の GrowableArray を格納するフィールド.
この中に, その ciMethodBlocks 用の全ての ciBlock オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciMethodBlocks::ciMethodBlocks()
* ciMethodBlocks::split_block_at()
* ciMethodBlocks::make_block_at()
* ciMethodBlocks::make_dummy_block()




### 詳細(Details)
See: [here](../doxygen/classciBlock.html) for details

---
