---
layout: default
title: 高レベル中間語(Ideal)/低レベル中間語(MachNode)の基底クラス (Node, DUIterator_Common, DUIterator, DUIterator_Fast, DUIterator_Last, SimpleDUIterator, Node_Array, Node_List, Unique_Node_List, Node_Stack, Node_Notes, TypeNode)
---
[Top](../index.html)

#### 高レベル中間語(Ideal)/低レベル中間語(MachNode)の基底クラス (Node, DUIterator_Common, DUIterator, DUIterator_Fast, DUIterator_Last, SimpleDUIterator, Node_Array, Node_List, Unique_Node_List, Node_Stack, Node_Notes, TypeNode)

これらは, C2 JIT Compiler 用の中間語を表すクラス.
より具体的に言うと, それら中間語を表すクラスの基底クラス.

### 概要(Summary)
(#Under Construction)



### クラス一覧(class list)

  * [Node](#noR0BGu-vs)
  * [DUIterator_Common](#noK6QgssUe)
  * [DUIterator](#no3_3L0NSC)
  * [DUIterator_Fast](#no8LC6zsgi)
  * [DUIterator_Last](#noG8Uvb-Jk)
  * [SimpleDUIterator](#noC6ZHOy6Z)
  * [Node_Array](#noJAmPvAIU)
  * [Node_List](#notfh4Ngye)
  * [Unique_Node_List](#noP5phE-FW)
  * [Node_Stack](#noVplmU8QA)
  * [Node_Notes](#noz_mo133P)
  * [TypeNode](#nozI-l9J4H)


---
## <a name="noR0BGu-vs" id="noR0BGu-vs">Node</a>

### 概要(Summary)
C2 JIT コンパイラが使用する中間語用のクラスの基底クラス.

高レベル中間語および低レベル中間語用のクラスは全て Node クラスのサブクラスとして表現される (See: [here](no3867ipg.html) for details).


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    //------------------------------Node-------------------------------------------
    // Nodes define actions in the program.  They create values, which have types.
    // They are both vertices in a directed graph and program primitives.  Nodes
    // are labeled; the label is the "opcode", the primitive function in the lambda
    // calculus sense that gives meaning to the Node.  Node inputs are ordered (so
    // that "a-b" is different from "b-a").  The inputs to a Node are the inputs to
    // the Node's function.  These inputs also define a Type equation for the Node.
    // Solving these Type equations amounts to doing dataflow analysis.
    // Control and data are uniformly represented in the graph.  Finally, Nodes
    // have a unique dense integer index which is used to index into side arrays
    // whenever I have phase-specific information.
    
    class Node {
```

(#Under Construction)




### 詳細(Details)
See: [here](../doxygen/classNode.html) for details

---
## <a name="noK6QgssUe" id="noK6QgssUe">DUIterator_Common</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#if OPTO_DU_ITERATOR_ASSERT 時にしか定義されない).

Node クラス用の補助クラス.
Node オブジェクト間の "Def-Use" 情報をたどるためのイテレータクラス(ValueObjクラス) (の基底クラス).

(Ideal は Def-Use 関係でつながれたグラフ構造(双方向にリンクされた有向グラフ)になっておりそこを辿る.
 より具体的に言うと Node オブジェクトの _out[] 配列内の要素を辿る).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    //-----------------------------------------------------------------------------
    // Iterators over DU info, and associated Node functions.
```


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    #if OPTO_DU_ITERATOR_ASSERT
    
    // Common code for assertion checking on DU iterators.
    class DUIterator_Common VALUE_OBJ_CLASS_SPEC {
```

### 備考(Notes)
明示的に #define しない場合, OPTO_DU_ITERATOR_ASSERT は #ifdef ASSERT 時にのみ 1 になる
(そのため #if OPTO_DU_ITERATOR_ASSERT は #ifdef ASSERT と同義).


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    #ifndef OPTO_DU_ITERATOR_ASSERT
    #ifdef ASSERT
    #define OPTO_DU_ITERATOR_ASSERT 1
    #else
    #define OPTO_DU_ITERATOR_ASSERT 0
    #endif
    #endif //OPTO_DU_ITERATOR_ASSERT
```




### 詳細(Details)
See: [here](../doxygen/classDUIterator__Common.html) for details

---
## <a name="no3_3L0NSC" id="no3_3L0NSC">DUIterator</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#if OPTO_DU_ITERATOR_ASSERT 時以外にはクラスとしては定義されず単なる typedef になる).

DUIterator_Common クラスの具象サブクラスの1つ.
"Def-Use" 情報をたどる処理に assert チェックをかかっているためバグの早期検出に役立つ.

なお DUIterator_Fast クラスと異なりイテレート処理中に要素が追加されても対応できる
(ただし, その場合 Node::refresh_out_pos() を呼ぶ必要がある).


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    //-----------------------------------------------------------------------------
    // Iterators over DU info, and associated Node functions.
```


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    // Default DU iterator.  Allows appends onto the out array.
    // Allows deletion from the out array only at the current point.
    // Usage:
    //  for (DUIterator i = x->outs(); x->has_out(i); i++) {
    //    Node* y = x->out(i);
    //    ...
    //  }
    // Compiles in product mode to a unsigned integer index, which indexes
    // onto a repeatedly reloaded base pointer of x->_out.  The loop predicate
    // also reloads x->_outcnt.  If you delete, you must perform "--i" just
    // before continuing the loop.  You must delete only the last-produced
    // edge.  You must delete only a single copy of the last-produced edge,
    // or else you must delete all copies at once (the first time the edge
    // is produced by the iterator).
    class DUIterator : public DUIterator_Common {
```


なお, #if OPTO_DU_ITERATOR_ASSERT 時以外は uint の typedef.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    #if OPTO_DU_ITERATOR_ASSERT
    class DUIterator;
    class DUIterator_Fast;
    class DUIterator_Last;
    #else
    typedef uint   DUIterator;
    typedef Node** DUIterator_Fast;
    typedef Node** DUIterator_Last;
    #endif
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* Node::outs()
* CallNode::extract_projections()
* RegionNode::Ideal()
* IdealGraphPrinter::walk_nodes()
* Block::implicit_null_check()
* PhaseIdealLoop::replace_parallel_iv()
* PhaseIdealLoop::build_loop_late_post()
* PhaseIdealLoop::sink_use()
* PhaseCCP::transform_once()
* PhaseIdealLoop::split_up()
* PhaseIdealLoop::do_split_if()
* SuperWord::remove_and_insert()
* SuperWord::co_locate_pack()

### 備考(Notes)
明示的に #define しない場合, OPTO_DU_ITERATOR_ASSERT は #ifdef ASSERT 時にのみ 1 になる
(そのため #if OPTO_DU_ITERATOR_ASSERT は #ifdef ASSERT と同義).


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    #ifndef OPTO_DU_ITERATOR_ASSERT
    #ifdef ASSERT
    #define OPTO_DU_ITERATOR_ASSERT 1
    #else
    #define OPTO_DU_ITERATOR_ASSERT 0
    #endif
    #endif //OPTO_DU_ITERATOR_ASSERT
```




### 詳細(Details)
See: [here](../doxygen/classDUIterator.html) for details

---
## <a name="no8LC6zsgi" id="no8LC6zsgi">DUIterator_Fast</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#if OPTO_DU_ITERATOR_ASSERT 時以外にはクラスとしては定義されず単なる typedef になる).

DUIterator_Common クラスの具象サブクラスの1つ.
"Def-Use" 情報をたどる処理に assert チェックをかかっているためバグの早期検出に役立つ.

なお DUIterator クラスと異なりイテレート処理中に要素が追加されると対応できない.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    //-----------------------------------------------------------------------------
    // Iterators over DU info, and associated Node functions.
```


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    // Faster DU iterator.  Disallows insertions into the out array.
    // Allows deletion from the out array only at the current point.
    // Usage:
    //  for (DUIterator_Fast imax, i = x->fast_outs(imax); i < imax; i++) {
    //    Node* y = x->fast_out(i);
    //    ...
    //  }
    // Compiles in product mode to raw Node** pointer arithmetic, with
    // no reloading of pointers from the original node x.  If you delete,
    // you must perform "--i; --imax" just before continuing the loop.
    // If you delete multiple copies of the same edge, you must decrement
    // imax, but not i, multiple times:  "--i, imax -= num_edges".
    class DUIterator_Fast : public DUIterator_Common {
```


なお, #if OPTO_DU_ITERATOR_ASSERT 時以外は Node** の typedef.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    #if OPTO_DU_ITERATOR_ASSERT
    class DUIterator;
    class DUIterator_Fast;
    class DUIterator_Last;
    #else
    typedef uint   DUIterator;
    typedef Node** DUIterator_Fast;
    typedef Node** DUIterator_Last;
    #endif
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

### 備考(Notes)
明示的に #define しない場合, OPTO_DU_ITERATOR_ASSERT は #ifdef ASSERT 時にのみ 1 になる
(そのため #if OPTO_DU_ITERATOR_ASSERT は #ifdef ASSERT と同義).


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    #ifndef OPTO_DU_ITERATOR_ASSERT
    #ifdef ASSERT
    #define OPTO_DU_ITERATOR_ASSERT 1
    #else
    #define OPTO_DU_ITERATOR_ASSERT 0
    #endif
    #endif //OPTO_DU_ITERATOR_ASSERT
```




### 詳細(Details)
See: [here](../doxygen/classDUIterator__Fast.html) for details

---
## <a name="noG8Uvb-Jk" id="noG8Uvb-Jk">DUIterator_Last</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#if OPTO_DU_ITERATOR_ASSERT 時以外にはクラスとしては定義されず単なる typedef になる).

DUIterator_Fast クラスのサブクラス.
こちらは要素を逆順に (= 最後から) たどる場合用.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    //-----------------------------------------------------------------------------
    // Iterators over DU info, and associated Node functions.
```


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    // Faster DU iterator.  Requires each successive edge to be removed.
    // Does not allow insertion of any edges.
    // Usage:
    //  for (DUIterator_Last imin, i = x->last_outs(imin); i >= imin; i -= num_edges) {
    //    Node* y = x->last_out(i);
    //    ...
    //  }
    // Compiles in product mode to raw Node** pointer arithmetic, with
    // no reloading of pointers from the original node x.
    class DUIterator_Last : private DUIterator_Fast {
```


なお, #if OPTO_DU_ITERATOR_ASSERT 時以外は Node** の typedef.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    #if OPTO_DU_ITERATOR_ASSERT
    class DUIterator;
    class DUIterator_Fast;
    class DUIterator_Last;
    #else
    typedef uint   DUIterator;
    typedef Node** DUIterator_Fast;
    typedef Node** DUIterator_Last;
    #endif
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

### 備考(Notes)
明示的に #define しない場合, OPTO_DU_ITERATOR_ASSERT は #ifdef ASSERT 時にのみ 1 になる
(そのため #if OPTO_DU_ITERATOR_ASSERT は #ifdef ASSERT と同義).


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    #ifndef OPTO_DU_ITERATOR_ASSERT
    #ifdef ASSERT
    #define OPTO_DU_ITERATOR_ASSERT 1
    #else
    #define OPTO_DU_ITERATOR_ASSERT 0
    #endif
    #endif //OPTO_DU_ITERATOR_ASSERT
```




### 詳細(Details)
See: [here](../doxygen/classDUIterator__Last.html) for details

---
## <a name="noC6ZHOy6Z" id="noC6ZHOy6Z">SimpleDUIterator</a>

### 概要(Summary)
Node クラス用の補助クラス.

Node オブジェクト間の "Def-Use" 情報をたどるためのイテレータクラス(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    // An Iterator that truly follows the iterator pattern.  Doesn't
    // support deletion but could be made to.
    //
    //   for (SimpleDUIterator i(n); i.has_next(); i.next()) {
    //     Node* m = i.get();
    //
    class SimpleDUIterator : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* final_graph_reshaping_impl()
* GraphKit::replace_call()
* PhaseIdealLoop::match_fill_loop()
* StringConcat::merge_add() (ただし, 現状では #if 0 でコメントアウトされているため使用されていない)
* StringConcat::eliminate_call()
* PhaseStringOpts::build_candidate()
* PhaseStringOpts::remove_dead_nodes()
* StringConcat::validate_control_flow()

### 内部構造(Internal structure)
内部的には, DUIterator_Fast クラスを用いて実装されている.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
      DUIterator_Fast i;
      DUIterator_Fast imax;
```




### 詳細(Details)
See: [here](../doxygen/classSimpleDUIterator.html) for details

---
## <a name="noJAmPvAIU" id="noJAmPvAIU">Node_Array</a>

### 概要(Summary)
Node クラス用のユーティリティ・クラス(ResourceObjクラス).

Node オブジェクトの配列を表す (= 整数値から Node オブジェクトへの写像).
なお配列長は必要に応じて自動的に拡張される.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    //-----------------------------------------------------------------------------
    // Map dense integer indices to Nodes.  Uses classic doubling-array trick.
    // Abstractly provides an infinite array of Node*'s, initialized to NULL.
    // Note that the constructor just zeros things, and since I use Arena
    // allocation I do not need a destructor to reclaim storage.
    class Node_Array : public ResourceObj {
```




### 詳細(Details)
See: [here](../doxygen/classNode__Array.html) for details

---
## <a name="notfh4Ngye" id="notfh4Ngye">Node_List</a>

### 概要(Summary)
Node_Array クラスのサブクラス.

配列の最後に要素を追加／削除する push()/pop() 等のメソッドが追加されている.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    class Node_List : public Node_Array {
```




### 詳細(Details)
See: [here](../doxygen/classNode__List.html) for details

---
## <a name="noP5phE-FW" id="noP5phE-FW">Unique_Node_List</a>

### 概要(Summary)
特殊な Node_List クラス.

既に格納済みの Node であれば push() しても何もしない.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    //------------------------------Unique_Node_List-------------------------------
    class Unique_Node_List : public Node_List {
```




### 詳細(Details)
See: [here](../doxygen/classUnique__Node__List.html) for details

---
## <a name="noVplmU8QA" id="noVplmU8QA">Node_Stack</a>

### 概要(Summary)
Node クラス用の補助クラス.

Node オブジェクトのスタックを表すユーティリティ・クラス.
push()/pop() の他, 添字(index)で指定した位置の要素の取得／変更もできる.
なお領域長は必要に応じて自動的に拡張される.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    //------------------------------Node_Stack-------------------------------------
    class Node_Stack {
```

### 内部構造(Internal structure)
メモリ領域はコンストラクタで指定した Arena 上に確保される. 
指定しない場合はカレントスレッドの ResourceArea 上に確保される. 


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
      Node_Stack(int size) {
        size_t max = (size > OptoNodeListSize) ? size : OptoNodeListSize;
        _a = Thread::current()->resource_area();
        _inodes = NEW_ARENA_ARRAY( _a, INode, max );
        _inode_max = _inodes + max;
        _inode_top = _inodes - 1; // stack is empty
      }
    
      Node_Stack(Arena *a, int size) : _a(a) {
        size_t max = (size > OptoNodeListSize) ? size : OptoNodeListSize;
        _inodes = NEW_ARENA_ARRAY( _a, INode, max );
        _inode_max = _inodes + max;
        _inode_top = _inodes - 1; // stack is empty
      }
```




### 詳細(Details)
See: [here](../doxygen/classNode__Stack.html) for details

---
## <a name="noz_mo133P" id="noz_mo133P">Node_Notes</a>

### 概要(Summary)
Node クラス用の補助クラス.

safepoint ではない箇所について, 
その箇所に対応する JVMState 情報(デバッグ情報／プロファイル情報)を関連づけるためのクラス.
1つの Node_Notes オブジェクトが 1つの Node オブジェクトに対応する.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    //-----------------------------Node_Notes--------------------------------------
    // Debugging or profiling annotations loosely and sparsely associated
    // with some nodes.  See Compile::node_notes_at for the accessor.
    class Node_Notes VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 Compile オブジェクトの _node_note_array フィールド
  
  (正確には, このフィールドは Node_Notes* の GrowableArray を格納するフィールド.
  各 Node オブジェクト (正確にはその Node::_idx フィールドの値) から対応する Node_Notes オブジェクトへの写像になっている)

* 各 Compile オブジェクトの _default_node_notes フィールド
  
  その時点での JVMState (を表す Note_Nodes オブジェクト) が格納されているフィールド

* 各 Matcher オブジェクトの _old_node_note_array フィールド
  
  (これは Compile::_node_note_array の値を一時的に待避しておくフィールド)
  
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ValueObj クラスなので「生成」というのは少し違和感があるが, 以下の箇所でのみ新しい値を持ったインスタンスが生成されている. 
他の使用箇所はコピーコンストラクタ, あるいは既に生成済みの値へのポインタ).

* Node_Notes::make()
* Node_Notes::clone()
* Compile::grow_node_notes()

そして, これらの関数は現在は以下のパスで呼び出されている. (#TODO 他のパス)

```
Compile::Init()
-> Node_Notes::make()

Parse::make_node_notes()
-> Node_Notes::clone()

LateInlineCallGenerator::do_late_inline()
-> Node_Notes::clone()

Compile::build_start_state()
-> Node_Notes::clone()

Compile::locate_node_notes()
-> Compile::grow_node_notes()
```

#### 使用箇所(where its instances are used)
以下の箇所で使用されている. (#TODO 他の使用箇所)

* NonSafepointEmitter::observe_instruction()
* IdealGraphPrinter::visit_node()
* Node::dump()

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
      JVMState* _jvms;
```




### 詳細(Details)
See: [here](../doxygen/classNode__Notes.html) for details

---
## <a name="nozI-l9J4H" id="nozI-l9J4H">TypeNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.

型情報を持つ Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
    //------------------------------TypeNode---------------------------------------
    // Node with a Type constant.
    class TypeNode : public Node {
```

### 内部構造(Internal structure)
スーパークラスである Node クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/node.hpp))
      const Type* const _type;
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classTypeNode.html) for details

---
