---
layout: default
title: 定数(CONstant), 条件付き転送(CONditional move), 型変換(CONvert)に関する高レベル中間語(Ideal)クラス (ConNode, ConINode, ConPNode, ConNNode, ConLNode, ConFNode, ConDNode, BinaryNode, CMoveNode, CMoveDNode, CMoveFNode, CMoveINode, CMoveLNode, CMovePNode, CMoveNNode, ConstraintCastNode, CastIINode, CastPPNode, CheckCastPPNode, EncodePNode, DecodeNNode, Conv2BNode, ConvD2FNode, ConvD2INode, ConvD2LNode, ConvF2DNode, ConvF2INode, ConvF2LNode, ConvI2DNode, ConvI2FNode, ConvI2LNode, ConvL2DNode, ConvL2FNode, ConvL2INode, CastX2PNode, CastP2XNode, MemMoveNode, ThreadLocalNode, LoadReturnPCNode, RoundFloatNode, RoundDoubleNode, Opaque1Node, Opaque2Node, PartialSubtypeCheckNode, MoveI2FNode, MoveL2DNode, MoveF2INode, MoveD2LNode, CountBitsNode, CountLeadingZerosINode, CountLeadingZerosLNode, CountTrailingZerosINode, CountTrailingZerosLNode, PopCountINode, PopCountLNode)
---
[Top](../index.html)

#### 定数(CONstant), 条件付き転送(CONditional move), 型変換(CONvert)に関する高レベル中間語(Ideal)クラス (ConNode, ConINode, ConPNode, ConNNode, ConLNode, ConFNode, ConDNode, BinaryNode, CMoveNode, CMoveDNode, CMoveFNode, CMoveINode, CMoveLNode, CMovePNode, CMoveNNode, ConstraintCastNode, CastIINode, CastPPNode, CheckCastPPNode, EncodePNode, DecodeNNode, Conv2BNode, ConvD2FNode, ConvD2INode, ConvD2LNode, ConvF2DNode, ConvF2INode, ConvF2LNode, ConvI2DNode, ConvI2FNode, ConvI2LNode, ConvL2DNode, ConvL2FNode, ConvL2INode, CastX2PNode, CastP2XNode, MemMoveNode, ThreadLocalNode, LoadReturnPCNode, RoundFloatNode, RoundDoubleNode, Opaque1Node, Opaque2Node, PartialSubtypeCheckNode, MoveI2FNode, MoveL2DNode, MoveF2INode, MoveD2LNode, CountBitsNode, CountLeadingZerosINode, CountLeadingZerosLNode, CountTrailingZerosINode, CountTrailingZerosLNode, PopCountINode, PopCountLNode)

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, 定数値, 条件付き転送, 型変換, 等を表すクラス.


