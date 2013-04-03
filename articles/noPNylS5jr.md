---
layout: default
title: SIMD 命令に関する高レベル中間語(Ideal)クラス (VectorNode, AddVBNode, AddVCNode, AddVSNode, AddVINode, AddVLNode, AddVFNode, AddVDNode, SubVBNode, SubVCNode, SubVSNode, SubVINode, SubVLNode, SubVFNode, SubVDNode, MulVFNode, MulVDNode, DivVFNode, DivVDNode, LShiftVBNode, LShiftVCNode, LShiftVSNode, LShiftVINode, URShiftVBNode, URShiftVCNode, URShiftVSNode, URShiftVINode, AndVNode, OrVNode, XorVNode, VectorLoadNode, Load16BNode, Load8BNode, Load4BNode, Load8CNode, Load4CNode, Load2CNode, Load8SNode, Load4SNode, Load2SNode, Load4INode, Load2INode, Load2LNode, Load4FNode, Load2FNode, Load2DNode, VectorStoreNode, Store16BNode, Store8BNode, Store4BNode, Store8CNode, Store4CNode, Store2CNode, Store4INode, Store2INode, Store2LNode, Store4FNode, Store2FNode, Store2DNode, Replicate16BNode, Replicate8BNode, Replicate4BNode, Replicate8CNode, Replicate4CNode, Replicate2CNode, Replicate8SNode, Replicate4SNode, Replicate2SNode, Replicate4INode, Replicate2INode, Replicate2LNode, Replicate4FNode, Replicate2FNode, Replicate2DNode, PackNode, PackBNode, PackCNode, PackSNode, PackINode, PackLNode, PackFNode, PackDNode, Pack2x1BNode, Pack2x2BNode, ExtractNode, ExtractBNode, ExtractCNode, ExtractSNode, ExtractINode, ExtractLNode, ExtractFNode, ExtractDNode)
---
[Top](../index.html)

#### SIMD 命令に関する高レベル中間語(Ideal)クラス (VectorNode, AddVBNode, AddVCNode, AddVSNode, AddVINode, AddVLNode, AddVFNode, AddVDNode, SubVBNode, SubVCNode, SubVSNode, SubVINode, SubVLNode, SubVFNode, SubVDNode, MulVFNode, MulVDNode, DivVFNode, DivVDNode, LShiftVBNode, LShiftVCNode, LShiftVSNode, LShiftVINode, URShiftVBNode, URShiftVCNode, URShiftVSNode, URShiftVINode, AndVNode, OrVNode, XorVNode, VectorLoadNode, Load16BNode, Load8BNode, Load4BNode, Load8CNode, Load4CNode, Load2CNode, Load8SNode, Load4SNode, Load2SNode, Load4INode, Load2INode, Load2LNode, Load4FNode, Load2FNode, Load2DNode, VectorStoreNode, Store16BNode, Store8BNode, Store4BNode, Store8CNode, Store4CNode, Store2CNode, Store4INode, Store2INode, Store2LNode, Store4FNode, Store2FNode, Store2DNode, Replicate16BNode, Replicate8BNode, Replicate4BNode, Replicate8CNode, Replicate4CNode, Replicate2CNode, Replicate8SNode, Replicate4SNode, Replicate2SNode, Replicate4INode, Replicate2INode, Replicate2LNode, Replicate4FNode, Replicate2FNode, Replicate2DNode, PackNode, PackBNode, PackCNode, PackSNode, PackINode, PackLNode, PackFNode, PackDNode, Pack2x1BNode, Pack2x2BNode, ExtractNode, ExtractBNode, ExtractCNode, ExtractSNode, ExtractINode, ExtractLNode, ExtractFNode, ExtractDNode)

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, 「SIMD 演算(ベクトル演算)」を表すクラス.

