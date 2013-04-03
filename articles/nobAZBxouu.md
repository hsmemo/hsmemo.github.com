---
layout: default
title: 加算に関する高レベル中間語(Ideal)クラス (AddNode, AddINode, AddLNode, AddFNode, AddDNode, AddPNode, OrINode, OrLNode, XorINode, XorLNode, MaxNode, MaxINode, MinINode)
---
[Top](../index.html)

#### 加算に関する高レベル中間語(Ideal)クラス (AddNode, AddINode, AddLNode, AddFNode, AddDNode, AddPNode, OrINode, OrLNode, XorINode, XorLNode, MaxNode, MaxINode, MinINode)

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, 主に「加算」を表すクラス.

なお単なる数値の加算だけでなく, 環(ring)上の加算とみなせるものは全てここで扱う
(例: XOR 演算, Max 演算, など).



### クラス一覧(class list)

  * [AddNode](#nocPvZ-SVQ)
  * [AddINode](#noAsSJJEGl)
  * [AddLNode](#nodPiz0TsI)
  * [AddFNode](#noc-Xatvl6)
  * [AddDNode](#no6W2gz9ji)
  * [AddPNode](#no8Xn1bKgM)
  * [OrINode](#nomLqH00VJ)
  * [OrLNode](#noXrboAXNt)
  * [XorINode](#no1h-UjeGr)
  * [XorLNode](#no4xowqHrG)
  * [MaxNode](#no3W7G9NV7)
  * [MaxINode](#no9JosD53X)
  * [MinINode](#noCCdGVhJW)


---
## <a name="nocPvZ-SVQ" id="nocPvZ-SVQ">AddNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.
「加算」を表す全ての Node クラスの基底クラス.

(なお単なる数値の加算だけでなく, 環(ring)上の加算とみなせるものは全てこのクラスのサブクラスで扱う) 
(例: XOR 演算, Max 演算, など).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    //------------------------------AddNode----------------------------------------
    // Classic Add functionality.  This covers all the usual 'add' behaviors for
    // an algebraic ring.  Add-integer, add-float, add-double, and binary-or are
    // all inherited from this class.  The various identity values are supplied
    // by virtual functions.
    class AddNode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      AddNode( Node *in1, Node *in2 ) : Node(0,in1,in2) {
        init_class_id(Class_Add);
      }
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classAddNode.html) for details

---
## <a name="noAsSJJEGl" id="noAsSJJEGl">AddINode</a>

### 概要(Summary)
AddNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の加算用.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // Add 2 integers
    class AddINode : public AddNode {
```

なお AddXNode という型も使われるが, `#ifdef _LP64` でない場合は, これは AddINode の別名


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define AddXNode     AddINode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      AddINode( Node *in1, Node *in2 ) : AddNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classAddINode.html) for details

---
## <a name="nodPiz0TsI" id="nodPiz0TsI">AddLNode</a>

### 概要(Summary)
AddNode クラスの具象サブクラスの1つ.
このクラスは long 値同士の加算用.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // Add 2 longs
    class AddLNode : public AddNode {
```

なお AddXNode という型も使われるが, `#ifdef _LP64` の場合は, これは AddLNode の別名.   


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define AddXNode     AddLNode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      AddLNode( Node *in1, Node *in2 ) : AddNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classAddLNode.html) for details

---
## <a name="noc-Xatvl6" id="noc-Xatvl6">AddFNode</a>

### 概要(Summary)
AddNode クラスの具象サブクラスの1つ.
このクラスは float 値同士の加算用.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // Add 2 floats
    class AddFNode : public AddNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      AddFNode( Node *in1, Node *in2 ) : AddNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classAddFNode.html) for details

---
## <a name="no6W2gz9ji" id="no6W2gz9ji">AddDNode</a>

### 概要(Summary)
AddNode クラスの具象サブクラスの1つ.
このクラスは double 値同士の加算用.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // Add 2 doubles
    class AddDNode : public AddNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      AddDNode( Node *in1, Node *in2 ) : AddNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classAddDNode.html) for details

---
## <a name="no8Xn1bKgM" id="no8Xn1bKgM">AddPNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.
ポインタ演算 (ポインタ+int) を表す.

なおコメントによると, 
「厳密に言うとポインタ演算は交換則などを満たさないので加算じゃないが, 
普通の人は加算を連想するので分かりやすさのためにここで定義している」, 
とのこと.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // Add pointer plus integer to get pointer.  NOT commutative, really.
    // So not really an AddNode.  Lives here, because people associate it with
    // an add.
    class AddPNode : public Node {
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).

その他の 3つは以下の通り
(ポインタ演算なのでポインタと int 値だけで表現はできるが, GC のためにポインタの base oop も保有している).

* Base : ポインタ値の base アドレス
* Address : 演算の対象になるポインタ値 (Base から派生した値)
* Offset  : 演算の対象になる int 値


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      AddPNode( Node *base, Node *ptr, Node *off ) : Node(0,base,ptr,off) {
        init_class_id(Class_AddP);
      }
```


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      enum { Control,               // When is it safe to do this add?
             Base,                  // Base oop, for GC purposes
             Address,               // Actually address, derived from base
             Offset } ;             // Offset added to address
```




### 詳細(Details)
See: [here](../doxygen/classAddPNode.html) for details

---
## <a name="nomLqH00VJ" id="nomLqH00VJ">OrINode</a>

### 概要(Summary)
AddNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の論理和(ビットOR)演算用.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // Logically OR 2 integers.  Included with the ADD nodes because it inherits
    // all the behavior of addition on a ring.
    class OrINode : public AddNode {
```

なお OrXNode という型も使われるが, `#ifdef _LP64` でない場合は, これは OrINode の別名.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define OrXNode      OrINode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      OrINode( Node *in1, Node *in2 ) : AddNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classOrINode.html) for details

---
## <a name="noXrboAXNt" id="noXrboAXNt">OrLNode</a>

### 概要(Summary)
AddNode クラスの具象サブクラスの1つ.
このクラスは long 値同士の論理和(ビットOR)演算用.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // Logically OR 2 longs.  Included with the ADD nodes because it inherits
    // all the behavior of addition on a ring.
    class OrLNode : public AddNode {
```

なお OrXNode という型も使われるが, `#ifdef _LP64` の場合は, これは OrLNode の別名.   


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define OrXNode      OrLNode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      OrLNode( Node *in1, Node *in2 ) : AddNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classOrLNode.html) for details

---
## <a name="no1h-UjeGr" id="no1h-UjeGr">XorINode</a>

### 概要(Summary)
AddNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の排他的論理和(ビットXOR)演算用.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // XOR'ing 2 integers
    class XorINode : public AddNode {
```

なお XorXNode という型も使われるが, `#ifdef _LP64` でない場合は, これは XorINode の別名.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define XorXNode     XorINode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      XorINode( Node *in1, Node *in2 ) : AddNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classXorINode.html) for details

---
## <a name="no4xowqHrG" id="no4xowqHrG">XorLNode</a>

### 概要(Summary)
AddNode クラスの具象サブクラスの1つ.
このクラスは long 値同士の排他的論理和(ビットXOR)演算用.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // XOR'ing 2 longs
    class XorLNode : public AddNode {
```

なお XorXNode という型も使われるが, `#ifdef _LP64` の場合は, これは XorLNode の別名.   


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define XorXNode     XorLNode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      XorLNode( Node *in1, Node *in2 ) : AddNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classXorLNode.html) for details

---
## <a name="no3W7G9NV7" id="no3W7G9NV7">MaxNode</a>

### 概要(Summary)
AddNode クラスのサブクラスの1つ.

「Max 演算」および「Min 演算」を表す全ての Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // Max (or min) of 2 values.  Included with the ADD nodes because it inherits
    // all the behavior of addition on a ring.  Only new thing is that we allow
    // 2 equal inputs to be equal.
    class MaxNode : public AddNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      MaxNode( Node *in1, Node *in2 ) : AddNode(in1,in2) {}
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classMaxNode.html) for details

---
## <a name="no9JosD53X" id="no9JosD53X">MaxINode</a>

### 概要(Summary)
MaxNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の Max 演算用.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // Maximum of 2 integers.  Included with the ADD nodes because it inherits
    // all the behavior of addition on a ring.
    class MaxINode : public MaxNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      MaxINode( Node *in1, Node *in2 ) : MaxNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classMaxINode.html) for details

---
## <a name="noCCdGVhJW" id="noCCdGVhJW">MinINode</a>

### 概要(Summary)
MaxNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の Min 演算用.


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
    // MINimum of 2 integers.  Included with the ADD nodes because it inherits
    // all the behavior of addition on a ring.
    class MinINode : public MaxNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/addnode.hpp))
      MinINode( Node *in1, Node *in2 ) : MaxNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classMinINode.html) for details

---
