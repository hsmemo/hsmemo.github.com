---
layout: default
title: methodDataOopDesc クラス関連のクラス (DataLayout, ProfileData, BitData, CounterData, JumpData, ReceiverTypeData, VirtualCallData, RetData, BranchData, ArrayData, MultiBranchData, ArgInfoData, methodDataOopDesc)
---
[Top](../index.html)

#### methodDataOopDesc クラス関連のクラス (DataLayout, ProfileData, BitData, CounterData, JumpData, ReceiverTypeData, VirtualCallData, RetData, BranchData, ArrayData, MultiBranchData, ArgInfoData, methodDataOopDesc)

これらは, JIT コンパイラのためのクラス.
より具体的に言うと, インタープリタ実行時のプロファイル情報を記録しておくためのクラス (See: [here](no2935fdD.html) for details).

### 概要(Summary)
JIT コンパイラは, インタープリタ実行時のプロファイル情報を用いて最適なコードを生成する.
これらのクラスはその情報を実行時に蓄えるために使用される.

なお, この機能は ProfileInterpreter オプションが指定されている場合にのみ使用される.
また, これらの情報はインタープリタ実行中と (TieredCompilation 時の) first-tier での実行中にだけ収集される.

(<= ところで, 現状でこの情報を活用しているのは C2 JIT Compiler のみ(?) (See: ProfileInterpreter))

なおコメントによると, 
「収集された情報は正確ではない可能性がある 
 (具体的には, カウンターがオーバーフローしたり, マルチスレッドで data race が生じる可能性がある).
 ただし不正確でも速度が落ちる可能性があるだけで correctness には影響しない.
 また, methodDataOop は生成された時間("creation_mileage")を覚えているので,
 これを使ってそのプロファイルデータの信頼性 ("maturity") を推測することはできる」
とのこと.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // The MethodData object collects counts and other profile information
    // during zeroth-tier (interpretive) and first-tier execution.
    // The profile is used later by compilation heuristics.  Some heuristics
    // enable use of aggressive (or "heroic") optimizations.  An aggressive
    // optimization often has a down-side, a corner case that it handles
    // poorly, but which is thought to be rare.  The profile provides
    // evidence of this rarity for a given method or even BCI.  It allows
    // the compiler to back out of the optimization at places where it
    // has historically been a poor choice.  Other heuristics try to use
    // specific information gathered about types observed at a given site.
    //
    // All data in the profile is approximate.  It is expected to be accurate
    // on the whole, but the system expects occasional inaccuraces, due to
    // counter overflow, multiprocessor races during data collection, space
    // limitations, missing MDO blocks, etc.  Bad or missing data will degrade
    // optimization quality but will not affect correctness.  Also, each MDO
    // is marked with its birth-date ("creation_mileage") which can be used
    // to assess the quality ("maturity") of its data.
    //
    // Short (<32-bit) counters are designed to overflow to a known "saturated"
    // state.  Also, certain recorded per-BCI events are given one-bit counters
    // which overflow to a saturated state which applied to all counters at
    // that BCI.  In other words, there is a small lattice which approximates
    // the ideal of an infinite-precision counter for each event at each BCI,
    // and the lattice quickly "bottoms out" in a state where all counters
    // are taken to be indefinitely large.
    //
    // The reader will find many data races in profile gathering code, starting
    // with invocation counter incrementation.  None of these races harm correct
    // execution of the compiled code.
