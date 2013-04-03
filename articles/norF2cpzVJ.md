---
layout: default
title: AbstractAssembler クラス関連のクラス (Label, RegisterOrConstant, AbstractAssembler, AbstractAssembler::InstructionMark)
---
[Top](../index.html)

#### AbstractAssembler クラス関連のクラス (Label, RegisterOrConstant, AbstractAssembler, AbstractAssembler::InstructionMark)

これらは, 動的コード生成を補佐するクラス.
より具体的に言うと, Assembler クラス (実行時にマシン語を出力するクラス) 関連のプラットフォーム非依存な部分を定義するクラス (See: [here](no7882z5r.html) for details).


```
    ((cite: hotspot/src/share/vm/asm/assembler.hpp))
    // This file contains platform-independent assembler declarations.
```



### クラス一覧(class list)

  * [AbstractAssembler](#nolVmAaxrq)
  * [Label](#noPmOghzr0)
  * [RegisterOrConstant](#noXpo3kLpi)
  * [AbstractAssembler::InstructionMark](#nofVdSsDBH)


---
## <a name="nolVmAaxrq" id="nolVmAaxrq">AbstractAssembler</a>

### 概要(Summary)
Assembler クラス (実行時にマシン語を出力するクラス) の基底クラス.

なお, 具体的な Assembler クラス (AbstractAssemblerのサブクラス) は cpu/ 下で定義されている
(See: hotspot/src/cpu/${cpu}/vm/assembler_${cpu}.cpp).


```
    ((cite: hotspot/src/share/vm/asm/assembler.hpp))
    // The Abstract Assembler: Pure assembler doing NO optimizations on the
    // instruction level; i.e., what you write is what you get.
    // The Assembler is generating code into a CodeBuffer.
    class AbstractAssembler : public ResourceObj  {
```

### 備考(Notes)
毎回 CodeBuffer のアクセサ経由で出力していると遅いため, 
出力先のバッファのアドレスを覚えておいてそのポインタ経由で出力している.
そして, 最後に set_code_end() で CodeBuffer の管理情報を修正している模様.


```
    ((cite: hotspot/src/share/vm/asm/assembler.cpp))
    // The AbstractAssembler is generating code into a CodeBuffer. To make code generation faster,
    // the assembler keeps a copy of the code buffers boundaries & modifies them when
    // emitting bytes rather than using the code buffers accessor functions all the time.
    // The code buffer is updated via set_code_end(...) after emitting a whole instruction.
```



### 詳細(Details)
See: [here](../doxygen/classAbstractAssembler.html) for details

---
## <a name="noPmOghzr0" id="noPmOghzr0">Label</a>

### 概要(Summary)
Assembler クラス用の補助クラス.

Assembler クラスが生成する branch 命令の飛び先を管理するクラス.


```
    ((cite: hotspot/src/share/vm/asm/assembler.hpp))
    /**
     * Labels represent destinations for control transfer instructions.  Such
     * instructions can accept a Label as their target argument.  A Label is
     * bound to the current location in the code stream by calling the
     * MacroAssembler's 'bind' method, which in turn calls the Label's 'bind'
     * method.  A Label may be referenced by an instruction before it's bound
     * (i.e., 'forward referenced').  'bind' stores the current code offset
     * in the Label object.
     *
     * If an instruction references a bound Label, the offset field(s) within
     * the instruction are immediately filled in based on the Label's code
     * offset.  If an instruction references an unbound label, that
     * instruction is put on a list of instructions that must be patched
     * (i.e., 'resolved') when the Label is bound.
     *
     * 'bind' will call the platform-specific 'patch_instruction' method to
     * fill in the offset field(s) for each unresolved instruction (if there
     * are any).  'patch_instruction' lives in one of the
     * cpu/<arch>/vm/assembler_<arch>* files.
     *
     * Instead of using a linked list of unresolved instructions, a Label has
     * an array of unresolved instruction code offsets.  _patch_index
     * contains the total number of forward references.  If the Label's array
     * overflows (i.e., _patch_index grows larger than the array size), a
     * GrowableArray is allocated to hold the remaining offsets.  (The cache
     * size is 4 for now, which handles over 99.5% of the cases)
     *
     * Labels may only be used within a single CodeSection.  If you need
     * to create references between code sections, use explicit relocations.
     */
    class Label VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
bind() で飛び先が束縛される.
(Label を使用するブランチ命令より後ろで bind してもきちんと設定してくれる)


```
    ((cite: hotspot/src/cpu/sparc/vm/assembler_sparc.cpp))
    void MacroAssembler::cond_inc(Assembler::Condition cond, address counter_ptr,
                                  Register Rtmp1, Register Rtmp2 /*, Register Rtmp3, Register Rtmp4 */) {
      Condition negated_cond = negate_condition(cond);
      Label L;
      brx(negated_cond, false, Assembler::pt, L);
      delayed()->nop();
      inc_counter(counter_ptr, Rtmp1, Rtmp2);
      bind(L);
    }
```




### 詳細(Details)
See: [here](../doxygen/classLabel.html) for details

---
## <a name="noXpo3kLpi" id="noXpo3kLpi">RegisterOrConstant</a>

### 概要(Summary)
Assembler クラス用の補助クラス.

即値とレジスタのどちらでも取れるようなコードを表現するためのクラス.


```
    ((cite: hotspot/src/share/vm/asm/assembler.hpp))
    // A union type for code which has to assemble both constant and
    // non-constant operands, when the distinction cannot be made
    // statically.
    class RegisterOrConstant VALUE_OBJ_CLASS_SPEC {
```



### 詳細(Details)
See: [here](../doxygen/classRegisterOrConstant.html) for details

---
## <a name="nofVdSsDBH" id="nofVdSsDBH">AbstractAssembler::InstructionMark</a>

### 概要(Summary)
コード生成作業中に使用される一時オブジェクト(StackObjクラス).
マシン語命令の境界を表す.

なお, x86 用のファイルでしか使われていないので
(より正確には hotspot/src/cpu/x86/vm/assembler_x86.cpp 内でしか使われていないので)
可変長命令でなければ必要ない模様 (? #TODO).


```
    ((cite: hotspot/src/share/vm/asm/assembler.hpp))
      // Instruction boundaries (required when emitting relocatable values).
      class InstructionMark: public StackObj {
```

### 使われ方(Usage)
命令を吐き出す前にインスタンスを生成して, そこからが命令の始まりだと示している.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
    void Assembler::adcl(Address dst, int32_t imm32) {
      InstructionMark im(this);
      prefix(dst);
      emit_arith_operand(0x81, rdx, dst, imm32);
    }
```

### 内部構造(Internal structure)
コンストラクタ内で AbstractAssembler::set_inst_mark() を呼び出し, 
デストラクタ内で AbstractAssembler::clear_inst_mark() を呼び出している.


```
    ((cite: hotspot/src/share/vm/asm/assembler.hpp))
        InstructionMark(AbstractAssembler* assm) : _assm(assm) {
          assert(assm->inst_mark() == NULL, "overlapping instructions");
          _assm->set_inst_mark();
        }
        ~InstructionMark() {
          _assm->clear_inst_mark();
        }
```

なお AbstractAssembler::set_inst_mark() と AbstractAssembler::clear_inst_mark() の中身は,
それぞれ CodeSection::set_mark() と CodeSection::clear_mark() を呼んでいるだけ.


```
    ((cite: hotspot/src/share/vm/asm/assembler.inline.hpp))
    inline void AbstractAssembler::set_inst_mark() {
      code_section()->set_mark();
    }
    
    
    inline void AbstractAssembler::clear_inst_mark() {
      code_section()->clear_mark();
    }
```

そして CodeSection::set_mark() と CodeSection::clear_mark() では
_mark というフィールドの値を更新しているだけ.


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.hpp))
      void    set_mark()                { _mark = _end; }
      void    clear_mark()              { _mark = NULL; }
```




### 詳細(Details)
See: [here](../doxygen/classAbstractAssembler_1_1InstructionMark.html) for details

---
