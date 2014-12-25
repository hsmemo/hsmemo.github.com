---
layout: default
title: 減算に関する高レベル中間語(Ideal)クラス (SubNode, SubINode, SubLNode, SubFPNode, SubFNode, SubDNode, CmpNode, CmpINode, CmpUNode, CmpPNode, CmpNNode, CmpLNode, CmpL3Node, CmpFNode, CmpF3Node, CmpDNode, CmpD3Node, BoolNode, AbsNode, AbsINode, AbsFNode, AbsDNode, CmpLTMaskNode, NegNode, NegFNode, NegDNode, CosDNode, SinDNode, TanDNode, AtanDNode, SqrtDNode, ExpDNode, LogDNode, Log10DNode, PowDNode, ReverseBytesINode, ReverseBytesLNode, ReverseBytesUSNode, ReverseBytesSNode)
---
[Top](../index.html)

#### 減算に関する高レベル中間語(Ideal)クラス (SubNode, SubINode, SubLNode, SubFPNode, SubFNode, SubDNode, CmpNode, CmpINode, CmpUNode, CmpPNode, CmpNNode, CmpLNode, CmpL3Node, CmpFNode, CmpF3Node, CmpDNode, CmpD3Node, BoolNode, AbsNode, AbsINode, AbsFNode, AbsDNode, CmpLTMaskNode, NegNode, NegFNode, NegDNode, CosDNode, SinDNode, TanDNode, AtanDNode, SqrtDNode, ExpDNode, LogDNode, Log10DNode, PowDNode, ReverseBytesINode, ReverseBytesLNode, ReverseBytesUSNode, ReverseBytesSNode)

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, 主に「減算」を表すクラス.

なお単なる数値の減算だけでなく, 減算に似たものは全てここで扱う
(例えば compare は {-1,0,1} への写像なので, 減算して符号だけ取得していると見なせる, 等).