```

現状では, 以下のバイトコードに対してプロファイル情報を取得している.

  * checkcast, instanceof, aastore,
  * invokespecial, invokestatic, invokevirtual, invokeinterface, invokedynamic,
  * goto, goto_w, jsr, jsr_w, ret,
    ifeq, ifne, iflt, ifge, ifgt, ifle, if_icmpeq, if_icmpne, if_icmplt, if_icmpge, if_icmpgt, if_icmple, if_acmpeq, if_acmpne, ifnull, ifnonnull,
    lookupswitch, tableswitch,

プロファイル情報はメソッド毎に集計されており, methodDataOopDesc オブジェクト内に格納されている.
この methodDataOopDesc オブジェクトは methodOopDesc 毎に存在する.
ただし, 実際に必要になるまで生成が遅延されるため methodDataOopDesc オブジェクトが存在しない methodOopDesc もある 
(See: methodOopDesc).

実際のプロファイル情報は DataLayout オブジェクトが格納している.
methodDataOopDesc オブジェクト内では, プロファイル情報は「DataLayout オブジェクトの配列」という形で格納されている.
1つの DataLayout オブジェクトはプロファイル取得対象のバイトコード 1つに対応する.
この配列はバイトコード順と同じになるようソートされている.
このため, この配列中にはそのメソッド内でプロファイル取得対象になっているバイトコードと同数の DataLayout オブジェクトがあり, 
バイトコードと同じ順で並んでいる.

なお, 実際にはバイトコード種別に応じて DataLayout オブジェクトの大きさは異なる.
そのため, 「配列」と言っても各要素の大きさが可変長な配列(heterogeneous array)になっている.
このような DataLayout オブジェクトの操作処理を簡単にするため, ProfileData クラス(とそのサブクラス)が用意されている.
DataLayout オブジェクトにアクセスする際には, バイトコード種別に応じた適切な ProfileData オブジェクトを用いるとよい.

なお, インタープリタ実行中には mdp というポインタが「次に実行されるプロファイル取得対象のバイトコード」に対応する DataLayout を指している
(mdp は具体的に言うと, sparc の場合は ImethodDataPtr レジスタ, x86 の場合はフレーム中の frame::interpreter_frame_mdx_offset の箇所が該当する.
まだ methodDataOopDesc が存在しないメソッドの場合は mdp には NULL がセットされる).

プロファイル取得処理は mdp を用いて以下のように行われる.

  1. メソッドのエントリ部で mdp に値がセットされる. この時点では, そのメソッドの最初の DataLayout へのポインタがセットされる.

     (なお, methodDataOopDesc が対応づけられていないメソッドの場合 mdp は NULL になる.
     ただし, そのメソッドの実行回数が閾値を超えた場合, この時点で methodDataOopDesc が生成されて mdp にセットされる.
     またこの段階では NULL であっても, 内部にループを含むメソッドの場合は実行途中で methodDataOopDesc が生成され mdp にセットされることもある)

  2. プロファイル取得対象のバイトコードが実行されると, mdp が指している DataLayout にプロファイル情報が記録される.
     そして, mdp は次の DataLayout へと進められる.

     (DataLayout はバイトコードと同じ順に並んでいるので, 基本的には1つ先に進めればいい.
     これは現在の DataLayout オブジェクトの大きさ分だけ mdp をインクリメントすることで行える.
     ただし, 分岐命令の場合は分岐が成立した場合は次の DataLayout が複数個先のものになることがあるので,
     その場合には「どれだけ進めればいいか」という情報("displacement" 情報)自体が分岐命令用の DataLayout オブジェクト内に格納されている)

     (なお, プロファイル取得対象以外のバイトコードの場合は, (当然ながら) 実行しても mdp は変化させない)

なお, methodDataOopDec 関係では以下のような用語が使われることがある.

  * "dp" ("data pointer")

    DataLayout オブジェクトの実際のアドレス

  * "di" ("data index")
    
    methodDataOopDesc の DataLayout 配列内における「対象の DataLayout オブジェクトのオフセット(先頭からの距離)」.
    単位は byte.

  * "displacement"

    分岐命令の場合(= 次の DataLayout が複数個先のものになる場合) の「どれだけ進めればいいか」という情報.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // The data entry area is a heterogeneous array of DataLayouts. Each
    // DataLayout in the array corresponds to a specific bytecode in the
    // method.  The entries in the array are sorted by the corresponding
    // bytecode.  Access to the data is via resource-allocated ProfileData,
    // which point to the underlying blocks of DataLayout structures.
    //
    // During interpretation, if profiling in enabled, the interpreter
    // maintains a method data pointer (mdp), which points at the entry
    // in the array corresponding to the current bci.  In the course of
    // intepretation, when a bytecode is encountered that has profile data
    // associated with it, the entry pointed to by mdp is updated, then the
    // mdp is adjusted to point to the next appropriate DataLayout.  If mdp
    // is NULL to begin with, the interpreter assumes that the current method
    // is not (yet) being profiled.
    //
    // In methodDataOop parlance, "dp" is a "data pointer", the actual address
    // of a DataLayout element.  A "di" is a "data index", the offset in bytes
    // from the base of the data entry array.  A "displacement" is the byte offset
    // in certain ProfileData objects that indicate the amount the mdp must be
    // adjusted in the event of a change in control flow.
```