なお, これらのクラスは SuperWord クラスによって生成される (See: SuperWord).
(<= たまにそうじゃないところで生成されることもあるが, 
まぁこれは SuperWord クラスが生成したものを2分木状に整形しているだけなので... (See: final_graph_reshaping_impl())

(#TODO ところでこれらの Node は使われてるか?? Load/Store/Replicate 以外はマシン語が出そうな気がしないが... 何か勘違いしてる??)


### クラス一覧(class list)

  * [VectorNode](#nonCiAEau-)
  * [AddVBNode](#noUM8Ktx8i)
  * [AddVCNode](#noNLJnUXq9)
  * [AddVSNode](#noCsYkx8qN)
  * [AddVINode](#nokwywxOyL)
  * [AddVLNode](#noNzLWu09P)
  * [AddVFNode](#no4MC8bdyW)
  * [AddVDNode](#no4NVULsXc)
  * [SubVBNode](#no3S-q6748)
  * [SubVCNode](#nomMHmLwtz)
  * [SubVSNode](#noBPW3pSgv)
  * [SubVINode](#nox75C21VM)
  * [SubVLNode](#noqvnMxllI)
  * [SubVFNode](#noIk2UKvui)
  * [SubVDNode](#noxdndiM4P)
  * [MulVFNode](#noymzGry2Y)
  * [MulVDNode](#nobMAuBtM_)
  * [DivVFNode](#noZicQJU-B)
  * [DivVDNode](#noDuimMabR)
  * [LShiftVBNode](#noyDmm-s62)
  * [LShiftVCNode](#noQ7tYBNH0)
  * [LShiftVSNode](#noLzTIRHA5)
  * [LShiftVINode](#noCugK4jJa)
  * [URShiftVBNode](#noZFMB9jgC)
  * [URShiftVCNode](#nodmepQdrw)
  * [URShiftVSNode](#noL_XdAxX5)
  * [URShiftVINode](#noPL5Wu_nD)
  * [AndVNode](#noJtfZhhgk)
  * [OrVNode](#noEErLK6gZ)
  * [XorVNode](#nogmn3VcUE)
  * [VectorLoadNode](#noRhu8-mMG)
  * [Load16BNode](#noLVjP4jWx)
  * [Load8BNode](#no9TUQh35I)
  * [Load4BNode](#no0HjI5sOV)
  * [Load8CNode](#noJEv68nAZ)
  * [Load4CNode](#nod9SNg-21)
  * [Load2CNode](#nosPdX_I8P)
  * [Load8SNode](#noFkKw4w5C)
  * [Load4SNode](#noPujOFQcN)
  * [Load2SNode](#noIcqrv8yZ)
  * [Load4INode](#noiUvV2EDQ)
  * [Load2INode](#no0zrvz3r-)
  * [Load2LNode](#nophEGXQcm)
  * [Load4FNode](#no4A4oJE1T)
  * [Load2FNode](#nobtsnWAnD)
  * [Load2DNode](#nopQQgNJvU)
  * [VectorStoreNode](#no6K1tNPfi)
  * [Store16BNode](#no7DWriQdE)
  * [Store8BNode](#nowRMKup0z)
  * [Store4BNode](#no9vJ7BmZN)
  * [Store8CNode](#nox9fTTdVQ)
  * [Store4CNode](#nof4Gbysjz)
  * [Store2CNode](#noiN1aabrS)
  * [Store4INode](#no8t8OzDgk)
  * [Store2INode](#noqpXB2AX5)
  * [Store2LNode](#noPuzQm80G)
  * [Store4FNode](#nocKEkI488)
  * [Store2FNode](#no-qVQt5UX)
  * [Store2DNode](#noAr7mPBlh)
  * [Replicate16BNode](#nobR8cDW6Y)
  * [Replicate8BNode](#no8zIbd3a5)
  * [Replicate4BNode](#no2SEax67u)
  * [Replicate8CNode](#noeHTdrG2Z)
  * [Replicate4CNode](#noSUM_ydtI)
  * [Replicate2CNode](#no04pAHMsy)
  * [Replicate8SNode](#noIQdxc7jo)
  * [Replicate4SNode](#noyapMWe88)
  * [Replicate2SNode](#nobA28RQ-2)
  * [Replicate4INode](#norOwM9fAI)
  * [Replicate2INode](#noDxshFxQN)
  * [Replicate2LNode](#nog1GVjz3_)
  * [Replicate4FNode](#no6Ar4QoMh)
  * [Replicate2FNode](#non0KK3oIi)
  * [Replicate2DNode](#no7qRCM6og)
  * [PackNode](#noGVCfuE66)
  * [PackBNode](#nonBkBlhcO)
  * [PackCNode](#novvJExAvo)
  * [PackSNode](#noCi5AYQVM)
  * [PackINode](#noxD7-VdLs)
  * [PackLNode](#noaV-oDvGu)
  * [PackFNode](#noQWaST5G2)
  * [PackDNode](#novz9ZgRAJ)
  * [Pack2x1BNode](#nox9jtQ1Om)
  * [Pack2x2BNode](#noODAUZ9YF)
  * [ExtractNode](#no4RydCtLh)
  * [ExtractBNode](#no0DnS8jkH)
  * [ExtractCNode](#noxFXPD_Lf)
  * [ExtractSNode](#noRduepW1S)
  * [ExtractINode](#no08PO3Wvx)
  * [ExtractLNode](#noKh5Xo_cf)
  * [ExtractFNode](#nopMBGvKlY)
  * [ExtractDNode](#noKLmz5kLP)


---
## <a name="nonCiAEau-" id="nonCiAEau-">VectorNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.
「SIMD 演算(ベクトル演算)」を表す全ての Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------VectorNode--------------------------------------
    // Vector Operation
    class VectorNode : public Node {
```

### 内部構造(Internal structure)
このクラスのサブクラスは単項演算または2項演算を表す.

単項演算の場合は (control input も含めて) 2つの入力ノードを持つ.
2項演算の場合は, (control input も含めて) 3つの入力ノードを持つ.

ただし, どちらの場合も control input は常に空 (0 が設定される).

(なお, vlen 引数はベクトル内の要素の「個数」を表す (合計のバイト数ではない))


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      VectorNode(Node* n1, uint vlen) : Node(NULL, n1), _length(vlen) {
        init_flags(Flag_is_Vector);
      }
      VectorNode(Node* n1, Node* n2, uint vlen) : Node(NULL, n1, n2), _length(vlen) {
        init_flags(Flag_is_Vector);
      }
```




### 詳細(Details)
See: [here](../doxygen/classVectorNode.html) for details

---
## <a name="noUM8Ktx8i" id="noUM8Ktx8i">AddVBNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは byte 値同士の SIMD 加算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------AddVBNode---------------------------------------
    // Vector add byte
    class AddVBNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      AddVBNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classAddVBNode.html) for details

---
## <a name="noNLJnUXq9" id="noNLJnUXq9">AddVCNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは char 値同士の SIMD 加算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------AddVCNode---------------------------------------
    // Vector add char
    class AddVCNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      AddVCNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classAddVCNode.html) for details

---
## <a name="noCsYkx8qN" id="noCsYkx8qN">AddVSNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは short 値同士の SIMD 加算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------AddVSNode---------------------------------------
    // Vector add short
    class AddVSNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      AddVSNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classAddVSNode.html) for details

---
## <a name="nokwywxOyL" id="nokwywxOyL">AddVINode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の SIMD 加算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------AddVINode---------------------------------------
    // Vector add int
    class AddVINode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      AddVINode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classAddVINode.html) for details

---
## <a name="noNzLWu09P" id="noNzLWu09P">AddVLNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは long 値同士の SIMD 加算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------AddVLNode---------------------------------------
    // Vector add long
    class AddVLNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      AddVLNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classAddVLNode.html) for details

---
## <a name="no4MC8bdyW" id="no4MC8bdyW">AddVFNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは float 値同士の SIMD 加算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------AddVFNode---------------------------------------
    // Vector add float
    class AddVFNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      AddVFNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classAddVFNode.html) for details

---
## <a name="no4NVULsXc" id="no4NVULsXc">AddVDNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは double 値同士の SIMD 加算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------AddVDNode---------------------------------------
    // Vector add double
    class AddVDNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      AddVDNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classAddVDNode.html) for details

---
## <a name="no3S-q6748" id="no3S-q6748">SubVBNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは byte 値同士の SIMD 減算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------SubVBNode---------------------------------------
    // Vector subtract byte
    class SubVBNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      SubVBNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classSubVBNode.html) for details

---
## <a name="nomMHmLwtz" id="nomMHmLwtz">SubVCNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは char 値同士の SIMD 減算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------SubVCNode---------------------------------------
    // Vector subtract char
    class SubVCNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      SubVCNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classSubVCNode.html) for details

---
## <a name="noBPW3pSgv" id="noBPW3pSgv">SubVSNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは short 値同士の SIMD 減算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------SubVSNode---------------------------------------
    // Vector subtract short
    class SubVSNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      SubVSNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classSubVSNode.html) for details

---
## <a name="nox75C21VM" id="nox75C21VM">SubVINode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の SIMD 減算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------SubVINode---------------------------------------
    // Vector subtract int
    class SubVINode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      SubVINode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classSubVINode.html) for details

---
## <a name="noqvnMxllI" id="noqvnMxllI">SubVLNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは long 値同士の SIMD 減算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------SubVLNode---------------------------------------
    // Vector subtract long
    class SubVLNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      SubVLNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classSubVLNode.html) for details

---
## <a name="noIk2UKvui" id="noIk2UKvui">SubVFNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは float 値同士の SIMD 減算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------SubVFNode---------------------------------------
    // Vector subtract float
    class SubVFNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      SubVFNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classSubVFNode.html) for details

---
## <a name="noxdndiM4P" id="noxdndiM4P">SubVDNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは double 値同士の SIMD 減算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------SubVDNode---------------------------------------
    // Vector subtract double
    class SubVDNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      SubVDNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classSubVDNode.html) for details

---
## <a name="noymzGry2Y" id="noymzGry2Y">MulVFNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは float 値同士の SIMD 乗算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------MulVFNode---------------------------------------
    // Vector multiply float
    class MulVFNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      MulVFNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classMulVFNode.html) for details

---
## <a name="nobMAuBtM_" id="nobMAuBtM_">MulVDNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは double 値同士の SIMD 乗算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------MulVDNode---------------------------------------
    // Vector multiply double
    class MulVDNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      MulVDNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classMulVDNode.html) for details

---
## <a name="noZicQJU-B" id="noZicQJU-B">DivVFNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは float 値同士の SIMD 除算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------DivVFNode---------------------------------------
    // Vector divide float
    class DivVFNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      DivVFNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classDivVFNode.html) for details

---
## <a name="noDuimMabR" id="noDuimMabR">DivVDNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは double 値同士の SIMD 除算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------DivVDNode---------------------------------------
    // Vector Divide double
    class DivVDNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      DivVDNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classDivVDNode.html) for details

---
## <a name="noyDmm-s62" id="noyDmm-s62">LShiftVBNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは byte 値同士の SIMD 左シフト演算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------LShiftVBNode---------------------------------------
    // Vector lshift byte
    class LShiftVBNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      LShiftVBNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classLShiftVBNode.html) for details

---
## <a name="noQ7tYBNH0" id="noQ7tYBNH0">LShiftVCNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは char 値同士の SIMD 左シフト演算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------LShiftVCNode---------------------------------------
    // Vector lshift chars
    class LShiftVCNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      LShiftVCNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classLShiftVCNode.html) for details

---
## <a name="noLzTIRHA5" id="noLzTIRHA5">LShiftVSNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは short 値同士の SIMD 左シフト演算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------LShiftVSNode---------------------------------------
    // Vector lshift shorts
    class LShiftVSNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      LShiftVSNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classLShiftVSNode.html) for details

---
## <a name="noCugK4jJa" id="noCugK4jJa">LShiftVINode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の SIMD 左シフト演算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------LShiftVINode---------------------------------------
    // Vector lshift ints
    class LShiftVINode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      LShiftVINode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classLShiftVINode.html) for details

---
## <a name="noZFMB9jgC" id="noZFMB9jgC">URShiftVBNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは byte 値同士の SIMD 論理右シフト演算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------URShiftVBNode---------------------------------------
    // Vector urshift bytes
    class URShiftVBNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      URShiftVBNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classURShiftVBNode.html) for details

---
## <a name="nodmepQdrw" id="nodmepQdrw">URShiftVCNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは char 値同士の SIMD 論理右シフト演算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------URShiftVCNode---------------------------------------
    // Vector urshift char
    class URShiftVCNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      URShiftVCNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classURShiftVCNode.html) for details

---
## <a name="noL_XdAxX5" id="noL_XdAxX5">URShiftVSNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは short 値同士の SIMD 論理右シフト演算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------URShiftVSNode---------------------------------------
    // Vector urshift shorts
    class URShiftVSNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      URShiftVSNode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classURShiftVSNode.html) for details

---
## <a name="noPL5Wu_nD" id="noPL5Wu_nD">URShiftVINode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の SIMD 論理右シフト演算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------URShiftVINode---------------------------------------
    // Vector urshift ints
    class URShiftVINode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      URShiftVINode(Node* in1, Node* in2, uint vlen) : VectorNode(in1,in2,vlen) {}
```




### 詳細(Details)
See: [here](../doxygen/classURShiftVINode.html) for details

---
## <a name="noJtfZhhgk" id="noJtfZhhgk">AndVNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスはベクトル値同士の SIMD 論理積(ビットAND)演算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------AndVNode---------------------------------------
    // Vector and
    class AndVNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      AndVNode(Node* in1, Node* in2, uint vlen, BasicType bt) : VectorNode(in1,in2,vlen), _bt(bt) {}
```




### 詳細(Details)
See: [here](../doxygen/classAndVNode.html) for details

---
## <a name="noEErLK6gZ" id="noEErLK6gZ">OrVNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスはベクトル値同士の SIMD 論理和(ビットOR)演算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------OrVNode---------------------------------------
    // Vector or
    class OrVNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      OrVNode(Node* in1, Node* in2, uint vlen, BasicType bt) : VectorNode(in1,in2,vlen), _bt(bt) {}
```




### 詳細(Details)
See: [here](../doxygen/classOrVNode.html) for details

---
## <a name="nogmn3VcUE" id="nogmn3VcUE">XorVNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.
このクラスはベクトル値同士の SIMD 排他的論理和(ビットXOR)演算用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------XorVNode---------------------------------------
    // Vector xor
    class XorVNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      XorVNode(Node* in1, Node* in2, uint vlen, BasicType bt) : VectorNode(in1,in2,vlen), _bt(bt) {}
```




### 詳細(Details)
See: [here](../doxygen/classXorVNode.html) for details

---
## <a name="noRhu8-mMG" id="noRhu8-mMG">VectorLoadNode</a>

### 概要(Summary)
LoadNode クラスのサブクラスの1つ.
SIMD 用の値のロードを表す全ての Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------VectorLoadNode--------------------------------------
    // Vector Load from memory
    class VectorLoadNode : public LoadNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      VectorLoadNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const Type *rt)
        : LoadNode(c,mem,adr,at,rt) {
          init_flags(Flag_is_Vector);
      }
```




### 詳細(Details)
See: [here](../doxygen/classVectorLoadNode.html) for details

---
## <a name="noLVjP4jWx" id="noLVjP4jWx">Load16BNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは byte 値 16 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load16BNode--------------------------------------
    // Vector load of 16 bytes (8bits signed) from memory
    class Load16BNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load16BNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt *ti = TypeInt::BYTE)
        : VectorLoadNode(c,mem,adr,at,vect_type(ti,16)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad16BNode.html) for details

---
## <a name="no9TUQh35I" id="no9TUQh35I">Load8BNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは byte 値 8 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load8BNode--------------------------------------
    // Vector load of 8 bytes (8bits signed) from memory
    class Load8BNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load8BNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt *ti = TypeInt::BYTE)
        : VectorLoadNode(c,mem,adr,at,vect_type(ti,8)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad8BNode.html) for details

---
## <a name="no0HjI5sOV" id="no0HjI5sOV">Load4BNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは byte 値 4 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load4BNode--------------------------------------
    // Vector load of 4 bytes (8bits signed) from memory
    class Load4BNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load4BNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt *ti = TypeInt::BYTE)
        : VectorLoadNode(c,mem,adr,at,vect_type(ti,4)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad4BNode.html) for details

---
## <a name="noJEv68nAZ" id="noJEv68nAZ">Load8CNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは char 値 8 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load8CNode--------------------------------------
    // Vector load of 8 chars (16bits unsigned) from memory
    class Load8CNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load8CNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt *ti = TypeInt::CHAR)
        : VectorLoadNode(c,mem,adr,at,vect_type(ti,8)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad8CNode.html) for details

---
## <a name="nod9SNg-21" id="nod9SNg-21">Load4CNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは char 値 4 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load4CNode--------------------------------------
    // Vector load of 4 chars (16bits unsigned) from memory
    class Load4CNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load4CNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt *ti = TypeInt::CHAR)
        : VectorLoadNode(c,mem,adr,at,vect_type(ti,4)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad4CNode.html) for details

---
## <a name="nosPdX_I8P" id="nosPdX_I8P">Load2CNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは char 値 2 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load2CNode--------------------------------------
    // Vector load of 2 chars (16bits unsigned) from memory
    class Load2CNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load2CNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt *ti = TypeInt::CHAR)
        : VectorLoadNode(c,mem,adr,at,vect_type(ti,2)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad2CNode.html) for details

---
## <a name="noFkKw4w5C" id="noFkKw4w5C">Load8SNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは short 値 8 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load8SNode--------------------------------------
    // Vector load of 8 shorts (16bits signed) from memory
    class Load8SNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load8SNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt *ti = TypeInt::SHORT)
        : VectorLoadNode(c,mem,adr,at,vect_type(ti,8)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad8SNode.html) for details

---
## <a name="noPujOFQcN" id="noPujOFQcN">Load4SNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは short 値 4 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load4SNode--------------------------------------
    // Vector load of 4 shorts (16bits signed) from memory
    class Load4SNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load4SNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt *ti = TypeInt::SHORT)
        : VectorLoadNode(c,mem,adr,at,vect_type(ti,4)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad4SNode.html) for details

---
## <a name="noIcqrv8yZ" id="noIcqrv8yZ">Load2SNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは short 値 2 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load2SNode--------------------------------------
    // Vector load of 2 shorts (16bits signed) from memory
    class Load2SNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load2SNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt *ti = TypeInt::SHORT)
        : VectorLoadNode(c,mem,adr,at,vect_type(ti,2)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad2SNode.html) for details

---
## <a name="noiUvV2EDQ" id="noiUvV2EDQ">Load4INode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは int 値 4 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load4INode--------------------------------------
    // Vector load of 4 integers (32bits signed) from memory
    class Load4INode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load4INode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt *ti = TypeInt::INT)
        : VectorLoadNode(c,mem,adr,at,vect_type(ti,4)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad4INode.html) for details

---
## <a name="no0zrvz3r-" id="no0zrvz3r-">Load2INode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは int 値 2 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load2INode--------------------------------------
    // Vector load of 2 integers (32bits signed) from memory
    class Load2INode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load2INode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt *ti = TypeInt::INT)
        : VectorLoadNode(c,mem,adr,at,vect_type(ti,2)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad2INode.html) for details

---
## <a name="nophEGXQcm" id="nophEGXQcm">Load2LNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは long 値 2 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load2LNode--------------------------------------
    // Vector load of 2 longs (64bits signed) from memory
    class Load2LNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load2LNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeLong *tl = TypeLong::LONG)
        : VectorLoadNode(c,mem,adr,at,vect_type(tl,2)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad2LNode.html) for details

---
## <a name="no4A4oJE1T" id="no4A4oJE1T">Load4FNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは float 値 4 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load4FNode--------------------------------------
    // Vector load of 4 floats (32bits) from memory
    class Load4FNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load4FNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const Type *t = Type::FLOAT)
        : VectorLoadNode(c,mem,adr,at,vect_type(t,4)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad4FNode.html) for details

---
## <a name="nobtsnWAnD" id="nobtsnWAnD">Load2FNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは float 値 2 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load2FNode--------------------------------------
    // Vector load of 2 floats (32bits) from memory
    class Load2FNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load2FNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const Type *t = Type::FLOAT)
        : VectorLoadNode(c,mem,adr,at,vect_type(t,2)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad2FNode.html) for details

---
## <a name="nopQQgNJvU" id="nopQQgNJvU">Load2DNode</a>

### 概要(Summary)
VectorLoadNode クラスの具象サブクラスの1つ.
このクラスは double 値 2 個のロード用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Load2DNode--------------------------------------
    // Vector load of 2 doubles (64bits) from memory
    class Load2DNode : public VectorLoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorLoadNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Load2DNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const Type *t = Type::DOUBLE)
        : VectorLoadNode(c,mem,adr,at,vect_type(t,2)) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoad2DNode.html) for details

---
## <a name="no6K1tNPfi" id="no6K1tNPfi">VectorStoreNode</a>

### 概要(Summary)
StoreNode クラスのサブクラスの1つ.
SIMD 用の値のストアを表す全ての Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------VectorStoreNode--------------------------------------
    // Vector Store to memory
    class VectorStoreNode : public StoreNode {
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      VectorStoreNode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : StoreNode(c,mem,adr,at,val) {
          init_flags(Flag_is_Vector);
      }
```




### 詳細(Details)
See: [here](../doxygen/classVectorStoreNode.html) for details

---
## <a name="no7DWriQdE" id="no7DWriQdE">Store16BNode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは byte 値 16 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store16BNode--------------------------------------
    // Vector store of 16 bytes (8bits signed) to memory
    class Store16BNode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store16BNode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore16BNode.html) for details

---
## <a name="nowRMKup0z" id="nowRMKup0z">Store8BNode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは byte 値 8 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store8BNode--------------------------------------
    // Vector store of 8 bytes (8bits signed) to memory
    class Store8BNode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store8BNode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore8BNode.html) for details

---
## <a name="no9vJ7BmZN" id="no9vJ7BmZN">Store4BNode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは byte 値 4 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store4BNode--------------------------------------
    // Vector store of 4 bytes (8bits signed) to memory
    class Store4BNode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store4BNode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore4BNode.html) for details

---
## <a name="nox9fTTdVQ" id="nox9fTTdVQ">Store8CNode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは char 値 8 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store8CNode--------------------------------------
    // Vector store of 8 chars (16bits signed/unsigned) to memory
    class Store8CNode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store8CNode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore8CNode.html) for details

---
## <a name="nof4Gbysjz" id="nof4Gbysjz">Store4CNode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは char 値 4 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store4CNode--------------------------------------
    // Vector store of 4 chars (16bits signed/unsigned) to memory
    class Store4CNode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store4CNode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore4CNode.html) for details

---
## <a name="noiN1aabrS" id="noiN1aabrS">Store2CNode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは char 値 2 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store2CNode--------------------------------------
    // Vector store of 2 chars (16bits signed/unsigned) to memory
    class Store2CNode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store2CNode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore2CNode.html) for details

---
## <a name="no8t8OzDgk" id="no8t8OzDgk">Store4INode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは int 値 4 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store4INode--------------------------------------
    // Vector store of 4 integers (32bits signed) to memory
    class Store4INode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store4INode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore4INode.html) for details

---
## <a name="noqpXB2AX5" id="noqpXB2AX5">Store2INode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは int 値 2 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store2INode--------------------------------------
    // Vector store of 2 integers (32bits signed) to memory
    class Store2INode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store2INode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore2INode.html) for details

---
## <a name="noPuzQm80G" id="noPuzQm80G">Store2LNode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは long 値 2 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store2LNode--------------------------------------
    // Vector store of 2 longs (64bits signed) to memory
    class Store2LNode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store2LNode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore2LNode.html) for details

---
## <a name="nocKEkI488" id="nocKEkI488">Store4FNode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは float 値 4 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store4FNode--------------------------------------
    // Vector store of 4 floats (32bits) to memory
    class Store4FNode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store4FNode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore4FNode.html) for details

---
## <a name="no-qVQt5UX" id="no-qVQt5UX">Store2FNode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは float 値 2 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store2FNode--------------------------------------
    // Vector store of 2 floats (32bits) to memory
    class Store2FNode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store2FNode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore2FNode.html) for details

---
## <a name="noAr7mPBlh" id="noAr7mPBlh">Store2DNode</a>

### 概要(Summary)
VectorStoredNode クラスの具象サブクラスの1つ.
このクラスは double 値 2 個のストア用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Store2DNode--------------------------------------
    // Vector store of 2 doubles (64bits) to memory
    class Store2DNode : public VectorStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorStoreNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> VectorStoreNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Store2DNode(Node* c, Node* mem, Node* adr, const TypePtr* at, Node* val)
        : VectorStoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStore2DNode.html) for details

---
## <a name="nobR8cDW6Y" id="nobR8cDW6Y">Replicate16BNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの byte 値を 16個複製し, byte 値 16 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate16BNode---------------------------------------
    // Replicate byte scalar to be vector of 16 bytes
    class Replicate16BNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate16BNode(Node* in1) : VectorNode(in1, 16) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate16BNode.html) for details

---
## <a name="no8zIbd3a5" id="no8zIbd3a5">Replicate8BNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの byte 値を 8個複製し, byte 値 8 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate8BNode---------------------------------------
    // Replicate byte scalar to be vector of 8 bytes
    class Replicate8BNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate8BNode(Node* in1) : VectorNode(in1, 8) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate8BNode.html) for details

---
## <a name="no2SEax67u" id="no2SEax67u">Replicate4BNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの byte 値を 4個複製し, byte 値 4 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate4BNode---------------------------------------
    // Replicate byte scalar to be vector of 4 bytes
    class Replicate4BNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate4BNode(Node* in1) : VectorNode(in1, 4) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate4BNode.html) for details

---
## <a name="noeHTdrG2Z" id="noeHTdrG2Z">Replicate8CNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの char 値を 8個複製し, char 値 8 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate8CNode---------------------------------------
    // Replicate char scalar to be vector of 8 chars
    class Replicate8CNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate8CNode(Node* in1) : VectorNode(in1, 8) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate8CNode.html) for details

---
## <a name="noSUM_ydtI" id="noSUM_ydtI">Replicate4CNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの char 値を 4個複製し, char 値 4 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate4CNode---------------------------------------
    // Replicate char scalar to be vector of 4 chars
    class Replicate4CNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate4CNode(Node* in1) : VectorNode(in1, 4) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate4CNode.html) for details

---
## <a name="no04pAHMsy" id="no04pAHMsy">Replicate2CNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの char 値を 2個複製し, char 値 2 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate2CNode---------------------------------------
    // Replicate char scalar to be vector of 2 chars
    class Replicate2CNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate2CNode(Node* in1) : VectorNode(in1, 2) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate2CNode.html) for details

---
## <a name="noIQdxc7jo" id="noIQdxc7jo">Replicate8SNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの short 値を 8個複製し, short 値 8 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate8SNode---------------------------------------
    // Replicate short scalar to be vector of 8 shorts
    class Replicate8SNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate8SNode(Node* in1) : VectorNode(in1, 8) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate8SNode.html) for details

---
## <a name="noyapMWe88" id="noyapMWe88">Replicate4SNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの short 値を 4個複製し, short 値 4 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate4SNode---------------------------------------
    // Replicate short scalar to be vector of 4 shorts
    class Replicate4SNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate4SNode(Node* in1) : VectorNode(in1, 4) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate4SNode.html) for details

---
## <a name="nobA28RQ-2" id="nobA28RQ-2">Replicate2SNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの short 値を 2個複製し, short 値 2 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate2SNode---------------------------------------
    // Replicate short scalar to be vector of 2 shorts
    class Replicate2SNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate2SNode(Node* in1) : VectorNode(in1, 2) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate2SNode.html) for details

---
## <a name="norOwM9fAI" id="norOwM9fAI">Replicate4INode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの int 値を 4個複製し, int 値 4 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate4INode---------------------------------------
    // Replicate int scalar to be vector of 4 ints
    class Replicate4INode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate4INode(Node* in1) : VectorNode(in1, 4) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate4INode.html) for details

---
## <a name="noDxshFxQN" id="noDxshFxQN">Replicate2INode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの int 値を 2個複製し, int 値 2 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate2INode---------------------------------------
    // Replicate int scalar to be vector of 2 ints
    class Replicate2INode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate2INode(Node* in1) : VectorNode(in1, 2) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate2INode.html) for details

---
## <a name="nog1GVjz3_" id="nog1GVjz3_">Replicate2LNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの long 値を 2個複製し, long 値 2 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate2LNode---------------------------------------
    // Replicate long scalar to be vector of 2 longs
    class Replicate2LNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate2LNode(Node* in1) : VectorNode(in1, 2) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate2LNode.html) for details

---
## <a name="no6Ar4QoMh" id="no6Ar4QoMh">Replicate4FNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの float 値を 4個複製し, float 値 4 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate4FNode---------------------------------------
    // Replicate float scalar to be vector of 4 floats
    class Replicate4FNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate4FNode(Node* in1) : VectorNode(in1, 4) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate4FNode.html) for details

---
## <a name="non0KK3oIi" id="non0KK3oIi">Replicate2FNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの float 値を 2個複製し, float 値 2 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate2FNode---------------------------------------
    // Replicate float scalar to be vector of 2 floats
    class Replicate2FNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate2FNode(Node* in1) : VectorNode(in1, 2) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate2FNode.html) for details

---
## <a name="no7qRCM6og" id="no7qRCM6og">Replicate2DNode</a>

### 概要(Summary)
VectorNode クラスの具象サブクラスの1つ.

このクラスは, 1つの double 値を 2個複製し, double 値 2 個からなるベクトル値を作成する処理用.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Replicate2DNode---------------------------------------
    // Replicate double scalar to be vector of 2 doubles
    class Replicate2DNode : public VectorNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
VectorNode::scalar2vector() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> VectorNode::scalar2vector()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Replicate2DNode(Node* in1) : VectorNode(in1, 2) {}
```




### 詳細(Details)
See: [here](../doxygen/classReplicate2DNode.html) for details

---
## <a name="noGVCfuE66" id="noGVCfuE66">PackNode</a>

### 概要(Summary)
VectorNode クラスのサブクラスの1つ.
このクラスは「SIMD 演算の対象となるデータ(スカラ値)を 1つのベクトル値にまとめる(パックする)処理」を表す全ての Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------PackNode---------------------------------------
    // Pack parent class (not for code generation).
    class PackNode : public VectorNode {
```

### 内部構造(Internal structure)
このクラスのサブクラスは単項演算または2項演算を表す.

単項演算の場合は (control input も含めて) 2つの入力ノードを持つ.
2項演算の場合は, (control input も含めて) 3つの入力ノードを持つ.

ただし, どちらの場合も control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      PackNode(Node* in1)  : VectorNode(in1, 1) {}
      PackNode(Node* in1, Node* n2)  : VectorNode(in1, n2, 2) {}
```




### 詳細(Details)
See: [here](../doxygen/classPackNode.html) for details

---
## <a name="nonBkBlhcO" id="nonBkBlhcO">PackBNode</a>

### 概要(Summary)
PackNode クラスの具象サブクラスの1つ.
このクラスは, 複数の byte 値を 1つのベクトル値にまとめる.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------PackBNode---------------------------------------
    // Pack byte scalars into vector
    class PackBNode : public PackNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
PackNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> PackNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      PackBNode(Node* in1)  : PackNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classPackBNode.html) for details

---
## <a name="novvJExAvo" id="novvJExAvo">PackCNode</a>

### 概要(Summary)
PackNode クラスの具象サブクラスの1つ.
このクラスは, 複数の char 値を 1つのベクトル値にまとめる.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------PackCNode---------------------------------------
    // Pack char scalars into vector
    class PackCNode : public PackNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
PackNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> PackNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      PackCNode(Node* in1)  : PackNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classPackCNode.html) for details

---
## <a name="noCi5AYQVM" id="noCi5AYQVM">PackSNode</a>

### 概要(Summary)
PackNode クラスの具象サブクラスの1つ.
このクラスは, 複数の short 値を 1つのベクトル値にまとめる.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------PackSNode---------------------------------------
    // Pack short scalars into a vector
    class PackSNode : public PackNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
PackNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> PackNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      PackSNode(Node* in1)  : PackNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classPackSNode.html) for details

---
## <a name="noxD7-VdLs" id="noxD7-VdLs">PackINode</a>

### 概要(Summary)
PackNode クラスの具象サブクラスの1つ.
このクラスは, 複数の int 値を 1つのベクトル値にまとめる.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------PackINode---------------------------------------
    // Pack integer scalars into a vector
    class PackINode : public PackNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* PackNode::make()
* final_graph_reshaping_impl()

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> PackNode::make()
```

```
Compile::Optimize()
-> Compile::final_graph_reshaping()
   -> final_graph_reshaping_walk()
      -> final_graph_reshaping_impl()
```

### 内部構造(Internal structure)
単項演算の場合は (control input も含めて) 2つの入力ノードを持つ.
2項演算の場合は, (control input も含めて) 3つの入力ノードを持つ.

どちらの場合も control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      PackINode(Node* in1)  : PackNode(in1) {}
      PackINode(Node* in1, Node* in2) : PackNode(in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classPackINode.html) for details

---
## <a name="noaV-oDvGu" id="noaV-oDvGu">PackLNode</a>

### 概要(Summary)
PackNode クラスの具象サブクラスの1つ.
このクラスは, 複数の long 値を 1つのベクトル値にまとめる.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------PackLNode---------------------------------------
    // Pack long scalars into a vector
    class PackLNode : public PackNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* PackNode::make()
* final_graph_reshaping_impl()

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> PackNode::make()
```

```
Compile::Optimize()
-> Compile::final_graph_reshaping()
   -> final_graph_reshaping_walk()
      -> final_graph_reshaping_impl()
```

### 内部構造(Internal structure)
単項演算の場合は (control input も含めて) 2つの入力ノードを持つ.
2項演算の場合は, (control input も含めて) 3つの入力ノードを持つ.

どちらの場合も control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      PackLNode(Node* in1)  : PackNode(in1) {}
      PackLNode(Node* in1, Node* in2) : PackNode(in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classPackLNode.html) for details

---
## <a name="noQWaST5G2" id="noQWaST5G2">PackFNode</a>

### 概要(Summary)
PackNode クラスの具象サブクラスの1つ.
このクラスは, 複数の float 値を 1つのベクトル値にまとめる.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------PackFNode---------------------------------------
    // Pack float scalars into vector
    class PackFNode : public PackNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* PackNode::make()
* final_graph_reshaping_impl()

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> PackNode::make()
```

```
Compile::Optimize()
-> Compile::final_graph_reshaping()
   -> final_graph_reshaping_walk()
      -> final_graph_reshaping_impl()
```

### 内部構造(Internal structure)
単項演算の場合は (control input も含めて) 2つの入力ノードを持つ.
2項演算の場合は, (control input も含めて) 3つの入力ノードを持つ.

どちらの場合も control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      PackFNode(Node* in1)  : PackNode(in1) {}
      PackFNode(Node* in1, Node* in2) : PackNode(in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classPackFNode.html) for details

---
## <a name="novz9ZgRAJ" id="novz9ZgRAJ">PackDNode</a>

### 概要(Summary)
PackNode クラスの具象サブクラスの1つ.
このクラスは, 複数の double 値を 1つのベクトル値にまとめる.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------PackDNode---------------------------------------
    // Pack double scalars into a vector
    class PackDNode : public PackNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* PackNode::make()
* final_graph_reshaping_impl()

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::vector_opd()
            -> PackNode::make()
```

```
Compile::Optimize()
-> Compile::final_graph_reshaping()
   -> final_graph_reshaping_walk()
      -> final_graph_reshaping_impl()
```

### 内部構造(Internal structure)
単項演算の場合は (control input も含めて) 2つの入力ノードを持つ.
2項演算の場合は, (control input も含めて) 3つの入力ノードを持つ.

どちらの場合も control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      PackDNode(Node* in1)  : PackNode(in1) {}
      PackDNode(Node* in1, Node* in2) : PackNode(in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classPackDNode.html) for details

---
## <a name="nox9jtQ1Om" id="nox9jtQ1Om">Pack2x1BNode</a>

### 概要(Summary)
PackNode クラスの具象サブクラスの1つ.
このクラスは, 2つの byte 値を 1つのベクトル値にまとめる.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    // The Pack2xN nodes assist code generation.  They are created from
    // Pack4C, etc. nodes in final_graph_reshape in the form of a
    // balanced, binary tree.
```


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Pack2x1BNode-----------------------------------------
    // Pack 2 1-byte integers into vector of 2 bytes
    class Pack2x1BNode : public PackNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
final_graph_reshaping_impl() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Compile::Optimize()
-> Compile::final_graph_reshaping()
   -> final_graph_reshaping_walk()
      -> final_graph_reshaping_impl()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ. ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Pack2x1BNode(Node *in1, Node* in2) : PackNode(in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classPack2x1BNode.html) for details

---
## <a name="noODAUZ9YF" id="noODAUZ9YF">Pack2x2BNode</a>

### 概要(Summary)
PackNode クラスの具象サブクラスの1つ.
このクラスは, 2つの 2Byte 値 (short/char/Pack2x1B) を 1つのベクトル値にまとめる.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    // The Pack2xN nodes assist code generation.  They are created from
    // Pack4C, etc. nodes in final_graph_reshape in the form of a
    // balanced, binary tree.
```


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------Pack2x2BNode---------------------------------------
    // Pack 2 2-byte integers into vector of 4 bytes
    class Pack2x2BNode : public PackNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
final_graph_reshaping_impl() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Compile::Optimize()
-> Compile::final_graph_reshaping()
   -> final_graph_reshaping_walk()
      -> final_graph_reshaping_impl()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ. ただし control input は常に空 (0 が設定される).


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      Pack2x2BNode(Node *in1, Node* in2) : PackNode(in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classPack2x2BNode.html) for details

---
## <a name="no4RydCtLh" id="no4RydCtLh">ExtractNode</a>

### 概要(Summary)
VectorNode クラスのサブクラスの1つ.
このクラスは「ベクトル値の中からスカラ値を取り出す処理(アンパック処理)」を表す全ての Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------ExtractNode---------------------------------------
    // Extract a scalar from a vector at position "pos"
    class ExtractNode : public Node {
```

### 内部構造(Internal structure)

```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      ExtractNode(Node* src, ConINode* pos) : Node(NULL, src, (Node*)pos) {
        assert(in(2)->get_int() >= 0, "positive constants");
      }
```




### 詳細(Details)
See: [here](../doxygen/classExtractNode.html) for details

---
## <a name="no0DnS8jkH" id="no0DnS8jkH">ExtractBNode</a>

### 概要(Summary)
ExtractNode クラスの具象サブクラスの1つ.
このクラスは, ベクトル値の中から byte 値を取り出す.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------ExtractBNode---------------------------------------
    // Extract a byte from a vector at position "pos"
    class ExtractBNode : public ExtractNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ExtractNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::insert_extracts()
            -> ExtractNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 2つは以下の通り.

* 2番目の入力Node : 対象のベクトル値
* 3番目の入力Node : 取り出したい要素の位置


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      ExtractBNode(Node* src, ConINode* pos) : ExtractNode(src, pos) {}
```




### 詳細(Details)
See: [here](../doxygen/classExtractBNode.html) for details

---
## <a name="noxFXPD_Lf" id="noxFXPD_Lf">ExtractCNode</a>

### 概要(Summary)
ExtractNode クラスの具象サブクラスの1つ.
このクラスは, ベクトル値の中から char 値を取り出す.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------ExtractCNode---------------------------------------
    // Extract a char from a vector at position "pos"
    class ExtractCNode : public ExtractNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ExtractNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::insert_extracts()
            -> ExtractNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 2つは以下の通り.

* 2番目の入力Node : 対象のベクトル値
* 3番目の入力Node : 取り出したい要素の位置


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      ExtractCNode(Node* src, ConINode* pos) : ExtractNode(src, pos) {}
```




### 詳細(Details)
See: [here](../doxygen/classExtractCNode.html) for details

---
## <a name="noRduepW1S" id="noRduepW1S">ExtractSNode</a>

### 概要(Summary)
ExtractNode クラスの具象サブクラスの1つ.
このクラスは, ベクトル値の中から short 値を取り出す.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------ExtractSNode---------------------------------------
    // Extract a short from a vector at position "pos"
    class ExtractSNode : public ExtractNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ExtractNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::insert_extracts()
            -> ExtractNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 2つは以下の通り.

* 2番目の入力Node : 対象のベクトル値
* 3番目の入力Node : 取り出したい要素の位置


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      ExtractSNode(Node* src, ConINode* pos) : ExtractNode(src, pos) {}
```




### 詳細(Details)
See: [here](../doxygen/classExtractSNode.html) for details

---
## <a name="no08PO3Wvx" id="no08PO3Wvx">ExtractINode</a>

### 概要(Summary)
ExtractNode クラスの具象サブクラスの1つ.
このクラスは, ベクトル値の中から int 値を取り出す.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------ExtractINode---------------------------------------
    // Extract an int from a vector at position "pos"
    class ExtractINode : public ExtractNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ExtractNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::insert_extracts()
            -> ExtractNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 2つは以下の通り.

* 2番目の入力Node : 対象のベクトル値
* 3番目の入力Node : 取り出したい要素の位置


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      ExtractINode(Node* src, ConINode* pos) : ExtractNode(src, pos) {}
```




### 詳細(Details)
See: [here](../doxygen/classExtractINode.html) for details

---
## <a name="noKh5Xo_cf" id="noKh5Xo_cf">ExtractLNode</a>

### 概要(Summary)
ExtractNode クラスの具象サブクラスの1つ.
このクラスは, ベクトル値の中から long 値を取り出す.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------ExtractLNode---------------------------------------
    // Extract a long from a vector at position "pos"
    class ExtractLNode : public ExtractNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ExtractNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::insert_extracts()
            -> ExtractNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 2つは以下の通り.

* 2番目の入力Node : 対象のベクトル値
* 3番目の入力Node : 取り出したい要素の位置


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      ExtractLNode(Node* src, ConINode* pos) : ExtractNode(src, pos) {}
```




### 詳細(Details)
See: [here](../doxygen/classExtractLNode.html) for details

---
## <a name="nopMBGvKlY" id="nopMBGvKlY">ExtractFNode</a>

### 概要(Summary)
ExtractNode クラスの具象サブクラスの1つ.
このクラスは, ベクトル値の中から float 値を取り出す.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------ExtractFNode---------------------------------------
    // Extract a float from a vector at position "pos"
    class ExtractFNode : public ExtractNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ExtractNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::insert_extracts()
            -> ExtractNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 2つは以下の通り.

* 2番目の入力Node : 対象のベクトル値
* 3番目の入力Node : 取り出したい要素の位置


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      ExtractFNode(Node* src, ConINode* pos) : ExtractNode(src, pos) {}
```




### 詳細(Details)
See: [here](../doxygen/classExtractFNode.html) for details

---
## <a name="noKLmz5kLP" id="noKLmz5kLP">ExtractDNode</a>

### 概要(Summary)
ExtractNode クラスの具象サブクラスの1つ.
このクラスは, ベクトル値の中から double 値を取り出す.


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
    //------------------------------ExtractDNode---------------------------------------
    // Extract a double from a vector at position "pos"
    class ExtractDNode : public ExtractNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ExtractNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> SuperWord::transform_loop()
   -> SuperWord::SLP_extract()
      -> SuperWord::output()
         -> SuperWord::insert_extracts()
            -> ExtractNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 2つは以下の通り.

* 2番目の入力Node : 対象のベクトル値
* 3番目の入力Node : 取り出したい要素の位置


```
    ((cite: hotspot/src/share/vm/opto/vectornode.hpp))
      ExtractDNode(Node* src, ConINode* pos) : ExtractNode(src, pos) {}
```





### 詳細(Details)
See: [here](../doxygen/classExtractDNode.html) for details

---
