---
layout: default
title: メソッド呼び出しに関する高レベル中間語(Ideal)クラス (StartNode, StartOSRNode, ParmNode, ReturnNode, RethrowNode, TailCallNode, TailJumpNode, JVMState, SafePointNode, SafePointScalarObjectNode, CallProjections, CallNode, CallJavaNode, CallStaticJavaNode, CallDynamicJavaNode, CallRuntimeNode, CallLeafNode, CallLeafNoFPNode, AllocateNode, AllocateArrayNode, AbstractLockNode, LockNode, UnlockNode)
---
[Top](../index.html)

#### メソッド呼び出しに関する高レベル中間語(Ideal)クラス (StartNode, StartOSRNode, ParmNode, ReturnNode, RethrowNode, TailCallNode, TailJumpNode, JVMState, SafePointNode, SafePointScalarObjectNode, CallProjections, CallNode, CallJavaNode, CallStaticJavaNode, CallDynamicJavaNode, CallRuntimeNode, CallLeafNode, CallLeafNoFPNode, AllocateNode, AllocateArrayNode, AbstractLockNode, LockNode, UnlockNode)

#Under Construction

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, 「メソッド呼び出し (及びそれに関連する処理)」 を表すクラス.

なおメモリ確保処理(AllocateNode/AllocateArrayNode)や同期排他処理(LockNode/UnlockNode)についても, 
ランタイム呼び出しを含むコードに展開される(可能性がある)ので CallNode のサブクラスとされている.


