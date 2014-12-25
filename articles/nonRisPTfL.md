---
layout: default
title: 制御構造(Control Flow Graph)に関する高レベル中間語(Ideal)クラス (RegionNode, JProjNode, PhiNode, GotoNode, CProjNode, MultiBranchNode, IfNode, IfTrueNode, IfFalseNode, PCTableNode, JumpNode, JumpProjNode, CatchNode, CatchProjNode, CreateExNode, NeverBranchNode)
---
[Top](../index.html)

#### 制御構造(Control Flow Graph)に関する高レベル中間語(Ideal)クラス (RegionNode, JProjNode, PhiNode, GotoNode, CProjNode, MultiBranchNode, IfNode, IfTrueNode, IfFalseNode, PCTableNode, JumpNode, JumpProjNode, CatchNode, CatchProjNode, CreateExNode, NeverBranchNode)

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, 制御構造(制御フロー)を表すクラス.

なお, これらのクラスは以下のような継承関係を持つ.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    class Node;
    class   RegionNode;
    class   TypeNode;
    class     PhiNode;
    class   GotoNode;
    class   MultiNode;
    class     MultiBranchNode;
    class       IfNode;
    class       PCTableNode;
    class         JumpNode;
    class         CatchNode;
    class       NeverBranchNode;
    class   ProjNode;
    class     CProjNode;
    class       IfTrueNode;
    class       IfFalseNode;
    class       CatchProjNode;
    class     JProjNode;
    class       JumpProjNode;
    class     SCMemProjNode;
```


### クラス一覧(class list)

  * [RegionNode](#no-g8Zg5w-)
  * [PhiNode](#noxBgu_Zu_)
  * [GotoNode](#no1RMnFNrR)
  * [MultiBranchNode](#nouShcDa9s)
  * [IfNode](#nozPFw2OFt)
  * [CProjNode](#nokdTO6Q5T)
  * [IfTrueNode](#no8QdpYqKx)
  * [IfFalseNode](#noxUGqmIgI)
  * [PCTableNode](#nolKHy50q0)
  * [JumpNode](#nonTqzBxvm)
  * [JProjNode](#noRoJOONBi)
  * [JumpProjNode](#noUPtbRN0_)
  * [CatchNode](#noH7Pr-W5D)
  * [CatchProjNode](#noYCf2NSTc)
  * [CreateExNode](#noY6ccTu-7)
  * [NeverBranchNode](#noHbNsGx_B)


---
## <a name="no-g8Zg5w-" id="no-g8Zg5w-">RegionNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.

複数のノードを入力としてそれらの制御情報を束ねるクラス.
概念的には「1つの basic block を表現する Node」
(入力は「その predecessor blocks にあたる Node」).

また RegionNode は対応する PhiNode を持つ.
この 2種類の Node によって分岐の合流点が表現される
(これらは predecessor blocks の制御情報とデータ情報に対応し, 
 RegionNode が制御情報を束ね, PhiNode がデータ情報を束ねる).

なお, ほとんどの演算系の Node は, その結果が必要になるまでなら別にいつ実行してもかまわないので,
どの RegionNode (= どの basic block) にも所属しないという扱いになる.
このため実際のバイトコード上における basic block のいくつかは所属するノードがいなくなり不要になる.
結果として RegionNode は主に If の合流点 (IfTrue や IfFalse の下) と loop の先頭箇所くらいにしか出てこない.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // The class of RegionNodes, which can be mapped to basic blocks in the
    // program.  Their inputs point to Control sources.  PhiNodes (described
    // below) have an input point to a RegionNode.  Merged data inputs to PhiNodes
    // correspond 1-to-1 with RegionNode inputs.  The zero input of a PhiNode is
    // the RegionNode, and the zero input of the RegionNode is itself.
    class RegionNode : public Node {
```

### 内部構造(Internal structure)
(control input も含めて) 可変個の入力ノードを持つ. 

それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input (ただし, 自分自身を指す)
* 2番目以降の入力Node : predecessor blocks にあたる Node


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      // Node layout (parallels PhiNode):
      enum { Region,                // Generally points to self.
             Control                // Control arcs are [1..len)
      };
```


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      RegionNode( uint required ) : Node(required) {
        init_class_id(Class_Region);
        init_req(0,this);
      }
```





### 詳細(Details)
See: [here](../doxygen/classRegionNode.html) for details

