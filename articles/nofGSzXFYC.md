---
layout: default
title: GenerateOopMap クラス関連のクラス (RetTableEntry, RetTable, CellTypeState, BasicBlock, GenerateOopMap, ResolveOopMapConflicts, GeneratePairingInfo, 及びそれらの補助クラス(ComputeCallStack, ComputeEntryStack, RelocCallback))
---
[Top](../index.html)

#### GenerateOopMap クラス関連のクラス (RetTableEntry, RetTable, CellTypeState, BasicBlock, GenerateOopMap, ResolveOopMapConflicts, GeneratePairingInfo, 及びそれらの補助クラス(ComputeCallStack, ComputeEntryStack, RelocCallback))

これらは, GC 用の補助構造(Oop Map) や JIT コンパイラ用の補助情報を計算するためのクラス群.
この計算に必要となるバイトコードの抽象実行機能を備えている (See: [here](no2114GzS.html) for details).


### クラス一覧(class list)

  * [GenerateOopMap](#nouOgbA6id)
  * [RetTable](#nosa_dyDy1)
  * [RetTableEntry](#noSRpPeDfr)
  * [CellTypeState](#noMHv2I4Rz)
  * [BasicBlock](#noZFGgNDzM)
  * [ComputeCallStack](#nolY2FsH21)
  * [ComputeEntryStack](#noe6WEkaeb)
  * [RelocCallback](#notbNuYazp)
  * [ResolveOopMapConflicts](#nos5s-m6w-)
  * [GeneratePairingInfo](#no43LwtiFY)


---
## <a name="nouOgbA6id" id="nouOgbA6id">GenerateOopMap</a>

### 概要(Summary)
Interpreter のスタックフレーム用の OopMap を計算するクラス (の基底クラス) (See: [here](no2114GzS.html) for details).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

(実際のサブクラスとしては OopMapForCacheEntry クラス等を参照)


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
    //
    //  GenerateOopMap
    //
    // Main class used to compute the pointer-maps in a MethodOop
    //
    class GenerateOopMap VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

  * ... (#TODO)

  * int             _bb_count
    
    basic block の個数

  * BitMap          _bb_hdr_bits;
    
    各 basic block の先頭を記録したビットマップ
    (アクセサは, GenerateOopMap::is_bb_header() と GenerateOopMap::set_bbmark_bit())

  * ... (#TODO)

  * _monitor_safe
  
    モニターを balanced な形で使用しているかどうか? 抽象実行中に以下のような状態が検出されると false になる.
  
    * monitorenter し過ぎている (初めに調べた最大回数を超えた)

    * monitorexit し過ぎている (「確保しているモニター数」がマイナスになった)

    * モニターを確保したままリターンしている.

    * モニターを確保したまま, 例外による強制脱出する可能性がある 

      (より正確に言うと, モニターを確保した状態で例外が発生する恐れがあるバイトコードを実行し, 
      かつそれが finally の対象範囲に入っていなければアウト.
      (See: GenerateOopMap::do_exception_edge()))
      
      (補足: JVMS にあるように, synchronized を実装する場合には, 
      (例外で飛び出る場合にもきちんと monitorexit するように)
      finally で synchronized 範囲を括り, 
      finally 内で montiorexit を実行するようにする必要がある)

      (finally の対象範囲であっても finally 内でちゃんと monitorexit してるとは限らないが, 
      その場合は上の「モニターを確保したままリターンしている」ルールの方で引っかかる)

  * ... (#TODO)


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // Main variables
      methodHandle _method;                     // The method we are examine
      RetTable     _rt;                         // Contains the return address mappings
      int          _max_locals;                 // Cached value of no. of locals
      int          _max_stack;                  // Cached value of max. stack depth
      int          _max_monitors;               // Cached value of max. monitor stack depth
      int          _has_exceptions;             // True, if exceptions exist for method
      bool         _got_error;                  // True, if an error occurred during interpretation.
      Handle       _exception;                  // Exception if got_error is true.
      bool         _did_rewriting;              // was bytecodes rewritten
      bool         _did_relocation;             // was relocation neccessary
      bool         _monitor_safe;               // The monitors in this method have been determined
                                                // to be safe.
```


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // Working Cell type state
      int            _state_len;                // Size of states
      CellTypeState *_state;                    // list of states
      char          *_state_vec_buf;            // Buffer used to print a readable version of a state
      int            _stack_top;
      int            _monitor_top;
```


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // Basicblock info
      BasicBlock *    _basic_blocks;             // Array of basicblock info
      int             _gc_points;
      int             _bb_count;
      BitMap          _bb_hdr_bits;
```


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // Create result set
      bool  _report_result;
      bool  _report_result_for_send;            // Unfortunatly, stackmaps for sends are special, so we need some extra
      BytecodeStream *_itr_send;                // variables to handle them properly.
```


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // Initvars
      GrowableArray<intptr_t> * _init_vars;
```


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // Conflicts rewrite logic
      bool      _conflict;                      // True, if a conflict occurred during interpretation
      int       _nof_refval_conflicts;          // No. of conflicts that require rewrites
      int *     _new_var_map;
```


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // List of bci's where a return address is on top of the stack
      GrowableArray<intptr_t> *_ret_adr_tos;
```




### 詳細(Details)
See: [here](../doxygen/classGenerateOopMap.html) for details

---
## <a name="nosa_dyDy1" id="nosa_dyDy1">RetTable</a>

### 概要(Summary)
GenerateOopMap クラス内で使用される補助クラス.

調査対象のコードにおける jsr 命令と ret 命令の対応を記録しておくためのクラス.
(なお, 実際の対応情報は内部に保持する RetTableEntry オブジェクトに記録されている (See: RetTableEntry))
(See: [here](no2114GzS.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
    //  RetTable
    //
    // Contains maping between jsr targets and there return addresses. One-to-many mapping
    //
```


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
    class RetTable VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 GenerateOopMap オブジェクトの _rt フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(GenerateOopMap クラスの _rt フィールドは, ポインタ型ではなく実体なので,
 GenerateOopMap オブジェクトの生成時に一緒に生成される)

(なお, RetTable の中身をセットする処理は, RetTable::compute_ret_table() で(のみ)行われている)

#### 使用箇所(where its instances are used)
GenerateOopMap::ret_jump_targets_do() で使用されている.




### 詳細(Details)
See: [here](../doxygen/classRetTable.html) for details

---
## <a name="noSRpPeDfr" id="noSRpPeDfr">RetTableEntry</a>

### 概要(Summary)
RetTable クラス内で使用される補助クラス.

調査対象のコードにおける jsr 命令と ret 命令の対応を記録しておくためのクラス.
1つの RetTableEntry オブジェクトが 1つの飛び先アドレスに対応する 
(つまり, 複数の jsr 命令の飛び先が同じであれば, それらの情報は 1つの RetTableEntry オブジェクト内に記録される)
(See: [here](no2114GzS.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
    class RetTableEntry : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 RetTable オブジェクトの _first フィールドに(のみ)格納されている.

(正確には, このフィールドは RetTableEntry の線形リストを格納するフィールド.
RetTableEntry オブジェクトは _next フィールドで次の RetTableEntry オブジェクトを指せる構造になっている.
その RetTable オブジェクト内で生成した RetTableEntry オブジェクトは全てこのフィールドの線形リストに格納されている)

#### 生成箇所(where its instances are created)
RetTable::add_jsr() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
以下の箇所で使用されている.

* GenerateOopMap::ret_jump_targets_do()
* 

### 内部構造(Internal structure)
1つの RetTableEntry オブジェクトに対応する jsr 命令は, 
jsrs フィールド内に記録されている.

```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      GrowableArray<intptr_t> * _jsrs;                     // List of return addresses  (bytecode index)
```




### 詳細(Details)
See: [here](../doxygen/classRetTableEntry.html) for details

---
## <a name="noMHv2I4Rz" id="noMHv2I4Rz">CellTypeState</a>

### 概要(Summary)
GenerateOopMap クラス内で使用される補助クラス.

バイトコードの抽象実行を行う際に, 
局所変数, オペランドスタック内のスロット, および確保しているモニター, 
の状態を管理するためのクラス.

それぞれの値がどういう型／どういう値になっているかを管理する.
1つの CellTypeState オブジェクトが 1つの局所変数/1つのスロット/1つのモニター に対応する
(See: [here](no2114GzS.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
    //
    // CellTypeState
    //
    class CellTypeState VALUE_OBJ_CLASS_SPEC {
```

なお, このオブジェクトは BasicBlock 内と GenerateOopMap 内で使用されている.

* BasicBlock オブジェクトが保持する CellTypeState には, その BasicBlock の開始時の状態が記録されている.
* GenerateOopMap オブジェクトが保持する CellTypeState は, 
  バイトコードの抽象実行時に, 現時点でそれぞれの値がどういう値になっているかを管理する.

また, 各 BasicBlock オブジェクト, 及び GenerateOopMap オブジェクトには, 
そのメソッド内で使用される stack/locals/monitors 1つにつき1個の CellTypeState が用意される.
このため全体では,
「(BasicBlock オブジェクトの個数 + 1) * (maxstack + maxlocals + maxmonitors)」個の CellTypeState が用意される.

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 BasicBlock オブジェクトの _state フィールド

  (正確には, このフィールドは CellTypeState の配列を格納するフィールド.
  この中に, その BasicBlock の開始時の状態を表す全ての CellTypeState オブジェクトが格納されている)

* 各 GenerateOopMap オブジェクトの _state フィールド

  (正確には, このフィールドは CellTypeState の配列を格納するフィールド.
  この中に, その GenerateOopMap での抽象実行における状態を表す全ての CellTypeState オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下のファクトリメソッドが用意されており, その中で(のみ)生成されている.
(ValueObj クラスなので「生成」というのは少し違和感があるが)

* CellTypeState::make_any()
* CellTypeState::make_bottom()
* CellTypeState::make_top()
* CellTypeState::make_addr()
* CellTypeState::make_slot_ref()
* CellTypeState::make_line_ref()
* CellTypeState::make_lock_ref()


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // Since some C++ constructors generate poor code for declarations of the
      // form...
      //
      //   CellTypeState vector[length];
      //
      // ...we avoid making a constructor for this class.  CellTypeState values
      // should be constructed using one of the make_* methods:
```

そして, これらのファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

* CellTypeState::make_any()

  以下のグローバル変数の定義 (= static 変数の初期化処理中)

  * CellTypeState::uninit
  * CellTypeState::ref
  * CellTypeState::value
  * CellTypeState::refUninit
  * CellTypeState::addr

* CellTypeState::make_bottom()

  CellTypeState::bottom グローバル変数の定義 (= static 変数の初期化処理中)

* CellTypeState::make_top()

  CellTypeState::top グローバル変数の定義 (= static 変数の初期化処理中)

* CellTypeState::make_addr()

  GenerateOopMap::do_jsr() 内

* CellTypeState::make_slot_ref()

  以下の関数内.

  * CellTypeState::merge() 内
  * ComputeEntryStack::do_object() 内
  * ComputeEntryStack::do_array() 内
  * ComputeEntryStack::compute_for_parameters() 内
  * GenerateOopMap::copy_state() 内
  * GenerateOopMap::initialize_vars() 内
  * GenerateOopMap::do_exception_edge() 内

* CellTypeState::make_line_ref()

  以下の関数内.

  * GenerateOopMap::sigchar_to_effect() 内
  * GenerateOopMap::interp1() 内
  * GenerateOopMap::pp_new_ref() 内
  * GenerateOopMap::do_monitorexit() 内
  * GenerateOopMap::do_ldc() 内
  * GenerateOopMap::do_multianewarray() 内
  * GenerateOopMap::do_method() 内

* CellTypeState::make_lock_ref()

  GenerateOopMap::do_monitorenter() 内
  

```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.cpp))
    CellTypeState CellTypeState::bottom      = CellTypeState::make_bottom();
    CellTypeState CellTypeState::uninit      = CellTypeState::make_any(uninit_value);
    CellTypeState CellTypeState::ref         = CellTypeState::make_any(ref_conflict);
    CellTypeState CellTypeState::value       = CellTypeState::make_any(val_value);
    CellTypeState CellTypeState::refUninit   = CellTypeState::make_any(ref_conflict | uninit_value);
    CellTypeState CellTypeState::top         = CellTypeState::make_top();
    CellTypeState CellTypeState::addr        = CellTypeState::make_any(addr_conflict);
```

### 内部構造(Internal structure)
定義されているフィールドはこれだけ.


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      unsigned int _state;
```

なお, この _state フィールド(32ビット)は, 「上位4ビット(BITS)」と「下位28ビット(INFO)」に分けて使用されている

```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // Masks for separating the BITS and INFO portions of a CellTypeState
      enum { info_mask            = right_n_bits(28),
             bits_mask            = (int)(~info_mask) };
```


上位4ビットの "BITS" には, 対応する値の型を記録している.

* ポインタ型 (ref_bit)
* プリミティブ型 (val_bit)
* returnAddress 型 (addr_bit)

(なお, live_bits_mask はこの3つのどれかが立っていることを調べるためのビットマスク)


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // These constant are used for manipulating the BITS portion of a
      // CellTypeState
      enum { uninit_bit           = (int)(nth_bit(31)),
             ref_bit              = nth_bit(30),
             val_bit              = nth_bit(29),
             addr_bit             = nth_bit(28),
             live_bits_mask       = (int)(bits_mask & ~uninit_bit) };
```

下位 28bit ビットには, さらにそれぞれの型に対する付属情報が付けられる.
これらの情報は, 情報の多さに応じた lattice をなす. lattice の最大限を top, 最小限を bottom と呼んでいる.

* top_info_bit

  top であることを示す.
  (なお, このビットがセットされている場合は残りのビットも全て 1 にすることになっている. そのため info_conflict ビットマスクとほぼ同義.
   (See: CellTypeState::is_valid_state()))

  (Top は全ての付属情報の union のような型. 付属情報が複数ありえて確定しないと言ってもいい.
   抽象実行中に付属情報の unification が失敗したことを意味する.
   (See: CellTypeState::merge()))

* not_bottom_info_bit

  bottom ではないことを示す.

  (bottom は何の付属情報も持たない型. 
  逆に言うと, このビットが 0 であれば何か付属情報を持っている, ということを意味する (See: CellTypeState::is_valid_state()))


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // These constants are used for manipulating the INFO portion of a
      // CellTypeState
      enum { top_info_bit         = nth_bit(27),
             not_bottom_info_bit  = nth_bit(26),
             info_data_mask       = right_n_bits(26),
             info_conflict        = info_mask };
```

付属情報としては, 以下のようなものが使用されている.

* ref_not_lock_bit
  
  monitorenter 命令でロックを取られている値かどうかを示す (ロックが取られていると 0 になる).
  
* ref_slot_bit
  
  reference 型用の付属情報. 
  1 なら "slot" reference, 0 なら "line" reference であることを示す.

* ref_data_mask 部分 (下位 24bit)
  
  reference 型用の付属情報. 以下のどれかが入る.

  * ロックを取られている値の場合: 対応する BCI      (See: CellTypeState::make_lock_ref())
  * "slot" reference の場合: 対応するスロット番号 (See: CellTypeState::make_slot_ref())
  * "line" reference の場合: 対応する BCI       (See: CellTypeState::make_line_ref())


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // Within the INFO data, these values are used to distinguish different
      // kinds of references.
      enum { ref_not_lock_bit     = nth_bit(25),  // 0 if this reference is locked as a monitor
             ref_slot_bit         = nth_bit(24),  // 1 if this reference is a "slot" reference,
                                                  // 0 if it is a "line" reference.
             ref_data_mask        = right_n_bits(24) };
```

### 備考(Notes)
よく使われる CellTypeState 値はグローバル変数として用意されている.

(名前の "CTS" は CellTypeState の略. それ以前の部分がその値の種別を示す("ref" または "r" はポインタ. "val" または "v" ならプリミティブ型))

また, 配列の場合には, 最後が常に CellTypeState::bottom で終わっている.
これは (配列長を伝える方法がないので) 代わりに bottom を終端の目印としているため (GenerateOopMap::ppload() 等を参照).


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.cpp))
    // Commonly used constants
    static CellTypeState epsilonCTS[1] = { CellTypeState::bottom };
    static CellTypeState   refCTS   = CellTypeState::ref;
    static CellTypeState   valCTS   = CellTypeState::value;
    static CellTypeState    vCTS[2] = { CellTypeState::value, CellTypeState::bottom };
    static CellTypeState    rCTS[2] = { CellTypeState::ref,   CellTypeState::bottom };
    static CellTypeState   rrCTS[3] = { CellTypeState::ref,   CellTypeState::ref,   CellTypeState::bottom };
    static CellTypeState   vrCTS[3] = { CellTypeState::value, CellTypeState::ref,   CellTypeState::bottom };
    static CellTypeState   vvCTS[3] = { CellTypeState::value, CellTypeState::value, CellTypeState::bottom };
    static CellTypeState  rvrCTS[4] = { CellTypeState::ref,   CellTypeState::value, CellTypeState::ref,   CellTypeState::bottom };
    static CellTypeState  vvrCTS[4] = { CellTypeState::value, CellTypeState::value, CellTypeState::ref,   CellTypeState::bottom };
    static CellTypeState  vvvCTS[4] = { CellTypeState::value, CellTypeState::value, CellTypeState::value, CellTypeState::bottom };
    static CellTypeState vvvrCTS[5] = { CellTypeState::value, CellTypeState::value, CellTypeState::value, CellTypeState::ref,   CellTypeState::bottom };
    static CellTypeState vvvvCTS[5] = { CellTypeState::value, CellTypeState::value, CellTypeState::value, CellTypeState::value, CellTypeState::bottom };
```




### 詳細(Details)
See: [here](../doxygen/classCellTypeState.html) for details

---
## <a name="noZFGgNDzM" id="noZFGgNDzM">BasicBlock</a>

### 概要(Summary)
GenerateOopMap クラス内で使用される補助クラス.

基本ブロック (Basic Block) を表すためのクラス.
1つの BasicBlock オブジェクトが 1つの basic block に対応する
(See: [here](no2114GzS.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
    //
    // BasicBlockStruct
    //
    class BasicBlock: ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 GenerateOopMap オブジェクトの _basic_blocks フィールドに(のみ)格納されている.

(正確には, このフィールドは BasicBlock の配列を格納するフィールド.
その GenerateOopMap オブジェクト用の BasicBlock オブジェクトは全てこの中に格納されている)

#### 生成箇所(where its instances are created)
メモリ領域は GenerateOopMap::init_basic_blocks() 内で(のみ)確保されている
(そのメモリ領域中に個別の BasicBlock オブジェクトを書き込む作業も GenerateOopMap::init_basic_blocks() 内で(のみ)行われている).

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

  * bool            _changed
    
    作業途中で使用されるフィールド.
    このフィールドが true であれば, (何か変更があったので) もう一度調査する必要があることを示す.

  * int             _bci
    
    その basic block の開始点.

  * int             _end_bci
    
    その basic block の終端.

  * int             _max_locals
    
    局所変数の数. (_state フィールド中の局所変数とオペランドスタックの境界を見つけるための情報)

  * int             _max_stack
    
    オペランドスタックの最大長. (_state フィールド中のオペランドスタックとモニターの境界を見つけるための情報)

  * CellTypeState*  _state
    
    この basic block の先頭地点における「局所変数, オペランドスタック内のスロット, および確保しているモニター」の状態.
    先頭から _max_locals 分が局所変数, 次の _max_stack 分がオペランドスタック内のスロット, 残りが確保しているモニターの状態を表す.

  * int             _stack_top
    
    この basic block の先頭地点における「オペランドスタック内の要素数」.
    なお, その basic block への到達可能性を記録する役割も担っている. 以下のような値を取る.

    * _dead_basic_block

      到達の可能性がないブロック(dead code). これが初期値. (See: GenerateOopMap::init_basic_blocks())

    * _unreached

      到達の可能性があるブロック. まだ一度も処理されていないため「オペランドスタック内の要素数」は不定.  (See: GenerateOopMap::mark_reachable_code())

    * 0 以上の値

      到達の可能性があるブロック. 値は「オペランドスタック内の要素数」を示す.
    
  * int             _monitor_top
    
    この basic block の先頭地点における「確保しているモニターの個数」.
    なお, 値が bad_monitors (-1) の場合は, 抽象実行でモニターの個数が確定しなかった(= モニターの状態がおかしい)ことを意味する.



```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      bool            _changed;                 // Reached a fixpoint or not
```


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      int             _bci;                     // Start of basic block
      int             _end_bci;                 // Bci of last instruction in basicblock
      int             _max_locals;              // Determines split between vars and stack
      int             _max_stack;               // Determines split between stack and monitors
      CellTypeState*  _state;                   // State (vars, stack) at entry.
      int             _stack_top;               // -1 indicates bottom stack value.
      int             _monitor_top;             // -1 indicates bottom monitor stack value.
```


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      enum Constants {
        _dead_basic_block = -2,
        _unreached        = -1                  // Alive but not yet reached by analysis
        // >=0                                  // Alive and has a merged state
      };
```


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
      // _monitor_top is set to this constant to indicate that a monitor matching
      // problem was encountered prior to this point in control flow.
      enum { bad_monitors = -1 };
```




### 詳細(Details)
See: [here](../doxygen/classBasicBlock.html) for details

---
## <a name="nolY2FsH21" id="nolY2FsH21">ComputeCallStack</a>

### 概要(Summary)
GenerateOopMap クラス内で使用される補助クラス.

メソッド呼び出しを抽象実行する際に使用される一時オブジェクト(ResourceObjクラス).
そのメソッドの引数や返値(に対応する値)を抽象オペランドスタックにセットする (See: [here](no2114GzS.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.cpp))
    // ComputeCallStack
    //
    // Specialization of SignatureIterator - compute the effects of a call
    //
    class ComputeCallStack : public SignatureIterator {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
以下の 2つのメソッドを備える.

* ComputeCallStack::compute_for_parameters()
  
  引数で渡された CellTypeState 配列に, 
  指定のメソッドの引数を表す CellTypeState を詰めて返す.
  また, 返値として「そのメソッドの引数のスロット数」を返す.

* ComputeCallStack::compute_for_returntype()
  
  引数で渡された CellTypeState 配列に, 
  指定のメソッドの返値を表す CellTypeState を詰めて返す.
  また, 返値として「そのメソッドの返値のスロット数 (void なら 0, long/double なら 2, それ以外なら 1)」を返す.

#### 使用箇所(where its instances are used)
GenerateOopMap::do_method() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classComputeCallStack.html) for details

---
## <a name="noe6WEkaeb" id="noe6WEkaeb">ComputeEntryStack</a>

### 概要(Summary)
GenerateOopMap クラス内で使用される補助クラス.

抽象実行の開始時に使用される一時オブジェクト(ResourceObjクラス).
そのメソッドの引数を抽象オペランドスタックにセットする (See: [here](no2114GzS.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.cpp))
    //=========================================================================================
    // ComputeEntryStack
    //
    // Specialization of SignatureIterator - in order to set up first stack frame
    //
    class ComputeEntryStack : public SignatureIterator {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
以下の 2つのメソッドを備える.

* ComputeEntryStack::compute_for_parameters()
  
  引数で渡された CellTypeState 配列に, 
  このメソッドの引数を表す CellTypeState を詰めて返す.
  また, 返値として「このメソッドの引数のスロット数」を返す.

* ComputeEntryStack::compute_for_returntype()
  
  こちらは使われていない...

#### 使用箇所(where its instances are used)
GenerateOopMap::methodsig_to_effect() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classComputeEntryStack.html) for details

---
## <a name="notbNuYazp" id="notbNuYazp">RelocCallback</a>

### 概要(Summary)
GenerateOopMap クラス内で使用される補助クラス.

メソッドを書き換えた際に, 
それに合わせて basic block の位置や jsr/ret の対応関係等を修正する処理で使われる一時オブジェクト(StackObjクラス)


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.cpp))
    class RelocCallback : public RelocatorListener {
```

### 使われ方(Usage)
GenerateOopMap::expand_current_instr() 内で(のみ)使用されている.
この関数は, 現在は以下のパスで(のみ)呼び出されている.

```
GenerateOopMap::compute_map()
-> GenerateOopMap::do_interpretation()
   -> GenerateOopMap::rewrite_refval_conflicts()
      -> GenerateOopMap::rewrite_refval_conflict()
         -> GenerateOopMap::rewrite_refval_conflict_inst()
            -> GenerateOopMap::rewrite_load_or_store()
               -> GenerateOopMap::expand_current_instr()
```




### 詳細(Details)
See: [here](../doxygen/classRelocCallback.html) for details

---
## <a name="nos5s-m6w-" id="nos5s-m6w-">ResolveOopMapConflicts</a>

### 概要(Summary)
GenerateOopMap クラスの具象サブクラスの1つ.

Rewriter クラス内で使用される補助クラス. #TODO


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
    //
    // Subclass of the GenerateOopMap Class that just do rewrites of the method, if needed.
    // It does not store any oopmaps.
    //
    class ResolveOopMapConflicts: public GenerateOopMap {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
ResolveOopMapConflicts::do_potential_rewrite() で, メソッドの書き換えを行う.

#### 使用箇所(where its instances are used)
Rewriter::rewrite_jsrs() 内で(のみ)使用されている.
この関数は, 現在は以下のパスで(のみ)呼び出されている.

```
instanceKlass::link_class_impl()
-> instanceKlass::relocate_and_link_methods()
   -> Rewriter::relocate_and_link(instanceKlassHandle this_oop, TRAPS)
      -> Rewriter::relocate_and_link(instanceKlassHandle this_oop, objArrayHandle methods, TRAPS)
         -> Rewriter::rewrite_jsrs()

VM_RedefineClasses::load_new_class_versions()
-> Rewriter::relocate_and_link(instanceKlassHandle this_oop, TRAPS)
   -> (同上)

MethodHandleCompiler::get_method_oop()
-> Rewriter::relocate_and_link(instanceKlassHandle this_oop, objArrayHandle methods, TRAPS)
   -> (同上)
```




### 詳細(Details)
See: [here](../doxygen/classResolveOopMapConflicts.html) for details

---
## <a name="no43LwtiFY" id="no43LwtiFY">GeneratePairingInfo</a>

### 概要(Summary)
GenerateOopMap クラスの具象サブクラスの1つ.

JIT コンパイラ用の補助クラス.
「monitor 命令が balance した使われ方をしているかどうか」を調べるためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/generateOopMap.hpp))
    //
    // Subclass used by the compiler to generate pairing infomation
    //
    class GeneratePairingInfo: public GenerateOopMap {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
monitor 命令のチェックは GeneratePairingInfo::compute_map() で行える.
チェック結果は GeneratePairingInfo::monitor_safe() で取得できる
(See: GenerateOopMap::_monitor_safe).

#### 使用箇所(where its instances are used)
ciMethod::has_balanced_monitors() 内で(のみ)使用されている.

### 内部構造(Internal structure)
処理のほとんどは, スーパークラスの GenerateOopMap::compute_map() が行っている.




### 詳細(Details)
See: [here](../doxygen/classGeneratePairingInfo.html) for details

---
