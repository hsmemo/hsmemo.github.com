---
layout: default
title: JIT Compiler ： 補足 ： C2 JIT コンパイラ用の前処理 ： AD ファイル(ADL ファイル)の内部構造
---
[Up](noVxQtU9lk.html) [Top](../index.html)

#### JIT Compiler ： 補足 ： C2 JIT コンパイラ用の前処理 ： AD ファイル(ADL ファイル)の内部構造

--- 
## 概要(Summary)
AD ファイルの中は, いくつかの意味のまとまり(「ブロック」)に分かれている.
以下にブロックの一覧を示す.

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table6348PaU -->
| block name | description |
|---|---|
| REGISTER DEFINITION BLOCK | レジスタの一覧, およびそれらの使い方(caller/callee save, etc)を規定するブロック |
| DEFINITION BLOCK | AD ファイルの他の箇所で参照する int 定数を定義するブロック |
| SOURCE BLOCK | ADL では記述しにくい内容を C++ で直接的に記述するためのブロック |
| ENCODING BLOCK | INSTRUCTIONS ブロック内で使用する補助情報(エンコーディングのクラス)を定義するブロック |
| FRAME | 対象のアーキテクチャにおけるスタックフレーム構造を指定するブロック |
| ATTRIBUTES | オペランドとインストラクションの属性のデフォルト値を定義する |
| OPERANDS | 低レベル中間語のオペランドノード (= MachOper のサブクラス) を定義するブロック |
| OPERAND CLASSES | 複数のオペランドをまとめて扱うためのグループ (operand class) を定義するブロック |
| PIPELINE | 対象の CPU のパイプライン情報を指定するブロック |
| INSTRUCTIONS | 低レベル中間語の命令ノード (= MachNode のサブクラス) を定義するブロック |
| PEEPHOLE RULES | 覗き穴式最適化用の情報を指定するブロック |
| SMARTSPILL RULES | ?? |
<!-- END RECEIVE ORGTBL table6348PaU -->

<!-- 
#+ORGTBL: SEND table6348PaU orgtbl-to-gfm :no-escape t
| block name                | description                                                                           |
|---------------------------+---------------------------------------------------------------------------------------|
| REGISTER DEFINITION BLOCK | レジスタの一覧, およびそれらの使い方(caller/callee save, etc)を規定するブロック       |
| DEFINITION BLOCK          | AD ファイルの他の箇所で参照する int 定数を定義するブロック                            |
| SOURCE BLOCK              | ADL では記述しにくい内容を C++ で直接的に記述するためのブロック                       |
| ENCODING BLOCK            | INSTRUCTIONS ブロック内で使用する補助情報(エンコーディングのクラス)を定義するブロック |
| FRAME                     | 対象のアーキテクチャにおけるスタックフレーム構造を指定するブロック                    |
| ATTRIBUTES                | オペランドとインストラクションの属性のデフォルト値を定義する                          |
| OPERANDS                  | 低レベル中間語のオペランドノード (= MachOper のサブクラス) を定義するブロック         |
| OPERAND CLASSES           | 複数のオペランドをまとめて扱うためのグループ (operand class) を定義するブロック       |
| PIPELINE                  | 対象の CPU のパイプライン情報を指定するブロック                                       |
| INSTRUCTIONS              | 低レベル中間語の命令ノード (= MachNode のサブクラス) を定義するブロック               |
| PEEPHOLE RULES            | 覗き穴式最適化用の情報を指定するブロック                                              |
| SMARTSPILL RULES          | ??                                                                                    |
-->

### 記法
それぞれのブロックは, 以下のように %{...%} でくくられた形で書かれる.

    register %{
      ...
    %}

## 参考(for your information)
AD ファイルのシンタックスについては hotspot/src/share/vm/adlc/Doc/Syntax.doc に説明がある (ただし, ファイルの生成日が 1997 年なので少し古いかもしれない)


```
    ((cite: hotspot/src/share/vm/adlc/Doc/Syntax.doc))
    Version 0.4 - September 19, 1997
```

