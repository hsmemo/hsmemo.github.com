---
layout: default
title: ConnectionGraph クラス関連のクラス (PointsToNode, ConnectionGraph)
---
[Top](../index.html)

#### ConnectionGraph クラス関連のクラス (PointsToNode, ConnectionGraph)

これらは, C2 JIT Compiler 用のクラス.
より具体的に言うと, エスケープ解析(escape analysis)を行うためのクラス.


### クラス一覧(class list)

  * [ConnectionGraph](#no4syLp690)
  * [PointsToNode](#noRXTzrM1y)


---
## <a name="no4syLp690" id="no4syLp690">ConnectionGraph</a>

### 概要(Summary)
Compile クラス用の補助クラス(ResourceObjクラス).

エスケープ解析(escape analysis)を行うためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/escape.hpp))
    class ConnectionGraph: public ResourceObj {
```

なおコメントによると, Choi らの "Escape Analysis for Java" を実装したもの, とのこと
(コメントでは "Jong-Deok Shoi" になっているが "Jong-Deok Choi" の typo では?? #TODO)
("Mauricio Seffano" も "Mauricio Serrano" の typo では?? #TODO)
(あと "Procedings" も "Proceedings" の typo だな)

        Jong-Deok Shoi, Manish Gupta, Mauricio Seffano,
        Vugranam C. Sreedhar, Sam Midkiff,
        "Escape Analysis for Java", Procedings of ACM SIGPLAN
        OOPSLA  Conference, November 1, 1999

ConnectionGraph は, この論文のアルゴリズムで用いられる "connection graph" (CG) を表すクラス.

なお, Ideal に local variable という概念はないが, 便宜上以下のノードを local variable と見なしている.

  * Phi       (pointer values)
  * LoadP, LoadN
  * Proj#5    (value returned from call nodes including allocations)
  * CheckCastPP, CastPP, EncodeP, DecodeN
  * Return    (GlobalEscape)


```cpp
    ((cite: hotspot/src/share/vm/opto/escape.hpp))
    //
    // Adaptation for C2 of the escape analysis algorithm described in:
    //
    // [Choi99] Jong-Deok Shoi, Manish Gupta, Mauricio Seffano,
    //          Vugranam C. Sreedhar, Sam Midkiff,
    //          "Escape Analysis for Java", Procedings of ACM SIGPLAN
    //          OOPSLA  Conference, November 1, 1999
    //
    // The flow-insensitive analysis described in the paper has been implemented.
    //
    // The analysis requires construction of a "connection graph" (CG) for
    // the method being analyzed.  The nodes of the connection graph are:
    //
    //     -  Java objects (JO)
    //     -  Local variables (LV)
    //     -  Fields of an object (OF),  these also include array elements
    //
    // The CG contains 3 types of edges:
    //
    //   -  PointsTo  (-P>)    {LV, OF} to JO
    //   -  Deferred  (-D>)    from {LV, OF} to {LV, OF}
    //   -  Field     (-F>)    from JO to OF
    //
    // The following  utility functions is used by the algorithm:
    //
    //   PointsTo(n) - n is any CG node, it returns the set of JO that n could
    //                 point to.
    //
    // The algorithm describes how to construct the connection graph
    // in the following 4 cases:
    //
    //          Case                  Edges Created
    //
    // (1)   p   = new T()              LV -P> JO
    // (2)   p   = q                    LV -D> LV
    // (3)   p.f = q                    JO -F> OF,  OF -D> LV
    // (4)   p   = q.f                  JO -F> OF,  LV -D> OF
    //
    // In all these cases, p and q are local variables.  For static field
    // references, we can construct a local variable containing a reference
    // to the static memory.
    //
    // C2 does not have local variables.  However for the purposes of constructing
    // the connection graph, the following IR nodes are treated as local variables:
    //     Phi    (pointer values)
    //     LoadP
    //     Proj#5 (value returned from callnodes including allocations)
    //     CheckCastPP, CastPP
    //
    // The LoadP, Proj and CheckCastPP behave like variables assigned to only once.
    // Only a Phi can have multiple assignments.  Each input to a Phi is treated
    // as an assignment to it.
    //
    // The following node types are JavaObject:
    //
    //     top()
    //     Allocate
    //     AllocateArray
    //     Parm  (for incoming arguments)
    //     CastX2P ("unsafe" operations)
    //     CreateEx
    //     ConP
    //     LoadKlass
    //     ThreadLocal
    //
    // AddP nodes are fields.
    //
    // After building the graph, a pass is made over the nodes, deleting deferred
    // nodes and copying the edges from the target of the deferred edge to the
    // source.  This results in a graph with no deferred edges, only:
    //
    //    LV -P> JO
    //    OF -P> JO (the object whose oop is stored in the field)
    //    JO -F> OF
    //
    // Then, for each node which is GlobalEscape, anything it could point to
    // is marked GlobalEscape.  Finally, for any node marked ArgEscape, anything
    // it could point to is marked ArgEscape.
    //
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Compile オブジェクトの _congraph フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
ConnectionGraph::do_analysis() 内で(のみ)生成されている
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Compile::Optimize()
-> ConnectionGraph::do_analysis()
```




### 詳細(Details)
See: [here](../doxygen/classConnectionGraph.html) for details

---
## <a name="noRXTzrM1y" id="noRXTzrM1y">PointsToNode</a>

### 概要(Summary)
ConnectionGraph クラス用の補助クラス.

"connection graph" (CG) 中での edge (PointsTo, Deferred, Field) を表すクラス.
1つの PointsToNode オブジェクトが 1つの Ideal オブジェクト(から出る全ての edge) に対応する.


```cpp
    ((cite: hotspot/src/share/vm/opto/escape.hpp))
    class PointsToNode {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ConnectionGraph オブジェクトの _nodes フィールドに(のみ)格納されている
(アクセサは ConnectionGraph::ptnode_adr()).
 
(正確には, このフィールドは PointsToNode の GrowableArray を格納するフィールド.
この中に, その ConnectionGraph 内で使用される全ての PointsToNode オブジェクトが格納されている)

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* (#TODO)

* GrowableArray<uint>* _edges
  
  実際の edge 情報を格納した GrowableArray.
  なお, edge の種別 (PointsTo, Deferred, Field) は uint 値の下位 2bit に埋め込まれている
  (この 2bit が PointsToNode::NodeType 型の定数値を示す).


```cpp
    ((cite: hotspot/src/share/vm/opto/escape.hpp))
      Node* _node;              // Ideal node corresponding to this PointsTo node.
      int   _offset;            // Object fields offsets.
      bool  _scalar_replaceable;// Not escaped object could be replaced with scalar
      bool  _hidden_alias;      // This node is an argument to a function.
                                // which may return it creating a hidden alias.
```


```cpp
    ((cite: hotspot/src/share/vm/opto/escape.hpp))
      NodeType             _type;
      EscapeState          _escape;
      GrowableArray<uint>* _edges;   // outgoing edges
```




### 詳細(Details)
See: [here](../doxygen/classPointsToNode.html) for details

---