### クラス一覧(class list)

  * [methodDataOopDesc](#noqr69NZ_d)
  * [DataLayout](#noF7I0iO63)
  * [ProfileData](#noHH913DoE)
  * [BitData](#noSGMUgEhr)
  * [CounterData](#noA6bcZgzq)
  * [JumpData](#noLYi85IXk)
  * [ReceiverTypeData](#nomg5co_oZ)
  * [VirtualCallData](#novob2g-jv)
  * [RetData](#noyGzi86R-)
  * [BranchData](#nosIa5VC4y)
  * [ArrayData](#noDzixDxzE)
  * [MultiBranchData](#noQvryC2T-)
  * [ArgInfoData](#noLF10mGj8)


---
## <a name="noqr69NZ_d" id="noqr69NZ_d">methodDataOopDesc</a>

### 概要(Summary)
JIT コンパイラのためのクラス.
より具体的に言うと, インタープリタ実行時のプロファイル情報を記録しておくためのクラス (See: [here](no2935fdD.html) for details).


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    class methodDataOopDesc : public oopDesc {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 methodOopDesc オブジェクトの _method_data フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
methodDataKlass::allocate() というファクトリメソッドが用意されており, その中で生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
methodOopDesc::build_interpreter_method_data()
-> oopFactory::new_methodData()
   -> methodDataKlass::allocate()

MethodHandleCompiler::get_method_oop()
-> oopFactory::new_methodData()
   -> (同上)
```

#### 情報の記録箇所(where information is recorded)
以下の関数内で情報の記録処理が行われている. (See: [here](no2935fdD.html) for details).

  * InterpreterMacroAssembler::profile_taken_branch()
  * InterpreterMacroAssembler::profile_not_taken_branch()
  * InterpreterMacroAssembler::profile_call()
  * InterpreterMacroAssembler::profile_final_call()
  * InterpreterMacroAssembler::profile_virtual_call()
  * InterpreterMacroAssembler::profile_ret()
  * InterpreterMacroAssembler::profile_null_seen()
  * InterpreterMacroAssembler::profile_typecheck()
  * InterpreterMacroAssembler::profile_typecheck_failed()
  * InterpreterMacroAssembler::profile_switch_default()
  * InterpreterMacroAssembler::profile_switch_case()

(Extra Data 及び ArgInfoData については...#TODO)

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* markOop  _mark  (下図中の header)

  oopDesc の共通ヘッダ (mark)

* wideKlassOop _metadata._klass (or narrowOop _metadata._compressed_klass)   (下図中の klass)

  oopDesc の共通ヘッダ (klass)

* methodOop _method (下図中の method)

  この methodDataOopDesc に対応する methodOopDesc オブジェクト.

* int _size (下図中の size of the methodDataOop)

  自分自身(= この methodDataOopDesc オブジェクト) の大きさ


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      // Size of this oop in bytes
      int _size;
```

* ... #TODO

* int _data_size

  自分自身の大きさから, ヘッダ長および Extra Data 分の大きさを除いたサイズ.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      // Size of _data array in bytes.  (Excludes header and extra_data fields.)
      int _data_size;
```

* intptr_t _data (下図中の Data entries...)

  実際に情報を格納する DataLayout オブジェクトが格納されている領域.
  初めの _data_size 分の領域は各バイトコードに対応する DataLayout オブジェクト.
  その後ろは Extra Data.
  Extra Data 部分は, ProfileTraps 用に任意個(?)の BitData が続いた後, 最後に ArgInfoData が入っている.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      // Beginning of the data entries
      intptr_t _data[1];
```


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // A methodDataOop holds information which has been collected about
    // a method.  Its layout looks like this:
    //
    // -----------------------------
    // | header                    |
    // | klass                     |
    // -----------------------------
    // | method                    |
    // | size of the methodDataOop |
    // -----------------------------
    // | Data entries...           |
    // |   (variable size)         |
    // |                           |
    // .                           .
    // .                           .
    // .                           .
    // |                           |
    // -----------------------------
```

### 備考(Notes)
なお, 実際の使用箇所では methodDataOop という別名(もしくはラッパークラス)で使われることが多い (See: methodDataOop).




### 詳細(Details)
See: [here](../doxygen/classmethodDataOopDesc.html) for details

---
## <a name="noF7I0iO63" id="noF7I0iO63">DataLayout</a>

### 概要(Summary)
methodDataOopDesc クラス用の補助クラス.

インタープリタ実行時にプログラム中の各バイトコードのプロファイル情報を記録するためのクラス.
1つの DataLayout オブジェクトがプログラム中の 1つのバイトコードに対応する (See: [here](no2935fdD.html) for details).

なお, DataLayout に格納されるデータはプロファイル対象のバイトコード毎に異なる.
そのため DataLayout オブジェクトの大きさも対応するバイトコード種別に応じて異なる.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // DataLayout
    //
    // Overlay for generic profiling data.
    class DataLayout VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 methodDataOopDesc オブジェクトの _data フィールドに(のみ)格納されている.

(正確には, このフィールドは DataLayout の配列(heterogeneous array)を格納するフィールド.
この中に, そのメソッド内の全てのバイトコード(ただしプロファイル対象になるものに限る)に対応する DataLayout オブジェクトが格納されている)

(なお, _data フィールドは型としては単なる intptr_t[] になっているが,
実際には DataLayout オブジェクトが格納されている)


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      // Beginning of the data entries
      intptr_t _data[1];
```

#### 生成箇所(where its instances are created)
methodDataKlass::allocate() 内で(のみ)生成されている

(より正確に言うと, この関数内で methodDataOopDesc オブジェクトが確保される際に 
methodDataOopDesc::_data フィールド用の領域 (= DataLayout オブジェクト) も一緒に確保されている) (See: [here](no2935fdD.html) for details).

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      // Every data layout begins with a header.  This header
      // contains a tag, which is used to indicate the size/layout
      // of the data, 4 bits of flags, which can be used in any way,
      // 4 bits of trap history (none/one reason/many reasons),
      // and a bci, which is used to tie this piece of data to a
      // specific bci in the bytecodes.
      union {
        intptr_t _bits;
        struct {
          u1 _tag;
          u1 _flags;
          u2 _bci;
        } _struct;
      } _header;
    
      // The data layout has an arbitrary number of cells, each sized
      // to accomodate a pointer or an integer.
      intptr_t _cells[1];
```

それぞれ以下の情報を格納している.

* どのバイトコードに対応するかという情報は _header._struct._tag に格納されている.
  
  (なお, _header._struct._tag には以下の定数値のどれかが格納される)


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      // Tag values
      enum {
        no_tag,
        bit_data_tag,
        counter_data_tag,
        jump_data_tag,
        receiver_type_data_tag,
        virtual_call_data_tag,
        ret_data_tag,
        branch_data_tag,
        multi_branch_data_tag,
        arg_info_data_tag
      };
```

* _cells フィールドがプロファイル情報を格納するフィールド.
  
  (ただしプロファイル情報が _header に収まるようなら使われないこともある)

  _cells フィールドの大きさは対応するバイトコード種別によって変わる
  (宣言では intptr_t[1] となっているがこれは実際の大きさではない).

* _header._struct._flags は u1 の大きさがあるが 4bit ずつに分けて利用されている

  前半 4bit は trap_state を格納している. 
  後半 4bit は任意の用途で使用できる flag フィールドとなっている.

  なお, trap_state は ProfileTraps オプションが指定されている場合にのみ使用される.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      enum {
        // The _struct._flags word is formatted as [trap_state:4 | flags:4].
        // The trap state breaks down further as [recompile:1 | reason:3].
        // This further breakdown is defined in deoptimization.cpp.
        // See Deoptimization::trap_state_reason for an assert that
        // trap_bits is big enough to hold reasons < Reason_RECORDED_LIMIT.
        //
        // The trap_state is collected only if ProfileTraps is true.
        trap_bits = 1+3,  // 3: enough to distinguish [0..Reason_RECORDED_LIMIT].
        trap_shift = BitsPerByte - trap_bits,
        trap_mask = right_n_bits(trap_bits),
        trap_mask_in_place = (trap_mask << trap_shift),
        flag_limit = trap_shift,
        flag_mask = right_n_bits(flag_limit),
        first_flag = 0
      };
```

* ...(#TODO)




### 詳細(Details)
See: [here](../doxygen/classDataLayout.html) for details

---
## <a name="noHH913DoE" id="noHH913DoE">ProfileData</a>

### 概要(Summary)
methodDataOopDesc クラス用のユーティリティ・クラス.

methodDataOopDesc 内の DataLayout オブジェクトにアクセスするための一時オブジェクト(ResourceObjクラス)の基底クラス.

DataLayout オブジェクトは, 対応するバイトコードによって中身や全体の大きさが変わるため, そのままだと扱いにくい.
この DataLayout オブジェクトの操作を簡単に行うために ProfileData クラス (とそのサブクラス) が用意されている.
DataLayout オブジェクトへのアクセス時には (そのバイトコード種別に応じた) ProfileData オブジェクトを介して行う
(See: [here](no2935fdD.html) for details).


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // ProfileData
    //
    // A ProfileData object is created to refer to a section of profiling
    // data in a structured way.
    class ProfileData : public ResourceObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

このクラスのサブクラスは以下のような継承関係を持つ.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // ProfileData class hierarchy
    class ProfileData;
    class   BitData;
    class     CounterData;
    class       ReceiverTypeData;
    class         VirtualCallData;
    class       RetData;
    class   JumpData;
    class     BranchData;
    class   ArrayData;
    class     MultiBranchData;
    class     ArgInfoData;
```

### 備考(Notes)
処理対象の DataLayout オブジェクトの大きさは cell_count() メソッドで取得できる.
このメソッドは ProfileData クラスの各サブクラスによって適切にオーバーライドされている.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      // How many cells are in this?
      virtual int cell_count() {
```




### 詳細(Details)
See: [here](../doxygen/classProfileData.html) for details

---
## <a name="noSGMUgEhr" id="noSGMUgEhr">BitData</a>

### 概要(Summary)
ProfileData クラスの具象サブクラスの1つ (See: [here](no2935fdD.html) for details).
以下の命令に関する DataLayout の操作を担当する.

  * checkcast, instanceof, aastore

なお対応する DataLayout オブジェクトには,
これらのバイトコードの実行時に「値が null になったことがあるかどうか(null_seen)」が記録される.

(なお, このクラスは (上記の用途については) TypeProfileCasts オプションが指定されていない場合にのみ使用される.
 TypeProfileCasts オプションが指定されている場合は, 代わりに ReceiverTypeData が使用される 
 (See: ReceiverTypeData))

(なお, そのほかに Extra Data 用の DataLayout を操作する役割もある(主に trap_state 情報のため). ... #TODO)


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // BitData
    //
    // A BitData holds a flag or two in its header.
    class BitData : public ProfileData {
```

### 内部構造(Internal structure)
このクラスが対応する DataLayout オブジェクトは cell フィールドを持たない.
代わりに情報は flag 部分に格納されている.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      enum {
        // null_seen:
        //  saw a null operand (cast/aastore/instanceof)
        null_seen_flag              = DataLayout::first_flag + 0
      };
```

そのため, この DataLayout オブジェクトには追加で必要なフィールドはない.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      enum { bit_cell_count = 0 };  // no additional data fields needed.
```


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      static int static_cell_count() {
        return bit_cell_count;
      }
    
      virtual int cell_count() {
        return static_cell_count();
      }
```




### 詳細(Details)
See: [here](../doxygen/classBitData.html) for details

---
## <a name="noA6bcZgzq" id="noA6bcZgzq">CounterData</a>

### 概要(Summary)
ProfileData クラスの具象サブクラスの1つ (See: [here](no2935fdD.html) for details).
以下の命令に関する DataLayout の操作を担当する.

  * invokespecial, invokestatic, invokedynamic

なお対応する DataLayout オブジェクトには,
これらのバイトコードが実行された回数(count)が記録される.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // CounterData
    //
    // A CounterData corresponds to a simple counter.
    class CounterData : public BitData {
```

### 内部構造(Internal structure)
このクラスが対応する DataLayout オブジェクトは count というセルを持つ (count_off というオフセットでアクセス可能).
このセルに実行回数が記録される.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      enum {
        count_off,
        counter_cell_count
      };
```


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      static int static_cell_count() {
        return counter_cell_count;
      }
    
      virtual int cell_count() {
        return static_cell_count();
      }
```




### 詳細(Details)
See: [here](../doxygen/classCounterData.html) for details

---
## <a name="noLYi85IXk" id="noLYi85IXk">JumpData</a>

### 概要(Summary)
ProfileData クラスの具象サブクラスの1つ (See: [here](no2935fdD.html) for details).
以下の命令に関する DataLayout の操作を担当する.

  * goto, goto_w, jsr, jsr_w

なお対応する DataLayout オブジェクトには,
これらのバイトコードによってジャンプが実行された回数(taken)が記録される.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // JumpData
    //
    // A JumpData is used to access profiling information for a direct
    // branch.  It is a counter, used for counting the number of branches,
    // plus a data displacement, used for realigning the data pointer to
    // the corresponding target bci.
    class JumpData : public ProfileData {
```

### 内部構造(Internal structure)
このクラスが対応する DataLayout オブジェクトは以下の 2つのセルを持つ.

* taken 
  
  ジャンプの実行回数を記録するためのセル
  (taken_off_set というオフセットでアクセス可能).

* displacement 
  
  ジャンプが実行された場合の次の DataLayout の位置(正確にはその位置と現在地との差分)が格納されている
  (displacement_off_set というオフセットでアクセス可能).


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      enum {
        taken_off_set,
        displacement_off_set,
        jump_cell_count
      };
```


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      static int static_cell_count() {
        return jump_cell_count;
      }
    
      virtual int cell_count() {
        return static_cell_count();
      }
```




### 詳細(Details)
See: [here](../doxygen/classJumpData.html) for details

---
## <a name="nomg5co_oZ" id="nomg5co_oZ">ReceiverTypeData</a>

### 概要(Summary)
ProfileData クラスの具象サブクラスの1つ (See: [here](no2935fdD.html) for details).
以下の命令に関する DataLayout の操作を担当する.

  * checkcast, instanceof, aastore

なお対応する DataLayout オブジェクトには,
これらのバイトコードの実行時に「どの型(クラス)のオブジェクトが何回使用されたか」という情報が (最大で TypeProfileWidth 種類だけ) 記録される.

また以下の情報も記録している.

* これらのバイトコードの実行時に「タイプエラーが発生した回数(count)」
* これらのバイトコードの実行時に「値が null になったことがあるかどうか(null_seen)」

(なお, このクラスは TypeProfileCasts オプションが指定されている場合にのみ使用される.
 TypeProfileCasts オプションが指定されていない場合は, 代わりに BitData が使用される 
 (See: BitData))


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // ReceiverTypeData
    //
    // A ReceiverTypeData is used to access profiling information about a
    // dynamic type check.  It consists of a counter which counts the total times
    // that the check is reached, and a series of (klassOop, count) pairs
    // which are used to store a type profile for the receiver of the check.
    class ReceiverTypeData : public CounterData {
```

### 内部構造(Internal structure)
このクラスが対応する DataLayout オブジェクトは
(スーパークラスである CounterData 用の DataLayout に含まれるセルに加えて)
以下のセルを持つ (それぞれ TypeProfileWidth 個ずつ持つ).

* receiver${N} : (N \in {0, ..., TypeProfileWidth-1})
  
  実際に観測された型を表すセル

* count${N} : (N \in {0, ..., TypeProfileWidth-1})
  
  対応する型が何度観測されたか(回数)を格納するセル

(なお, 現状の実装では情報は早いもの順で TypeProfileWidth 種類だけ格納され, 全部埋まってしまうと新しい型の情報は入らなくなる
 (= 古い情報が purge されることはない). (See: InterpreterMacroAssembler::record_klass_in_profile_helper()))

また, 実行時のタイプチェックが失敗した場合には「タイプエラーが発生した回数」情報がインクリメントされる.
この情報の格納場所についてはスーパークラスである CounterData の count セルを参照.

なお, 値が NULL だった場合には型情報やタイプエラーの発生回数情報は記録されない代わりに (というか記録できないが...) 
「値が null になったことがある(null_seen)」情報が記録される.
この情報の格納場所についてはスーパークラスである BitData 参照.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      enum {
        receiver0_offset = counter_cell_count,
        count0_offset,
        receiver_type_row_cell_count = (count0_offset + 1) - receiver0_offset
      };
```


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      static int static_cell_count() {
        return counter_cell_count + (uint) TypeProfileWidth * receiver_type_row_cell_count;
      }
    
      virtual int cell_count() {
        return static_cell_count();
      }
```




### 詳細(Details)
See: [here](../doxygen/classReceiverTypeData.html) for details

---
## <a name="novob2g-jv" id="novob2g-jv">VirtualCallData</a>

### 概要(Summary)
ProfileData クラスの具象サブクラスの1つ (See: [here](no2935fdD.html) for details).
以下の命令に関する DataLayout の操作を担当する.

  * invokevirtual, invokeinterface

(なおコメントによると, 現状では ReceiverTypeData クラスと全く同じとのこと)


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // VirtualCallData
    //
    // A VirtualCallData is used to access profiling information about a
    // virtual call.  For now, it has nothing more than a ReceiverTypeData.
    class VirtualCallData : public ReceiverTypeData {
```

### 内部構造(Internal structure)
このクラスが対応する DataLayout オブジェクトは, (上記のコメントの通り)
スーパークラスである ReceiverTypeData 用の DataLayout と全く同じセルを持つ.

ただし count セルに格納される情報は少し異なる.
さらに count セルの情報は呼び出すメソッドに final 修飾子が付いているかどうかによって変わる.

* count 

    * final 修飾子が付いている場合 : 

      対応するバイトコードが実行された回数 (= メソッド呼び出しが実行された回数) を格納する
      (See: InterpreterMacroAssembler::profile_final_call())

    * final 修飾子が付いていない場合 : 

      メソッド呼び出し時に receiver が NULL であった回数を格納する
      (See: InterpreterMacroAssembler::profile_virtual_call())

* receiver${N} : (N \in {0, ..., TypeProfileWidth-1})
  
  実際に観測された型を表すセル

* count${N} : (N \in {0, ..., TypeProfileWidth-1})
  
  対応する型が何度観測されたか(回数)を格納するセル


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      static int static_cell_count() {
        // At this point we could add more profile state, e.g., for arguments.
        // But for now it's the same size as the base record type.
        return ReceiverTypeData::static_cell_count();
      }
    
      virtual int cell_count() {
        return static_cell_count();
      }
```

### 備考(Notes)
ReceiverTypeData クラスとは異なり, このクラスは TypeProfileCasts オプションの値に関わらず使用される.




### 詳細(Details)
See: [here](../doxygen/classVirtualCallData.html) for details

---
## <a name="noyGzi86R-" id="noyGzi86R-">RetData</a>

### 概要(Summary)
ProfileData クラスの具象サブクラスの1つ (See: [here](no2935fdD.html) for details).
以下の命令に関する DataLayout の操作を担当する.

  * ret

なお対応する DataLayout オブジェクトには,
ret バイトコードの実行時に「どのリターンアドレスに何回リターンしたか」という情報が (最大で BciProfileWidth 種類だけ) 記録される.
また, 「対応する ret バイトコードが何回実行されたか(回数)」も記録される.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // RetData
    //
    // A RetData is used to access profiling information for a ret bytecode.
    // It is composed of a count of the number of times that the ret has
    // been executed, followed by a series of triples of the form
    // (bci, count, di) which count the number of times that some bci was the
    // target of the ret and cache a corresponding data displacement.
    class RetData : public CounterData {
```

### 内部構造(Internal structure)
このクラスが対応する DataLayout オブジェクトは,
(スーパークラスである CounterData 用の DataLayout に含まれるセルに加えて)
以下のセルを持つ (それぞれ BciProfileWidth 個ずつ持つ).

* bci${N} : (N \in {0, ..., BciProfileWidth-1})
  
  観測されたリターンアドレスを格納するセル
  
* count${N} : (N \in {0, ..., BciProfileWidth-1})
  
  対応するリターンアドレスに何度リターンしたか(回数)を格納するセル

* displacement${N} : (N \in {0, ..., BciProfileWidth-1})
  
  対応するリターンアドレスにリターンした場合の次の DataLayout の位置(正確にはその位置と現在地との差分)を格納するセル

(なお, 現状の実装では情報は早いもの順で BciProfileWidth 種類だけ格納され, 
 全部埋まってしまうと新しいリターンアドレスの情報は入らなくなる
 (= 古い情報が purge されることはない). (See: RetData::fixup_ret()))

また, 対応する ret バイトコードが実行される度に「実行回数」情報がインクリメントされる.
この情報の格納場所についてはスーパークラスである CounterData の count セルを参照.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      enum {
        bci0_offset = counter_cell_count,
        count0_offset,
        displacement0_offset,
        ret_row_cell_count = (displacement0_offset + 1) - bci0_offset
      };
```


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      static int static_cell_count() {
        return counter_cell_count + (uint) BciProfileWidth * ret_row_cell_count;
      }
    
      virtual int cell_count() {
        return static_cell_count();
      }
```




### 詳細(Details)
See: [here](../doxygen/classRetData.html) for details

---
## <a name="nosIa5VC4y" id="nosIa5VC4y">BranchData</a>

### 概要(Summary)
ProfileData クラスの具象サブクラスの1つ (See: [here](no2935fdD.html) for details).
以下の命令に関する DataLayout の操作を担当する.

  * ifeq, ifne, iflt, ifge, ifgt, ifle, if_icmpeq, if_icmpne, if_icmplt, if_icmpge, if_icmpgt, if_icmple, if_acmpeq, if_acmpne, ifnull, ifnonnull

なお対応する DataLayout オブジェクトには,
これらのバイトコードによってジャンプした回数(taken), ジャンプしなかった回数(not_taken)が記録される.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // BranchData
    //
    // A BranchData is used to access profiling data for a two-way branch.
    // It consists of taken and not_taken counts as well as a data displacement
    // for the taken case.
    class BranchData : public JumpData {
```

### 内部構造(Internal structure)
このクラスが対応する DataLayout オブジェクトは
(スーパークラスである JumpData 用の DataLayout に含まれるセルに加えて)
以下のセルを持つ.

* not_taken
  
  ジャンプが不成立だった回数を記録するセル
  (not_taken_off_set というオフセットでアクセス可能).


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      enum {
        not_taken_off_set = jump_cell_count,
        branch_cell_count
      };
```


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      static int static_cell_count() {
        return branch_cell_count;
      }
    
      virtual int cell_count() {
        return static_cell_count();
      }
```




### 詳細(Details)
See: [here](../doxygen/classBranchData.html) for details

---
## <a name="noDzixDxzE" id="noDzixDxzE">ArrayData</a>

### 概要(Summary)
ProfileData クラスのサブクラスの1つ (See: [here](no2935fdD.html) for details).

扱う DataLayout オブジェクトの大きさが可変長である ProfileData クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // ArrayData
    //
    // A ArrayData is a base class for accessing profiling data which does
    // not have a statically known size.  It consists of an array length
    // and an array start.
    class ArrayData : public ProfileData {
```

### 内部構造(Internal structure)
このクラスが対応する DataLayout オブジェクトは以下のセルを持つ.

* array_len
  
  可変長部分の大きさ(セル数) を格納しているセル

(なお, array_start_off_set は可変長部分の開始地点を示す. 
つまり, 先頭に array_len セルがあり, それ以降のセルに実際の情報が格納される.)


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      enum {
        array_len_off_set,
        array_start_off_set
      };
```


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      int array_len() {
        return int_at_unchecked(array_len_off_set);
      }
    
      virtual int cell_count() {
        return array_len() + 1;
      }
```




### 詳細(Details)
See: [here](../doxygen/classArrayData.html) for details

---
## <a name="noQvryC2T-" id="noQvryC2T-">MultiBranchData</a>

### 概要(Summary)
ArrayData クラスの具象サブクラスの1つ (See: [here](no2935fdD.html) for details).
以下の命令に関する DataLayout の操作を担当する.

  * lookupswitch, tableswitch

なお対応する DataLayout オブジェクトには,
これらのバイトコード実行時にどのジャンプ先に何回ジャンプしたか(default_count, relative_count)が記録される.


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    // MultiBranchData
    //
    // A MultiBranchData is used to access profiling information for
    // a multi-way branch (*switch bytecodes).  It consists of a series
    // of (count, displacement) pairs, which count the number of times each
    // case was taken and specify the data displacment for each branch target.
    class MultiBranchData : public ArrayData {
```

### 内部構造(Internal structure)
このクラスが対応する DataLayout オブジェクトは,
(スーパークラスである ArrayData 用の DataLayout に含まれるセルに加えて)
以下のセルを持つ
(relative_count と relative_displacement については, 分岐先の個数に等しい数だけ持つ.
なおメモリ上では, 先に default_count と default_displacement が配置され, 
その後に relative_count と relative_displacement が分岐の個数分だけ並ぶ)

* default_count
  
  デフォルトケースに分岐した回数を格納するセル

* default_displacement
  
  デフォルトケースに分岐した場合の次の DataLayout の位置(正確にはその位置と現在地との差分)を格納するセル
  
* relative_count${N} : (N \in {0, ..., 分岐の個数-1})
  
  対応する分岐先に分岐した回数を格納するセル

* relative_displacement${N} : (N \in {0, ..., 分岐の個数-1})
  
  対応する分岐先に分岐した場合の次の DataLayout の位置(正確にはその位置と現在地との差分)を格納するセル


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      enum {
        default_count_off_set,
        default_disaplacement_off_set,
        case_array_start
      };
      enum {
        relative_count_off_set,
        relative_displacement_off_set,
        per_case_cell_count
      };
```




### 詳細(Details)
See: [here](../doxygen/classMultiBranchData.html) for details

---
## <a name="noLF10mGj8" id="noLF10mGj8">ArgInfoData</a>

### 概要(Summary)
ArrayData クラスの具象サブクラスの1つ (See: [here](no2935fdD.html) for details).

メソッドの引数情報を蓄える DataLayout オブジェクトの操作を担当する.
(現状では Escape Analysis の結果を蓄えるためにのみ使用されている?? #TODO)


```
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
    class ArgInfoData : public ArrayData {
```

### 内部構造(Internal structure)
このクラスが対応する DataLayout オブジェクトは,
ArrayData で規定されているセルを持つ
(= 先頭に array_len セルがあり, それ以降の aray_len 個のセルに実際の引数情報が格納される).




### 詳細(Details)
See: [here](../doxygen/classArgInfoData.html) for details

---