---
## <a name="noxBgu_Zu_" id="noxBgu_Zu_">PhiNode</a>

### 概要(Summary)
TypeNode クラスの具象サブクラスの1つ.

SSA(静的単一代入形式) の φ 関数を表す Node.
分岐の合流点における data flow のマージを行う.

また PhiNode は対応する RegionNode を持つ.
この 2種類の Node によって分岐の合流点が表現される
(これらは predecessor blocks の制御情報とデータ情報に対応し, 
 RegionNode が制御情報を束ね, PhiNode がデータ情報を束ねる).
(<= より正確に言うと, 分岐の合流点は 1つの RegionNode, およびその地点で live な値と同数の PhiNode で表現される)

(なお, PhiNode は対応する RegionNode の入力ノードと同数の入力ノードを持つ. 
 それぞれの入力ノードが RegionNode の対応する predecessor blocks からの値を示す)


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // PhiNodes merge values from different Control paths.  Slot 0 points to the
    // controlling RegionNode.  Other slots map 1-for-1 with incoming control flow
    // paths to the RegionNode.  For speed reasons (to avoid another pass) we
    // can turn PhiNodes into copys in-place by NULL'ing out their RegionNode
    // input in slot 0.
    class PhiNode : public TypeNode {
```

### 使われ方(Usage)
#### 使用例(usage examples)
実際の PhiNode の使用例
(ここでは, 対応する RegionNode に併せて作成され, 
その後 RegionNode にパスがセットされるたびに 
PhiNode にもそのパスに対応する値がセットされている)


```cpp
    ((cite: hotspot/src/share/vm/opto/library_call.cpp))
        // Make the merge point
        RegionNode* result_rgn = new (C, 4) RegionNode(4);
        Node*       result_phi = new (C, 4) PhiNode(result_rgn, TypeInt::INT);
    ...
          result_phi->init_req(2, intcon(-1));
          result_rgn->init_req(2, if_gt);
    ...
            result_phi->init_req(3, intcon(0));
            result_rgn->init_req(3, if_zero);
    ...
          result_phi->init_req(1, result);
          result_rgn->init_req(1, control());
```

### 内部構造(Internal structure)
(control input も含めて) 可変個の入力ノードを持つ. 

それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input (対応する RegionNode)
* 2番目以降の入力Node : merge 対象のデータ (= φ 関数の引数)


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      // Node layout (parallels RegionNode):
      enum { Region,                // Control input is the Phi's region.
             Input                  // Input values are [1..len)
      };
```


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      PhiNode( Node *r, const Type *t, const TypePtr* at = NULL,
               const int iid = TypeOopPtr::InstanceTop,
               const int iidx = Compile::AliasIdxTop,
               const int ioffs = Type::OffsetTop )
        : TypeNode(t,r->req()),
          _adr_type(at),
          _inst_id(iid),
          _inst_index(iidx),
          _inst_offset(ioffs)
      {
        init_class_id(Class_Phi);
        init_req(0, r);
        verify_adr_type();
      }
```





### 詳細(Details)
See: [here](../doxygen/classPhiNode.html) for details

---
## <a name="no1RMnFNrR" id="no1RMnFNrR">GotoNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
goto (無条件分岐) を表す.

なお, これは PhaseCFG によって生成されるノード
(PhaseCFG によってノードが CFG の形になり, 全ての basic block は Goto か If か Return で終わるようになる.
このため, これまでは明示的に出てきていなかった basic block 間をつなぐ Goto がここで導入される).

なお, 正確には PhaseCFG() は MachNode 用の処理なので,
まず Matcher::match_tree() で GotoNode に対応する MachNode が作られ, それが使い回される.
なので, 実際には GotoNode 自体が使われることはない (使われるのは GotoNode に対応する MachNode だけ).


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // GotoNodes perform direct branches.
    class GotoNode : public Node {
```

### 使われ方(Usage)
#### 使用例(usage examples)
(PhaseCFG::_goto に格納されている GotoNode (に対応する MachNode) (後述) が clone() メソッドでコピーされ, 
basic block の間に挿入されている)


```cpp
    ((cite: hotspot/src/share/vm/opto/block.cpp))
    uint PhaseCFG::build_cfg() {
    ...
          Node *g = _goto->clone(); // Force it to end in a Goto
          g->set_req(0, proj);
          np->set_req(idx, g);
```

#### インスタンスの格納場所(where its instances are stored)
各 PhaseCFG オブジェクトの _goto フィールドに(のみ)格納されている.

(正確には GotoNode に対応する MachNode がこのフィールドに格納されている (See: PhaseCFG::PhaseCFG()))

#### 生成箇所(where its instances are created)
PhaseCFG::PhaseCFG() 内で(のみ)生成されている.

(そして, PhaseCFG 内のその後の処理でこの Node がコピーされて使い回される.
 なお, この段階では GotoNode の飛び先を示す input 0 には自分自身が入っている.
 これは実際に使う段階で正しい飛び先に設定し直される.)


```cpp
    ((cite: hotspot/src/share/vm/opto/block.cpp))
    PhaseCFG::PhaseCFG( Arena *a, RootNode *r, Matcher &m ) :
    ...
      Node *x = new (C, 1) GotoNode(NULL);
      x->init_req(0, x);
      _goto = m.match_tree(x);
      assert(_goto != NULL, "");
      _goto->set_req(0,_goto);
```

### 内部構造(Internal structure)
入力ノードは control input のみ. control input は GotoNode の飛び元を示す.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      GotoNode( Node *control ) : Node(control) {
        init_flags(Flag_is_Goto);
      }
```





### 詳細(Details)
See: [here](../doxygen/classGotoNode.html) for details

---
## <a name="nouShcDa9s" id="nouShcDa9s">MultiBranchNode</a>

### 概要(Summary)
MultiNode クラスのサブクラスの1つ.

複数の control-flow path を作る Node (= 条件分岐を表す Node) の基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // This class defines a MultiBranchNode, a MultiNode which yields multiple
    // control values. These are distinguished from other types of MultiNodes
    // which yield multiple values, but control is always and only projection #0.
    class MultiBranchNode : public MultiNode {
```

### 内部構造(Internal structure)
このクラス自体は入力ノードを規定しない (= 入力ノードはない).


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      MultiBranchNode( uint required ) : MultiNode(required) {
        init_class_id(Class_MultiBranch);
      }
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classMultiBranchNode.html) for details

