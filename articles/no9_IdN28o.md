---
layout: default
title: MultiNode に関する高レベル中間語(Ideal)クラス (MultiNode, ProjNode)
---
[Top](../index.html)

#### MultiNode に関する高レベル中間語(Ideal)クラス (MultiNode, ProjNode)

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, 複数の出力を持つ中間語を表すクラス.


### クラス一覧(class list)

  * [MultiNode](#noxOocynYo)
  * [ProjNode](#nozhA-wIn1)


---
## <a name="noxOocynYo" id="noxOocynYo">MultiNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.

複数の出力を出す Node クラスの基底クラス. 

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/multnode.hpp))
    //------------------------------MultiNode--------------------------------------
    // This class defines a MultiNode, a Node which produces many values.  The
    // values are wrapped up in a tuple Type, i.e. a TypeTuple.
    class MultiNode : public Node {
```

### 内部構造(Internal structure)
このクラス自体は入力ノードを規定しない (= 入力ノードはない).


```
    ((cite: hotspot/src/share/vm/opto/multnode.hpp))
      MultiNode( uint required ) : Node(required) {
        init_class_id(Class_Multi);
      }
```

### 備考(Notes)
なお, MultiNode の出力の型は TypeTuple として扱われる.




### 詳細(Details)
See: [here](../doxygen/classMultiNode.html) for details

---
## <a name="nozhA-wIn1" id="nozhA-wIn1">ProjNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.

MultiNode (やそのサブクラス) が表すタプルから要素を1つ取り出すための Node.


```
    ((cite: hotspot/src/share/vm/opto/multnode.hpp))
    //------------------------------ProjNode---------------------------------------
    // This class defines a Projection node.  Projections project a single element
    // out of a tuple (or Signature) type.  Only MultiNodes produce TypeTuple
    // results.
    class ProjNode : public Node {
```

### 使われ方(Usage)
以下の箇所で(のみ)生成されている.

* LibraryCallKit::inline_trig()
* DivModINode::make()
* DivModLNode::make()
* GraphKit::gen_stub()
* GraphKit::set_all_memory_call()
* GraphKit::set_edges_for_java_call()
* GraphKit::set_results_for_java_call()
* GraphKit::set_predefined_output_for_runtime_call()
* ... (#TODO)

### 内部構造(Internal structure)
control input は持たない. 入力ノードは 1つで, 処理対象の MultiNode を指す.


```
    ((cite: hotspot/src/share/vm/opto/multnode.hpp))
      ProjNode( Node *src, uint con, bool io_use = false )
        : Node( src ), _con(con), _is_io_use(io_use)
      {
        init_class_id(Class_Proj);
        // Optimistic setting. Need additional checks in Node::is_dead_loop_safe().
        if (con != TypeFunc::Memory || src->is_Start())
          init_flags(Flag_is_dead_loop_safe);
        debug_only(check_con());
      }
```





### 詳細(Details)
See: [here](../doxygen/classProjNode.html) for details

---
