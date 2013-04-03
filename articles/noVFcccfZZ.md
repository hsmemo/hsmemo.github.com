---
layout: default
title: PhaseRemoveUseless クラス, PhaseTransform クラスとそのサブクラス, 及びそれらの補助クラス (NodeHash, Type_Array, PhaseRemoveUseless, PhaseTransform, PhaseValues, PhaseGVN, PhaseIterGVN, PhaseCCP, PhasePeephole)
---
[Top](../index.html)

#### PhaseRemoveUseless クラス, PhaseTransform クラスとそのサブクラス, 及びそれらの補助クラス (NodeHash, Type_Array, PhaseRemoveUseless, PhaseTransform, PhaseValues, PhaseGVN, PhaseIterGVN, PhaseCCP, PhasePeephole)

これらは, C2 JIT Compiler 内の処理フェーズを表すクラス.
より具体的に言うと, コードに対する種々の最適化処理を表すクラス.


### クラス一覧(class list)

  * [PhaseRemoveUseless](#noHhDL8k_2)
  * [PhaseTransform](#novm9xznoa)
  * [PhaseValues](#noNsmLLTKm)
  * [PhaseGVN](#noYBhD3Wsj)
  * [PhaseIterGVN](#no5wIRXinX)
  * [PhaseCCP](#noiJh6UJAA)
  * [PhasePeephole](#no6jczUH35)
  * [NodeHash](#noNY2zkc1f)
  * [Type_Array](#noaRpaTONN)


---
## <a name="noHhDL8k_2" id="noHhDL8k_2">PhaseRemoveUseless</a>

### 概要(Summary)
Phase クラスの具象サブクラスの1つ.

無用命令の削除(dead code elimination)を行う.


```
    ((cite: hotspot/src/share/vm/opto/phaseX.hpp))
    //------------------------------PhaseRemoveUseless-----------------------------
    // Remove useless nodes from GVN hash-table, worklist, and graph
    class PhaseRemoveUseless : public Phase {
```

### 使われ方(Usage)
Compile::Compile( ciEnv* ci_env, C2Compiler* compiler, ciMethod* target, int osr_bci, bool subsume_loads, bool do_escape_analysis ) 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPhaseRemoveUseless.html) for details

---
## <a name="novm9xznoa" id="novm9xznoa">PhaseTransform</a>

### 概要(Summary)
Phase クラスのサブクラスの1つ.
コードを解析し新しい形に変形する Phase クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/phaseX.hpp))
    //------------------------------PhaseTransform---------------------------------
    // Phases that analyze, then transform.  Constructing the Phase object does any
    // global or slow analysis.  The results are cached later for a fast
    // transformation pass.  When the Phase object is deleted the cached analysis
    // results are deleted.
    class PhaseTransform : public Phase {
```

### 使われ方(Usage)
使用する際には, transform() メソッドをオーバーライドしたサブクラスを作ればいい.


```
    ((cite: hotspot/src/share/vm/opto/phaseX.hpp))
      // Return a node which computes the same function as this node, but
      // in a faster or cheaper fashion.
      virtual Node *transform( Node *n ) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classPhaseTransform.html) for details

---
## <a name="noNsmLLTKm" id="noNsmLLTKm">PhaseValues</a>

### 概要(Summary)
PhaseTransform クラスのサブクラスの1つ.

Value Numbering を行う.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/phaseX.hpp))
    //------------------------------PhaseValues------------------------------------
    // Phase infrastructure to support values
    class PhaseValues : public PhaseTransform {
```




### 詳細(Details)
See: [here](../doxygen/classPhaseValues.html) for details

---
## <a name="noYBhD3Wsj" id="noYBhD3Wsj">PhaseGVN</a>

### 概要(Summary)
PhaseValues クラスの具象サブクラス.

Global Value Numbering を行う.


```
    ((cite: hotspot/src/share/vm/opto/phaseX.hpp))
    //------------------------------PhaseGVN---------------------------------------
    // Phase for performing local, pessimistic GVN-style optimizations.
    class PhaseGVN : public PhaseValues {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* Compile::Compile( ciEnv* ci_env, C2Compiler* compiler, ciMethod* target, int osr_bci, bool subsume_loads, bool do_escape_analysis )
* Compile::Compile( ciEnv* ci_env, TypeFunc_generator generator, address stub_function, const char *stub_name, int is_fancy_jump, bool pass_tls, bool save_arg_registers, bool return_pc )




### 詳細(Details)
See: [here](../doxygen/classPhaseGVN.html) for details

---
## <a name="no5wIRXinX" id="no5wIRXinX">PhaseIterGVN</a>

### 概要(Summary)
PhaseValues クラスのサブクラス.

Global Value Numbering を行う.


```
    ((cite: hotspot/src/share/vm/opto/phaseX.hpp))
    //------------------------------PhaseIterGVN-----------------------------------
    // Phase for iteratively performing local, pessimistic GVN-style optimizations.
    // and ideal transformations on the graph.
    class PhaseIterGVN : public PhaseGVN {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* Compile::Optimize()
* PhaseIterGVN::optimize() (ただし #ifndef PRODUCT の場合にのみ使用される)




### 詳細(Details)
See: [here](../doxygen/classPhaseIterGVN.html) for details

---
## <a name="noiJh6UJAA" id="noiJh6UJAA">PhaseCCP</a>

### 概要(Summary)
PhaseIterGVN クラスのサブクラス.

定数伝播 (Conditional Constant Propagation) を行う.


```
    ((cite: hotspot/src/share/vm/opto/phaseX.hpp))
    //------------------------------PhaseCCP---------------------------------------
    // Phase for performing global Conditional Constant Propagation.
    // Should be replaced with combined CCP & GVN someday.
    class PhaseCCP : public PhaseIterGVN {
```

### 使われ方(Usage)
Compile::Optimize() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPhaseCCP.html) for details

---
## <a name="no6jczUH35" id="no6jczUH35">PhasePeephole</a>

### 概要(Summary)
PhaseTransform クラスの具象サブクラスの1つ.

のぞき穴式最適化 (peephole optimization) を行う.


```
    ((cite: hotspot/src/share/vm/opto/phaseX.hpp))
    //------------------------------PhasePeephole----------------------------------
    // Phase for performing peephole optimizations on register allocated basic blocks.
    class PhasePeephole : public PhaseTransform {
```

### 使われ方(Usage)
Compile::Code_Gen() 内で(のみ)使用されている.

(ただし, OptoPeephole オプションが指定されている時にしか使用されない)




### 詳細(Details)
See: [here](../doxygen/classPhasePeephole.html) for details

---
## <a name="noNY2zkc1f" id="noNY2zkc1f">NodeHash</a>

### 概要(Summary)
PhaseValues クラス内で使用される補助クラス(StackObjクラス).

Node をキーとし Node を値とするハッシュテーブル.
Value Numbering 時に同じ値を持つ Node を探すために使用される.


```
    ((cite: hotspot/src/share/vm/opto/phaseX.hpp))
    //-----------------------------------------------------------------------------
    // Expandable closed hash-table of nodes, initialized to NULL.
    // Note that the constructor just zeros things
    // Storage is reclaimed when the Arena's lifetime is over.
    class NodeHash : public StackObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 PhaseValues オブジェクトの _table フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(PhaseValues クラスの _table フィールドは, ポインタ型ではなく実体なので,
 PhaseValues オブジェクトの生成時に一緒に生成される)




### 詳細(Details)
See: [here](../doxygen/classNodeHash.html) for details

---
## <a name="noaRpaTONN" id="noaRpaTONN">Type_Array</a>

### 概要(Summary)
PhaseTransform クラス内で使用される補助クラス(StackObjクラス).

Type クラス用のユーティリティ・クラス.
Type オブジェクトの配列を表す (= 整数値から Type オブジェクトへの写像).
なお配列長は必要に応じて自動的に拡張される.


```
    ((cite: hotspot/src/share/vm/opto/phaseX.hpp))
    //-----------------------------------------------------------------------------
    // Map dense integer indices to Types.  Uses classic doubling-array trick.
    // Abstractly provides an infinite array of Type*'s, initialized to NULL.
    // Note that the constructor just zeros things, and since I use Arena
    // allocation I do not need a destructor to reclaim storage.
    // Despite the general name, this class is customized for use by PhaseTransform.
    class Type_Array : public StackObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 PhaseTransform オブジェクトの _types フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(PhaseTransform クラスの _types フィールドは, ポインタ型ではなく実体なので,
 PhaseTransform オブジェクトの生成時に一緒に生成される)




### 詳細(Details)
See: [here](../doxygen/classType__Array.html) for details

---
