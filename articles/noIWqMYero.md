---
layout: default
title: 乗算に関する高レベル中間語(Ideal)クラス (MulNode, MulINode, MulLNode, MulFNode, MulDNode, MulHiLNode, AndINode, AndLNode, LShiftINode, LShiftLNode, RShiftINode, RShiftLNode, URShiftINode, URShiftLNode)
---
[Top](../index.html)

#### 乗算に関する高レベル中間語(Ideal)クラス (MulNode, MulINode, MulLNode, MulFNode, MulDNode, MulHiLNode, AndINode, AndLNode, LShiftINode, LShiftLNode, RShiftINode, RShiftLNode, URShiftINode, URShiftLNode)

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, 主に「乗算」を表すクラス.

なお単なる数値の乗算だけでなく, 環(ring)上の乗算とみなせるものは全てここで扱う 
(例: AND 演算など).

(ついでに左シフトも混ざっているがまぁこれは掛け算と見なせるか. 
 だけど右シフトはどちらかというと割り算のような気もするが... 論理右シフトも unsigned な割り算だが...)


### クラス一覧(class list)

  * [MulNode](#nomsxQSSLs)
  * [MulINode](#noWoy9_M3X)
  * [MulLNode](#noRdzUFJFN)
  * [MulFNode](#noBHA3cg0h)
  * [MulDNode](#noMjmdHfpC)
  * [MulHiLNode](#now6IVXvoy)
  * [AndINode](#no0rvAsBAp)
  * [AndLNode](#no_JzgDvVS)
  * [LShiftINode](#noiPSbYba0)
  * [LShiftLNode](#nouXk0HiWc)
  * [RShiftINode](#nov6FraIBk)
  * [RShiftLNode](#noQ-ql2hx7)
  * [URShiftINode](#noLBydmEML)
  * [URShiftLNode](#nobMb4ma6v)


---
## <a name="nomsxQSSLs" id="nomsxQSSLs">MulNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.
「乗算」を表す全ての Node クラスの基底クラス.

(なお単なる数値の乗算だけでなく, 環(ring)上の乗算とみなせるものは全てこのクラスのサブクラスで扱う)
(例: AND 演算など).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Classic MULTIPLY functionality.  This covers all the usual 'multiply'
    // behaviors for an algebraic ring.  Multiply-integer, multiply-float,
    // multiply-double, and binary-and are all inherited from this class.  The
    // various identity values are supplied by virtual functions.
    class MulNode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      MulNode( Node *in1, Node *in2 ): Node(0,in1,in2) {
        init_class_id(Class_Mul);
      }
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classMulNode.html) for details

---
## <a name="noWoy9_M3X" id="noWoy9_M3X">MulINode</a>

### 概要(Summary)
MulNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の乗算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Multiply 2 integers
    class MulINode : public MulNode {
```

なお MulXNode という型も使われるが, 
`#ifdef _LP64` でない場合は, これは MulINode の別名.


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define MulXNode     MulINode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      MulINode( Node *in1, Node *in2 ) : MulNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classMulINode.html) for details

---
## <a name="noRdzUFJFN" id="noRdzUFJFN">MulLNode</a>

### 概要(Summary)
MulNode クラスの具象サブクラスの1つ.
このクラスは long 値同士の乗算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Multiply 2 longs
    class MulLNode : public MulNode {
```

なお MulXNode という型も使われるが, 
`#ifdef _LP64` の場合は, これは MulLNode の別名.


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define MulXNode     MulLNode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      MulLNode( Node *in1, Node *in2 ) : MulNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classMulLNode.html) for details

---
## <a name="noBHA3cg0h" id="noBHA3cg0h">MulFNode</a>

### 概要(Summary)
MulNode クラスの具象サブクラスの1つ.
このクラスは float 値同士の乗算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Multiply 2 floats
    class MulFNode : public MulNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      MulFNode( Node *in1, Node *in2 ) : MulNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classMulFNode.html) for details

---
## <a name="noMjmdHfpC" id="noMjmdHfpC">MulDNode</a>

### 概要(Summary)
MulNode クラスの具象サブクラスの1つ.
このクラスは double 値同士の乗算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Multiply 2 doubles
    class MulDNode : public MulNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      MulDNode( Node *in1, Node *in2 ) : MulNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classMulDNode.html) for details

---
## <a name="now6IVXvoy" id="now6IVXvoy">MulHiLNode</a>

### 概要(Summary)
long 値に対する除算(DivLNode, ModLNode)を最適化するための Node クラス.

MulNode クラスの具象サブクラスの1つ.
このクラスは 64bit 同士の乗算結果の上位64bitを表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Upper 64 bits of a 64 bit by 64 bit multiply
    class MulHiLNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
long_by_long_mulhi() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
DivLNode::Ideal()
-> transform_long_divide()
   -> long_by_long_mulhi() 

ModLNode::Ideal()
-> transform_long_divide()
   -> long_by_long_mulhi() 
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      MulHiLNode( Node *in1, Node *in2 ) : Node(0,in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classMulHiLNode.html) for details

---
## <a name="no0rvAsBAp" id="no0rvAsBAp">AndINode</a>

### 概要(Summary)
MulINode クラスのサブクラス.
このクラスは int 値同士の論理積(ビットAND)演算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Logically AND 2 integers.  Included with the MUL nodes because it inherits
    // all the behavior of multiplication on a ring.
    class AndINode : public MulINode {
```

なお AndXNode という型も使われるが, 
`#ifdef _LP64` でない場合は, これは AndINode の別名.


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define AndXNode     AndINode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      AndINode( Node *in1, Node *in2 ) : MulINode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classAndINode.html) for details

---
## <a name="no_JzgDvVS" id="no_JzgDvVS">AndLNode</a>

### 概要(Summary)
MulLNode クラスのサブクラス.
このクラスは long 値同士の論理積(ビットAND)演算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Logically AND 2 longs.  Included with the MUL nodes because it inherits
    // all the behavior of multiplication on a ring.
    class AndLNode : public MulLNode {
```

なお AndXNode という型も使われるが, 
`#ifdef _LP64` の場合は, これは AndLNode の別名.


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define AndXNode     AndLNode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      AndLNode( Node *in1, Node *in2 ) : MulLNode(in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classAndLNode.html) for details

---
## <a name="noiPSbYba0" id="noiPSbYba0">LShiftINode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは int 値に対する左シフト演算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Logical shift left
    class LShiftINode : public Node {
```

なお LShiftXNode という型も使われるが, `#ifdef _LP64` でない場合は, これは LShiftINode の別名.


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define LShiftXNode  LShiftINode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      LShiftINode( Node *in1, Node *in2 ) : Node(0,in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classLShiftINode.html) for details

---
## <a name="nouXk0HiWc" id="nouXk0HiWc">LShiftLNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは long 値に対する左シフト演算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Logical shift left
    class LShiftLNode : public Node {
```

なお LShiftXNode という型も使われるが, `#ifdef _LP64` の場合は, これは LShiftLNode の別名.   


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define LShiftXNode  LShiftLNode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      LShiftLNode( Node *in1, Node *in2 ) : Node(0,in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classLShiftLNode.html) for details

---
## <a name="nov6FraIBk" id="nov6FraIBk">RShiftINode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは int 値に対する算術右シフト演算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Signed shift right
    class RShiftINode : public Node {
```

なお RShiftXNode という型も使われるが, `#ifdef _LP64` でない場合は, これは RShiftINode の別名.


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define RShiftXNode  RShiftINode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      RShiftINode( Node *in1, Node *in2 ) : Node(0,in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classRShiftINode.html) for details

---
## <a name="noQ-ql2hx7" id="noQ-ql2hx7">RShiftLNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは long 値に対する算術右シフト演算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Signed shift right
    class RShiftLNode : public Node {
```

なお RShiftXNode という型も使われるが, `#ifdef _LP64` の場合は, これは RShiftLNode の別名.   


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define RShiftXNode  RShiftLNode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      RShiftLNode( Node *in1, Node *in2 ) : Node(0,in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classRShiftLNode.html) for details

---
## <a name="noLBydmEML" id="noLBydmEML">URShiftINode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは int 値に対する論理右シフト演算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Logical shift right
    class URShiftINode : public Node {
```

なお URShiftXNode という型も使われるが, `#ifdef _LP64` でない場合は, これは URShiftINode の別名.


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define URShiftXNode URShiftINode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      URShiftINode( Node *in1, Node *in2 ) : Node(0,in1,in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classURShiftINode.html) for details

---
## <a name="nobMb4ma6v" id="nobMb4ma6v">URShiftLNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは long 値に対する論理右シフト演算用.


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
    // Logical shift right
    class URShiftLNode : public Node {
```

なお URShiftXNode という型も使われるが, `#ifdef _LP64` の場合は, これは URShiftLNode の別名.   


```cpp
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define URShiftXNode URShiftLNode
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.
ただし control input は常に空 (0 が設定される).


```cpp
    ((cite: hotspot/src/share/vm/opto/mulnode.hpp))
      URShiftLNode( Node *in1, Node *in2 ) : Node(0,in1,in2) {}
```





### 詳細(Details)
See: [here](../doxygen/classURShiftLNode.html) for details

---