```
    ((cite: hotspot/src/share/vm/adlc/Doc/Syntax.doc))
    This document specifies the syntax and associated semantics for the JavaSoft
    HotSpot Architecture Description Language.  This language is used to describe
    the architecture of a processor, and is the input to the ADL Compiler.  The
    ADL Compiler compiles an ADL file into code which is incorporated into the
    Optimizing Just In Time Compiler (OJIT) to generate efficient and correct code
    for the target architecture.  The ADL describes three bassic different types
    of architectural features.  It describes the instruction set (and associated
    operands) of the target architecture.  It describes the register set of the
    target architecture along with relevant information for the register allocator.
    Finally, it describes the architecture's pipeline for scheduling purposes.
    The ADL is used to create an architecture description file for a target
    architecture.  The architecture description file along with some additional
    target specific oracles, written in C++, represent the principal effort in
    porting the OJIT to a new target architecture.
```


## 備考(Notes)
なお, AD ファイル中では C++ と同じ書式でコメントを記入できる (//, /*...*/).


## 詳細
### REGISTER DEFINITION BLOCK
register の一覧とその使い方等を定義 (caller/callee save, etc).
定義した内容は matcher と register allocator から参照される.

    * reg_def
      各レジスタを定義する.

        reg_def ${name} ( ${register save type},  ${C convention save type},
                          ${ideal register type}, ${encoding}, ${vm name} );

      例:
        reg_def R_G0 ( NS,  NS, Op_RegI,  0, G0->as_VMReg());

      * 1つ目と2つ目のパラメータ(... save type)は, caller/callee save を指定
        (1つ目 : Java メソッド呼び出し時の callee save,
         2つ目 : C 関数呼び出し時の callee save)

        指定するパラメータは4種類
        * SOC(Save-On-Call)  : caller save
        * SOE(Save-On-Entry) : callee save
        * NS (No-Save)       : どちらも save しない
        * AS (Always-Save)   : どちらも save する

        (補足: NS という指定は (sparc の G0 のように実際に save/restore が意味がないものだけでなく)
         メソッドのentry/exitで明示的に save/restore しているので C2 compiler が考慮する必要がない, というケースにも使われる模様.
         sparc 版では callee save レジスタが全て NS(No-Save)と指定されていたりするが,
         これはメソッドの entry/exit で明示的に save/restore 命令を呼んでいるためだと思われる. 
         以下の関数を参照.
           * MachPrologNode::emit() @ sparc.ad
           * MachEpilogNode::emit() @ sparc.ad
        )

      * 3つ目のパラメータ(ideal register type)は, 対応する高レベル中間語のレジスタ種別を記述.
        これは save/restore 時の命令に関係する. (Op_RegI なら LoadI/StoreI, Op_RegP なら LoadP/StoreP. そのレジスタが int もポインタも入れられるなら Op_RegI とする)
        (参考: hotspot/src/share/vm/opto/opcodes.hpp)
      * 4つ目のパラメータ(encoding)は, hotspot/src/cpu/sparc/vm/register_sparc.hpp で定義しているレジスタ番号を指定 (このビットパターンが実際の出力に使われる).
      * 5つ目のパラメータ(vm name)は, 対応する VMreg 名??

      sparc 版では 64bit 汎用レジスタを上位/下位に分けて 2つ定義している (VMReg が 32bit 単位での管理であることに対応(?)).  (<= 何故 encoding は 128 なんだろうか?? #TODO)
        reg_def R_G0H( NS,  NS, Op_RegI,128, G0->as_VMReg()->next());
        reg_def R_G0 ( NS,  NS, Op_RegI,  0, G0->as_VMReg());

    * alloc_class
      レジスタを使用する優先度(レジスタ割り当ての優先度)を指定する.

    * reg_class
      MachOper を定義する際に使用するレジスタのクラスを定義するためのもの.
      MachOper は (レジスタオペランドの場合) 使用してよいレジスタの「クラス」を指定できる (例えば, このオペランド種別では R0 は使えない, といった指定ができる).
      (operand block の "constraint" も参照)

### DEFINITION BLOCK	(definitions %{ ... %})
(他の箇所で使用するために)int定数を定義しておける箇所.
(なお, 使える値は [0, 0x7FFFFFFF] の範囲)

    以下のような感じで定義すると,

       int_def  <name>         ( <int_value>, <expression>);

    ad_${architecture}.hpp に以下のようなコードが作られ,

      #define  <name>   (<expression>)
      // value == <int_value>

    ad_${architecture}.cpp の adlc_verification() に以下のような assert が作られる.

      assert( <name> == <int_value>, "Expect (<expression>) to equal <int_value>");

### SOURCE BLOCK	(source_hpp %{ ... %} および source %{ .. %} )
ADL では記述しにくいことを C++ で直接的に記述するためのブロック.
(定数や関数などを定義する. これらは AD ファイルの残りの箇所から利用できる)

    例: sparc.ad では以下のような内容が定義されている (x86_64.ad もほぼ同様).

       * source_hpp %{...%} の記述内容
         source %{...%} 内の関数で外に export したいもののプロトタイプ宣言や, 外で使いたい定数の定義.

       * source %{...%} の記述内容
         * enc_class 内で使う補助関数の定義,
         * instruct (の predicate) 内で使う補助関数の定義,
         * MachNode のメソッド/フィールド定義
           MachCall* の ret_addr_offset()  (各 MachCall が生成する命令列の長さを返す関数. リターンアドレスをどう設定すればいいかを教えるためのもの)
           instruct %{...%} では書きにくいもの(??)のメソッド/フィールド定義 (emit(), size(), format(), etc)
         * 高レベル中間語のメソッド定義
           SafePointNode::needs_polling_address_input(),
           BoxLockNode::emit(), BoxLockNode::size()
         * Matcher の定数/メソッド定義  (参考: hotspot/src/share/vm/opto/matcher.hpp)
           ... (ありすぎるので省略)
         * share 部で型宣言されている関数の定義 (参考: hotspot/src/share/vm/opto/output.cpp)
           size_java_to_interp(),
           reloc_java_to_interp(),
           size_exception_handler(),
           size_deopt_handler(),
           emit_exception_handler(),
           emit_deopt_handler(),

         * ??
           x86_64 では以下の関数が定義されているが, どこにも使用点が見つからない...
           build_address()

### ENCODING BLOCK	( encode %{ ... %} )
MachNode を定義する際に使用するエンコーディングのクラスを定義するためのもの.

    ENCODING BLOCK で定義した内容は, INSTRUCTIONS ブロックで使われる. 
    (MachNode(低レベル中間語)からマシン語への変換規則を定義するのに使われる模様)

    使われ方としては, ENCODING BLOCK で「ビットパターン出力用の関数」を定義しておき,
    INSTRUCTIONS ブロック (MachNode の定義) で使用する関数を指定する, という形になっている. 
    (大抵の CPU の命令ビットパターンはいくつかの種類に分類できるので, こういう風に分けた方が綺麗に記述できるのだと思われる)

    それぞれの関数は以下のようなフォーマットで定義する (詳細は, instruct block の "ins_encode" 参照).

      enc_class 関数名( 引数 ... ) %{
        関数ボディ部
      %}

    定義の例:
      enc_class form3_mem_prefetch_write( memory mem ) %{
        emit_form3_mem_reg(cbuf, this, $primary, -1,
                           $mem$$base, $mem$$disp, $mem$$index, 2/*prefetch function many-writes*/);
      %}

    備考:
    * (関数の引数で渡される) オペランドの値を取得する必要がある場合, 以下のような書式で取得できる.
        $dst$$Register , $mask$$constant, ...

      このようなアクセサは, operand block の "interface" パラメータに応じて自動生成される.
      現在は, 以下の4種類の interface type がサポートされており, 対応する関数が自動生成される.
        * REG_INTER
          MachOper からレジスタ番号を取り出す関数($$Register)が自動生成される. (??)
          (なお, この関数($$Register = reg())はレジスタ割り当てクラスを受け取る. そしてこの関数内でレジスタ割り当てが行われる模様)
        * CONST_INTER
          
        * MEMORY_INTER
          Base Register, Index Register, the Scale Value, the Offset をそれぞれ返す4つの関数が自動生成される.
        * COND_INTER
          各比較条件に対応する 6つの関数が自動生成される (equal, not_equal, less, greater_equal, less_equal, greater).

    * また, if 文や guarantee() なども使用できる模様.
      
      例: 
         enc_class form3_g0_rs2_rd_move( iRegI rs2, iRegI rd ) %{
           // Encode a reg-reg copy.  If it is useless, then empty encoding.
           if( $rs2$$reg != $rd$$reg )
             emit3( cbuf, Assembler::arith_op, $rd$$reg, Assembler::or_op3, 0, 0, $rs2$$reg );
         %}

    * また, 複数の masm 出力文を書くこともできる模様.

      例: 
         enc_class idiv_imm(iRegIsafe src1, immI13 imm, iRegIsafe dst) %{
             MacroAssembler _masm(&cbuf);

             Register Rdividend = reg_to_register_object($src1$$reg);
             int divisor = $imm$$constant;
             Register Rresult = reg_to_register_object($dst$$reg);

             __ sra(Rdividend, 0, Rdividend);
             __ sdivx(Rdividend, divisor, Rresult);
         %}


### FRAME	( frame %{ ... %} )
スタックフレームの構造を定義する (ついでに calling convention も規定している模様. 例えば, このレジスタに引数を入れる/返り値を入れる, といった設定等).

  * stack_direction

      スタックが伸びる向き(上向き/下向き). (なお, ネイティブの場合も Java メソッドの場合も, 方向は同じと想定)


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // What direction does stack grow in (assumed to be same for native & Java)
```

  * inline_cache_reg, interpreter_method_oop_reg

      (この2つは, JIT 生成コードとインタープリタとの間の calling convention 設定用)


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // These two registers define part of the calling convention
      // between compiled code and the interpreter.
```


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      inline_cache_reg(R_G5);                // Inline Cache Register or methodOop for I2C
      interpreter_method_oop_reg(R_G5);      // Method Oop Register when calling interpreter
```

  * cisc_spilling_operand_name

      (現在は x86 専用?)
      演算にも使える memory address オペランド (MachOper クラス) を指定する模様.
      (memory address を使って演算ができれば
      add RT,RA,RB のようなコードを add RT,[SP+offset],RB のようなかたちでも実現できるので,
      spill のオーバーヘッドが小さくなる, ということらしい)


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // Optional: name the operand used by cisc-spilling to access [stack_pointer + offset]
```

  * sync_stack_slots

      monitor オブジェクト1個あたりのスタックスロット数. (See: GraphKit::next_monitor())


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // Number of stack slots consumed by a Monitor enter
```

  * frame_pointer

      スタックポインタとして使用するレジスタ. JIT 生成コード用.


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // Compiled code's Frame Pointer
```

  * interpreter_frame_pointer

      スタックポインタとして使用するレジスタ. インタープリタ用. (frame_pointer と同じでもよい. 同じ場合は省略可能?)


```
    ((cite: hotspot/src/cpu/x86/vm/x86_64.ad))
      // Interpreter stores its frame pointer in a register which is
      // stored to the stack by I2CAdaptors.
      // I2CAdaptors convert from interpreted java to compiled java.
```

  * stack_alignment

      スタックのアラインメント制約(何バイトアラインメントか).


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // Stack alignment requirement
```

  * in_preserve_stack_slots

      各スタックフレームにおいて, 先頭部分(FP側)に確保しなければいけないスタックスロット数.
      x86 ではリターンアドレスと rbp を待避するので 4スロット(64bit時)又は2スロット(32bit時) (※).
      sparc では該当する規定はないため 0 スロット.
      
      (※ 正確には, develop オプションである VerifyStackAtCalls が指定されている場合は, 
          その作業用の 1word が追加されるため, 6スロット(64bit時) 又は 3スロット(32bit時))


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // Number of stack slots between incoming argument block and the start of
      // a new frame.  The PROLOG must add this many slots to the stack.  The
      // EPILOG must remove this many slots.
```

  * varargs_C_out_slots_killed

      (printf のような可変長引数を持つネイティブ関数呼び出し時に使われる模様)
      可変長の場合に, out_preserve_stack_slots に加えてどれだけスタックスロットが潰されるかを指定.

      備考: なおこれらに関連して, SharedRuntime::out_preserve_stack_slots() 関数も定義する必要がある模様.
            この関数は, 各スタックフレームの末尾部分(SP側)に確保しなければいけないスタックスロット数を示す.
            sparc ではレジスタウィンドウの overflow/underflow 時用の領域が必要なため 32スロット(64bit時) または 16スロット(32bit時) (レジスタ16本(iレジスタ及びlレジスタ)の待避用). 
            x86 では該当する規定はないため 0 スロット.


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // Number of outgoing stack slots killed above the out_preserve_stack_slots
      // for calls to C.  Supports the var-args backing area for register parms.
      // ADLC doesn't support parsing expressions, so I folded the math by hand.
```

  * return_addr

      リターンアドレスの配置場所を指定.
      sparc では i7 レジスタ, x86 ではスタックフレーム中の先頭箇所.

      (備考: REG ... と書くとレジスタ, STACK ... と書くとスタック上, という指定になる)


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // The after-PROLOG location of the return address.  Location of
      // return address specifies a type (REG or STACK) and a number
      // representing the register number (i.e. - use a register name) or
      // stack slot.
```

  * calling_convention

      JIT 生成コード用の calling convention を指定.

      現状では, calling convention を表す VMRegPair の計算処理を記述する模様.
      sparc でも x86 でも SharedRuntime::java_calling_convention() の呼び出し処理が指定されている.


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // Body of function which returns an OptoRegs array locating
      // arguments either in registers or in stack slots for calling
      // java
```

  * c_calling_convention

      ネイティブの calling convention を指定
      (JIT 生成したスタブからネイティブメソッドを呼び出したり, JIT 生成コードが HotSpot のランタイムを呼び出す際に使用).

      現状では, calling convention を表す VMRegPair の計算処理を記述する模様.
      sparc でも x86 でも SharedRuntime::c_calling_convention() の呼び出し処理が指定されている.


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // Body of function which returns an OptoRegs array locating
      // arguments either in registers or in stack slots for callin
      // C.
```

  * return_value

      JIT コンパイルされたメソッドの返り値の場所を指定.
      (備考: sparc でも x86 でも c_return_value と同じにしている模様)


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // Location of compiled Java return values.  Same as C
```

  * c_return_value

      ネイティブコード及びインタープリタ実行されたメソッドの返り値の場所を指定.


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
      // Location of native (C/C++) and interpreter return values.  This is specified to
      // be the  same as Java.  In the 32-bit VM, long values are actually returned from
      // native calls in O0:O1 and returned to the interpreter in I0:I1.  The copying
      // to and from the register pairs is done by the appropriate call and epilog
      // opcodes.  This simplifies the register allocator.
```

#### 備考
各プラットフォームにおけるスタックフレームの構造は以下の通り.

* sparc


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
    //----------FRAME--------------------------------------------------------------
    // Definition of frame structure and management information.
    //
    //  S T A C K   L A Y O U T    Allocators stack-slot number
    //                             |   (to get allocators register number
    //  G  Owned by    |        |  v    add VMRegImpl::stack0)
    //  r   CALLER     |        |
    //  o     |        +--------+      pad to even-align allocators stack-slot
    //  w     V        |  pad0  |        numbers; owned by CALLER
    //  t   -----------+--------+----> Matcher::_in_arg_limit, unaligned
    //  h     ^        |   in   |  5
    //        |        |  args  |  4   Holes in incoming args owned by SELF
    //  |     |        |        |  3
    //  |     |        +--------+
    //  V     |        | old out|      Empty on Intel, window on Sparc
    //        |    old |preserve|      Must be even aligned.
    //        |     SP-+--------+----> Matcher::_old_SP, 8 (or 16 in LP64)-byte aligned
    //        |        |   in   |  3   area for Intel ret address
    //     Owned by    |preserve|      Empty on Sparc.
    //       SELF      +--------+
    //        |        |  pad2  |  2   pad to align old SP
    //        |        +--------+  1
    //        |        | locks  |  0
    //        |        +--------+----> VMRegImpl::stack0, 8 (or 16 in LP64)-byte aligned
    //        |        |  pad1  | 11   pad to align new SP
    //        |        +--------+
    //        |        |        | 10
    //        |        | spills |  9   spills
    //        V        |        |  8   (pad0 slot for callee)
    //      -----------+--------+----> Matcher::_out_arg_limit, unaligned
    //        ^        |  out   |  7
    //        |        |  args  |  6   Holes in outgoing args owned by CALLEE
    //     Owned by    +--------+
    //      CALLEE     | new out|  6   Empty on Intel, window on Sparc
    //        |    new |preserve|      Must be even-aligned.
    //        |     SP-+--------+----> Matcher::_new_SP, even aligned
    //        |        |        |
    //
    // Note 1: Only region 8-11 is determined by the allocator.  Region 0-5 is
    //         known from SELF's arguments and the Java calling convention.
    //         Region 6-7 is determined per call site.
    // Note 2: If the calling convention leaves holes in the incoming argument
    //         area, those holes are owned by SELF.  Holes in the outgoing area
    //         are owned by the CALLEE.  Holes should not be nessecary in the
    //         incoming area, as the Java calling convention is completely under
    //         the control of the AD file.  Doubles can be sorted and packed to
    //         avoid holes.  Holes in the outgoing arguments may be nessecary for
    //         varargs C calling conventions.
    // Note 3: Region 0-3 is even aligned, with pad2 as needed.  Region 3-5 is
    //         even aligned with pad0 as needed.
    //         Region 6 is even aligned.  Region 6-7 is NOT even aligned;
    //         region 6-11 is even aligned; it may be padded out more so that
    //         the region from SP to FP meets the minimum stack alignment.
```

* x86_64


```
    ((cite: hotspot/src/cpu/x86/vm/x86_64.ad))
    //----------FRAME--------------------------------------------------------------
    // Definition of frame structure and management information.
    //
    //  S T A C K   L A Y O U T    Allocators stack-slot number
    //                             |   (to get allocators register number
    //  G  Owned by    |        |  v    add OptoReg::stack0())
    //  r   CALLER     |        |
    //  o     |        +--------+      pad to even-align allocators stack-slot
    //  w     V        |  pad0  |        numbers; owned by CALLER
    //  t   -----------+--------+----> Matcher::_in_arg_limit, unaligned
    //  h     ^        |   in   |  5
    //        |        |  args  |  4   Holes in incoming args owned by SELF
    //  |     |        |        |  3
    //  |     |        +--------+
    //  V     |        | old out|      Empty on Intel, window on Sparc
    //        |    old |preserve|      Must be even aligned.
    //        |     SP-+--------+----> Matcher::_old_SP, even aligned
    //        |        |   in   |  3   area for Intel ret address
    //     Owned by    |preserve|      Empty on Sparc.
    //       SELF      +--------+
    //        |        |  pad2  |  2   pad to align old SP
    //        |        +--------+  1
    //        |        | locks  |  0
    //        |        +--------+----> OptoReg::stack0(), even aligned
    //        |        |  pad1  | 11   pad to align new SP
    //        |        +--------+
    //        |        |        | 10
    //        |        | spills |  9   spills
    //        V        |        |  8   (pad0 slot for callee)
    //      -----------+--------+----> Matcher::_out_arg_limit, unaligned
    //        ^        |  out   |  7
    //        |        |  args  |  6   Holes in outgoing args owned by CALLEE
    //     Owned by    +--------+
    //      CALLEE     | new out|  6   Empty on Intel, window on Sparc
    //        |    new |preserve|      Must be even-aligned.
    //        |     SP-+--------+----> Matcher::_new_SP, even aligned
    //        |        |        |
    //
    // Note 1: Only region 8-11 is determined by the allocator.  Region 0-5 is
    //         known from SELF's arguments and the Java calling convention.
    //         Region 6-7 is determined per call site.
    // Note 2: If the calling convention leaves holes in the incoming argument
    //         area, those holes are owned by SELF.  Holes in the outgoing area
    //         are owned by the CALLEE.  Holes should not be nessecary in the
    //         incoming area, as the Java calling convention is completely under
    //         the control of the AD file.  Doubles can be sorted and packed to
    //         avoid holes.  Holes in the outgoing arguments may be nessecary for
    //         varargs C calling conventions.
    // Note 3: Region 0-3 is even aligned, with pad2 as needed.  Region 3-5 is
    //         even aligned with pad0 as needed.
    //         Region 6 is even aligned.  Region 6-7 is NOT even aligned;
    //         region 6-11 is even aligned; it may be padded out more so that
    //         the region from SP to FP meets the minimum stack alignment.
    // Note 4: For I2C adapters, the incoming FP may not meet the minimum stack
    //         alignment.  Region 11, pad1, may be dynamically extended so that
    //         SP meets the minimum alignment.
```


### ATTRIBUTES	( op_attrib ...,  ins_attrib ... )
(注: この部分は %{...%} という形式ではない)

オペランドとインストラクションの属性のデフォルト値を定義する (実行コスト, 命令長, 等など).

### OPERANDS	( operand ${opername}(...) %{ ... %} )
低レベル中間語のオペランドノード (= MachOper のサブクラス) を定義する.

    * predicate, match
      高レベル中間語から MachOper(低レベル中間語) への変換規則を指定. これらの規則を元に matcher が作られる.
      match は, マッチする高レベル中間語のノードパターンを指定する.
      predicate は, さらに補助的な条件を指定する (例: 値がこの範囲の即値, など)
      (#TODO: base match, chain match)
    * constraint
      (レジスタオペランドの場合にのみ指定?) 
      利用してよいレジスタの「クラス」を指定 (これをもとにレジスタ割り当てが行われる)
    * interface
      オペランドの種別を指定.
      (現在は 4種類の interface type をサポート (REG_INTER, CONST_INTER, MEMORY_INTER, COND_INTER). 
      これにより対応するアクセサ関数が自動生成される)
    * format
      デバッグ用の情報. 各 MachOper に対応するアセンブラニーモニック.

### OPERAND CLASSES	( opclass ${opclassname}( ... )  )
複数の Operands をまとめて扱える機能.

(1つの instruction で複数種類のオペランドが利用できる場合に, まとめて定義できるようになるのがうれしい模様)

    指定例: メモリのアドレッシングモード3種をグルーピングした例 (レジスタ, レジスタ+即値, レジスタ+レジスタ)
      opclass memory( indirect, indOffset13, indIndex );

### PIPELINE	( pipeline %{ ... %} )
CPU のパイプライン情報を定義する.

    * attributes
      各種情報の定義
      (命令長が固定長か可変長か, 遅延スロットの有無, バンドル中の最大命令数, 命令長(バイト数), 1回あたりの命令フェッチのバイト数, NOP に相当する命令, など)

    * RESOURCES
      利用可能な演算機の定義.

      例: 
        resources(A0, A1, MS, BR, FA, FM, IDIV, FDIV, IALU = A0 | A1);
        (なお, 最後の  "IALU = A0 | A1"  は,  IALU を A0 または A1 のエイリアスと定義する, という意味になる模様.
        これにより「加算命令は A0 もしくは A1 を使用する」といった設定が簡単に記述可能)

    * PIPELINE DESCRIPTION
      パイプラインステージの定義.
      (順番は特に関係なく, とにかく存在するパイプラインステージを全て列挙すればよい模様)

    * PIPELINE CLASSES
      命令がパイプラインの各ステージで使用するリソースのクラスを定義するためのもの.

      (この PIPELINE CLASSES で「リソース利用のパターン」を定義しておき,
      instruct block (個々の MachNode の定義) の方でその命令がどのパターンに該当するかを指定する, という記述方法を取っている)


### INSTRUCTIONS	( instruct ${instname}(...) %{ ... %} )
低レベル中間語の命令ノード (= MachNode のサブクラス) を定義する.

    * predicate, match
      高レベル中間語から MachNode(低レベル中間語) への変換規則.

    * ins_encode
      MachNode からマシン語への変換規則.  (この内容は ADLC によって XXXXMachNode::emit() に埋め込まれる(？#TODO))

    * opcode
      オペコード (MachNode からマシン語への変換規則)
      (なお, ins_encode 内でオペコードまで指定している場合は必要ない模様)

    * effect
      レジスタの kill/use/def.

    * size
      命令長

    * format
      デバッグ用の情報. 各 MachNode に対応するアセンブラニーモニック.

    * ins_pipe
      対象命令の PIPELINE CLASS. (PIPELINE CLASSES 参照)

    * ins_cost
      対象命令の実行コスト (処理時間).
      省略すると ins_attrib ins_cost で設定したデフォルト値が使われる.
      (<= デフォルト値から変更している命令としては, メモリアクセス, ブランチ(パイプラインストール), 等)

    * expand
      1つの MachNode を複数の MachNode に分割したい場合の変換規則.
      (通常の変換規則では, 複数個の高レベル中間語に対して 1つの低レベル中間語が対応付けられる.
       expand を用いると, 反対に, 1つ以上の高レベル中間語に複数の低レベル中間語を対応付けることが可能)


* 指定例1:


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
    // Addition Instructions
    // Register Addition
    instruct addI_reg_reg(iRegI dst, iRegI src1, iRegI src2) %{
      match(Set dst (AddI src1 src2));
    
      size(4);
      format %{ "ADD    $src1,$src2,$dst" %}
      ins_encode %{
        __ add($src1$$Register, $src2$$Register, $dst$$Register);
      %}
      ins_pipe(ialu_reg_reg);
    %}
```

* 指定例2: expand の使用例


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
    // Integer DIV with 10
    instruct divI_10( iRegI dst, iRegIsafe src, immI10 div ) %{
      match(Set dst (DivI src div));
      ins_cost((6+6)*DEFAULT_COST);
      expand %{
        iRegIsafe tmp1;               // Killed temps;
        iRegIsafe tmp2;               // Killed temps;
        iRegI tmp3;                   // Killed temps;
        iRegI tmp4;                   // Killed temps;
        loadConI_x66666667( tmp1 );   // SET  0x66666667 -> tmp1
        mul_hi( tmp2, src, tmp1 );    // MUL  hibits(src * tmp1) -> tmp2
        sra_31( tmp3, src );          // SRA  src,31 -> tmp3
        sra_reg_2( tmp4, tmp2 );      // SRA  tmp2,2 -> tmp4
        subI_reg_reg( dst,tmp4,tmp3); // SUB  tmp4 - tmp3 -> dst
      %}
    %}
```

* 指定例3: 分岐(nullチェック)のマッチング   (CmpP と If の組合せパターンでマッチ)


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
    instruct branchCon_regP(cmpOp_reg cmp, iRegP op1, immP0 null, label labl) %{
      match(If cmp (CmpP op1 null));
      predicate(can_branch_register(_kids[0]->_leaf, _kids[1]->_leaf));
      effect(USE labl);
    
      size(8);
      ins_cost(BRANCH_COST);
      format %{ "BR$cmp   $op1,$labl" %}
      ins_encode( enc_bpr( labl, cmp, op1 ) );
      ins_pc_relative(1);
      ins_pipe(br_reg);
    %}
```

* 指定例4: 右シフト&L2I処理の最適化   (シフト幅が 32~64 bit の算術右シフトの後で L2I, というパターンを最適化)
  
  (シフト幅が32~64の場合, L2I の際には元々の64bit目で符号拡張することになるので, 単なる64bit算術右シフトに最適化できる)


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
    // Register Shift Right Immediate
    instruct shrI_reg_imm5(iRegI dst, iRegI src1, immU5 src2) %{
      match(Set dst (URShiftI src1 src2));
    
      size(4);
      format %{ "SRL    $src1,$src2,$dst" %}
      opcode(Assembler::srl_op3, Assembler::arith_op);
      ins_encode( form3_rs1_imm5_rd( src1, src2, dst ) );
      ins_pipe(ialu_reg_imm);
    %}
```


### PEEPHOLE RULES	( peephole %{ ... %} )
覗き穴式最適化の定義.

(備考: sparc 用のファイルでは何も定義されていない)

### SMARTSPILL RULES
?? (レジスタのspillコードに関する最適化を定義する模様)

(備考: sparc, x86 ともに, 現状では何も定義されていない)

(備考: sparc の方は, 命令セットが RISC 系なので特に定義することはないとのこと)


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
    // SPARC will probably not have any of these rules due to RISC instruction set.
```