### クラス一覧(class list)

  * [ConNode](#noAE-VpV83)
  * [ConINode](#nokpgHl5Rn)
  * [ConPNode](#no6X4ZYCCu)
  * [ConNNode](#nox4Oriyq_)
  * [ConLNode](#no2veWrk90)
  * [ConFNode](#nozC4iqdOF)
  * [ConDNode](#noTTjqfcRo)
  * [CMoveNode](#nootG922Eg)
  * [CMoveDNode](#nocaU-vl9u)
  * [CMoveFNode](#noNN4NX3nX)
  * [CMoveINode](#nod1j2jN-k)
  * [CMoveLNode](#noDBYpkqL_)
  * [CMovePNode](#noQBtq8xIl)
  * [CMoveNNode](#nojoc3kk7Y)
  * [BinaryNode](#nouERuTyR3)
  * [ConstraintCastNode](#noaGiokyt8)
  * [CastIINode](#noJw3sTUKU)
  * [CastPPNode](#noVo698t7v)
  * [CheckCastPPNode](#noCU4S27Wf)
  * [EncodePNode](#no3LGW7TEk)
  * [DecodeNNode](#no5-IrkDzZ)
  * [Conv2BNode](#nodLi3yVDQ)
  * [ConvD2FNode](#noKO842L7Y)
  * [ConvD2INode](#noU_y9OQpG)
  * [ConvD2LNode](#noYpoMWwhc)
  * [ConvF2DNode](#noI4f67GEn)
  * [ConvF2INode](#noKcSZyROo)
  * [ConvF2LNode](#no-MiSbROz)
  * [ConvI2DNode](#nohxO3OuSQ)
  * [ConvI2FNode](#noAoFNDHfk)
  * [ConvI2LNode](#noQTrT7AjP)
  * [ConvL2DNode](#noLfYGf7jF)
  * [ConvL2FNode](#norFN5YISL)
  * [ConvL2INode](#noP7zcrC9M)
  * [CastX2PNode](#nooMX9hmqK)
  * [CastP2XNode](#nodIa08Xuy)
  * [MemMoveNode](#nojQhc4hHo)
  * [ThreadLocalNode](#noTiURCuOP)
  * [LoadReturnPCNode](#no2jRgTbt6)
  * [RoundFloatNode](#no1RErZt4U)
  * [RoundDoubleNode](#noJfnBSOuR)
  * [Opaque1Node](#noRivbXU3y)
  * [Opaque2Node](#nok977F7tP)
  * [PartialSubtypeCheckNode](#noHcX9a0BL)
  * [MoveI2FNode](#noG6jNDmMI)
  * [MoveL2DNode](#noJffb6jFh)
  * [MoveF2INode](#no2C9sxj3H)
  * [MoveD2LNode](#nouPP1wCTu)
  * [CountBitsNode](#nodnWi3vC5)
  * [CountLeadingZerosINode](#nohrygcPCE)
  * [CountLeadingZerosLNode](#norRE_hCm-)
  * [CountTrailingZerosINode](#noxR-_uXrE)
  * [CountTrailingZerosLNode](#no7FKt0yAm)
  * [PopCountINode](#no1VNnY0h4)
  * [PopCountLNode](#noznprNiVz)


---
## <a name="noAE-VpV83" id="noAE-VpV83">ConNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.
「定数値」を表す全ての Node クラスの基底クラス.

なお, このクラスは abstract class ではない.
(不定な値を表す際に使用されている. 例えば void や long/double の ２スロット目などに使われる).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConNode----------------------------------------
    // Simple constants
    class ConNode : public TypeNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ConNode::make()
* StartNode::match()
* Compile::Init()

### 内部構造(Internal structure)
入力ノードは control input のみ. そして control input は常に RootNode.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConNode( const Type *t ) : TypeNode(t,1) {
        init_req(0, (Node*)Compile::current()->root());
        init_flags(Flag_is_Con);
      }
```

(なお, コンストラクタで指定された定数値 (t引数) は TypeNode のコンストラクタに引き渡される)




### 詳細(Details)
See: [here](../doxygen/classConNode.html) for details

---
## <a name="nokpgHl5Rn" id="nokpgHl5Rn">ConINode</a>

### 概要(Summary)
ConNode クラスのサブクラスの1つ.
このクラスは int 型の定数用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConINode---------------------------------------
    // Simple integer constants
    class ConINode : public ConNode {
```

なお ConXNode という型も使われるが, 
`#ifdef _LP64` でない場合は, これは ConINode の別名.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define ConXNode     ConINode
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ConNode::make()
* ConINode::make()
* ModINode::Ideal()
* split_if()
* IfNode::Ideal()
* MemBarNode::Ideal()

### 内部構造(Internal structure)
入力ノードは control input のみ. そして control input は常に RootNode.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConINode( const TypeInt *t ) : ConNode(t) {}
```




### 詳細(Details)
See: [here](../doxygen/classConINode.html) for details

---
## <a name="no6X4ZYCCu" id="no6X4ZYCCu">ConPNode</a>

### 概要(Summary)
ConNode クラスのサブクラスの1つ.
このクラスはポインタ型の定数用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConPNode---------------------------------------
    // Simple pointer constants
    class ConPNode : public ConNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ConNode::make()
* ConPNode::make()

### 内部構造(Internal structure)
入力ノードは control input のみ. そして control input は常に RootNode.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConPNode( const TypePtr *t ) : ConNode(t) {}
```




### 詳細(Details)
See: [here](../doxygen/classConPNode.html) for details

---
## <a name="nox4Oriyq_" id="nox4Oriyq_">ConNNode</a>

### 概要(Summary)
ConNode クラスのサブクラスの1つ.
このクラスは narrow oop 型の定数用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConNNode--------------------------------------
    // Simple narrow oop constants
    class ConNNode : public ConNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ConNode::make()

### 内部構造(Internal structure)
入力ノードは control input のみ. そして control input は常に RootNode.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConNNode( const TypeNarrowOop *t ) : ConNode(t) {}
```




### 詳細(Details)
See: [here](../doxygen/classConNNode.html) for details

---
## <a name="no2veWrk90" id="no2veWrk90">ConLNode</a>

### 概要(Summary)
ConNode クラスのサブクラスの1つ.
このクラスは long 型の定数用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConLNode---------------------------------------
    // Simple long constants
    class ConLNode : public ConNode {
```

なお ConXNode という型も使われるが, 
`#ifdef _LP64` の場合は, これは ConLNode の別名.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define ConXNode     ConLNode
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ConNode::make()
* ConLNode::make()
* ModLNode::Ideal()

### 内部構造(Internal structure)
入力ノードは control input のみ. そして control input は常に RootNode.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConLNode( const TypeLong *t ) : ConNode(t) {}
```




### 詳細(Details)
See: [here](../doxygen/classConLNode.html) for details

---
## <a name="nozC4iqdOF" id="nozC4iqdOF">ConFNode</a>

### 概要(Summary)
ConNode クラスのサブクラスの1つ.
このクラスは float 型の定数用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConFNode---------------------------------------
    // Simple float constants
    class ConFNode : public ConNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ConNode::make()
* ConFNode::make()

### 内部構造(Internal structure)
入力ノードは control input のみ. そして control input は常に RootNode.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConFNode( const TypeF *t ) : ConNode(t) {}
```




### 詳細(Details)
See: [here](../doxygen/classConFNode.html) for details

---
## <a name="noTTjqfcRo" id="noTTjqfcRo">ConDNode</a>

### 概要(Summary)
ConNode クラスのサブクラスの1つ.
このクラスは double 型の定数用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConDNode---------------------------------------
    // Simple double constants
    class ConDNode : public ConNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ConNode::make()
* ConDNode::make()

### 内部構造(Internal structure)
入力ノードは control input のみ. そして control input は常に RootNode.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConDNode( const TypeD *t ) : ConNode(t) {}
```




### 詳細(Details)
See: [here](../doxygen/classConDNode.html) for details

---
## <a name="nootG922Eg" id="nootG922Eg">CMoveNode</a>

### 概要(Summary)
TypeNode クラスのサブクラスの1つ.
全ての CMove*Node クラスの基底クラス
(なお CMove*Node は条件に応じて 2つの値のどちらかを出力する Node. 
 C 言語の3項演算子のようなもの (<= あるいは関数型言語の if 式のようなもの)).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CMoveNode--------------------------------------
    // Conditional move
    class CMoveNode : public TypeNode {
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 3つは以下の通り.

* 2番目の入力Node : どちらの値を選ぶかを示す条件
* 3番目の入力Node : 条件が true の場合に出力する Node
* 4番目の入力Node : 条件が false の場合に出力する Node


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      enum { Control,               // When is it safe to do this cmove?
             Condition,             // Condition controlling the cmove
             IfFalse,               // Value if condition is false
             IfTrue };              // Value if condition is true
```


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CMoveNode( Node *bol, Node *left, Node *right, const Type *t ) : TypeNode(t,4)
      {
        init_class_id(Class_CMove);
        // all inputs are nullified in Node::Node(int)
        // init_req(Control,NULL);
        init_req(Condition,bol);
        init_req(IfFalse,left);
        init_req(IfTrue,right);
      }
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classCMoveNode.html) for details

---
## <a name="nocaU-vl9u" id="nocaU-vl9u">CMoveDNode</a>

### 概要(Summary)
CMoveNode クラスの具象サブクラスの1つ.
このクラスは 2つの double 値から 1つを選ぶ場合用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CMoveDNode-------------------------------------
    class CMoveDNode : public CMoveNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
CMoveNode::make() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 3つは以下の通り.

* 2番目の入力Node : どちらの値を選ぶかを示す条件
* 3番目の入力Node : 条件が true の場合に出力する Node
* 4番目の入力Node : 条件が false の場合に出力する Node


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CMoveDNode( Node *bol, Node *left, Node *right, const Type* t) : CMoveNode(bol,left,right,t){}
```




### 詳細(Details)
See: [here](../doxygen/classCMoveDNode.html) for details

---
## <a name="noNN4NX3nX" id="noNN4NX3nX">CMoveFNode</a>

### 概要(Summary)
CMoveNode クラスの具象サブクラスの1つ.
このクラスは 2つの float 値から 1つを選ぶ場合用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CMoveFNode-------------------------------------
    class CMoveFNode : public CMoveNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
CMoveNode::make() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 3つは以下の通り.

* 2番目の入力Node : どちらの値を選ぶかを示す条件
* 3番目の入力Node : 条件が true の場合に出力する Node
* 4番目の入力Node : 条件が false の場合に出力する Node


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CMoveFNode( Node *bol, Node *left, Node *right, const Type* t ) : CMoveNode(bol,left,right,t) {}
```




### 詳細(Details)
See: [here](../doxygen/classCMoveFNode.html) for details

---
## <a name="nod1j2jN-k" id="nod1j2jN-k">CMoveINode</a>

### 概要(Summary)
CMoveNode クラスの具象サブクラスの1つ.
このクラスは 2つの int 値から 1つを選ぶ場合用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CMoveINode-------------------------------------
    class CMoveINode : public CMoveNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* CMoveNode::make()
* ModINode::Ideal()
* PhaseIdealLoop::do_unroll()

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 3つは以下の通り.

* 2番目の入力Node : どちらの値を選ぶかを示す条件
* 3番目の入力Node : 条件が true の場合に出力する Node
* 4番目の入力Node : 条件が false の場合に出力する Node


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CMoveINode( Node *bol, Node *left, Node *right, const TypeInt *ti ) : CMoveNode(bol,left,right,ti){}
```




### 詳細(Details)
See: [here](../doxygen/classCMoveINode.html) for details

---
## <a name="noDBYpkqL_" id="noDBYpkqL_">CMoveLNode</a>

### 概要(Summary)
CMoveNode クラスの具象サブクラスの1つ.
このクラスは 2つの long 値から 1つを選ぶ場合用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CMoveLNode-------------------------------------
    class CMoveLNode : public CMoveNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* CMoveNode::make()
* ModLNode::Ideal()

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 3つは以下の通り.

* 2番目の入力Node : どちらの値を選ぶかを示す条件
* 3番目の入力Node : 条件が true の場合に出力する Node
* 4番目の入力Node : 条件が false の場合に出力する Node


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CMoveLNode(Node *bol, Node *left, Node *right, const TypeLong *tl ) : CMoveNode(bol,left,right,tl){}
```




### 詳細(Details)
See: [here](../doxygen/classCMoveLNode.html) for details

---
## <a name="noQBtq8xIl" id="noQBtq8xIl">CMovePNode</a>

### 概要(Summary)
CMoveNode クラスの具象サブクラスの1つ.
このクラスは 2つのポインタ値から 1つを選ぶ場合用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CMovePNode-------------------------------------
    class CMovePNode : public CMoveNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
CMoveNode::make() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り

* 1番目の入力Node : control input
* 2番目の入力Node : どちらの値を選ぶかを示す条件
* 3番目の入力Node : 条件が true の場合に出力する Node
* 4番目の入力Node : 条件が false の場合に出力する Node


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CMovePNode( Node *c, Node *bol, Node *left, Node *right, const TypePtr* t ) : CMoveNode(bol,left,right,t) { init_req(Control,c); }
```




### 詳細(Details)
See: [here](../doxygen/classCMovePNode.html) for details

---
## <a name="nojoc3kk7Y" id="nojoc3kk7Y">CMoveNNode</a>

### 概要(Summary)
CMoveNode クラスの具象サブクラスの1つ.
このクラスは 2つの narrow oop 値から 1つを選ぶ場合用.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CMoveNNode-------------------------------------
    class CMoveNNode : public CMoveNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
CMoveNode::make() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り

* 1番目の入力Node : control input
* 2番目の入力Node : どちらの値を選ぶかを示す条件
* 3番目の入力Node : 条件が true の場合に出力する Node
* 4番目の入力Node : 条件が false の場合に出力する Node


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CMoveNNode( Node *c, Node *bol, Node *left, Node *right, const Type* t ) : CMoveNode(bol,left,right,t) { init_req(Control,c); }
```




### 詳細(Details)
See: [here](../doxygen/classCMoveNNode.html) for details

---
## <a name="nouERuTyR3" id="nouERuTyR3">BinaryNode</a>

### 概要(Summary)
CMoveNode クラス(及びそのサブクラス)を補佐する Node クラス.
低レベル中間語で使用される.

CMoveNode は 4つの入力を必要とするが, Matcher は二分木を要求する.
そこで BinaryNode を使って "(CMove (Binary bol cmp) (Binary src1 src2))" という二分木として表現する.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------BinaryNode-------------------------------------
    // Place holder for the 2 conditional inputs to a CMove.  CMove needs 4
    // inputs: the Bool (for the lt/gt/eq/ne bits), the flags (result of some
    // compare), and the 2 values to select between.  The Matcher requires a
    // binary tree so we break it down like this:
    //     (CMove (Binary bol cmp) (Binary src1 src2))
    class BinaryNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
Matcher::find_shared() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      BinaryNode( Node *n1, Node *n2 ) : Node(0,n1,n2) { }
```




### 詳細(Details)
See: [here](../doxygen/classBinaryNode.html) for details

---
## <a name="noaGiokyt8" id="noaGiokyt8">ConstraintCastNode</a>

### 概要(Summary)
TypeNode クラスの具象サブクラスの1つ.

Ideal の型情報を扱うための便宜的な Node クラスの基底クラス(?).
型に含まれている「取り得る値の範囲(value range)情報」を修正したい場合に使用される模様.

(C2 内で使用される型情報(Type クラス)は, 型だけでなく取り得る値の範囲も保持している (See: Type).
この取り得る値の範囲が変わる場合に ConstraintCastNode (のサブクラス) で表現される模様.
例えば, 「Null の可能性があるポインタ型」は Null チェックが終わった後は「Null の可能性がないポインタ型」にキャストされる, 等)

(<= 実際問題 AD ファイルでは何も出力されない(enc が空になっている)ので, 実質的な処理には対応していない模様)

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConstraintCastNode-----------------------------
    // cast to a different range
    class ConstraintCastNode: public TypeNode {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConstraintCastNode (Node *n, const Type *t ): TypeNode(t,2) {
        init_class_id(Class_ConstraintCast);
        init_req(1, n);
      }
```




### 詳細(Details)
See: [here](../doxygen/classConstraintCastNode.html) for details

---
## <a name="noJw3sTUKU" id="noJw3sTUKU">CastIINode</a>

### 概要(Summary)
ConstraintCastNode クラスの具象サブクラスの1つ.

このクラスは, 整数型の取り得る値の範囲が変わった場合用 (? #TODO).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CastIINode-------------------------------------
    // cast integer to integer (different range)
    class CastIINode: public ConstraintCastNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* Parse::adjust_map_after_if()
* AllocateArrayNode::make_ideal_length()
* LibraryCallKit::generate_negative_guard()
* LibraryCallKit::generate_nonpositive_guard()

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CastIINode (Node *n, const Type *t ): ConstraintCastNode(n,t) {}
```




### 詳細(Details)
See: [here](../doxygen/classCastIINode.html) for details

---
## <a name="noVo698t7v" id="noVo698t7v">CastPPNode</a>

### 概要(Summary)
ConstraintCastNode クラスの具象サブクラスの1つ.

このクラスは, ポインタ型の取り得る値の範囲が変わった場合用 (? #TODO).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CastPPNode-------------------------------------
    // cast pointer to pointer (different type)
    class CastPPNode: public ConstraintCastNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* Parse::adjust_map_after_if()
* GraphKit::cast_not_null()
* LibraryCallKit::inline_array_copyOf()

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CastPPNode (Node *n, const Type *t ): ConstraintCastNode(n, t) {}
```




### 詳細(Details)
See: [here](../doxygen/classCastPPNode.html) for details

---
## <a name="noCU4S27Wf" id="noCU4S27Wf">CheckCastPPNode</a>

### 概要(Summary)
TypeNode クラスの具象サブクラスの1つ.
オブジェクトの型を検査する処理 (checkcast 相当の処理) を表す Node.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CheckCastPPNode--------------------------------
    // for _checkcast, cast pointer to pointer (different type), without JOIN,
    class CheckCastPPNode: public TypeNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* GraphKit::type_check_receiver()
* GraphKit::gen_checkcast()
* GraphKit::set_output_for_allocation()
* Parse::catch_inline_exceptions()
* Parse::return_current()
* Parse::do_multianewarray()
* LibraryCallKit::inline_string_equals()

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CheckCastPPNode( Node *c, Node *n, const Type *t ) : TypeNode(t,2) {
        init_class_id(Class_CheckCastPP);
        init_req(0, c);
        init_req(1, n);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCheckCastPPNode.html) for details

---
## <a name="no3LGW7TEk" id="no3LGW7TEk">EncodePNode</a>

### 概要(Summary)
UseCompressedOops 機能のための Node.
ポインタを narrow oop に変換する処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------EncodeP--------------------------------
    // Encodes an oop pointers into its compressed form
    // Takes an extra argument which is the real heap base as a long which
    // may be useful for code generation in the backend.
    class EncodePNode : public TypeNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* StoreNode::make()
* LoadNKlassNode::Identity()
* PhiNode::Ideal()
* LibraryCallKit::inline_unsafe_CAS()

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      EncodePNode(Node* value, const Type* type):
        TypeNode(type, 2) {
        init_class_id(Class_EncodeP);
        init_req(0, NULL);
        init_req(1, value);
      }
```




### 詳細(Details)
See: [here](../doxygen/classEncodePNode.html) for details

---
## <a name="no5-IrkDzZ" id="no5-IrkDzZ">DecodeNNode</a>

### 概要(Summary)
UseCompressedOops 機能のための Node.
narrow oop をポインタに戻す処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------DecodeN--------------------------------
    // Converts a narrow oop into a real oop ptr.
    // Takes an extra argument which is the real heap base as a long which
    // may be useful for code generation in the backend.
    class DecodeNNode : public TypeNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* LoadNode::make()
* LoadKlassNode::make()
* PhaseMacroExpand::scalar_replacement()
* PhiNode::Ideal()
* final_graph_reshaping_impl()

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      DecodeNNode(Node* value, const Type* type):
        TypeNode(type, 2) {
        init_class_id(Class_DecodeN);
        init_req(0, NULL);
        init_req(1, value);
      }
```




### 詳細(Details)
See: [here](../doxygen/classDecodeNNode.html) for details

---
## <a name="nodLi3yVDQ" id="nodLi3yVDQ">Conv2BNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
int やポインタ値から boolean への型変換処理を表す (0 なら 0, 非0 なら 1 になる).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------Conv2BNode-------------------------------------
    // Convert int/pointer to a Boolean.  Map zero to zero, all else to 1.
    class Conv2BNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      Conv2BNode( Node *i ) : Node(0,i) {}
```




### 詳細(Details)
See: [here](../doxygen/classConv2BNode.html) for details

---
## <a name="noKO842L7Y" id="noKO842L7Y">ConvD2FNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
double 値から float への型変換処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvD2FNode------------------------------------
    // Convert double to float
    class ConvD2FNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvD2FNode( Node *in1 ) : Node(0,in1) {}
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvD2FNode.html) for details

---
## <a name="noU_y9OQpG" id="noU_y9OQpG">ConvD2INode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
double 値から int への型変換処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvD2INode------------------------------------
    // Convert Double to Integer
    class ConvD2INode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvD2INode( Node *in1 ) : Node(0,in1) {}
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvD2INode.html) for details

---
## <a name="noYpoMWwhc" id="noYpoMWwhc">ConvD2LNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
double 値から lont への型変換処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvD2LNode------------------------------------
    // Convert Double to Long
    class ConvD2LNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvD2LNode( Node *dbl ) : Node(0,dbl) {}
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvD2LNode.html) for details

---
## <a name="noI4f67GEn" id="noI4f67GEn">ConvF2DNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
float 値から double への型変換処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvF2DNode------------------------------------
    // Convert Float to a Double.
    class ConvF2DNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvF2DNode( Node *in1 ) : Node(0,in1) {}
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvF2DNode.html) for details

---
## <a name="noKcSZyROo" id="noKcSZyROo">ConvF2INode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
float 値から int への型変換処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvF2INode------------------------------------
    // Convert float to integer
    class ConvF2INode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvF2INode( Node *in1 ) : Node(0,in1) {}
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvF2INode.html) for details

---
## <a name="no-MiSbROz" id="no-MiSbROz">ConvF2LNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
float 値から lont への型変換処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvF2LNode------------------------------------
    // Convert float to long
    class ConvF2LNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvF2LNode( Node *in1 ) : Node(0,in1) {}
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvF2LNode.html) for details

---
## <a name="nohxO3OuSQ" id="nohxO3OuSQ">ConvI2DNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
int 値から double への型変換処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvI2DNode------------------------------------
    // Convert Integer to Double
    class ConvI2DNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvI2DNode( Node *in1 ) : Node(0,in1) {}
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvI2DNode.html) for details

---
## <a name="noAoFNDHfk" id="noAoFNDHfk">ConvI2FNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
int 値から float への型変換処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvI2FNode------------------------------------
    // Convert Integer to Float
    class ConvI2FNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvI2FNode( Node *in1 ) : Node(0,in1) {}
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvI2FNode.html) for details

---
## <a name="noQTrT7AjP" id="noQTrT7AjP">ConvI2LNode</a>

### 概要(Summary)
TypeNode クラスの具象サブクラスの1つ.
int 値から long への型変換処理を表す.

(他の Conv*Node は Node のサブクラスなのにこいつは TypeNode のサブクラス.
 中身が int 範囲の (上 32bit は 0 の) long 値だと分かっていれば最適化できることがある?? #TODO)


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvI2LNode------------------------------------
    // Convert integer to long
    class ConvI2LNode : public TypeNode {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvI2LNode(Node *in1, const TypeLong* t = TypeLong::INT)
        : TypeNode(t, 2)
      { init_req(1, in1); }
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvI2LNode.html) for details

---
## <a name="noLfYGf7jF" id="noLfYGf7jF">ConvL2DNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
long 値から double への型変換処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvL2DNode------------------------------------
    // Convert Long to Double
    class ConvL2DNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvL2DNode( Node *in1 ) : Node(0,in1) {}
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvL2DNode.html) for details

---
## <a name="norFN5YISL" id="norFN5YISL">ConvL2FNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
long 値から float への型変換処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvL2FNode------------------------------------
    // Convert Long to Float
    class ConvL2FNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvL2FNode( Node *in1 ) : Node(0,in1) {}
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvL2FNode.html) for details

---
## <a name="noP7zcrC9M" id="noP7zcrC9M">ConvL2INode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
long 値から int への型変換処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ConvL2INode------------------------------------
    // Convert long to integer
    class ConvL2INode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ConvL2INode( Node *in1 ) : Node(0,in1) {}
```

### 備考(Notes)
なお, ファイル中では Conv*Node はアルファベット順に定義しているとのこと.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    // The conversions operations are all Alpha sorted.  Please keep it that way!
```




### 詳細(Details)
See: [here](../doxygen/classConvL2INode.html) for details

---
## <a name="nooMX9hmqK" id="nooMX9hmqK">CastX2PNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
ポインタと同ビット幅の整数値(int/long)からポインタへのキャスト処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CastX2PNode-------------------------------------
    // convert a machine-pointer-sized integer to a raw pointer
    class CastX2PNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CastX2PNode( Node *n ) : Node(NULL, n) {}
```




### 詳細(Details)
See: [here](../doxygen/classCastX2PNode.html) for details

---
## <a name="nodIa08Xuy" id="nodIa08Xuy">CastP2XNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
ポインタからポインタと同ビット幅の整数値(int/long)へのキャスト処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------CastP2XNode-------------------------------------
    // Used in both 32-bit and 64-bit land.
    // Used for card-marks and unsafe pointer math.
    class CastP2XNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
なお, コンストラクタで指定することにより 0 以外の control input を設定できる.

(何で制御依存が必要なのかよく分からない. 実質何もしないノード(やるとしてもレジスタ間コピー程度のノード)に見えるが... #TODO)


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CastP2XNode( Node *ctrl, Node *n ) : Node(ctrl, n) {}
```




### 詳細(Details)
See: [here](../doxygen/classCastP2XNode.html) for details

---
## <a name="nojQhc4hHo" id="nojQhc4hHo">MemMoveNode</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

(メモリ間コピーを表す Node のようだが... #TODO)


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------MemMoveNode------------------------------------
    // Memory to memory move.  Inserted very late, after allocation.
    class MemMoveNode : public Node {
```

### 内部構造(Internal structure)

```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      MemMoveNode( Node *dst, Node *src ) : Node(0,dst,src) {}
```




### 詳細(Details)
See: [here](../doxygen/classMemMoveNode.html) for details

---
## <a name="noTiURCuOP" id="noTiURCuOP">ThreadLocalNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
カレントスレッドに対応する ThreadLocalStorage オブジェクトのアドレスを表す (See: ThreadLocalStorage).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------ThreadLocalNode--------------------------------
    // Ideal Node which returns the base of ThreadLocalStorage.
    class ThreadLocalNode : public Node {
```

### 内部構造(Internal structure)
入力ノードは control input のみ. そして control input は常に RootNode.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      ThreadLocalNode( ) : Node((Node*)Compile::current()->root()) {}
```




### 詳細(Details)
See: [here](../doxygen/classThreadLocalNode.html) for details

---
## <a name="no2jRgTbt6" id="no2jRgTbt6">LoadReturnPCNode</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------LoadReturnPCNode-------------------------------
    class LoadReturnPCNode: public Node {
```

### 内部構造(Internal structure)

```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      LoadReturnPCNode(Node *c) : Node(c) { }
```




### 詳細(Details)
See: [here](../doxygen/classLoadReturnPCNode.html) for details

---
## <a name="no1RErZt4U" id="no1RErZt4U">RoundFloatNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
float 値を IEEE 754 で規定された 32bit のフォーマットに変換する処理を表す
(この Node は x87 のような独自フォーマットが使われている場合にのみ意味を持つ).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //-----------------------------RoundFloatNode----------------------------------
    class RoundFloatNode: public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
GraphKit::precision_rounding() 内で(のみ)生成されている.

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      RoundFloatNode(Node* c, Node *in1): Node(c, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classRoundFloatNode.html) for details

---
## <a name="noJfnBSOuR" id="noJfnBSOuR">RoundDoubleNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
double 値を IEEE 754 で規定された 64bit のフォーマットに変換する処理を表す
(この Node は x87 のような独自フォーマットが使われている場合にのみ意味を持つ).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //-----------------------------RoundDoubleNode---------------------------------
    class RoundDoubleNode: public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* GraphKit::dprecision_rounding()
* GraphKit::dstore_rounding()
* LibraryCallKit::pop_math_arg()

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      RoundDoubleNode(Node* c, Node *in1): Node(c, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classRoundDoubleNode.html) for details

---
## <a name="noRivbXU3y" id="noRivbXU3y">Opaque1Node</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
最適化されたくない箇所を最適化から保護するための Node (?? #TODO).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------Opaque1Node------------------------------------
    // A node to prevent unwanted optimizations.  Allows constant folding.
    // Stops value-numbering, Ideal calls or Identity functions.
    class Opaque1Node : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* GraphKit::add_predicate_impl()
* PhaseIdealLoop::clone_predicate()
* PhaseIdealLoop::insert_pre_post_loops()
* PhaseIdealLoop::create_slow_version_of_loop()

### 内部構造(Internal structure)
(control input も含めて) 2つまたは3つの入力ノードを持つ. ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      Opaque1Node( Compile* C, Node *n ) : Node(0,n) {
        // Put it on the Macro nodes list to removed during macro nodes expansion.
        init_flags(Flag_is_macro);
        C->add_macro_node(this);
      }
      // Special version for the pre-loop to hold the original loop limit
      // which is consumed by range check elimination.
      Opaque1Node( Compile* C, Node *n, Node* orig_limit ) : Node(0,n,orig_limit) {
        // Put it on the Macro nodes list to removed during macro nodes expansion.
        init_flags(Flag_is_macro);
        C->add_macro_node(this);
      }
```




### 詳細(Details)
See: [here](../doxygen/classOpaque1Node.html) for details

---
## <a name="nok977F7tP" id="nok977F7tP">Opaque2Node</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
最適化されたくない箇所を最適化から保護するための Node (?? #TODO).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //------------------------------Opaque2Node------------------------------------
    // A node to prevent unwanted optimizations.  Allows constant folding.  Stops
    // value-numbering, most Ideal calls or Identity functions.  This Node is
    // specifically designed to prevent the pre-increment value of a loop trip
    // counter from being live out of the bottom of the loop (hence causing the
    // pre- and post-increment values both being live and thus requiring an extra
    // temp register and an extra move).  If we "accidentally" optimize through
    // this kind of a Node, we'll get slightly pessimal, but correct, code.  Thus
    // it's OK to be slightly sloppy on optimizations here.
    class Opaque2Node : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* PhaseIdealLoop::do_unroll()
* PhaseIdealLoop::reorg_offsets()

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      Opaque2Node( Compile* C, Node *n ) : Node(0,n) {
        // Put it on the Macro nodes list to removed during macro nodes expansion.
        init_flags(Flag_is_macro);
        C->add_macro_node(this);
      }
```




### 詳細(Details)
See: [here](../doxygen/classOpaque2Node.html) for details

---
## <a name="noHcX9a0BL" id="noHcX9a0BL">PartialSubtypeCheckNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
2つのクラス(klassOop)がサブタイプ関係にあるかどうかを判定する処理を表す.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //----------------------PartialSubtypeCheckNode--------------------------------
    // The 2nd slow-half of a subtype check.  Scan the subklass's 2ndary superklass
    // array for an instance of the superklass.  Set a hidden internal cache on a
    // hit (cache is checked with exposed code in gen_subtype_check()).  Return
    // not zero for a miss or zero for a hit.
    class PartialSubtypeCheckNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
GraphKit::gen_subtype_check() 内で(のみ)生成されている.

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      PartialSubtypeCheckNode(Node* c, Node* sub, Node* super) : Node(c,sub,super) {}
```




### 詳細(Details)
See: [here](../doxygen/classPartialSubtypeCheckNode.html) for details

---
## <a name="noG6jNDmMI" id="noG6jNDmMI">MoveI2FNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

整数値から浮動小数値へのキャストを表す.
なお, この変換では値(ビットのレイアウト)は変わらず型が変わるのみ
(= java.lang.Float.intBitsToFloat() 用の Node).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    class MoveI2FNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_fp_conversions() 内で(のみ)生成されている.

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      MoveI2FNode( Node *value ) : Node(0,value) {}
```




### 詳細(Details)
See: [here](../doxygen/classMoveI2FNode.html) for details

---
## <a name="noJffb6jFh" id="noJffb6jFh">MoveL2DNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

整数値から浮動小数値へのキャストを表す.
なお, この変換では値(ビットのレイアウト)は変わらず型が変わるのみ
(= java.lang.Double.longBitsToDouble() 用の Node).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    class MoveL2DNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_fp_conversions() 内で(のみ)生成されている.

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      MoveL2DNode( Node *value ) : Node(0,value) {}
```




### 詳細(Details)
See: [here](../doxygen/classMoveL2DNode.html) for details

---
## <a name="no2C9sxj3H" id="no2C9sxj3H">MoveF2INode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.

浮動小数値から整数値へのキャストを表す.
なお, この変換では値(ビットのレイアウト)は変わらず型が変わるのみ.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    class MoveF2INode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* LibraryCallKit::inline_fp_conversions()
  
  (java.lang.Float.floatToRawIntBits() の変換用)

* PhaseIdealLoop::intrinsify_fill()

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      MoveF2INode( Node *value ) : Node(0,value) {}
```




### 詳細(Details)
See: [here](../doxygen/classMoveF2INode.html) for details

---
## <a name="nouPP1wCTu" id="nouPP1wCTu">MoveD2LNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.

浮動小数値から整数値へのキャストを表す.
なお, この変換では値(ビットのレイアウト)は変わらず型が変わるのみ.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    class MoveD2LNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* LibraryCallKit::inline_fp_conversions()
  
  (java.lang.Float.floatToRawIntBits() の変換用)

* PhaseIdealLoop::intrinsify_fill()

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      MoveD2LNode( Node *value ) : Node(0,value) {}
```




### 詳細(Details)
See: [here](../doxygen/classMoveD2LNode.html) for details

---
## <a name="nodnWi3vC5" id="nodnWi3vC5">CountBitsNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス (の基底クラス).

Node クラスのサブクラスの1つ.
全ての CountLeadingZero*Node/CountTrailingZero*Node/PopCount*Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //---------- CountBitsNode -----------------------------------------------------
    class CountBitsNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CountBitsNode(Node* in1) : Node(0, in1) {}
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classCountBitsNode.html) for details

---
## <a name="nohrygcPCE" id="nohrygcPCE">CountLeadingZerosINode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

CountBitsNode クラスの具象サブクラスの1つ.
このクラスは int 値に対する count leading zero 演算用
(= java.lang.Integer.numberOfLeadingZeros() 用).

 Long.numberOfLeadingZeros(long)


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //---------- CountLeadingZerosINode --------------------------------------------
    // Count leading zeros (0-bit count starting from MSB) of an integer.
    class CountLeadingZerosINode : public CountBitsNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_numberOfLeadingZeros() 内で(のみ)生成されている.

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CountLeadingZerosINode(Node* in1) : CountBitsNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classCountLeadingZerosINode.html) for details

---
## <a name="norRE_hCm-" id="norRE_hCm-">CountLeadingZerosLNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

CountBitsNode クラスの具象サブクラスの1つ.
このクラスは long 値に対する count leading zero 演算用
(= java.lang.Long.numberOfLeadingZeros() 用).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //---------- CountLeadingZerosLNode --------------------------------------------
    // Count leading zeros (0-bit count starting from MSB) of a long.
    class CountLeadingZerosLNode : public CountBitsNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_numberOfLeadingZeros() 内で(のみ)生成されている.

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CountLeadingZerosLNode(Node* in1) : CountBitsNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classCountLeadingZerosLNode.html) for details

---
## <a name="noxR-_uXrE" id="noxR-_uXrE">CountTrailingZerosINode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

CountBitsNode クラスの具象サブクラスの1つ.
このクラスは int 値に対する count trailing zero 演算用
(= java.lang.Integer.numberOfTrailingZeros() 用).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //---------- CountTrailingZerosINode -------------------------------------------
    // Count trailing zeros (0-bit count starting from LSB) of an integer.
    class CountTrailingZerosINode : public CountBitsNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_numberOfTrailingZeros() 内で(のみ)生成されている.

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CountTrailingZerosINode(Node* in1) : CountBitsNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classCountTrailingZerosINode.html) for details

---
## <a name="no7FKt0yAm" id="no7FKt0yAm">CountTrailingZerosLNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

CountBitsNode クラスの具象サブクラスの1つ.
このクラスは long 値に対する count trailing zero 演算用
(= java.lang.Integer.numberOfTrailingZeros() 用).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //---------- CountTrailingZerosLNode -------------------------------------------
    // Count trailing zeros (0-bit count starting from LSB) of a long.
    class CountTrailingZerosLNode : public CountBitsNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_numberOfTrailingZeros() 内で(のみ)生成されている.

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      CountTrailingZerosLNode(Node* in1) : CountBitsNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classCountTrailingZerosLNode.html) for details

---
## <a name="no1VNnY0h4" id="no1VNnY0h4">PopCountINode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

CountBitsNode クラスの具象サブクラスの1つ.
このクラスは int 値に対する population count 演算用
(= java.lang.Integer.bitCount() 用).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //---------- PopCountINode -----------------------------------------------------
    // Population count (bit count) of an integer.
    class PopCountINode : public CountBitsNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_bitCount() 内で(のみ)生成されている.

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      PopCountINode(Node* in1) : CountBitsNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classPopCountINode.html) for details

---
## <a name="noznprNiVz" id="noznprNiVz">PopCountLNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

CountBitsNode クラスの具象サブクラスの1つ.
このクラスは long 値に対する population count 演算用
(= java.lang.Long.bitCount() 用).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
    //---------- PopCountLNode -----------------------------------------------------
    // Population count (bit count) of a long.
    class PopCountLNode : public CountBitsNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_bitCount() 内で(のみ)生成されている.

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/connode.hpp))
      PopCountLNode(Node* in1) : CountBitsNode(in1) {}
```





### 詳細(Details)
See: [here](../doxygen/classPopCountLNode.html) for details

---