### クラス一覧(class list)

  * [StartNode](#no8nFFoztD)
  * [StartOSRNode](#noxnKXwLPk)
  * [ParmNode](#no7f6EeUli)
  * [ReturnNode](#no0oEAekev)
  * [RethrowNode](#noFdhOlbTF)
  * [TailCallNode](#noglGEEviC)
  * [TailJumpNode](#nonPm2Rr4B)
  * [JVMState](#noMaEU8bxt)
  * [SafePointNode](#noCKQNqAFt)
  * [SafePointScalarObjectNode](#nobumpzV_e)
  * [CallProjections](#nooyGoI1mp)
  * [CallNode](#noUlJUqcR2)
  * [CallJavaNode](#noqXQK9-1i)
  * [CallStaticJavaNode](#noH1pnBPw-)
  * [CallDynamicJavaNode](#nouX0r54yW)
  * [CallRuntimeNode](#noNLFq3OMl)
  * [CallLeafNode](#nonpB5-T5V)
  * [CallLeafNoFPNode](#no1Cwz3OmX)
  * [AllocateNode](#no9uzUVdJC)
  * [AllocateArrayNode](#noboPzs-gf)
  * [AbstractLockNode](#nomJNgXEN9)
  * [LockNode](#noKcwIvEGl)
  * [UnlockNode](#noNsDFFtXO)


---
## <a name="no8nFFoztD" id="no8nFFoztD">StartNode</a>

### 概要(Summary)
MultiNode クラスの具象サブクラスの1つ.
メソッドの開始点を表す Node.

RootNode の直下に存在し, メソッド開始時点でのプログラムの状態を表す.

MultiNode クラスなので出力は複数存在し, 
control flow (制御情報)とメソッドの引数(データ情報)からなる Tuple になっている
(なお個々の要素は ProjNode のサブクラスである ParmNode で取り出す).


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // The method start node
    class StartNode : public MultiNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* Compile::Compile( ciEnv* ci_env, C2Compiler* compiler, ciMethod* target, int osr_bci, bool subsume_loads, bool do_escape_analysis )
* GraphKit::gen_stub()

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input (自分自身を指す)
* 2番目の入力Node : RootNode を指す (正確には, 型の上ではどんな Node も設定可能になっているが, 実際には RootNode しか設定されない)


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      StartNode( Node *root, const TypeTuple *domain ) : MultiNode(2), _domain(domain) {
        init_class_id(Class_Start);
        init_flags(Flag_is_block_start);
        init_req(0,this);
        init_req(1,root);
      }
```




### 詳細(Details)
See: [here](../doxygen/classStartNode.html) for details

---
## <a name="noxnKXwLPk" id="noxnKXwLPk">StartOSRNode</a>

### 概要(Summary)
特殊な StartNode クラス.

OSR (On Stack Replacement) の場合には StartNode の代わりにこのクラスが使用される.
役割は StartNode とほぼ同様.


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // The method start node for on stack replacement code
    class StartOSRNode : public StartNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
Compile::Compile( ciEnv* ci_env, C2Compiler* compiler, ciMethod* target, int osr_bci, bool subsume_loads, bool do_escape_analysis ) 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 2つの入力ノードを持つ. それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input (自分自身を指す)
* 2番目の入力Node : RootNode を指す (正確には, 型の上ではどんな Node も設定可能になっているが, 実際には RootNode しか設定されない)


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      StartOSRNode( Node *root, const TypeTuple *domain ) : StartNode(root, domain) {}
```




### 詳細(Details)
See: [here](../doxygen/classStartOSRNode.html) for details

---
## <a name="no7f6EeUli" id="no7f6EeUli">ParmNode</a>

### 概要(Summary)
ProjNode クラスのサブクラスの1つ. 

StartNode/StartOSRNode から引数を取り出すためのノード. 


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Incoming parameters
    class ParmNode : public ProjNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* Compile::build_start_state()
  
  コンパイル対象のメソッドの引数を表す.

* GraphKit::gen_stub()
  
  生成対象のスタブの引数を表す.

* AllocateArrayNode::Ideal()
  
  HaltNode の引数 (frameptr) を取得するために生成されている.
  
* PhaseIdealLoop::build_loop_tree_impl()

  HaltNode の引数 (frameptr) を取得するために生成されている.

### 内部構造(Internal structure)
入力ノードは control input のみ. control input は対応する StartNode/StartOSRNode を示す.

(なお, コンストラクタには対応する StartNode/StartOSRNode と何番目の引数かを示す数字が渡される)


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      ParmNode( StartNode *src, uint con ) : ProjNode(src,con) {
        init_class_id(Class_Parm);
      }
```




### 詳細(Details)
See: [here](../doxygen/classParmNode.html) for details

---
## <a name="no0oEAekev" id="no0oEAekev">ReturnNode</a>

### 概要(Summary)
リターン(及びそれによるメソッドの終了処理)を表す Node.


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Return from subroutine node
    class ReturnNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* Compile::return_values()

  コンパイル対象のメソッドからのリターンを表す.

* GraphKit::gen_stub()
  
  生成対象のスタブからのリターンを表す.

### 内部構造(Internal structure)
(control input も含めて) 5つまたは6つの入力ノードを持つ. 返値がなければ 5個, あれば 6個になる.
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : TypeFunc::Control (control input)
* 2番目の入力Node : TypeFunc::I_O (I/O state)(#TODO)
* 3番目の入力Node : TypeFunc::Memory (memory state)
* 4番目の入力Node : TypeFunc::FramePtr (#TODO)
* 5番目の入力Node : TypeFunc::ReturnAdr (#TODO)
* 6番目の入力Node : TypeFunc::Parms (返値)


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.cpp))
    ReturnNode::ReturnNode(uint edges, Node *cntrl, Node *i_o, Node *memory, Node *frameptr, Node *retadr ) : Node(edges) {
      init_req(TypeFunc::Control,cntrl);
      init_req(TypeFunc::I_O,i_o);
      init_req(TypeFunc::Memory,memory);
      init_req(TypeFunc::FramePtr,frameptr);
      init_req(TypeFunc::ReturnAdr,retadr);
    }
```




### 詳細(Details)
See: [here](../doxygen/classReturnNode.html) for details

---
## <a name="noFdhOlbTF" id="noFdhOlbTF">RethrowNode</a>

### 概要(Summary)
throws 宣言されている例外による大域脱出処理(及びそれによるメソッドの終了処理)を表す Node.


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Rethrow of exception at call site.  Ends a procedure before rethrowing;
    // ends the current basic block like a ReturnNode.  Restores registers and
    // unwinds stack.  Rethrow happens in the caller's method.
    class RethrowNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
Compile::rethrow_exceptions() でのみ生成されている.

### 内部構造(Internal structure)
(control input も含めて) 6つの入力ノードを持つ.
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : TypeFunc::Control (control input)
* 2番目の入力Node : TypeFunc::I_O (I/O state)(#TODO)
* 3番目の入力Node : TypeFunc::Memory (memory state)
* 4番目の入力Node : TypeFunc::FramePtr (#TODO)
* 5番目の入力Node : TypeFunc::ReturnAdr (#TODO)
* 6番目の入力Node : TypeFunc::Parms (送出する例外オブジェクト)


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.cpp))
    RethrowNode::RethrowNode(
      Node* cntrl,
      Node* i_o,
      Node* memory,
      Node* frameptr,
      Node* ret_adr,
      Node* exception
    ) : Node(TypeFunc::Parms + 1) {
      init_req(TypeFunc::Control  , cntrl    );
      init_req(TypeFunc::I_O      , i_o      );
      init_req(TypeFunc::Memory   , memory   );
      init_req(TypeFunc::FramePtr , frameptr );
      init_req(TypeFunc::ReturnAdr, ret_adr);
      init_req(TypeFunc::Parms    , exception);
    }
```




### 詳細(Details)
See: [here](../doxygen/classRethrowNode.html) for details

---
## <a name="noglGEEviC" id="noglGEEviC">TailCallNode</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Pop stack frame and jump indirect
    class TailCallNode : public ReturnNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
GraphKit::gen_stub() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 7つの入力ノードを持つ.
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : TypeFunc::Control (control input)
* 2番目の入力Node : TypeFunc::I_O (I/O state)(#TODO)
* 3番目の入力Node : TypeFunc::Memory (memory state)
* 4番目の入力Node : TypeFunc::FramePtr (#TODO)
* 5番目の入力Node : TypeFunc::ReturnAdr (#TODO)
* 6番目の入力Node : TypeFunc::Parms (飛び先のアドレス)
* 7番目の入力Node : TypeFunc::Parms+1 (飛び先への引数)


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      TailCallNode( Node *cntrl, Node *i_o, Node *memory, Node *frameptr, Node *retadr, Node *target, Node *moop )
        : ReturnNode( TypeFunc::Parms+2, cntrl, i_o, memory, frameptr, retadr ) {
        init_req(TypeFunc::Parms, target);
        init_req(TypeFunc::Parms+1, moop);
      }
```




### 詳細(Details)
See: [here](../doxygen/classTailCallNode.html) for details

---
## <a name="nonPm2Rr4B" id="nonPm2Rr4B">TailJumpNode</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Pop stack frame and jump indirect
    class TailJumpNode : public ReturnNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
GraphKit::gen_stub() でのみ生成されている.

### 内部構造(Internal structure)
(control input も含めて) 7つの入力ノードを持つ.
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : TypeFunc::Control (control input)
* 2番目の入力Node : TypeFunc::I_O (I/O state)(#TODO)
* 3番目の入力Node : TypeFunc::Memory (memory state)
* 4番目の入力Node : TypeFunc::FramePtr (#TODO)
* 5番目の入力Node : TypeFunc::ReturnAdr (#TODO)
* 6番目の入力Node : TypeFunc::Parms (飛び先のアドレス)
* 7番目の入力Node : TypeFunc::Parms+1 (飛び先への引数)


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      TailJumpNode( Node *cntrl, Node *i_o, Node *memory, Node *frameptr, Node *target, Node *ex_oop)
        : ReturnNode(TypeFunc::Parms+2, cntrl, i_o, memory, frameptr, Compile::current()->top()) {
        init_req(TypeFunc::Parms, target);
        init_req(TypeFunc::Parms+1, ex_oop);
      }
```




### 詳細(Details)
See: [here](../doxygen/classTailJumpNode.html) for details

---
## <a name="noMaEU8bxt" id="noMaEU8bxt">JVMState</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // A linked list of JVMState nodes captures the whole interpreter state,
    // plus GC roots, for all active calls at some call site in this compilation
    // unit.  (If there is no inlining, then the list has exactly one link.)
    // This provides a way to map the optimized program back into the interpreter,
    // or to let the GC mark the stack.
    class JVMState : public ResourceObj {
```



### 詳細(Details)
See: [here](../doxygen/classJVMState.html) for details

---
## <a name="noCKQNqAFt" id="noCKQNqAFt">SafePointNode</a>

### 概要(Summary)


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // A SafePointNode is a subclass of a MultiNode for convenience (and
    // potential code sharing) only - conceptually it is independent of
    // the Node semantics.
    class SafePointNode : public MultiNode {
```

### 使われ方(Usage)
以下の箇所で(のみ)生成されている.

* Parse::create_entry_map()
* LateInlineCallGenerator::do_late_inline()
* GraphKit::gen_stub()
* GraphKit::transfer_exceptions_into_jvms()
* Compile::build_start_state()
* Parse::add_safepoint()
* PhaseStringOpts::replace_string_concat()

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      SafePointNode(uint edges, JVMState* jvms,
                    // A plain safepoint advertises no memory effects (NULL):
                    const TypePtr* adr_type = NULL)
        : MultiNode( edges ),
          _jvms(jvms),
          _oop_map(NULL),
          _adr_type(adr_type)
      {
        init_class_id(Class_SafePoint);
      }
```




### 詳細(Details)
See: [here](../doxygen/classSafePointNode.html) for details

---
## <a name="nobumpzV_e" id="nobumpzV_e">SafePointScalarObjectNode</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // A SafePointScalarObjectNode represents the state of a scalarized object
    // at a safepoint.
    
    class SafePointScalarObjectNode: public TypeNode {
```

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.cpp))
    SafePointScalarObjectNode::SafePointScalarObjectNode(const TypeOopPtr* tp,
    #ifdef ASSERT
                                                         AllocateNode* alloc,
    #endif
                                                         uint first_index,
                                                         uint n_fields) :
      TypeNode(tp, 1), // 1 control input -- seems required.  Get from root.
    #ifdef ASSERT
      _alloc(alloc),
    #endif
      _first_index(first_index),
      _n_fields(n_fields)
    {
      init_class_id(Class_SafePointScalarObject);
    }
```




### 詳細(Details)
See: [here](../doxygen/classSafePointScalarObjectNode.html) for details

---
## <a name="nooyGoI1mp" id="nooyGoI1mp">CallProjections</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Simple container for the outgoing projections of a call.  Useful
    // for serious surgery on calls.
    class CallProjections : public StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classCallProjections.html) for details

---
## <a name="noUlJUqcR2" id="noUlJUqcR2">CallNode</a>

### 概要(Summary)
SafePointNode クラスのサブクラスの1つ.

「メソッド呼び出し処理」を表す全ての Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Call nodes now subsume the function of debug nodes at callsites, so they
    // contain the functionality of a full scope chain of debug nodes.
    class CallNode : public SafePointNode {
```

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      CallNode(const TypeFunc* tf, address addr, const TypePtr* adr_type)
        : SafePointNode(tf->domain()->cnt(), NULL, adr_type),
          _tf(tf),
          _entry_point(addr),
          _cnt(COUNT_UNKNOWN)
      {
        init_class_id(Class_Call);
        init_flags(Flag_is_Call);
      }
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classCallNode.html) for details

---
## <a name="noqXQK9-1i" id="noqXQK9-1i">CallJavaNode</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Make a static or dynamic subroutine call node using Java calling
    // convention.  (The "Java" calling convention is the compiler's calling
    // convention, as opposed to the interpreter's or that of native C.)
    class CallJavaNode : public CallNode {
```

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      CallJavaNode(const TypeFunc* tf , address addr, ciMethod* method, int bci)
        : CallNode(tf, addr, TypePtr::BOTTOM),
          _method(method), _bci(bci),
          _optimized_virtual(false),
          _method_handle_invoke(false)
      {
        init_class_id(Class_CallJava);
      }
```





### 詳細(Details)
See: [here](../doxygen/classCallJavaNode.html) for details

---
## <a name="noH1pnBPw-" id="noH1pnBPw-">CallStaticJavaNode</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Make a direct subroutine call using Java calling convention (for static
    // calls and optimized virtual calls, plus calls to wrappers for run-time
    // routines); generates static stub.
    class CallStaticJavaNode : public CallJavaNode {
```

### 使われ方(Usage)
以下の箇所で(のみ)生成されている.

* DirectCallGenerator::generate()
* DynamicCallGenerator::generate()
* GraphKit::make_runtime_call()
* LibraryCallKit::generate_method_call()
* PhaseMacroExpand::make_slow_call()
* Compile::call_zap_node()
* StringConcat::convert_uncommon_traps()

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      CallStaticJavaNode(const TypeFunc* tf, address addr, ciMethod* method, int bci)
        : CallJavaNode(tf, addr, method, bci), _name(NULL) {
        init_class_id(Class_CallStaticJava);
      }
      CallStaticJavaNode(const TypeFunc* tf, address addr, const char* name, int bci,
                         const TypePtr* adr_type)
        : CallJavaNode(tf, addr, NULL, bci), _name(name) {
        init_class_id(Class_CallStaticJava);
        // This node calls a runtime stub, which often has narrow memory effects.
        _adr_type = adr_type;
      }
```




### 詳細(Details)
See: [here](../doxygen/classCallStaticJavaNode.html) for details

---
## <a name="nouX0r54yW" id="nouX0r54yW">CallDynamicJavaNode</a>

### 概要(Summary)
CallJavaNode クラスの具象サブクラスの1つ. 

このクラスは, ダイナミックディスパッチが必要なメソッド呼び出し用
(e.g. invokevirtual, invokeinterface).


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Make a dispatched call using Java calling convention.
    class CallDynamicJavaNode : public CallJavaNode {
```

### 使われ方(Usage)
以下の箇所で(のみ)生成されている.

* VirtualCallGenerator::generate()
* LibraryCallKit::generate_method_call()

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      CallDynamicJavaNode( const TypeFunc *tf , address addr, ciMethod* method, int vtable_index, int bci ) : CallJavaNode(tf,addr,method,bci), _vtable_index(vtable_index) {
        init_class_id(Class_CallDynamicJava);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCallDynamicJavaNode.html) for details

---
## <a name="noNLFq3OMl" id="noNLFq3OMl">CallRuntimeNode</a>

### 概要(Summary)
CallNode のサブクラス. ("a direct subroutine call node into compiled C++ code") (#TODO)

  * CallRuntimeNode  : 下記以外の一般的なケース用
  * CallLeafNode     : safepoint が内部で発生しない場合用.
  * CallLeafNoFPNode : #TODO


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Make a direct subroutine call node into compiled C++ code.
    class CallRuntimeNode : public CallNode {
```

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      CallRuntimeNode(const TypeFunc* tf, address addr, const char* name,
                      const TypePtr* adr_type)
        : CallNode(tf, addr, adr_type),
          _name(name)
      {
        init_class_id(Class_CallRuntime);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCallRuntimeNode.html) for details

---
## <a name="nonpB5-T5V" id="nonpB5-T5V">CallLeafNode</a>

### 概要(Summary)
CallRuntimeNode のサブクラス. ("a direct subroutine call node into compiled C++ code, without safepoints")

  * CallLeafNode     : safepoint が内部で発生しない場合用.


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // Make a direct subroutine call node into compiled C++ code, without
    // safepoints
    class CallLeafNode : public CallRuntimeNode {
```


### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      CallLeafNode(const TypeFunc* tf, address addr, const char* name,
                   const TypePtr* adr_type)
        : CallRuntimeNode(tf, addr, name, adr_type)
      {
        init_class_id(Class_CallLeaf);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCallLeafNode.html) for details

---
## <a name="no1Cwz3OmX" id="no1Cwz3OmX">CallLeafNoFPNode</a>

### 概要(Summary)
特殊な CallLeafNode クラス. 

浮動小数を使わないケース用("not using floating point or using it in the same manner as the generated code").


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // CallLeafNode, not using floating point or using it in the same manner as
    // the generated code
    class CallLeafNoFPNode : public CallLeafNode {
```

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      CallLeafNoFPNode(const TypeFunc* tf, address addr, const char* name,
                       const TypePtr* adr_type)
        : CallLeafNode(tf, addr, name, adr_type)
      {
      }
```




### 詳細(Details)
See: [here](../doxygen/classCallLeafNoFPNode.html) for details

---
## <a name="no9uzUVdJC" id="no9uzUVdJC">AllocateNode</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // High-level memory allocation
    //
    //  AllocateNode and AllocateArrayNode are subclasses of CallNode because they will
    //  get expanded into a code sequence containing a call.  Unlike other CallNodes,
    //  they have 2 memory projections and 2 i_o projections (which are distinguished by
    //  the _is_io_use flag in the projection.)  This is needed when expanding the node in
    //  order to differentiate the uses of the projection on the normal control path from
    //  those on the exception return path.
    //
    class AllocateNode : public CallNode {
```


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      AllocateNode(Compile* C, const TypeFunc *atype, Node *ctrl, Node *mem, Node *abio,
                   Node *size, Node *klass_node, Node *initial_test);
```


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      enum {
        // Output:
        RawAddress  = TypeFunc::Parms,    // the newly-allocated raw address
        // Inputs:
        AllocSize   = TypeFunc::Parms,    // size (in bytes) of the new object
        KlassNode,                        // type (maybe dynamic) of the obj.
        InitialTest,                      // slow-path test (may be constant)
        ALength,                          // array length (or TOP if none)
        ParmLimit
      };
```

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.cpp))
    AllocateNode::AllocateNode(Compile* C, const TypeFunc *atype,
                               Node *ctrl, Node *mem, Node *abio,
                               Node *size, Node *klass_node, Node *initial_test)
      : CallNode(atype, NULL, TypeRawPtr::BOTTOM)
    {
      init_class_id(Class_Allocate);
      init_flags(Flag_is_macro);
      _is_scalar_replaceable = false;
      Node *topnode = C->top();
    
      init_req( TypeFunc::Control  , ctrl );
      init_req( TypeFunc::I_O      , abio );
      init_req( TypeFunc::Memory   , mem );
      init_req( TypeFunc::ReturnAdr, topnode );
      init_req( TypeFunc::FramePtr , topnode );
      init_req( AllocSize          , size);
      init_req( KlassNode          , klass_node);
      init_req( InitialTest        , initial_test);
      init_req( ALength            , topnode);
      C->add_macro_node(this);
    }
```




### 詳細(Details)
See: [here](../doxygen/classAllocateNode.html) for details

---
## <a name="noboPzs-gf" id="noboPzs-gf">AllocateArrayNode</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    //
    // High-level array allocation
    //
    class AllocateArrayNode : public AllocateNode {
```

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      AllocateArrayNode(Compile* C, const TypeFunc *atype, Node *ctrl, Node *mem, Node *abio,
                        Node* size, Node* klass_node, Node* initial_test,
                        Node* count_val
                        )
        : AllocateNode(C, atype, ctrl, mem, abio, size, klass_node,
                       initial_test)
      {
        init_class_id(Class_AllocateArray);
        set_req(AllocateNode::ALength,        count_val);
      }
```




### 詳細(Details)
See: [here](../doxygen/classAllocateArrayNode.html) for details

---
## <a name="nomJNgXEN9" id="nomJNgXEN9">AbstractLockNode</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    class AbstractLockNode: public CallNode {
```

### 内部構造(Internal structure)
通常の CallNode の入力ノードに加えて, 以下のような 3つの入力ノードを持つ.


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      Node *   obj_node() const       {return in(TypeFunc::Parms + 0); }
      Node *   box_node() const       {return in(TypeFunc::Parms + 1); }
      Node *   fastlock_node() const  {return in(TypeFunc::Parms + 2); }
```


```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      AbstractLockNode(const TypeFunc *tf)
        : CallNode(tf, NULL, TypeRawPtr::BOTTOM),
          _coarsened(false),
          _eliminate(false)
      {
    #ifndef PRODUCT
        _counter = NULL;
    #endif
      }
```




### 詳細(Details)
See: [here](../doxygen/classAbstractLockNode.html) for details

---
## <a name="noKcwIvEGl" id="noKcwIvEGl">LockNode</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // High-level lock operation
    //
    // This is a subclass of CallNode because it is a macro node which gets expanded
    // into a code sequence containing a call.  This node takes 3 "parameters":
    //    0  -  object to lock
    //    1 -   a BoxLockNode
    //    2 -   a FastLockNode
    //
    class LockNode : public AbstractLockNode {
```


```cpp
    ((cite: hotspot/src/share/vm/opto/graphKit.cpp))
    FastLockNode* GraphKit::shared_lock(Node* obj) {
    ...
      LockNode *lock = new (C, tf->domain()->cnt()) LockNode(C, tf);
    
      lock->init_req( TypeFunc::Control, control() );
      lock->init_req( TypeFunc::Memory , mem );
      lock->init_req( TypeFunc::I_O    , top() )     ;   // does no i/o
      lock->init_req( TypeFunc::FramePtr, frameptr() );
      lock->init_req( TypeFunc::ReturnAdr, top() );
    
      lock->init_req(TypeFunc::Parms + 0, obj);
      lock->init_req(TypeFunc::Parms + 1, box);
      lock->init_req(TypeFunc::Parms + 2, flock);
```

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      LockNode(Compile* C, const TypeFunc *tf) : AbstractLockNode( tf ) {
        init_class_id(Class_Lock);
        init_flags(Flag_is_macro);
        C->add_macro_node(this);
      }
```




### 詳細(Details)
See: [here](../doxygen/classLockNode.html) for details

---
## <a name="noNsDFFtXO" id="noNsDFFtXO">UnlockNode</a>

### 概要(Summary)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
    // High-level unlock operation
    class UnlockNode : public AbstractLockNode {
```


```cpp
    ((cite: hotspot/src/share/vm/opto/graphKit.cpp))
    void GraphKit::shared_unlock(Node* box, Node* obj) {
    ...
      UnlockNode *unlock = new (C, tf->domain()->cnt()) UnlockNode(C, tf);
      uint raw_idx = Compile::AliasIdxRaw;
      unlock->init_req( TypeFunc::Control, control() );
      unlock->init_req( TypeFunc::Memory , memory(raw_idx) );
      unlock->init_req( TypeFunc::I_O    , top() )     ;   // does no i/o
      unlock->init_req( TypeFunc::FramePtr, frameptr() );
      unlock->init_req( TypeFunc::ReturnAdr, top() );
    
      unlock->init_req(TypeFunc::Parms + 0, obj);
      unlock->init_req(TypeFunc::Parms + 1, box);
```

### 内部構造(Internal structure)

```cpp
    ((cite: hotspot/src/share/vm/opto/callnode.hpp))
      UnlockNode(Compile* C, const TypeFunc *tf) : AbstractLockNode( tf ) {
        init_class_id(Class_Unlock);
        init_flags(Flag_is_macro);
        C->add_macro_node(this);
      }
```





### 詳細(Details)
See: [here](../doxygen/classUnlockNode.html) for details

---
