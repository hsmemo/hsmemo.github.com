---
layout: default
title: RootNode に関する高レベル中間語(Ideal)クラス (RootNode, HaltNode)
---
[Top](../index.html)

#### RootNode に関する高レベル中間語(Ideal)クラス (RootNode, HaltNode)

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, Ideal によるグラフ構造全体の管理を行うクラス.


### クラス一覧(class list)

  * [RootNode](#noqMdbirgh)
  * [HaltNode](#noiz4g_ue-)


---
## <a name="noqMdbirgh" id="noqMdbirgh">RootNode</a>

### 概要(Summary)
LoopNode クラスのサブクラスの1つ.
グラフ処理のアルゴリズムを簡単にするための便宜的なノード.

Ideal によるグラフ構造の頂点を表す (どのグラフも RootNode を頂点ノードとする).
また, コンパイル対象のメソッドから exit しようとする Node を示す役割もある (そういった Node は全て RootNode を出力先とする).


```cpp
    ((cite: hotspot/src/share/vm/opto/rootnode.hpp))
    // The one-and-only before-all-else and after-all-else RootNode.  The RootNode
    // represents what happens if the user runs the whole program repeatedly.  The
    // RootNode produces the initial values of I/O and memory for the program or
    // procedure start.
    class RootNode : public LoopNode {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Compile オブジェクトの _root フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
Compile::Init() 内で(のみ)生成されている.


```cpp
    ((cite: hotspot/src/share/vm/opto/compile.cpp))
    void Compile::Init(int aliaslevel) {
    ...
      set_root(new (this, 3) RootNode());
```

### 内部構造(Internal structure)
LoopNode のサブクラスなので生成直後は (control input も含めて) 3つの入力ノードを持つ.
ただし, コンストラクタ内で del_req() で第3および第2入力ノードを削ってしまうため, 
初期状態では入力ノードは1つだけ(自分自身のみ)になる.


```cpp
    ((cite: hotspot/src/share/vm/opto/rootnode.hpp))
      RootNode( ) : LoopNode(0,0) {
        init_class_id(Class_Root);
        del_req(2);
        del_req(1);
      }
```

その後の処理パスで, Node::add_req() によって, 
ReturnNode, RethrowNode, TailCallNode, TailJumpNode, HaltNode (他にもある? #TODO) が追加される模様 (#TODO).

(<= Ideal のグラフは双方向につながっているので, 
これらの Node の出力先として使われるということは, これらの Node を入力に持つということ)
(See: 
Compile::return_values(), Compile::rethrow_exceptions(), 
GraphKit::gen_stub(), 
AllocateArrayNode::Ideal(), GraphKit::uncommon_trap())




### 詳細(Details)
See: [here](../doxygen/classRootNode.html) for details

---
## <a name="noiz4g_ue-" id="noiz4g_ue-">HaltNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.

このクラスは決して到達しえないパスを示す.


```cpp
    ((cite: hotspot/src/share/vm/opto/rootnode.hpp))
    // Throw an exception & die
    class HaltNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* uncommon trap 処理で SharedRuntime::uncommon_trap_blob() を呼び出した後の制御フロー (決して返ってこないので実行されない)
  
  GraphKit::uncommon_trap()

* NeverBranchNode が生成した偽のブランチ(決して分岐されない)の分岐先
  
  PhaseIdealLoop::build_loop_tree_impl()

* 配列確保処理で確保する長さが負値の場合のフォールスルーパス (決してフォールスルーすることはない)

  AllocateArrayNode::Ideal()

### 内部構造(Internal structure)
(control input も含めて) 5つの入力ノードを持つ. それぞれの入力の意味は以下の通り.

* 1番目の入力Node : TypeFunc::Control (control input)
* 2番目の入力Node : TypeFunc::I_O (I/O state)(#TODO)
* 3番目の入力Node : TypeFunc::Memory (memory state)
* 4番目の入力Node : TypeFunc::FramePtr (#TODO)
* 5番目の入力Node : TypeFunc::ReturnAdr (#TODO)


```cpp
    ((cite: hotspot/src/share/vm/opto/rootnode.cpp))
    HaltNode::HaltNode( Node *ctrl, Node *frameptr ) : Node(TypeFunc::Parms) {
      Node* top = Compile::current()->top();
      init_req(TypeFunc::Control,  ctrl        );
      init_req(TypeFunc::I_O,      top);
      init_req(TypeFunc::Memory,   top);
      init_req(TypeFunc::FramePtr, frameptr    );
      init_req(TypeFunc::ReturnAdr,top);
    }
```




### 詳細(Details)
See: [here](../doxygen/classHaltNode.html) for details

---
