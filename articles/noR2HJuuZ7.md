---
layout: default
title: OptoReg クラスおよび OptoRegPair クラス (OptoReg, OptoRegPair)
---
[Top](../index.html)

#### OptoReg クラスおよび OptoRegPair クラス (OptoReg, OptoRegPair)

これらは, C2 JIT Compiler 用のユーティリティ・クラス.
より具体的に言うと, C2 JIT コンパイラが(レジスタ割り当てなどで)使用する仮想的なレジスタを表すクラス.

(より正確に言うと, VMReg と同様, レジスタだけでなくスタック上のスロットも含めて統一的に扱うためのクラス)

なお OptoRegPair の宣言部にあるコメントによると, 
レジスタを表すクラスとしてはマシン依存のレジスタを表現する VMReg 及び VMRegPair があるが,
C2 内では calling convention を扱うコード以外は 
OptoReg と OptoRegPair を使用することにして, あまりマシン依存しないようにしている模様
(これにより, アーキテクチャによってはレジスタではない condition flag や, 
 中間語の Control, Memory, I/O 等もアーキテクチャ依存せずにとりあえず「レジスタ」として扱えている模様).



### クラス一覧(class list)

  * [OptoReg](#noHeZHM90H)
  * [OptoRegPair](#noediOVbla)


---
## <a name="noHeZHM90H" id="noHeZHM90H">OptoReg</a>

### 概要(Summary)
レジスタに関する情報を扱うための定義や関数を納めた名前空間
(このクラスは AllStatic ではないが (というか何故か ValueObj になっているが) static な定義しか持たない).


```
    ((cite: hotspot/src/share/vm/opto/optoreg.hpp))
    //------------------------------OptoReg----------------------------------------
    // We eventually need Registers for the Real World.  Registers are essentially
    // non-SSA names.  A Register is represented as a number.  Non-regular values
    // (e.g., Control, Memory, I/O) use the Special register.  The actual machine
    // registers (as described in the ADL file for a machine) start at zero.
    // Stack-slots (spill locations) start at the nest Chunk past the last machine
    // register.
    //
    // Note that stack spill-slots are treated as a very large register set.
    // They have all the correct properties for a Register: not aliased (unique
    // named).  There is some simple mapping from a stack-slot register number
    // to the actual location on the stack; this mapping depends on the calling
    // conventions and is described in the ADL.
    //
    // Note that Name is not enum. C++ standard defines that the range of enum
    // is the range of smallest bit-field that can represent all enumerators
    // declared in the enum. The result of assigning a value to enum is undefined
    // if the value is outside the enumeration's valid range. OptoReg::Name is
    // typedef'ed as int, because it needs to be able to represent spill-slots.
    //
    class OptoReg VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
C2 JIT Compiler 関連の様々な箇所で使用されている (#TODO).

(レジスタ割り当て処理(PhaseChaitin)や低レベル中間語への変換処理(Matcher)など).

### 内部構造(Internal structure)
OptoReg は (実質上) AllStatic でありレジスタ自体を表すわけではない.
実際の 1つ1つのレジスタは OptoReg::Name 型の値として扱う. 
なお, 現状ではこれは単なる int 値 
(上記のコメントによると「enum にしたかったが 
C++ の enum 型は定義時に列挙した項目以外を書き込むと undefined となっており, 
spill-slot を扱うには不便だったから」とのこと)


```
    ((cite: hotspot/src/share/vm/opto/optoreg.hpp))
      typedef int Name;
```




### 詳細(Details)
See: [here](../doxygen/classOptoReg.html) for details

---
## <a name="noediOVbla" id="noediOVbla">OptoRegPair</a>

### 概要(Summary)
その名の通り OptoReg::Name 2つからなるクラス.
VMRegPair と同様, OptoReg 1つが 32bit 長なので, 64bit 長の割り当て領域を表す.

また, 値が32bitである場合は「片方が不正(Badという定数値を入れる)」という値で表現できるのも VMRegPair と同様.


```
    ((cite: hotspot/src/share/vm/opto/optoreg.hpp))
    //---------------------------OptoRegPair-------------------------------------------
    // Pairs of 32-bit registers for the allocator.
    // This is a very similar class to VMRegPair. C2 only interfaces with VMRegPair
    // via the calling convention code which is shared between the compilers.
    // Since C2 uses OptoRegs for register allocation it is more efficient to use
    // VMRegPair internally for nodes that can contain a pair of OptoRegs rather
    // than use VMRegPair and continually be converting back and forth. So normally
    // C2 will take in a VMRegPair from the calling convention code and immediately
    // convert them to an OptoRegPair and stay in the OptoReg world. The only over
    // conversion between OptoRegs and VMRegs is for debug info and oopMaps. This
    // is not a high bandwidth spot and so it is not an issue.
    // Note that onde other consequence of staying in the OptoReg world with OptoRegPairs
    // is that there are "physical" OptoRegs that are not representable in the VMReg
    // world, notably flags. [ But by design there is "space" in the VMReg world
    // for such registers they just may not be concrete ]. So if we were to use VMRegPair
    // then the VMReg world would have to have a representation for these registers
    // so that a OptoReg->VMReg->OptoReg would reproduce ther original OptoReg. As it
    // stands if you convert a flag (condition code) to a VMReg you will get VMRegImpl::Bad
    // and converting that will return OptoReg::Bad losing the identity of the OptoReg.
    
    class OptoRegPair {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 Matcher オブジェクトの _parm_regs フィールド   
  
  (正確には, このフィールドは OptoRegPair の配列を格納するフィールド.
  この中に, その Matcher 内で使用される全ての OptoRegPair オブジェクトが格納されている)

* 各 PhaseRegAlloc オブジェクトの _node_regs フィールド
  
  (正確には, このフィールドは OptoRegPair の配列を格納するフィールド.
  この中に, その PhaseRegAlloc 内で使用される全ての OptoRegPair オブジェクトが格納されている)
  
#### 生成箇所(where its instances are created)
* Matcher::_parm_regs フィールドの配列用のメモリ領域は
  Matcher::match() 内で(のみ)確保されている. 

* PhaseRegAlloc::_node_regs フィールドの配列用のメモリ領域は
  PhaseRegAlloc::alloc_node_regs() 内で(のみ)確保されている. 





### 詳細(Details)
See: [here](../doxygen/classOptoRegPair.html) for details

---