---
## <a name="nozPFw2OFt" id="nozPFw2OFt">IfNode</a>

### 概要(Summary)
MultiBranchNode クラスの具象サブクラスの1つ.

boolean 値に基づく 2分岐を表す Node.
それぞれの control flow は IfTrueNode と IfFalseNode で取り出せる.

(なお, どちらに分岐しやすいかといった情報も保持している)


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // Output selected Control, based on a boolean test
    class IfNode : public MultiBranchNode {
```

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : 分岐判断に使用する bool 値


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      IfNode( Node *control, Node *b, float p, float fcnt )
        : MultiBranchNode(2), _prob(p), _fcnt(fcnt) {
        init_class_id(Class_If);
        init_req(0,control);
        init_req(1,b);
      }
```

### 備考(Notes)
IfNode はコンストラクタで 2種類の分岐予測のヒント情報を受け取る. 
それぞれ以下のフィールドに格納されている.

* float _prob
* float _fcnt


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      float _prob;                  // Probability of true path being taken.
      float _fcnt;                  // Frequency counter
```

これらは ...(#TODO)


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      // Degrees of branch prediction probability by order of magnitude:
      // PROB_UNLIKELY_1e(N) is a 1 in 1eN chance.
      // PROB_LIKELY_1e(N) is a 1 - PROB_UNLIKELY_1e(N)
    #define PROB_UNLIKELY_MAG(N)    (1e- ## N ## f)
    #define PROB_LIKELY_MAG(N)      (1.0f-PROB_UNLIKELY_MAG(N))
    
      // Maximum and minimum branch prediction probabilties
      // 1 in 1,000,000 (magnitude 6)
      //
      // Although PROB_NEVER == PROB_MIN and PROB_ALWAYS == PROB_MAX
      // they are used to distinguish different situations:
      //
      // The name PROB_MAX (PROB_MIN) is for probabilities which correspond to
      // very likely (unlikely) but with a concrete possibility of a rare
      // contrary case.  These constants would be used for pinning
      // measurements, and as measures for assertions that have high
      // confidence, but some evidence of occasional failure.
      //
      // The name PROB_ALWAYS (PROB_NEVER) is to stand for situations for which
      // there is no evidence at all that the contrary case has ever occurred.
    
    #define PROB_NEVER              PROB_UNLIKELY_MAG(6)
    #define PROB_ALWAYS             PROB_LIKELY_MAG(6)
    
    #define PROB_MIN                PROB_UNLIKELY_MAG(6)
    #define PROB_MAX                PROB_LIKELY_MAG(6)
    
      // Static branch prediction probabilities
      // 1 in 10 (magnitude 1)
    #define PROB_STATIC_INFREQUENT  PROB_UNLIKELY_MAG(1)
    #define PROB_STATIC_FREQUENT    PROB_LIKELY_MAG(1)
    
      // Fair probability 50/50
    #define PROB_FAIR               (0.5f)
    
      // Unknown probability sentinel
    #define PROB_UNKNOWN            (-1.0f)
    
      // Probability "constructors", to distinguish as a probability any manifest
      // constant without a names
    #define PROB_LIKELY(x)          ((float) (x))
    #define PROB_UNLIKELY(x)        (1.0f - (float)(x))
    
      // Other probabilities in use, but without a unique name, are documented
      // here for lack of a better place:
      //
      // 1 in 1000 probabilities (magnitude 3):
      //     threshold for converting to conditional move
      //     likelihood of null check failure if a null HAS been seen before
      //     likelihood of slow path taken in library calls
      //
      // 1 in 10,000 probabilities (magnitude 4):
      //     threshold for making an uncommon trap probability more extreme
      //     threshold for for making a null check implicit
      //     likelihood of needing a gc if eden top moves during an allocation
      //     likelihood of a predicted call failure
      //
      // 1 in 100,000 probabilities (magnitude 5):
      //     threshold for ignoring counts when estimating path frequency
      //     likelihood of FP clipping failure
      //     likelihood of catching an exception from a try block
      //     likelihood of null check failure if a null has NOT been seen before
      //
      // Magic manifest probabilities such as 0.83, 0.7, ... can be found in
      // gen_subtype_check() and catch_inline_exceptions().
```




### 詳細(Details)
See: [here](../doxygen/classIfNode.html) for details

---
## <a name="nokdTO6Q5T" id="nokdTO6Q5T">CProjNode</a>

### 概要(Summary)
ProjNode クラスの具象サブクラスの1つ.

複数の control-flow path を作る Node を入力とし, 
その中から 1つの control-flow path を取り出す Node.

(なお abstract class かと思いきや, NeverBranchNode と組み合わせて PhaseIdealLoop::build_loop_tree_impl() 内で生成されている)

(ところで JProjNode との違いは?? こちらは abstract class ではないが... #TODO)


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // control projection for node that produces multiple control-flow paths
    class CProjNode : public ProjNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
PhaseIdealLoop::build_loop_tree_impl() 内で(のみ)生成されている
(NeverBranchNode の true 分岐/false 分岐を表すための 2個が生成される).

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> PhaseIdealLoop::build_loop_tree()
   -> PhaseIdealLoop::build_loop_tree_impl()
```

### 内部構造(Internal structure)
入力ノードは control input のみ. control input は処理対象の MultiNode を指す.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      CProjNode( Node *ctrl, uint idx ) : ProjNode(ctrl,idx) {}
```





### 詳細(Details)
See: [here](../doxygen/classCProjNode.html) for details

---
## <a name="no8QdpYqKx" id="no8QdpYqKx">IfTrueNode</a>

### 概要(Summary)
CProjNode クラスのサブクラスの1つ.

IfNode から条件が true だった場合の control flow を取り出す Node.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    class IfTrueNode : public CProjNode {
```

### 内部構造(Internal structure)
入力ノードは control input のみ. control input は対応する IfNode を示す.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      IfTrueNode( IfNode *ifnode ) : CProjNode(ifnode,1) {
        init_class_id(Class_IfTrue);
      }
```




### 詳細(Details)
See: [here](../doxygen/classIfTrueNode.html) for details

---
## <a name="noxUGqmIgI" id="noxUGqmIgI">IfFalseNode</a>

### 概要(Summary)
CProjNode クラスのサブクラスの1つ.

IfNode から条件が false だった場合の control flow を取り出す Node.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    class IfFalseNode : public CProjNode {
```

### 内部構造(Internal structure)
入力ノードは control input のみ. control input は対応する IfNode を示す.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      IfFalseNode( IfNode *ifnode ) : CProjNode(ifnode,0) {
        init_class_id(Class_IfFalse);
      }
```





### 詳細(Details)
See: [here](../doxygen/classIfFalseNode.html) for details

---
## <a name="nolKHy50q0" id="nolKHy50q0">PCTableNode</a>

### 概要(Summary)
MultiBranchNode クラスのサブクラスの1つ.

テーブルを用いた分岐を表す Node の基底クラス 
(より具体的に言うと lookupswitch/tableswitch や例外ハンドリングを表現するために使用される).
index を受け取り, 対応する飛び先に分岐する. index が範囲外だった場合は undefined.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // Build an indirect branch table.  Given a control and a table index,
    // control is passed to the Projection matching the table index.  Used to
    // implement switch statements and exception-handling capabilities.
    // Undefined behavior if passed-in index is not inside the table.
    class PCTableNode : public MultiBranchNode {
```

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : 分岐判断に使用する int 値 (index)


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      PCTableNode( Node *ctrl, Node *idx, uint size ) : MultiBranchNode(2), _size(size) {
        init_class_id(Class_PCTable);
        init_req(0, ctrl);
        init_req(1, idx);
      }
```




### 詳細(Details)
See: [here](../doxygen/classPCTableNode.html) for details

---
## <a name="nonTqzBxvm" id="nonTqzBxvm">JumpNode</a>

### 概要(Summary)
PCTableNode クラスの具象サブクラスの1つ. 

lookupswitch/tableswitch バイトコードを表現するための Node.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // Indirect branch.  Uses PCTable above to implement a switch statement.
    // It emits as a table load and local branch.
    class JumpNode : public PCTableNode {
```

### 使われ方(Usage)
Parse::create_jump_tables() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Parse::do_tableswitch()
-> Parse::jump_switch_ranges()
   -> Parse::create_jump_tables()

Parse::do_lookupswitch()
-> Parse::jump_switch_ranges()
   -> Parse::create_jump_tables()
```

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : 分岐判断に使用する int 値 (index)


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      JumpNode( Node* control, Node* switch_val, uint size) : PCTableNode(control, switch_val, size) {
        init_class_id(Class_Jump);
      }
```




### 詳細(Details)
See: [here](../doxygen/classJumpNode.html) for details

---
## <a name="noRoJOONBi" id="noRoJOONBi">JProjNode</a>

### 概要(Summary)
ProjNode クラスのサブクラスの1つ.

複数の control-flow path を作る Node を入力とし, 
その中から 1つの control-flow path を取り出す Node の基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

(ところで CProjNode との違いは?? こちらは lookupswitch/tableswitch 用に当たるようだが...?? #TODO)


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // jump projection for node that produces multiple control-flow paths
    class JProjNode : public ProjNode {
```

### 内部構造(Internal structure)
入力ノードは control input のみ. control input は処理対象の MultiNode を指す.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      JProjNode( Node* ctrl, uint idx ) : ProjNode(ctrl,idx) {}
```





### 詳細(Details)
See: [here](../doxygen/classJProjNode.html) for details

---
## <a name="noUPtbRN0_" id="noUPtbRN0_">JumpProjNode</a>

### 概要(Summary)
JProjNode クラスの具象サブクラスの1つ. 

JumpNode から control flow を 1つ取り出す Node.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    class JumpProjNode : public JProjNode {
```

### 使われ方(Usage)
Parse::create_jump_tables() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Parse::do_tableswitch()
-> Parse::jump_switch_ranges()
   -> Parse::create_jump_tables()

Parse::do_lookupswitch()
-> Parse::jump_switch_ranges()
   -> Parse::create_jump_tables()
```

### 内部構造(Internal structure)
入力ノードは control input のみ. control input は処理対象の JumpNode を指す
(正確には, control input は型の上ではどんな Node も設定可能になっているが, 実際には JumpNode しか設定されない).


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      JumpProjNode(Node* jumpnode, uint proj_no, int dest_bci, int switch_val)
        : JProjNode(jumpnode, proj_no), _dest_bci(dest_bci), _proj_no(proj_no), _switch_val(switch_val) {
        init_class_id(Class_JumpProj);
      }
```

### 備考(Notes)
(なお, このクラスは product オプションである UseJumpTables が true の場合にしか使用されない 
(See: Parse::create_jump_tables()))




### 詳細(Details)
See: [here](../doxygen/classJumpProjNode.html) for details

---
## <a name="noH7Pr-W5D" id="noH7Pr-W5D">CatchNode</a>

### 概要(Summary)
PCTableNode クラスの具象サブクラスの1つ. 

例外ハンドリングを補佐するための Node
(なおこのクラスはテーブルを作るだけ. テーブルのルックアップと分岐は RethrowNode が行う).


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // Helper node to fork exceptions.  "Catch" catches any exceptions thrown by
    // a just-prior call.  Looks like a PCTableNode but emits no code - just the
    // table.  The table lookup and branch is implemented by RethrowNode.
    class CatchNode : public PCTableNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* Parse::catch_call_exceptions()
* GraphKit::make_slow_call_ex()

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : I/O state (#TODO)


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      CatchNode( Node *ctrl, Node *idx, uint size ) : PCTableNode(ctrl,idx,size){
        init_class_id(Class_Catch);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCatchNode.html) for details

---
## <a name="noYCf2NSTc" id="noYCf2NSTc">CatchProjNode</a>

### 概要(Summary)
CProjNode クラスのサブクラスの1つ.

CatchNode 中の飛び先を 1つ取り出す Node (= 例外ハンドラでのキャッチを表す Node).


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // CatchProjNode controls which exception handler is targetted after a call.
    // It is passed in the bci of the target handler, or no_handler_bci in case
    // the projection doesn't lead to an exception handler.
    class CatchProjNode : public CProjNode {
```

### 使われ方(Usage)
以下の箇所で(のみ)生成されている.

* Parse::catch_call_exceptions()
* GraphKit::make_slow_call_ex()

### 内部構造(Internal structure)
入力ノードは control input のみ. control input は処理対象の CatchNode を指す
(正確には, control input は型の上ではどんな Node も設定可能になっているが, 実際には CatchNode しか設定されない).


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      CatchProjNode(Node* catchnode, uint proj_no, int handler_bci)
        : CProjNode(catchnode, proj_no), _handler_bci(handler_bci) {
        init_class_id(Class_CatchProj);
        assert(proj_no != fall_through_index || handler_bci < 0, "fall through case must have bci < 0");
      }
```




### 詳細(Details)
See: [here](../doxygen/classCatchProjNode.html) for details

---
## <a name="noY6ccTu-7" id="noY6ccTu-7">CreateExNode</a>

### 概要(Summary)
TypeNode クラスの具象サブクラスの1つ.

例外オブジェクト(java.lang.Throwable)の作成処理を表す Node.
CatchProjNode とセットで使用される.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // Helper node to create the exception coming back from a call
    class CreateExNode : public TypeNode {
```

### 使われ方(Usage)
以下の箇所で(のみ)生成されている.

* Parse::catch_call_exceptions()
* GraphKit::make_slow_call_ex()

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input (= 対応する CatchProjNode) 
  (正確には, control input は型の上ではどんな Node も設定可能になっているが, 実際には CatchProjNode しか設定されない)
* 2番目の入力Node : I/O state (#TODO)


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      CreateExNode(const Type* t, Node* control, Node* i_o) : TypeNode(t, 2) {
        init_req(0, control);
        init_req(1, i_o);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCreateExNode.html) for details

---
## <a name="noHbNsGx_B" id="noHbNsGx_B">NeverBranchNode</a>

### 概要(Summary)
MultiBranchNode クラスの具象サブクラスの1つ.
決して実行されない分岐を表す.

なお, これは PhaseIdealLoop によって生成されるノード.
無限ループがあった場合に, そこから抜け出るパスがあるように見せかけて 
CFG をとりあえず構築するための Node, である模様 (#TODO).


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
    // The never-taken branch.  Used to give the appearance of exiting infinite
    // loops to those algorithms that like all paths to be reachable.  Encodes
    // empty.
    class NeverBranchNode : public MultiBranchNode {
```

### 使われ方(Usage)
PhaseIdealLoop::build_loop_tree_impl() 内で(のみ)生成されている
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
PhaseIdealLoop::build_and_optimize()
-> PhaseIdealLoop::build_loop_tree()
   -> PhaseIdealLoop::build_loop_tree_impl()
```

### 内部構造(Internal structure)
入力ノードは control input のみ. control input は NeverBranchNode の飛び元を示す.


```cpp
    ((cite: hotspot/src/share/vm/opto/cfgnode.hpp))
      NeverBranchNode( Node *ctrl ) : MultiBranchNode(1) { init_req(0,ctrl); }
```




### 詳細(Details)
See: [here](../doxygen/classNeverBranchNode.html) for details

---
