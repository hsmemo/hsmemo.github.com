---
layout: default
title: PhaseIdealLoop クラス, 及びループに関する高レベル中間語(Ideal)クラス (LoopNode, CountedLoopNode, CountedLoopEndNode, LoopLimitNode, IdealLoopTree, PhaseIdealLoop, LoopTreeIterator)
---
[Top](../index.html)

#### PhaseIdealLoop クラス, 及びループに関する高レベル中間語(Ideal)クラス (LoopNode, CountedLoopNode, CountedLoopEndNode, LoopLimitNode, IdealLoopTree, PhaseIdealLoop, LoopTreeIterator)

これらは, C2 JIT Compiler 内の処理フェーズおよび関連する高レベル中間語を表すクラス.
より具体的に言うと, ループ最適化処理を表すクラス.

### 概要(Summary)
bytecode 命令にループを表す命令はないが, ループを検出することは最適化を行う上では極めて重要になる.
そこでパースが終わった後, C2 はループの検出を行う.


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
    //
    //                  I D E A L I Z E D   L O O P S
    //
    // Idealized loops are the set of loops I perform more interesting
    // transformations on, beyond simple hoisting.
```

パース終了時のループは「自分自身が入力となっている RegionNode」という形になっている.
C2 は regular loops の loop header となっている RegionNode を検出し, LoopNode で置き換える.

また, ループの中でも制御変数が固定の初期値から始まり固定の増分幅で変化していくもの(Fortran の DO ループ的なもの)については, 
特に最適化しやすいので特別に CountedLoopNode というノードに変換する.
なお, CountedLoopNode に関連したノードとして以下のノードがある.

* CountedLoopEndNode 
  
  CountedLoopNode が表すループの終了判定を示す特殊な IfNode.

* LoopLimitNode 
  
  CountedLoopNode の終了時の制御変数の値を表す Node
  (終値ぴったりで終わるとは限らず, 初期値と増分によっては終値を微妙に超えることがある. LoopLimitNode は正確な終了時の値を計算する).


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
    //------------------------------Counted Loops----------------------------------
    // Counted loops are all trip-counted loops, with exactly 1 trip-counter exit
    // path (and maybe some other exit paths).  The trip-counter exit is always
    // last in the loop.  The trip-counter have to stride by a constant;
    // the exit value is also loop invariant.
    
    // CountedLoopNodes and CountedLoopEndNodes come in matched pairs.  The
    // CountedLoopNode has the incoming loop control and the loop-back-control
    // which is always the IfTrue before the matching CountedLoopEndNode.  The
    // CountedLoopEndNode has an incoming control (possibly not the
    // CountedLoopNode if there is control flow in the loop), the post-increment
    // trip-counter value, and the limit.  The trip-counter value is always of
    // the form (Op old-trip-counter stride).  The old-trip-counter is produced
    // by a Phi connected to the CountedLoopNode.  The stride is constant.
    // The Op is any commutable opcode, including Add, Mul, Xor.  The
    // CountedLoopEndNode also takes in the loop-invariant limit value.
    
    // From a CountedLoopNode I can reach the matching CountedLoopEndNode via the
    // loop-back control.  From CountedLoopEndNodes I can reach CountedLoopNodes
    // via the old-trip-counter from the Op node.
```



