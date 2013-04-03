---
layout: default
title: 除算に関する高レベル中間語(Ideal)クラス (DivINode, DivLNode, DivFNode, DivDNode, ModINode, ModLNode, ModFNode, ModDNode, DivModNode, DivModINode, DivModLNode)
---
[Top](../index.html)

#### 除算に関する高レベル中間語(Ideal)クラス (DivINode, DivLNode, DivFNode, DivDNode, ModINode, ModLNode, ModFNode, ModDNode, DivModNode, DivModINode, DivModLNode)

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, 主に「除算」を表すクラス.

(なお, 他の四則演算と異なり, Div や Mod 自体には abstract な基底クラスは存在しない. ただし DivMod には存在する)


### クラス一覧(class list)

  * [DivINode](#noXxAp_Xlb)
  * [DivLNode](#notMiyRj5X)
  * [DivFNode](#noznEVKrav)
  * [DivDNode](#no_o0vkIh4)
  * [ModINode](#nopXX4eUO4)
  * [ModLNode](#noYNosWHmq)
  * [ModFNode](#nouIyCREim)
  * [ModDNode](#noSJm33cik)
  * [DivModNode](#noFENo454d)
  * [DivModINode](#nozEFy2lIE)
  * [DivModLNode](#noLarJBcCb)


---
## <a name="noXxAp_Xlb" id="noXxAp_Xlb">DivINode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは int 値同士の除算用.

なお, semantics は idiv バイトコード命令に合わせている.
そのため MinInt/-1 == MinInt としなければいけないことに注意.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
    // Integer division
    // Note: this is division as defined by JVMS, i.e., MinInt/-1 == MinInt.
    // On processors which don't naturally support this special case (e.g., x86),
    // the matcher or runtime system must take care of this.
    class DivINode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる 
(これは 0 割りを検出するための 0 チェックに制御依存するため.
 もちろん必ず 0 ではないと分かっているパスでは省略されるため, control input が 0 になることもある)


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
      DivINode( Node *c, Node *dividend, Node *divisor ) : Node(c, dividend, divisor ) {}
```




### 詳細(Details)
See: [here](../doxygen/classDivINode.html) for details

---
## <a name="notMiyRj5X" id="notMiyRj5X">DivLNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは long 値同士の除算用.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
    // Long division
    class DivLNode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる 
(これは 0 割りを検出するための 0 チェックに制御依存するため.
 もちろん必ず 0 ではないと分かっているパスでは省略されるため, control input が 0 になることもある)


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
      DivLNode( Node *c, Node *dividend, Node *divisor ) : Node(c, dividend, divisor ) {}
```




### 詳細(Details)
See: [here](../doxygen/classDivLNode.html) for details

---
## <a name="noznEVKrav" id="noznEVKrav">DivFNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは float 値同士の除算用.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
    // Float division
    class DivFNode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
      DivFNode( Node *c, Node *dividend, Node *divisor ) : Node(c, dividend, divisor) {}
```




### 詳細(Details)
See: [here](../doxygen/classDivFNode.html) for details

---
## <a name="no_o0vkIh4" id="no_o0vkIh4">DivDNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは double 値同士の除算用.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
    // Double division
    class DivDNode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
      DivDNode( Node *c, Node *dividend, Node *divisor ) : Node(c,dividend, divisor) {}
```




### 詳細(Details)
See: [here](../doxygen/classDivDNode.html) for details

---
## <a name="nopXX4eUO4" id="nopXX4eUO4">ModINode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは int 値同士の剰余演算用.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
    // Integer modulus
    class ModINode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる 
(これは 0 割りを検出するための 0 チェックに制御依存するため.
 もちろん必ず 0 ではないと分かっているパスでは省略されるため, control input が 0 になることもある)


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
      ModINode( Node *c, Node *in1, Node *in2 ) : Node(c,in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classModINode.html) for details

---
## <a name="noYNosWHmq" id="noYNosWHmq">ModLNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは long 値同士の剰余演算用.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
    // Long modulus
    class ModLNode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる 
(これは 0 割りを検出するための 0 チェックに制御依存するため.
 もちろん必ず 0 ではないと分かっているパスでは省略されるため, control input が 0 になることもある)


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
      ModLNode( Node *c, Node *in1, Node *in2 ) : Node(c,in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classModLNode.html) for details

---
## <a name="nouIyCREim" id="nouIyCREim">ModFNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは float 値同士の剰余演算用.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
    // Float Modulus
    class ModFNode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
      ModFNode( Node *c, Node *in1, Node *in2 ) : Node(c,in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classModFNode.html) for details

---
## <a name="noSJm33cik" id="noSJm33cik">ModDNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
このクラスは double 値同士の剰余演算用.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
    // Double Modulus
    class ModDNode : public Node {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.
ただし現状では 0 しか指定されていない.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
      ModDNode( Node *c, Node *in1, Node *in2 ) : Node(c, in1, in2) {}
```




### 詳細(Details)
See: [here](../doxygen/classModDNode.html) for details

---
## <a name="noFENo454d" id="noFENo454d">DivModNode</a>

### 概要(Summary)
MultiNode クラスのサブクラスの1つ.
出力として「除算の結果と剰余算の結果の双方を出す」 Node クラスの基底クラス.

(<= 除算と剰余算は共通部が多いので, 両方の結果が必要な場合は一度に計算すると高速化できる.
 DivModNode はそれを表現するためのクラス)

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
    // Division with remainder result.
    class DivModNode : public MultiNode {
```

### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
      DivModNode( Node *c, Node *dividend, Node *divisor );
```




### 詳細(Details)
See: [here](../doxygen/classDivModNode.html) for details

---
## <a name="nozEFy2lIE" id="nozEFy2lIE">DivModINode</a>

### 概要(Summary)
DivModNode クラスの具象サブクラスの1つ.
このクラスは int 値同士の除算/剰余演算用.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
    // Integer division with remainder result.
    class DivModINode : public DivModNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
DivModINode::make() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Compile::Optimize()
-> Compile::final_graph_reshaping()
   -> final_graph_reshaping_walk()
      -> final_graph_reshaping_impl()
         -> DivModINode::make()
```

(<= final_graph_reshaping_impl() 内で引数が同じ DivINode と ModINode が見つかれば, その2つがマージされて作成される)

なお, DivModINode::make() では以下のように処理が行われる.

* DivModINode の入力ノードは, 元々存在していた DivINode または ModINode の入力ノードがそのまま使われる.
* DivModINode だけでなく2つの ProjNode も併せて作られ (これらが Div の結果と Mod の結果をそれぞれ取り出す),
  元々存在していた DivINode と ModINode が ProjNode で置き換えられる)

#### 参考(for your information): DivModINode::make()
See: [here](no17119QyN.html) for details
### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる 
(これは 0 割りを検出するための 0 チェックに制御依存するため.
 もちろん必ず 0 ではないと分かっているパスでは省略されるため, control input が 0 になることもある)


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
      DivModINode( Node *c, Node *dividend, Node *divisor ) : DivModNode(c, dividend, divisor) {}
```

### 備考(Notes)
なお, このクラスは product オプションである UseDivMod が true の場合にしか作成されない.
ただしデフォルトでは true.


```
    ((cite: hotspot/src/share/vm/opto/c2_globals.hpp))
      product(bool, UseDivMod, true,                                            \
              "Use combined DivMod instruction if available")                   \
```





### 詳細(Details)
See: [here](../doxygen/classDivModINode.html) for details

---
## <a name="noLarJBcCb" id="noLarJBcCb">DivModLNode</a>

### 概要(Summary)
DivModNode クラスの具象サブクラスの1つ.
このクラスは long 値同士の除算/剰余演算用.


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
    // Long division with remainder result.
    class DivModLNode : public DivModNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
DivModLNode::make() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Compile::Optimize()
-> Compile::final_graph_reshaping()
   -> final_graph_reshaping_walk()
      -> final_graph_reshaping_impl()
         -> DivModLNode::make()
```

(<= final_graph_reshaping_impl() 内で引数が同じ DivLNode と ModLNode が見つかれば, その2つがマージされて作成される)

なお, DivModLNode::make() では以下のように処理が行われる.

* DivModLNode の入力ノードは, 元々存在していた DivLNode または ModLNode の入力ノードがそのまま使われる.
* DivModLNode だけでなく2つの ProjNode も併せて作られ (これらが Div の結果と Mod の結果をそれぞれ取り出す),
  元々存在していた DivLNode と ModLNode が ProjNode で置き換えられる)

#### 参考(for your information): DivModLNode::make()
See: [here](no17119d8T.html) for details
### 内部構造(Internal structure)
2項演算を表すノードなので, (control input も含めて) 3つの入力ノードを持つ.

なお, コンストラクタで指定することにより 0 以外の control input を設定できる 
(これは 0 割りを検出するための 0 チェックに制御依存するため.
 もちろん必ず 0 ではないと分かっているパスでは省略されるため, control input が 0 になることもある)


```
    ((cite: hotspot/src/share/vm/opto/divnode.hpp))
      DivModLNode( Node *c, Node *dividend, Node *divisor ) : DivModNode(c, dividend, divisor) {}
```

### 備考(Notes)
なお, このクラスは product オプションである UseDivMod が true の場合にしか作成されない.
ただしデフォルトでは true.


```
    ((cite: hotspot/src/share/vm/opto/c2_globals.hpp))
      product(bool, UseDivMod, true,                                            \
              "Use combined DivMod instruction if available")                   \
```




### 詳細(Details)
See: [here](../doxygen/classDivModLNode.html) for details

---
