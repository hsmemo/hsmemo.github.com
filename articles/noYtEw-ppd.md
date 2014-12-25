---
layout: default
title: RegMask クラス 
---
[Top](../index.html)

#### RegMask クラス 



---
## <a name="noLfsS6Jza" id="noLfsS6Jza">RegMask</a>

### 概要(Summary)
C2 JIT Compiler の低レベル中間語の処理 (主に Matcher クラスの処理とレジスタ割り当て処理? #TODO) で使用されるユーティリティ・クラス.

「レジスタの集合」を表すクラス.
より具体的に言うと AD ファイルで定義したレジスタが全て収まる大きさのビットマップを表す
(なおここで言うレジスタとは, VMReg 等と同様, 実際のレジスタだけでなくスタックフレーム上のスロットも含む模様).

メソッドとしては union (OR()メソッド), intersection (AND()メソッド), 等を備える.

なお, レジスタの個数はアーキテクチャ依存なので,
ADLC が定義する RM_SIZE マクロ及び FORALL_BODY マクロを利用して定義されている.

(なお, このクラスはValueObjクラスだが NEW_RESOURCE_ARRAY() で確保されることもある)


```cpp
    ((cite: hotspot/src/share/vm/opto/regmask.hpp))
    //------------------------------RegMask----------------------------------------
    // The ADL file describes how to print the machine-specific registers, as well
    // as any notion of register classes.  We provide a register mask, which is
    // just a collection of Register numbers.
    
    // The ADLC defines 2 macros, RM_SIZE and FORALL_BODY.
    // RM_SIZE is the size of a register mask in words.
    // FORALL_BODY replicates a BODY macro once per word in the register mask.
    // The usage is somewhat clumsy and limited to the regmask.[h,c]pp files.
    // However, it means the ADLC can redefine the unroll macro and all loops
    // over register masks will be unrolled by the correct amount.
    
    class RegMask VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* RegMask クラスの Empty フィールド (static フィールド)
  
* 各 Compile オブジェクトの _FIRST_STACK_mask フィールド
   
* 各 BoxLockNode オブジェクトの _inmask フィールド

* Matcher クラスの mreg2regmask フィールド (static フィールド)
  
  (正確には, このフィールドは RegMask の配列を格納するフィールド.
  配列長は _last_Mach_Reg. なお _last_Mach_Reg は ADLC が定義する定数)

* Matcher クラスの STACK_ONLY_mask フィールド (static フィールド)

* Matcher クラスの c_frame_ptr_mask フィールド (static フィールド)

* 各 Matcher オブジェクトの _return_value_mask フィールド

* 各 Matcher オブジェクトの _calling_convention_mask フィールド
  
  (正確には, このフィールドは RegMask の配列を格納するフィールド)
    
* 各 Matcher オブジェクトの _return_addr_mask フィールド

* 各 Matcher オブジェクトの _return_value_mask フィールド

* 各 Matcher オブジェクトの idealreg2spillmask フィールド
  
  (正確には, このフィールドは RegMask のポインタの配列を格納するフィールド)

* 各 LRG オブジェクトの _mask フィールド
  
* 各 MachProjNode オブジェクトの _rout フィールド

* 各 MachReturnNode オブジェクトの _in_rms フィールド

  変換前の Ideal が ReturnNode, RethrowNode, TailCallNode, TailJumpNode, HaltNode のいずれかである MachReturnNode, 
  もしくは MachSafePointNode の場合に使用される
  (See: Matcher::Fixup_Save_On_Entry(), Matcher::match_sfpt())

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ValueObj クラスなので「生成」というのは少し違和感があるが, 以下の箇所でのみ新しい値を持ったインスタンスが生成されている. 
他の使用箇所はコピーコンストラクタ, あるいは既に生成済みの値へのポインタ).

* (上記のフィールドのうちで 
  Matcher::_calling_convention_mask, Matcher::idealreg2spillmask, MachReturnNode::_in_rms 
  以外のものは全てポインタ型ではなく実体なので,
  格納しているオブジェクトの生成時に一緒に (あるいは static フィールドの場合は初期段階で) 生成される)

* init_input_masks()
  
  (MachReturnNode::_in_rms フィールドの初期化用) (NEW_RESOURCE_ARRAY() で確保)
  
  (正確にはこれはファクトリメソッド. Matcher::Fixup_Save_On_Entry() から(のみ)呼び出されている)

* Matcher::init_first_stack_mask()
  
  (Matcher::idealreg2spillmask フィールドの初期化用)

* ? Node::out_RegMask()
  
  (new されてはいるが, ShouldNotReachHere() でつぶされているので実質生成していない...)

* ? Node::in_RegMask()
  
  (new されてはいるが, ShouldNotReachHere() でつぶされているので実質生成していない...)

* CallNode::match()
  
* DivModINode::match()
  
* DivModLNode::match()

* Block::sched_call()

* Matcher::match( )
  
  (Matcher::_calling_convention_mask の初期化用)

* PhaseChaitin::Split() (局所変数として生成)

* Matcher::match_sfpt()
  
  (MachSafePointNode の MachReturnNode::_in_rms フィールドの初期化用)
  (NEW_RESOURCE_ARRAY() で確保)
  
### 内部構造(Internal structure)

内部には以下の配列を保持している
(型は int 配列だが, 実際にはこのメモリ領域全体を1つのビットマップとして扱う. レジスタ1つに付き 1bit が割り当てられている).


```cpp
    ((cite: hotspot/src/share/vm/opto/regmask.hpp))
        // Array of Register Mask bits.  This array is large enough to cover
        // all the machine registers and all parameters that need to be passed
        // on the stack (stack registers) up to some interesting limit.  Methods
        // that need more parameters will NOT be compiled.  On Intel, the limit
        // is something like 90+ parameters.
        int _A[RM_SIZE];
```





### 詳細(Details)
See: [here](../doxygen/classRegMask.html) for details

---
