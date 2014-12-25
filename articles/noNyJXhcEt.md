---
layout: default
title: PhaseCFG, PhaseBlockLayout クラス関連のクラス (Block_Array, Block_List, CFGElement, Block, PhaseCFG, UnionFind, BlockProbPair, CFGLoop, CFGEdge, Trace, PhaseBlockLayout)
---
[Top](../index.html)

#### PhaseCFG, PhaseBlockLayout クラス関連のクラス (Block_Array, Block_List, CFGElement, Block, PhaseCFG, UnionFind, BlockProbPair, CFGLoop, CFGEdge, Trace, PhaseBlockLayout)

これらは, C2 JIT Compiler 内の処理フェーズを表すクラス.
より具体的に言うと, CFG(Control Flow Graph)の計算処理および基本ブロック(basic block)の並び替え処理を表すクラス.


### クラス一覧(class list)

  * [PhaseCFG](#nom5HKoLga)
  * [CFGElement](#noznRhUpOh)
  * [Block](#noHFWIqNI2)
  * [Block_Array](#no3zWbFIf4)
  * [Block_List](#no2I5HUU1O)
  * [CFGLoop](#not6-r9-rO)
  * [BlockProbPair](#no_VvVAa39)
  * [PhaseBlockLayout](#no3I_DdyZc)
  * [CFGEdge](#nopcGwj3MP)
  * [Trace](#nonDu459rZ)
  * [UnionFind](#nopgbl2MDw)


---
## <a name="nom5HKoLga" id="nom5HKoLga">PhaseCFG</a>

### 概要(Summary)
Phase クラスの具象サブクラスの1つ.

低レベル中間語のコードに対して CFG(Control Flow Graph) 情報を計算する.

(また, CFG として適切な形に変形する処理も行っている模様 (適切に GotoNode を挿入する, 等))

(また, 関連する最適化として Global Code Motion 処理を行う機能も備える)


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
    //------------------------------PhaseCFG---------------------------------------
    // Build an array of Basic Block pointers, one per Node.
    class PhaseCFG : public Phase {
```

### 使われ方(Usage)
Compile::Code_Gen() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPhaseCFG.html) for details

---
## <a name="noznRhUpOh" id="noznRhUpOh">CFGElement</a>

### 概要(Summary)
PhaseCFG クラス内で使用される一時オブジェクト(ResourceObjクラス)の基底クラス.

PhaseCFG が生成する CFG(Control Flow Graph) 中の要素を表す.
1つの CFGElement オブジェクトが 1つの basic block もしくは 1つのループに対応する.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
    class CFGElement : public ResourceObj {
```

### 内部構造(Internal structure)
定義されているフィールドはこれだけ.


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
      float _freq; // Execution frequency (estimate)
```





### 詳細(Details)
See: [here](../doxygen/classCFGElement.html) for details

---
## <a name="noHFWIqNI2" id="noHFWIqNI2">Block</a>

### 概要(Summary)
CFGElement クラスの具象サブクラスの1つ.

このクラスは basic block を表す.
1つの Block オブジェクトが 1つの basic block に対応する.


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
    //------------------------------Block------------------------------------------
    // This class defines a Basic Block.
    // Basic blocks are used during the output routines, and are not used during
    // any optimization pass.  They are created late in the game.
    class Block : public CFGElement {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 PhaseCFG オブジェクトの _bbs フィールド
  
  各 basic block の開始点となる Node や終端点となる Node (正確にはそれらの Node::_idx フィールドの値) から 
  Block オブジェクトへの写像

  (正確には, このフィールドは Block_Array オブジェクトを格納するフィールド)

* 各 Block オブジェクトの _succs フィールド
  
  その Block オブジェクトの successor に当たる Block オブジェクト群を格納したフィールド

  (正確には, このフィールドは Block_Array オブジェクトを格納するフィールド)
  
  (格納している Block オブジェクト自体は, PhaseCFG::_bbs に格納されているものと重複).

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* PhaseCFG::build_cfg()
* PhaseCFG::insert_goto_at()




### 詳細(Details)
See: [here](../doxygen/classBlock.html) for details

---
## <a name="no3zWbFIf4" id="no3zWbFIf4">Block_Array</a>

### 概要(Summary)
Block クラス用のユーティリティ・クラス(ResourceObjクラス).

Block オブジェクトの配列を表す (= 整数値から Block オブジェクトへの写像).
なお配列長は必要に応じて自動的に拡張される.


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
    //------------------------------Block_Array------------------------------------
    // Map dense integer indices to Blocks.  Uses classic doubling-array trick.
    // Abstractly provides an infinite array of Block*'s, initialized to NULL.
    // Note that the constructor just zeros things, and since I use Arena
    // allocation I do not need a destructor to reclaim storage.
    class Block_Array : public ResourceObj {
```




### 詳細(Details)
See: [here](../doxygen/classBlock__Array.html) for details

---
## <a name="no2I5HUU1O" id="no2I5HUU1O">Block_List</a>

### 概要(Summary)
Block_Array クラスのサブクラス.

配列の最後に要素を追加／削除する push()/pop() 等のメソッドが追加されている.


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
    class Block_List : public Block_Array {
```




### 詳細(Details)
See: [here](../doxygen/classBlock__List.html) for details

---
## <a name="not6-r9-rO" id="not6-r9-rO">CFGLoop</a>

### 概要(Summary)
CFGElement クラスの具象サブクラスの1つ.

このクラスはループ構造を表す.
1つの CFGLoop オブジェクトが 1つのループに対応する.


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
    //------------------------------CFGLoop-------------------------------------------
    class CFGLoop : public CFGElement {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 PhaseCFG オブジェクトの _root_loop フィールド
  
  (正確には, このフィールドは CFGLoop の木構造を格納するフィールド.
  CFGLoop オブジェクトは 
  _parent フィールド, _sibling フィールド, 及び _child フィールドで次の CFGLoop オブジェクトを指せる構造になっている.
  その PhaseCFG オブジェクト内で生成された CFGLoop オブジェクトは全てこの木構造に格納されている)

* 各 Block オブジェクトの _loop フィールド
  
  (その Block オブジェクトから開始されるループを表す.
  なおループの先頭になっていない Block オブジェクトの場合は 
  PhaseCFG::_root_loop が指している CFGLoop オブジェクトが格納される)
  
  (格納している CFGLoop オブジェクト自体は, PhaseCFG::_root_loop に格納されているものと重複)

#### 生成箇所(where its instances are created)
PhaseCFG::create_loop_tree() 内で(のみ)生成されている.





### 詳細(Details)
See: [here](../doxygen/classCFGLoop.html) for details

---
## <a name="no_VvVAa39" id="no_VvVAa39">BlockProbPair</a>

### 概要(Summary)
CFGLoop クラス内で使用される補助クラス(ValueObjクラス).

名前の通り, Block オブジェクトとそこへの遷移確率(probability)の組(pair)を表すクラス.
その CFGLoop オブジェクトの successor に当たる basic block を (その block への遷移確率とともに) 記録しておくために使われる.
1つの BlockProbPair オブジェクトが 1つの successor block に対応する.

(ところでここに書かれているコメントは OrderedPair クラスのものでは?? #TODO)


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
    //----------------------------BlockProbPair---------------------------
    // Ordered pair of Node*.
    class BlockProbPair VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CFGLoop オブジェクトの _exits フィールドに(のみ)格納されている.
 
(正確には, このフィールドは BlockProbPair の GrowableArray を格納するフィールド.
この中に, その CFGLoop 用の全ての BlockProbPair オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* CFGLoop::compute_freq()
* CFGLoop::update_succ_freq()

### 内部構造(Internal structure)
定義されているフィールドはこれだけ
(そして, メソッドはこれらのフィールドへの getter メソッド(アクセサメソッド)のみ).


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
      Block* _target;      // block target
      float  _prob;        // probability of edge to block
```





### 詳細(Details)
See: [here](../doxygen/classBlockProbPair.html) for details

---
## <a name="no3I_DdyZc" id="no3I_DdyZc">PhaseBlockLayout</a>

### 概要(Summary)
Phase クラスの具象サブクラスの1つ.

basic block 間の順番を (それらの遷移関係や実行される頻度に応じて) 並び替える.


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
    //------------------------------PhaseBlockLayout-------------------------------
    // Rearrange blocks into some canonical order, based on edges and their frequencies
    class PhaseBlockLayout : public Phase {
```

### 使われ方(Usage)
Compile::Code_Gen() 内で(のみ)使用されている.





### 詳細(Details)
See: [here](../doxygen/classPhaseBlockLayout.html) for details

---
## <a name="nopcGwj3MP" id="nopcGwj3MP">CFGEdge</a>

### 概要(Summary)
PhaseBlockLayout クラス内で使用される補助クラス.

CFG(Control Flow Graph) 中での edge を表すクラス
(= 2つの Block オブジェクト間での分岐またはフォールスルーによる遷移関係を示す).
1つの CFGEdge オブジェクトが 1つの edge に対応する.


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
    //----------------------------------CFGEdge------------------------------------
    // A edge between two basic blocks that will be embodied by a branch or a
    // fall-through.
    class CFGEdge : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 PhaseBlockLayout オブジェクトの edges フィールドに(のみ)格納されている.

(正確には, このフィールドは CFGEdge の GrowableArray を格納するフィールド.
この中に, その PhaseBlockLayout 内で使用される全ての CFGEdge オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
PhaseBlockLayout::find_edges() 内で(のみ)生成されている.





### 詳細(Details)
See: [here](../doxygen/classCFGEdge.html) for details

---
## <a name="nonDu459rZ" id="nonDu459rZ">Trace</a>

### 概要(Summary)
PhaseBlockLayout クラス内で使用される補助クラス.

Block オブジェクトを並び替える処理で使用される一時オブジェクト(ResourceObjクラス).
Block オブジェクトを順番づけるための双方向リストになっている.
1つの Trace オブジェクトが 1つの双方向リストを表す.


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
    //-----------------------------------Trace-------------------------------------
    // An ordered list of basic blocks.
    class Trace : public ResourceObj {
```

(なお使われ方としては, 
最初に Block オブジェクトと同数の Trace オブジェクトが作成され, 
関連性の強い Block 間の Trace 同士が 1つの Trace オブジェクトに併合されていくという感じになる.
最後に, 併合されずに残った Trace 間で順番が付けられて Block の並び替えが完了する)

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 PhaseBlockLayout オブジェクトの traces フィールドに(のみ)格納されている.

(正確には, このフィールドは Trace のポインタの配列を格納するフィールド.
この中に, その PhaseBlockLayout 内で使用される全ての Trace オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
PhaseBlockLayout::find_edges() 内で(のみ)生成されている.

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* uint _id

* Block ** _next_list
  
  (正確には, このフィールドは Block のポインタの配列を格納するフィールド)

  ある Block オブジェクトに対して, 双方向リスト中での1つ後ろの Block を示すフィールド
  (より具体的に言うと, 並び替える前のその Block の番号 (Block::_pre_order フィールドの値) から1つ後ろの Block への写像)

  (なおこの配列は, 同一の PhaseBlockLayout オブジェクト内で生成された全ての Trace オブジェクト間で共有される 
  (See: PhaseBlockLayout::find_edges()))

* Block ** _prev_list
  
  (正確には, このフィールドは Block のポインタの配列を格納するフィールド)

  ある Block オブジェクトに対して, 双方向リスト中での1つ前の Block を示すフィールド
  (より具体的に言うと, 並び替える前のその Block の番号 (Block::_pre_order フィールドの値) から1つ前の Block への写像)

  (なおこの配列は, 同一の PhaseBlockLayout オブジェクト内で生成された全ての Trace オブジェクト間で共有される 
  (See: PhaseBlockLayout::find_edges()))

* Block * _first
  
  双方向リストの先頭の Block を指すポインタ.

* Block * _last

  双方向リストの最後の Block を指すポインタ.  


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
      uint _id;             // Unique Trace id (derived from initial block)
      Block ** _next_list;  // Array mapping index to next block
      Block ** _prev_list;  // Array mapping index to previous block
      Block * _first;       // First block in the trace
      Block * _last;        // Last block in the trace
```




### 詳細(Details)
See: [here](../doxygen/classTrace.html) for details

---
## <a name="nopgbl2MDw" id="nopgbl2MDw">UnionFind</a>

### 概要(Summary)
PhaseBlockLayout クラス内で使用される補助クラス(ResourceObjクラス).

各 Block オブジェクトがどの Trace オブジェクトに所属しているかを記録しておくためのクラス
(= Block オブジェクトから Trace オブジェクトへの写像).

なお Trace オブジェクトは併合されることがあるため Union 処理も必要になる.
Union 処理とルックアップ処理 (Find 処理) を高速に行うため, 
内部的には Tarjan (1975) の UnionFind アルゴリズム(collapsing 有り)を用いている.


```cpp
    ((cite: hotspot/src/share/vm/opto/block.hpp))
    //------------------------------UnionFind--------------------------------------
    // Map Block indices to a block-index for a cfg-cover.
    // Array lookup in the optimized case.
    class UnionFind : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 PhaseBlockLayout オブジェクトの uf フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
PhaseBlockLayout::PhaseBlockLayout() 内で(のみ)生成されている.





### 詳細(Details)
See: [here](../doxygen/classUnionFind.html) for details

---