### クラス一覧(class list)

  * [LoopNode](#noDZWLRdzP)
  * [CountedLoopNode](#nobxEgpZ_d)
  * [CountedLoopEndNode](#noSatpZXKj)
  * [LoopLimitNode](#noye7LEbQw)
  * [PhaseIdealLoop](#no90KSI6zh)
  * [IdealLoopTree](#no4z3-ZZ3y)
  * [LoopTreeIterator](#noIElp8H_g)


---
## <a name="noDZWLRdzP" id="noDZWLRdzP">LoopNode</a>

### 概要(Summary)
RegionNode クラスのサブクラス.

ループになっている制御構造を表す Node.


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
    // Simple loop header.  Fall in path on left, loop-back path on right.
    class LoopNode : public RegionNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* IdealLoopTree::beautify_loops()
* IdealLoopTree::split_outer_loop()
* PhaseIdealLoop::partial_peel()

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
PhaseIdealLoop::build_and_optimize()
-&gt; IdealLoopTree::beautify_loops()
   -&gt; IdealLoopTree::split_outer_loop()
-&gt; IdealLoopTree::iteration_split()
   -&gt; IdealLoopTree::iteration_split_impl()
      -&gt; PhaseIdealLoop::partial_peel()
</pre></div>

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り (この enum の順で入力される).

* 1番目の入力Node : control input (ただし, 自分自身を指す)
* 2番目の入力Node : ループの入口に入ってくるパスを示す control input
* 3番目の入力Node : ループのバックエッジを示す control input


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
      // Names for edge indices
      enum { Self=0, EntryControl, LoopBackControl };
```


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
      LoopNode( Node *entry, Node *backedge ) : RegionNode(3), _loop_flags(0), _unswitch_count(0) {
        init_class_id(Class_Loop);
        init_req(EntryControl, entry);
        init_req(LoopBackControl, backedge);
      }
```




### 詳細(Details)
See: [here](../doxygen/classLoopNode.html) for details

---
## <a name="nobxEgpZ_d" id="nobxEgpZ_d">CountedLoopNode</a>

### 概要(Summary)
特殊な LoopNode クラス.

制御変数が固定の初期値から始まり固定の増分幅で変化していくループを表す Node.


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
    // CountedLoopNodes head simple counted loops.  CountedLoopNodes have as
    // inputs the incoming loop-start control and the loop-back control, so they
    // act like RegionNodes.  They also take in the initial trip counter, the
    // loop-invariant stride and the loop-invariant limit value.  CountedLoopNodes
    // produce a loop-body control and the trip counter value.  Since
    // CountedLoopNodes behave like RegionNodes I still have a standard CFG model.
    
    class CountedLoopNode : public LoopNode {
```

### 使われ方(Usage)
PhaseIdealLoop::is_counted_loop() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
PhaseIdealLoop::build_and_optimize()
-&gt; IdealLoopTree::counted_loop()
   -&gt; PhaseIdealLoop::is_counted_loop()
</pre></div>

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input (ただし, 自分自身を指す)
* 2番目の入力Node : ループの入口に入ってくるパスを示す control input
* 3番目の入力Node : ループのバックエッジを示す control input


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
      CountedLoopNode( Node *entry, Node *backedge )
        : LoopNode(entry, backedge), _main_idx(0), _trip_count(max_juint),
          _profile_trip_cnt(COUNT_UNKNOWN), _unrolled_count_log2(0),
          _node_count_before_unroll(0) {
        init_class_id(Class_CountedLoop);
        // Initialize _trip_count to the largest possible value.
        // Will be reset (lower) if the loop's trip count is known.
      }
```




### 詳細(Details)
See: [here](../doxygen/classCountedLoopNode.html) for details

---
## <a name="noSatpZXKj" id="noSatpZXKj">CountedLoopEndNode</a>

### 概要(Summary)
CountedLoopNode クラスと併用される Node.

CountedLoopNode が表すループの終了判定を示す特殊な IfNode.


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
    // CountedLoopEndNodes end simple trip counted loops.  They act much like
    // IfNodes.
    class CountedLoopEndNode : public IfNode {
```

### 使われ方(Usage)
PhaseIdealLoop::is_counted_loop() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
PhaseIdealLoop::build_and_optimize()
-&gt; IdealLoopTree::counted_loop()
   -&gt; PhaseIdealLoop::is_counted_loop()
</pre></div>

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : ループの終了判定に使用する bool 値

(<= 基本的に IfNode と同じ. 
 というか PhaseIdealLoop::is_counted_loop() 内で IfNode を置き換えて作成するので, 入力はそのまま引き継ぐ).


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
      CountedLoopEndNode( Node *control, Node *test, float prob, float cnt )
        : IfNode( control, test, prob, cnt) {
        init_class_id(Class_CountedLoopEnd);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCountedLoopEndNode.html) for details

---
## <a name="noye7LEbQw" id="noye7LEbQw">LoopLimitNode</a>

### 概要(Summary)
CountedLoopNode クラスと併用される Node.

CountedLoopNode の終了時の制御変数の値を表す
(終値ぴったりで終わるとは限らず, 初期値と増分によっては終値を微妙に超えることがある. 
LoopLimitNode は正確な終了時の値を計算する).


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
    // Counted Loop limit node which represents exact final iterator value:
    // trip_count = (limit - init_trip + stride - 1)/stride
    // final_value= trip_count * stride + init_trip.
    // Use HW instructions to calculate it when it can overflow in integer.
    // Note, final_value should fit into integer since counted loop has
    // limit check: limit <= max_int-stride.
    class LoopLimitNode : public Node {
```

### 使われ方(Usage)
PhaseIdealLoop::exact_limit() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ. ただし control input は常に空 (0 が設定される). 

その他の 3つは以下の通り.

* 2番目の入力Node : 制御変数の初期値
* 3番目の入力Node : 制御変数の終値
* 4番目の入力Node : 制御変数の増分幅


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
      LoopLimitNode( Compile* C, Node *init, Node *limit, Node *stride ) : Node(0,init,limit,stride) {
        // Put it on the Macro nodes list to optimize during macro nodes expansion.
        init_flags(Flag_is_macro);
        C->add_macro_node(this);
      }
```




### 詳細(Details)
See: [here](../doxygen/classLoopLimitNode.html) for details

---
## <a name="no90KSI6zh" id="no90KSI6zh">PhaseIdealLoop</a>

### 概要(Summary)
Phase クラスの具象サブクラスの1つ.

ループの最適化を行う.


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
    // Computes the mapping from Nodes to IdealLoopTrees.  Organizes IdealLoopTrees into a
    // loop tree.  Drives the loop-based transformations on the ideal graph.
    class PhaseIdealLoop : public PhaseTransform {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* Compile::Optimize()
* PhaseIdealLoop::verify()




### 詳細(Details)
See: [here](../doxygen/classPhaseIdealLoop.html) for details

---
## <a name="no4z3-ZZ3y" id="no4z3-ZZ3y">IdealLoopTree</a>

### 概要(Summary)
PhaseIdealLoop クラス内で使用される補助クラス(ResourceObjクラス).

コード中のループ構造を (そのネスト構造も含めて) 表すためのクラス.
1つの IdealLoopTree オブジェクトが 1つのループに対応する.


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
    class IdealLoopTree : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 PhaseIdealLoop オブジェクトの _ltree_root フィールド
  
  (正確には, このフィールドは IdealLoopTree の木構造を格納するフィールド.
  IdealLoopTree オブジェクトは 
  _parent フィールド, _next フィールド, 及び _child フィールドで次の IdealLoopTree オブジェクトを指せる構造になっている.
  その PhaseIdealLoop 用の IdealLoopTree オブジェクトは全てこの木構造に格納されている)

* 各 PhaseIdealLoop オブジェクトの PhaseTransform::_nodes フィールド
   
  各ループの開始点となる Node や終端点となる Node (正確にはそれらの Node::_idx フィールドの値) から 
  IdealLoopTree オブジェクトへの写像

  (正確には, このフィールドは Node_Array オブジェクトを格納するフィールド.
  IdealLoopTree オブジェクトは Node オブジェクトではないが Node* にキャストして格納されている)

  (格納している IdealLoopTree オブジェクト自体は, PhaseIdealLoop::_ltree_root に格納されているものと重複)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* PhaseIdealLoop::build_and_optimize()
* PhaseIdealLoop::build_loop_tree_impl()
* IdealLoopTree::merge_many_backedges()

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
PhaseIdealLoop::build_and_optimize()
-&gt; PhaseIdealLoop::build_loop_tree()
   -&gt; PhaseIdealLoop::build_loop_tree_impl()
-&gt; IdealLoopTree::beautify_loops()
   -&gt; IdealLoopTree::merge_many_backedges()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classIdealLoopTree.html) for details

---
## <a name="noIElp8H_g" id="noIElp8H_g">LoopTreeIterator</a>

### 概要(Summary)
PhaseIdealLoop クラス内で使用される補助クラス.

ある IdealLoopTree を起点としてその子要素をたどるためのイテレータクラス(StackObjクラス).
なお, 辿る順番は preorder で left-to-right.


```cpp
    ((cite: hotspot/src/share/vm/opto/loopnode.hpp))
    // Iterate over the loop tree using a preorder, left-to-right traversal.
    //
    // Example that visits all counted loops from within PhaseIdealLoop
    //
    //  for (LoopTreeIterator iter(_ltree_root); !iter.done(); iter.next()) {
    //   IdealLoopTree* lpt = iter.current();
    //   if (!lpt->is_counted()) continue;
    //   ...
    class LoopTreeIterator : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* PhaseIdealLoop::do_intrinsify_fill()
* PhaseIdealLoop::build_and_optimize()





### 詳細(Details)
See: [here](../doxygen/classLoopTreeIterator.html) for details

---
