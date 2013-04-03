---
layout: default
title: SuperWord クラス(とその補助クラス) (DepEdge, DepMem, DepGraph, DepPreds, DepSuccs, SWNodeInfo, SuperWord, SWPointer, OrderedPair)
---
[Top](../index.html)

#### SuperWord クラス(とその補助クラス) (DepEdge, DepMem, DepGraph, DepPreds, DepSuccs, SWNodeInfo, SuperWord, SWPointer, OrderedPair)

これらは, C2 JIT Compiler 用のクラス.
より具体的に言うと, ループのベクトル化処理で使用される補助クラス.


### クラス一覧(class list)

  * [SuperWord](#noomiJ4wdu)
  * [DepGraph](#nocyFtNMLb)
  * [DepMem](#noYGxtfmCU)
  * [DepEdge](#noqNdNHSRy)
  * [DepPreds](#no1mNOk1PL)
  * [DepSuccs](#novuAYf2PV)
  * [SWNodeInfo](#noYpQQj1zO)
  * [SWPointer](#no0SJkvx7S)
  * [OrderedPair](#noTM3g0Yr9)


---
## <a name="noomiJ4wdu" id="noomiJ4wdu">SuperWord</a>

### 概要(Summary)
PhaseIdealLoop クラス内で使用される補助クラス(ResourceObjクラス).
ループのベクトル化 (= ベクトル演算(SIMD演算)への変形処理, vectorization) を行う.

(なお, このクラスは ResourceObj クラスだが局所変数としてのみ生成されている)


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
    //
    //                  S U P E R W O R D   T R A N S F O R M
    //
    // SuperWords are short, fixed length vectors.
    //
    // Algorithm from:
    //
    // Exploiting SuperWord Level Parallelism with
    //   Multimedia Instruction Sets
    // by
    //   Samuel Larsen and Saman Amarasighe
    //   MIT Laboratory for Computer Science
    // date
    //   May 2000
    // published in
    //   ACM SIGPLAN Notices
    //   Proceedings of ACM PLDI '00,  Volume 35 Issue 5
    //
    // Definition 3.1 A Pack is an n-tuple, <s1, ...,sn>, where
    // s1,...,sn are independent isomorphic statements in a basic
    // block.
    //
    // Definition 3.2 A PackSet is a set of Packs.
    //
    // Definition 3.3 A Pair is a Pack of size two, where the
    // first statement is considered the left element, and the
    // second statement is considered the right element.
```

なおコメントによると, Larsen らの "Superword Level Parallelism" を実装したもの, とのこと
(ところで "Saman Amarasighe" は "Saman Amarasinghe" の typo?? #TODO)

        Samuel Larsen and Saman Amarasighe, MIT Laboratory for Computer Science,
        Exploiting SuperWord Level Parallelism with Multimedia Instruction Sets,
        PLDI 2000


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
    // -----------------------------SuperWord---------------------------------
    // Transforms scalar operations into packed (superword) operations.
    class SuperWord : public ResourceObj {
```

### 使われ方(Usage)
PhaseIdealLoop::build_and_optimize() 内で(のみ)(局所変数として)生成されている.

### 備考(Notes)
なお, このクラスは product オプションである UseSuperWord が true の場合にしか使用されない. 
ただしデフォルトでは true.


```
    ((cite: hotspot/src/share/vm/opto/c2_globals.hpp))
      product(bool, UseSuperWord, true,                                         \
              "Transform scalar operations into superword operations")          \
```




### 詳細(Details)
See: [here](../doxygen/classSuperWord.html) for details

---
## <a name="nocyFtNMLb" id="nocyFtNMLb">DepGraph</a>

### 概要(Summary)
SuperWord クラス内で使用される補助クラス(ValueObjクラス).

メモリアクセス間の依存グラフ(dependency graph)を表す.


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
    //------------------------------DepGraph---------------------------
    class DepGraph VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 SuperWord オブジェクトの _dg フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(SuperWord クラスの _dg フィールドは, ポインタ型ではなく実体なので,
 SuperWord オブジェクトの生成時に一緒に生成される)




### 詳細(Details)
See: [here](../doxygen/classDepGraph.html) for details

---
## <a name="noYGxtfmCU" id="noYGxtfmCU">DepMem</a>

### 概要(Summary)
DepGraph クラス内で使用される補助クラス.

依存グラフ(dependency graph)内の node を表すクラス.
1つの DepMem オブジェクトが 1つの node に対応する.


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
    //------------------------------DepMem---------------------------
    // A node in the dependence graph.  _in_head starts the threaded list of
    // incoming edges, and _out_head starts the list of outgoing edges.
    class DepMem : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 DepGraph オブジェクトの _root フィールド

* 各 DepGraph オブジェクトの _tail フィールド

* 各 DepGraph オブジェクトの _map フィールド
  
  (正確には, このフィールドは DepMem の GrowableArray を格納するフィールド.
  各 Node オブジェクト (正確にはその Node::_idx フィールドの値) から対応する DepMem オブジェクトへの写像になっている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* DepGraph::DepGraph()
* DepGraph::make_node()




### 詳細(Details)
See: [here](../doxygen/classDepMem.html) for details

---
## <a name="noqNdNHSRy" id="noqNdNHSRy">DepEdge</a>

### 概要(Summary)
DepGraph クラス内で使用される補助クラス.

依存グラフ(dependency graph)内の edge を表すクラス.
1つの DepEdge オブジェクトが 1つの edge に対応する.


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
    //------------------------------DepEdge---------------------------
    // An edge in the dependence graph.  The edges incident to a dependence
    // node are threaded through _next_in for incoming edges and _next_out
    // for outgoing edges.
    class DepEdge : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 DepMem オブジェクトの _out_head フィールド
* 各 DepMem オブジェクトの _in_head フィールド

#### 生成箇所(where its instances are created)
DepGraph::make_edge() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classDepEdge.html) for details

---
## <a name="no1mNOk1PL" id="no1mNOk1PL">DepPreds</a>

### 概要(Summary)
SuperWord クラス内で使用される補助クラス.

指定の Node オブジェクトについて, 
対応する DepMem の DepMem::_in_head 中の edge が指す Node, 
もしくはその Node の Node::_in 配列から指されている Node をたどるためのイテレータクラス(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
    //------------------------------DepPreds---------------------------
    // Iterator over predecessors in the dependence graph and
    // non-memory-graph inputs of ideal nodes.
    class DepPreds : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* SuperWord::independent_path()
* SuperWord::compute_max_depth()




### 詳細(Details)
See: [here](../doxygen/classDepPreds.html) for details

---
## <a name="novuAYf2PV" id="novuAYf2PV">DepSuccs</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

指定の Node オブジェクトについて, 
対応する DepMem の DepMem::_out_head 中の edge が指す Node, 
もしくはその Node の Node::_out 配列から指されている Node をたどるためのイテレータクラス(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
    //------------------------------DepSuccs---------------------------
    // Iterator over successors in the dependence graph and
    // non-memory-graph outputs of ideal nodes.
    class DepSuccs : public StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classDepSuccs.html) for details

---
## <a name="noYpQQj1zO" id="noYpQQj1zO">SWNodeInfo</a>

### 概要(Summary)
SuperWord クラス内で使用される補助クラス.

各 Node オブジェクトについて SuperWord クラスが必要とする情報を格納しておくためのクラス.
1つの SWNodeInfo オブジェクトが 1つの Node オブジェクトに対応する.


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
    // -----------------------------SWNodeInfo---------------------------------
    // Per node info needed by SuperWord
    class SWNodeInfo VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 SuperWord オブジェクトの _node_info フィールド

  (正確には, このフィールドは SWNodeInfo の GrowableArray を格納するフィールド.
  各 Node オブジェクト (正確にはその Node::_idx フィールドの値を SuperWord::bb_idx() で写像した値) 
  から対応する SWNodeInfo オブジェクトへの写像になっている)

* SWNodeInfo クラスの initial フィールド (static フィールド)
  
#### 生成箇所(where its instances are created)
* (SuperWord クラスの _node_info フィールドは, ポインタ型ではなく実体なので,
  SuperWord オブジェクトの生成時に一緒に生成される)

* (SWNodeInfo クラスの initial フィールドは, ポインタ型ではなく実体なので,
  初期段階で自動的に生成される)

### 内部構造(Internal structure)
単なる構造体のようなクラス.
内部には以下の public フィールドのみを持つ (そしてメソッドはない).


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
      int         _alignment; // memory alignment for a node
      int         _depth;     // Max expression (DAG) depth from block start
      const Type* _velt_type; // vector element type
      Node_List*  _my_pack;   // pack containing this node
```




### 詳細(Details)
See: [here](../doxygen/classSWNodeInfo.html) for details

---
## <a name="no0SJkvx7S" id="no0SJkvx7S">SWPointer</a>

### 概要(Summary)
SuperWord クラス内で使用される一時オブジェクト(ValueObjクラス).

MemNode からそのアクセス先のアドレス情報を抽出するためのクラス.

(なお, このクラスは StackObjクラスではないが現状では局所変数としてのみ生成されている)


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
    //------------------------------SWPointer---------------------------
    // Information about an address for dependence checking and vector alignment
    class SWPointer VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* SWPointer::scaled_iv()
* SuperWord::find_adjacent_refs()
* SuperWord::find_align_to_ref()
* SuperWord::dependence_graph()
* SuperWord::are_adjacent_refs()
* SuperWord::memory_alignment()
* SuperWord::align_initial_loop_index()




### 詳細(Details)
See: [here](../doxygen/classSWPointer.html) for details

---
## <a name="noTM3g0Yr9" id="noTM3g0Yr9">OrderedPair</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか使用されない).

(なお, このクラスは (デバッグ時であることに加えて) SuperWordRTDepCheck オプションが指定されている場合にしか使用されない)

SuperWord クラス内で使用される補助クラス(ValueObjクラス).
2 個の Node オブジェクトからなる組(pair)を表す.


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
    //------------------------------OrderedPair---------------------------
    // Ordered pair of Node*.
    class OrderedPair VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 SuperWord オブジェクトの _disjoint_ptrs フィールド

  (正確には, このフィールドは OrderedPair の GrowableArray を格納するフィールド.
  この中に, その SuperWord 内で生成された全ての OrderedPair オブジェクトが格納されている)

* OrderedPair クラスの initial フィールド (static フィールド)

#### 生成箇所(where its instances are created)
SuperWord::dependence_graph() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
SuperWord::dependence_graph() 内で(のみ)使用されている (ただし #ifndef PRODUCT の場合にのみ使用される).

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```
    ((cite: hotspot/src/share/vm/opto/superword.hpp))
      Node* _p1;
      Node* _p2;
```




### 詳細(Details)
See: [here](../doxygen/classOrderedPair.html) for details

---