(<= なぜか double の数学関数や, 整数のバイト反転処理も入っているが...?? #TODO)

なお, Cmp*Node の出力は {-1, 0, 1} のどれかを示す condition code だが, 
名前に "3" を含むものは {-1, 0, 1} の int 値を返す
(JVM の *cmp バイトコード命令が int 値を返すので,
 その値が branch の判定のみにしか使われないときは condition code,
 int 値としても使われるときは int, と最適化している模様 #TODO)

なお, 各 Sub*Node の先頭は「negative と add に置き換え可能」というコメントが書かれている 
(例: "NOTE: SubINode should be taken away and replaced by add and negate"). 
ただし, 逆に NegNode は SubNode で代用できないことに注意 (See: NegFNode, NegDNode).



### クラス一覧(class list)

  * [SubNode](#no5McEuB3p)
  * [SubINode](#noFdS1xIoY)
  * [SubLNode](#no9gVUYWSP)
  * [SubFPNode](#noK2P4uIwe)
  * [SubFNode](#nonndtCTqe)
  * [SubDNode](#noZuUoyjna)
  * [CmpNode](#no6gOgABuu)
  * [CmpINode](#not0Nj5f0z)
  * [CmpUNode](#nosjaSeEaS)
  * [CmpPNode](#no9evRjc74)
  * [CmpNNode](#no7JudFNW9)
  * [CmpLNode](#no-WvLtEJV)
  * [CmpL3Node](#noFJjPg1PQ)
  * [CmpFNode](#noopmSsoy3)
  * [CmpF3Node](#no3CIJfthv)
  * [CmpDNode](#nojmHr7AFY)
  * [CmpD3Node](#novbzQXhhR)
  * [BoolNode](#noYyR9k3zL)
  * [AbsNode](#noGfrwe4sF)
  * [AbsINode](#noP2Pohcqi)
  * [AbsFNode](#noYoRcZxWc)
  * [AbsDNode](#noKxUKseR2)
  * [CmpLTMaskNode](#no33uJpfIq)
  * [NegNode](#nobQtZjI2k)
  * [NegFNode](#noDy1yIpRW)
  * [NegDNode](#nolmRn74wb)
  * [CosDNode](#nox191LHVV)
  * [SinDNode](#noeiRxF8sG)
  * [TanDNode](#nooMjjMZdc)
  * [AtanDNode](#notUO5uOKl)
  * [SqrtDNode](#nobRqZCB7o)
  * [ExpDNode](#novZ1Qj5tO)
  * [LogDNode](#no3-JIH33p)
  * [Log10DNode](#norCSV-IRQ)
  * [PowDNode](#noRFzeWbtb)
  * [ReverseBytesINode](#noVphMlG-3)
  * [ReverseBytesLNode](#noc6op4gEp)
  * [ReverseBytesUSNode](#noifYl1RzB)
  * [ReverseBytesSNode](#no48QvolG1)


---
## <a name="no5McEuB3p" id="no5McEuB3p">SubNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.
「減算」を表す全ての Node クラスの基底クラス.

(なお単なる数値の減算だけでなく, 減算に似たものは全てこのクラスのサブクラスで扱う)
(例えば compare は {-1,0,1} への写像なので, 減算して符号だけ取得していると見なせる, 等).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Class SUBTRACTION functionality.  This covers all the usual 'subtract'
    // behaviors.  Subtract-integer, -float, -double, binary xor, compare-integer,
    // -float, and -double are all inherited from this class.  The compare
    // functions behave like subtract functions, except that all negative answers
    // are compressed into -1, and all positive answers compressed to 1.
    class SubNode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      SubNode( Node *in1, Node *in2 ) : Node(0,in1,in2) {
        init_class_id(Class_Sub);
      }
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classSubNode.html) for details

---
## <a name="noFdS1xIoY" id="noFdS1xIoY">SubINode</a>

### 概要(Summary)
SubNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の減算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // NOTE: SubINode should be taken away and replaced by add and negate
```


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Subtract 2 integers
    class SubINode : public SubNode {
```

なお SubXNode という型も使われるが, `#ifdef _LP64` でない場合は, これは SubINode の別名.


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define SubXNode     SubINode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      SubINode( Node *in1, Node *in2 ) : SubNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classSubINode.html) for details

---
## <a name="no9gVUYWSP" id="no9gVUYWSP">SubLNode</a>

### 概要(Summary)
SubNode クラスの具象サブクラスの1つ.
このクラスは long 値同士の減算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Subtract 2 integers
    class SubLNode : public SubNode {
```

なお SubXNode という型も使われるが, `#ifdef _LP64` の場合は, これは SubLNode の別名.   


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define SubXNode     SubLNode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      SubLNode( Node *in1, Node *in2 ) : SubNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classSubLNode.html) for details

---
## <a name="noK2P4uIwe" id="noK2P4uIwe">SubFPNode</a>

### 概要(Summary)
SubNode クラスのサブクラスの1つ.
このクラスは float/double 値同士の減算用.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // NOTE: SubFPNode should be taken away and replaced by add and negate
```


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Subtract 2 floats or doubles
    class SubFPNode : public SubNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      SubFPNode( Node *in1, Node *in2 ) : SubNode(in1,in2) {}
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classSubFPNode.html) for details

---
## <a name="nonndtCTqe" id="nonndtCTqe">SubFNode</a>

### 概要(Summary)
SubFPNode クラスの具象サブクラスの1つ.
このクラスは float 値同士の減算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // NOTE: SubFNode should be taken away and replaced by add and negate
```


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Subtract 2 doubles
    class SubFNode : public SubFPNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      SubFNode( Node *in1, Node *in2 ) : SubFPNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classSubFNode.html) for details

---
## <a name="noZuUoyjna" id="noZuUoyjna">SubDNode</a>

### 概要(Summary)
SubFPNode クラスの具象サブクラスの1つ.
このクラスは double 値同士の減算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // NOTE: SubDNode should be taken away and replaced by add and negate
```


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Subtract 2 doubles
    class SubDNode : public SubFPNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      SubDNode( Node *in1, Node *in2 ) : SubFPNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classSubDNode.html) for details

---
## <a name="no6gOgABuu" id="no6gOgABuu">CmpNode</a>

### 概要(Summary)
SubNode クラスのサブクラスの1つ.
このクラスは比較演算用. 
結果としては {-1, 0, 1} のどれかを示す condition code を返す.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Compare 2 values, returning condition codes (-1, 0 or 1).
    class CmpNode : public SubNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpNode( Node *in1, Node *in2 ) : SubNode(in1,in2) {
        init_class_id(Class_Cmp);
      }
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classCmpNode.html) for details

---
## <a name="not0Nj5f0z" id="not0Nj5f0z">CmpINode</a>

### 概要(Summary)
CmpNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の比較(符号あり)用.

結果としては {-1, 0, 1} のどれかを示す condition code を返す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Compare 2 signed values, returning condition codes (-1, 0 or 1).
    class CmpINode : public CmpNode {
```

なお CmpXNode という型も使われるが, `#ifdef _LP64` でない場合は, これは CmpINode の別名.


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define CmpXNode     CmpINode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpINode( Node *in1, Node *in2 ) : CmpNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classCmpINode.html) for details

---
## <a name="nosjaSeEaS" id="nosjaSeEaS">CmpUNode</a>

### 概要(Summary)
CmpNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の比較(符号なし)用.

結果としては {-1, 0, 1} のどれかを示す condition code を返す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Compare 2 unsigned values (integer or pointer), returning condition codes (-1, 0 or 1).
    class CmpUNode : public CmpNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpUNode( Node *in1, Node *in2 ) : CmpNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classCmpUNode.html) for details

---
## <a name="no9evRjc74" id="no9evRjc74">CmpPNode</a>

### 概要(Summary)
CmpNode クラスの具象サブクラスの1つ.
このクラスはポインタ値同士の比較用.

結果としては {-1, 0, 1} のどれかを示す condition code を返す
(といってもポインタ比較なので -1 と 1 の区別は要らない気もするが...#TODO)


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Compare 2 pointer values, returning condition codes (-1, 0 or 1).
    class CmpPNode : public CmpNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpPNode( Node *in1, Node *in2 ) : CmpNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classCmpPNode.html) for details

---
## <a name="no7JudFNW9" id="no7JudFNW9">CmpNNode</a>

### 概要(Summary)
CmpNode クラスの具象サブクラスの1つ.
このクラスは narrow oop 値同士の比較用.

結果としては {-1, 0, 1} のどれかを示す condition code を返す
(といってもポインタ比較なので -1 と 1 の区別は要らない気もするが...#TODO)


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Compare 2 narrow oop values, returning condition codes (-1, 0 or 1).
    class CmpNNode : public CmpNode {
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
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpNNode( Node *in1, Node *in2 ) : CmpNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classCmpNNode.html) for details

---
## <a name="no-WvLtEJV" id="no-WvLtEJV">CmpLNode</a>

### 概要(Summary)
CmpNode クラスの具象サブクラスの1つ.
このクラスは long 値同士の比較用.

結果としては {-1, 0, 1} のどれかを示す condition code を返す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Compare 2 long values, returning condition codes (-1, 0 or 1).
    class CmpLNode : public CmpNode {
```

なお CmpXNode という型も使われるが, `#ifdef _LP64` の場合は, これは CmpLNode の別名.   


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define CmpXNode     CmpLNode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpLNode( Node *in1, Node *in2 ) : CmpNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classCmpLNode.html) for details

---
## <a name="noFJjPg1PQ" id="noFJjPg1PQ">CmpL3Node</a>

### 概要(Summary)
CmpLNode クラスのサブクラス.

スーパークラスと同じく long 値同士の比較用だが, 
このクラスは結果として {-1, 0, 1} の int 値を返す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Compare 2 long values, returning integer value (-1, 0 or 1).
    class CmpL3Node : public CmpLNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).

なお BoolNode と組み合わせられない (= condition code を返さない) ので正確には Cmp*Node ではない.
そのため _class_id フィールドには Class_Cmp ではなく Class_Sub を持つようにしている.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpL3Node( Node *in1, Node *in2 ) : CmpLNode(in1,in2) {
        // Since it is not consumed by Bools, it is not really a Cmp.
        init_class_id(Class_Sub);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCmpL3Node.html) for details

---
## <a name="noopmSsoy3" id="noopmSsoy3">CmpFNode</a>

### 概要(Summary)
CmpNode クラスの具象サブクラスの1つ.
このクラスは float 値同士の比較用.

結果としては {-1, 0, 1} のどれかを示す condition code を返す.

なお, semantics は fcmpl バイトコード命令に合わせている.
そのためどちらかが NaN 値であれば -1 が返される.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Compare 2 float values, returning condition codes (-1, 0 or 1).
    // This implements the Java bytecode fcmpl, so unordered returns -1.
    // Operands may not commute.
    class CmpFNode : public CmpNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpFNode( Node *in1, Node *in2 ) : CmpNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classCmpFNode.html) for details

---
## <a name="no3CIJfthv" id="no3CIJfthv">CmpF3Node</a>

### 概要(Summary)
CmpFNode クラスのサブクラス.

スーパークラスと同じく float 値同士の比較用だが, 
このクラスは結果として {-1, 0, 1} の int 値を返す.

なお, semantics は fcmpl バイトコード命令に合わせている.
そのためどちらかが NaN 値であれば -1 が返される.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Compare 2 float values, returning integer value (-1, 0 or 1).
    // This implements the Java bytecode fcmpl, so unordered returns -1.
    // Operands may not commute.
    class CmpF3Node : public CmpFNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).

なお BoolNode と組み合わせられない (= condition code を返さない) ので正確には Cmp*Node ではない.
そのため _class_id フィールドには Class_Cmp ではなく Class_Sub を持つようにしている.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpF3Node( Node *in1, Node *in2 ) : CmpFNode(in1,in2) {
        // Since it is not consumed by Bools, it is not really a Cmp.
        init_class_id(Class_Sub);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCmpF3Node.html) for details

---
## <a name="nojmHr7AFY" id="nojmHr7AFY">CmpDNode</a>

### 概要(Summary)
CmpNode クラスの具象サブクラスの1つ.
このクラスは double 値同士の比較用.

結果としては {-1, 0, 1} のどれかを示す condition code を返す.

なお, semantics は dcmpl バイトコード命令に合わせている.
そのためどちらかが NaN 値であれば -1 が返される.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Compare 2 double values, returning condition codes (-1, 0 or 1).
    // This implements the Java bytecode dcmpl, so unordered returns -1.
    // Operands may not commute.
    class CmpDNode : public CmpNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpDNode( Node *in1, Node *in2 ) : CmpNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classCmpDNode.html) for details

---
## <a name="novbzQXhhR" id="novbzQXhhR">CmpD3Node</a>

### 概要(Summary)
CmpDNode クラスのサブクラス.

スーパークラスと同じく double 値同士の比較用だが, 
このクラスは結果として {-1, 0, 1} の int 値を返す.

なお, semantics は dcmpl バイトコード命令に合わせている.
そのためどちらかが NaN 値であれば -1 が返される.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Compare 2 double values, returning integer value (-1, 0 or 1).
    // This implements the Java bytecode dcmpl, so unordered returns -1.
    // Operands may not commute.
    class CmpD3Node : public CmpDNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).

なお BoolNode と組み合わせられない (= condition code を返さない) ので正確には Cmp*Node ではない.
そのため _class_id フィールドには Class_Cmp ではなく Class_Sub を持つようにしている.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpD3Node( Node *in1, Node *in2 ) : CmpDNode(in1,in2) {
        // Since it is not consumed by Bools, it is not really a Cmp.
        init_class_id(Class_Sub);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCmpD3Node.html) for details

---
## <a name="noYyR9k3zL" id="noYyR9k3zL">BoolNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.

Cmp*Node が出力する condition code から bool 値への変換を表す Node クラス
(Cmp*Node で condition code を生成する時点では比較条件(equal, greater than, 等)は指定されておらず, 
BoolNode で bool へ変換する際に指定する).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // A Node to convert a Condition Codes to a Logical result.
    class BoolNode : public Node {
```

### 使われ方(Usage)
#### 使用例(usage examples)

```cpp
    ((cite: hotspot/src/share/vm/opto/graphKit.cpp))
        Node* chk = _gvn.transform( new (C, 3) CmpINode(should_post_flag, intcon(0)) );
        Node* tst = _gvn.transform( new (C, 2) BoolNode(chk, BoolTest::eq) );
```

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される). 
control input 以外の入力ノードは, 変換対象の condition code を示す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      BoolNode( Node *cc, BoolTest::mask t): _test(t), Node(0,cc) {
        init_class_id(Class_Bool);
      }
```




### 詳細(Details)
See: [here](../doxygen/classBoolNode.html) for details

---
## <a name="noGfrwe4sF" id="noGfrwe4sF">AbsNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.
「絶対値演算」を表す全ての Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Abstract class for absolute value.  Mostly used to get a handy wrapper
    // for finding this pattern in the graph.
    class AbsNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される). 


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      AbsNode( Node *value ) : Node(0,value) {}
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classAbsNode.html) for details

---
## <a name="noP2Pohcqi" id="noP2Pohcqi">AbsINode</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

AbsNode クラスの具象サブクラスの1つ.
このクラスは int 値を対象とした絶対値演算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Absolute value an integer.  Since a naive graph involves control flow, we
    // "match" it in the ideal world (so the control flow can be removed).
    class AbsINode : public AbsNode {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される). 


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      AbsINode( Node *in1 ) : AbsNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classAbsINode.html) for details

---
## <a name="noYoRcZxWc" id="noYoRcZxWc">AbsFNode</a>

### 概要(Summary)
AbsNode クラスの具象サブクラスの1つ.
このクラスは float 値を対象とした絶対値演算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Absolute value a float, a common float-point idiom with a cheap hardware
    // implemention on most chips.  Since a naive graph involves control flow, we
    // "match" it in the ideal world (so the control flow can be removed).
    class AbsFNode : public AbsNode {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される). 


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      AbsFNode( Node *in1 ) : AbsNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classAbsFNode.html) for details

---
## <a name="noKxUKseR2" id="noKxUKseR2">AbsDNode</a>

### 概要(Summary)
AbsNode クラスの具象サブクラスの1つ.
このクラスは double 値を対象とした絶対値演算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Absolute value a double, a common float-point idiom with a cheap hardware
    // implemention on most chips.  Since a naive graph involves control flow, we
    // "match" it in the ideal world (so the control flow can be removed).
    class AbsDNode : public AbsNode {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される). 


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      AbsDNode( Node *in1 ) : AbsNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classAbsDNode.html) for details

---
## <a name="no33uJpfIq" id="no33uJpfIq">CmpLTMaskNode</a>

### 概要(Summary)
PhiNode の最適化のための Node クラス.

2つの入力ノード間に p < q の関係が成り立てば -1, そうでなければ 0 を返す.

"(P < Q) ? X+Y : X" というパターンの式を "(sgn(P-Q))&Y) + X" に置き換えるために使用される.

(なお, このノードは (名前に反して) CmpNode のサブクラスではなく Node のサブクラス)


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // If p < q, return -1 else return 0.  Nice for flow-free idioms.
    class CmpLTMaskNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
is_cond_add() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
PhiNode::Ideal()
-> is_cond_add()
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CmpLTMaskNode( Node *p, Node *q ) : Node(0, p, q) {}
```




### 詳細(Details)
See: [here](../doxygen/classCmpLTMaskNode.html) for details

---
## <a name="nobQtZjI2k" id="nobQtZjI2k">NegNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.
「符号反転処理」を表す全ての Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    class NegNode : public Node {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      NegNode( Node *in1 ) : Node(0,in1) {}
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classNegNode.html) for details

---
## <a name="noDy1yIpRW" id="noDy1yIpRW">NegFNode</a>

### 概要(Summary)
NegNode クラスの具象サブクラスの1つ.
このクラスは float 値に対する符号反転用.

なおコメントによると
「浮動小数の場合, 符号反転はゼロからの減算で代用できないので別の Node を作った.
駄目なのは 0.0 の反転で, -0.0 になるべきだがゼロからの減算だと +0.0 になる.
なお, 符号反転を減算で置き換えることは出来ないが, 逆に減算を符号反転と加算で置き換えることは出来る」, 
とのこと.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Negate value a float.  Negating 0.0 returns -0.0, but subtracting from
    // zero returns +0.0 (per JVM spec on 'fneg' bytecode).  As subtraction
    // cannot be used to replace negation we have to implement negation as ideal
    // node; note that negation and addition can replace subtraction.
    class NegFNode : public NegNode {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      NegFNode( Node *in1 ) : NegNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classNegFNode.html) for details

---
## <a name="nolmRn74wb" id="nolmRn74wb">NegDNode</a>

### 概要(Summary)
NegNode クラスの具象サブクラスの1つ.
このクラスは double 値に対する符号反転用.

なおコメントによると
「浮動小数の場合, 符号反転はゼロからの減算で代用できないので別の Node を作った.
駄目なのは 0.0 の反転で, -0.0 になるべきだがゼロからの減算だと +0.0 になる.
なお, 符号反転を減算で置き換えることは出来ないが, 逆に減算を符号反転と加算で置き換えることは出来る」, 
とのこと.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Negate value a double.  Negating 0.0 returns -0.0, but subtracting from
    // zero returns +0.0 (per JVM spec on 'dneg' bytecode).  As subtraction
    // cannot be used to replace negation we have to implement negation as ideal
    // node; note that negation and addition can replace subtraction.
    class NegDNode : public NegNode {
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      NegDNode( Node *in1 ) : NegNode(in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classNegDNode.html) for details

---
## <a name="nox191LHVV" id="nox191LHVV">CosDNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.
double 値に対する cos (コサイン) 演算を表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Cosinus of a double
    class CosDNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_trig() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_math_native()
      -> LibraryCallKit::inline_trig() 
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      CosDNode( Node *in1  ) : Node(0, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classCosDNode.html) for details

---
## <a name="noeiRxF8sG" id="noeiRxF8sG">SinDNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.
double 値に対する sin (サイン) 演算を表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Sinus of a double
    class SinDNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_trig() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_math_native()
      -> LibraryCallKit::inline_trig() 
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      SinDNode( Node *in1  ) : Node(0, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classSinDNode.html) for details

---
## <a name="nooMjjMZdc" id="nooMjjMZdc">TanDNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.
double 値に対する tan (タンジェント) 演算を表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // tangens of a double
    class TanDNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_trig() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_math_native()
      -> LibraryCallKit::inline_trig() 
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      TanDNode(Node *in1  ) : Node(0, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classTanDNode.html) for details

---
## <a name="notUO5uOKl" id="notUO5uOKl">AtanDNode</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

2つの double 値に対する atan2 (アークタンジェント) 演算を表す模様
(LibraryCallKit::inline_math_native() 内に該当する case 節だけはあるが, まだ未実装とのこと)


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // arcus tangens of a double
    class AtanDNode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      AtanDNode(Node *c, Node *in1, Node *in2  ) : Node(c, in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classAtanDNode.html) for details

---
## <a name="nobRqZCB7o" id="nobRqZCB7o">SqrtDNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.
double 値に対する sqrt (平方根) 演算を表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // square root a double
    class SqrtDNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_sqrt() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_math_native()
      -> LibraryCallKit::inline_sqrt()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない
(control input が指定できるのは, 境界値での挙動が java.lang.Math.sqrt() とは異なるアーキテクチャ向けだと思われる).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      SqrtDNode(Node *c, Node *in1  ) : Node(c, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classSqrtDNode.html) for details

---
## <a name="novZ1Qj5tO" id="novZ1Qj5tO">ExpDNode</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

double 値に対する exp (指数) 演算を表す模様
(LibraryCallKit::inline_math_native() 内に該当する case 節だけはあるが, 
x86 用の AD ファイルのバグで一時的に無効化されており現在は使われていない).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    //  Exponentiate a double
    class ExpDNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_exp() 内で(のみ)生成されている.

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない
(control input が指定できるのは, 境界値での挙動が java.lang.Math.exp() とは異なるアーキテクチャ向けだと思われる).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      ExpDNode( Node *c, Node *in1 ) : Node(c, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classExpDNode.html) for details

---
## <a name="no3-JIH33p" id="no3-JIH33p">LogDNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.
double 値に対する log_e (自然対数) 演算を表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Log_e of a double
    class LogDNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_trans() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_math_native()
      -> LibraryCallKit::inline_trans()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      LogDNode( Node *in1 ) : Node(0, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classLogDNode.html) for details

---
## <a name="norCSV-IRQ" id="norCSV-IRQ">Log10DNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.
double 値に対する log_10 (常用対数) 演算を表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Log_10 of a double
    class Log10DNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_trans() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_math_native()
      -> LibraryCallKit::inline_trans()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      Log10DNode( Node *in1 ) : Node(0, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classLog10DNode.html) for details

---
## <a name="noRFzeWbtb" id="noRFzeWbtb">PowDNode</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

double 値に対する power (累乗) 演算を表す模様
(LibraryCallKit::inline_math_native() 内に該当する case 節だけはあるが, 
x86 用の AD ファイルのバグで一時的に無効化されており現在は使われていない).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // Raise a double to a double power
    class PowDNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_pow() 内で(のみ)生成されている.

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない
(control input が指定できるのは, 境界値での挙動が java.lang.Math.pow() とは異なるアーキテクチャ向けだと思われる).


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      PowDNode(Node *c, Node *in1, Node *in2  ) : Node(c, in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classPowDNode.html) for details

---
## <a name="noVphMlG-3" id="noVphMlG-3">ReverseBytesINode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.
int 値に対するバイト順の反転演算(java.lang.Integer.reverseBytes())を表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // reverse bytes of an integer
    class ReverseBytesINode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_reverseBytes() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_reverseBytes()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      ReverseBytesINode(Node *c, Node *in1) : Node(c, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classReverseBytesINode.html) for details

---
## <a name="noc6op4gEp" id="noc6op4gEp">ReverseBytesLNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.
long 値に対するバイト順の反転演算(java.lang.Long.reverseBytes())を表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // reverse bytes of a long
    class ReverseBytesLNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_reverseBytes() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_reverseBytes()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      ReverseBytesLNode(Node *c, Node *in1) : Node(c, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classReverseBytesLNode.html) for details

---
## <a name="noifYl1RzB" id="noifYl1RzB">ReverseBytesUSNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.
unsigned short (= char) 値に対するバイト順の反転演算(java.lang.Character.reverseBytes())を表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // reverse bytes of an unsigned short / char
    class ReverseBytesUSNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_reverseBytes() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_reverseBytes()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      ReverseBytesUSNode(Node *c, Node *in1) : Node(c, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classReverseBytesUSNode.html) for details

---
## <a name="no48QvolG1" id="no48QvolG1">ReverseBytesSNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.
short 値に対するバイト順の反転演算(java.lang.Short.reverseBytes())を表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
    // reverse bytes of a short
    class ReverseBytesSNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_reverseBytes() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_reverseBytes()
```

### 内部構造(Internal structure)
単項演算を表すノードなので (control input も含めて) 2つの入力ノードを持つ. 

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない.


```cpp
    ((cite: hotspot/src/share/vm/opto/subnode.hpp))
      ReverseBytesSNode(Node *c, Node *in1) : Node(c, in1) {}
```




### 詳細(Details)
See: [here](../doxygen/classReverseBytesSNode.html) for details

---
