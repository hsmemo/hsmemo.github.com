---
layout: default
title: NativeInstruction クラス関連のクラス (NativeInstruction, NativeCall, NativeMovConstReg, NativeMovConstRegPatching, NativeMovRegMem, NativeMovRegMemPatching, NativeLoadAddress, NativeJump, NativeGeneralJump, NativePopReg, NativeIllegalInstruction, NativeReturn, NativeReturnX, NativeTstRegMem)
---
[Top](../index.html)

#### NativeInstruction クラス関連のクラス (NativeInstruction, NativeCall, NativeMovConstReg, NativeMovConstRegPatching, NativeMovRegMem, NativeMovRegMemPatching, NativeLoadAddress, NativeJump, NativeGeneralJump, NativePopReg, NativeIllegalInstruction, NativeReturn, NativeReturnX, NativeTstRegMem)

これらは, HotSpot が動的に生成したマシン語を再度書き換えるためのクラス.
命令中のオペランドの値を読み取ったり, オペランドの値を変更する機能を提供する.

### 概要(Summary)
マシン語の再書き換え処理は, 
生成したマシン語語内のアドレスの再配置処理(relocation 処理)で行われる (See: [here](no2935-q0.html) for details).
(他に行われている箇所はある?? #TODO)


なお, これらのクラスが使用されるのは JIT コンパイラが生成したコード
(なぜなら, Interpreter 用のコード中には書き換えなければならないようなアドレスはないため.
 例えば JIT 生成コードでは相対ジャンプがあるが, Interpreter では絶対アドレスでのジャンプしかない).
(<= 本当か?? #TODO)

なお, これらのクラスは以下のような継承関係を持つ.

  * NativeInstruction
      * NativeCall
      * NativeMovConstReg
          * NativeMovConstRegPatching
      * NativeMovRegMem
          * NativeMovRegMemPatching
          * NativeLoadAddress
      * NativeJump
      * NativeGeneralJump
      * NativePopReg
      * NativeIllegalInstruction
      * NativeReturn
      * NativeReturnX
      * NativeTstRegMem
      
(コメントには存在しないクラスの名前も書かれていたりするが...)


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    // We have interfaces for the following instructions:
    // - NativeInstruction
    // - - NativeCall
    // - - NativeMovConstReg
    // - - NativeMovConstRegPatching
    // - - NativeMovRegMem
    // - - NativeMovRegMemPatching
    // - - NativeJump
    // - - NativeIllegalOpCode
    // - - NativeGeneralJump
    // - - NativeReturn
    // - - NativeReturnX (return with argument)
    // - - NativePushConst
    // - - NativeTstRegMem
```



### クラス一覧(class list)

  * [NativeInstruction](#nobaaJj9S_)
  * [NativeCall](#noRcuyuda4)
  * [NativeMovConstReg](#noX3lcfW28)
  * [NativeMovConstRegPatching](#noIpnLLPbe)
  * [NativeMovRegMem](#noIQmfHndh)
  * [NativeMovRegMemPatching](#nonZ5tyRqA)
  * [NativeLoadAddress](#nojU6rWW2Z)
  * [NativeJump](#no2MJfQH9i)
  * [NativeGeneralJump](#noW1wmKmKp)
  * [NativePopReg](#nohwRE9Se1)
  * [NativeIllegalInstruction](#noCUiS_L9l)
  * [NativeReturn](#now1oaSB6A)
  * [NativeReturnX](#no1O3n_53G)
  * [NativeTstRegMem](#no-335Jizv)


---
## <a name="nobaaJj9S_" id="nobaaJj9S_">NativeInstruction</a>

### 概要(Summary)
一度生成したマシン語を修正するためのクラス (の基底クラス).


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    // The base class for different kinds of native instruction abstractions.
    // Provides the primitive operations to manipulate code relative to this.
    
    class NativeInstruction VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
nativeInstruction_at() 関数で, 
ポインタ値(差し先は何らかのマシン語)から NativeInstruction オブジェクトを生成できる
(生成というか, 実際にはキャストしているだけで何もしてないが...).

生成した NativeInstruction オブジェクトからは, 
NativeInstruction::is_*() メソッドでそのマシン語の種別を取得できる.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
      bool is_nop()                        { return ubyte_at(0) == nop_instruction_code; }
      bool is_dtrace_trap();
      inline bool is_call();
      inline bool is_illegal();
      inline bool is_return();
      inline bool is_jump();
      inline bool is_cond_jump();
      inline bool is_safepoint_poll();
      inline bool is_mov_literal64();
```

#### 参考(for your information): nativeInstruction_at()
See: [here](no2935K7n.html) for details
### 内部構造(Internal structure)
内部には 1つのポインタ値だけを持つ

なおポインタ型のフィールドを持つわけではなく, 
ポインタ値を NativeInstruction* にキャストして使用する (See: nativeInstruction_at()).

(つまり, NativeInstruction クラス内ではこのポインタ値に "this" でアクセスできる)

(あるいは, このポインタ値を起点とした相対アドレス位置にアクセスするための addr_at() というメソッドも用意されている)




### 詳細(Details)
See: [here](../doxygen/classNativeInstruction.html) for details

---
## <a name="noRcuyuda4" id="noRcuyuda4">NativeCall</a>

### 概要(Summary)
NativeInstruction クラスのサブクラスの1つ.
このクラスは call 命令(関数呼び出し命令)を処理する.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    // The NativeCall is an abstraction for accessing/manipulating native call imm32/rel32off
    // instructions (used to manipulate inline caches, primitive & dll calls, etc.).
    
    class NativeCall: public NativeInstruction {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
nativeCall_at() 関数で, 
ポインタ値(差し先は call 命令のマシン語)から NativeCall オブジェクトを生成できる
(生成というか, 実際にはキャストしているだけで何もしてないが...).

生成した NativeCall オブジェクトからは, 
NativeCall::destination() メソッドでジャンプ先のアドレスが取得できる.
また, NativeCall::set_destination() でジャンプ先のアドレスを書き換えることもできる.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
      bool is_nop()                        { return ubyte_at(0) == nop_instruction_code; }
      bool is_dtrace_trap();
      inline bool is_call();
      inline bool is_illegal();
      inline bool is_return();
      inline bool is_jump();
      inline bool is_cond_jump();
      inline bool is_safepoint_poll();
      inline bool is_mov_literal64();
```

#### 参考(for your information): nativeCall_at()
See: [here](no2935XFu.html) for details
### 内部構造(Internal structure)
(スーパークラスである NativeInstruction クラスと同じく) 内部には 1つのポインタ値だけを持つ (See: NativeInstruction).




### 詳細(Details)
See: [here](../doxygen/classNativeCall.html) for details

---
## <a name="noX3lcfW28" id="noX3lcfW28">NativeMovConstReg</a>

### 概要(Summary)
NativeInstruction クラスのサブクラスの1つ.
このクラスは "mov reg, imm32" 命令(レジスタに32bit即値をセットする命令)を処理する.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    // An interface for accessing/manipulating native mov reg, imm32 instructions.
    // (used to manipulate inlined 32bit data dll calls, etc.)
    class NativeMovConstReg: public NativeInstruction {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
nativeMovConstReg_at() 関数で, 
ポインタ値(差し先は mov 命令のマシン語)から NativeMovConstReg オブジェクトを生成できる
(生成というか, 実際にはキャストしているだけで何もしてないが...).

生成した NativeMovConstReg オブジェクトからは, 
NativeMovConstReg::data() メソッドで即値部分のデータが取得できる.
また, NativeMovConstReg::set_data() で即値部分を書き換えることもできる.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
      bool is_nop()                        { return ubyte_at(0) == nop_instruction_code; }
      bool is_dtrace_trap();
      inline bool is_call();
      inline bool is_illegal();
      inline bool is_return();
      inline bool is_jump();
      inline bool is_cond_jump();
      inline bool is_safepoint_poll();
      inline bool is_mov_literal64();
```

#### 参考(for your information): nativeMovConstReg_at()
See: [here](no2935kP0.html) for details
### 内部構造(Internal structure)
(スーパークラスである NativeInstruction クラスと同じく) 内部には 1つのポインタ値だけを持つ (See: NativeInstruction).




### 詳細(Details)
See: [here](../doxygen/classNativeMovConstReg.html) for details

---
## <a name="noIpnLLPbe" id="noIpnLLPbe">NativeMovConstRegPatching</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    class NativeMovConstRegPatching: public NativeMovConstReg {
```




### 詳細(Details)
See: [here](../doxygen/classNativeMovConstRegPatching.html) for details

---
## <a name="noIQmfHndh" id="noIQmfHndh">NativeMovRegMem</a>

### 概要(Summary)
NativeInstruction クラスのサブクラスの1つ.
このクラスは, メモリの load/store 命令を処理する.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    // An interface for accessing/manipulating native moves of the form:
    //      mov[b/w/l/q] [reg + offset], reg   (instruction_code_reg2mem)
    //      mov[b/w/l/q] reg, [reg+offset]     (instruction_code_mem2reg
    //      mov[s/z]x[w/b/q] [reg + offset], reg
    //      fld_s  [reg+offset]
    //      fld_d  [reg+offset]
    //      fstp_s [reg + offset]
    //      fstp_d [reg + offset]
    //      mov_literal64  scratch,<pointer> ; mov[b/w/l/q] 0(scratch),reg | mov[b/w/l/q] reg,0(scratch)
    //
    // Warning: These routines must be able to handle any instruction sequences
    // that are generated as a result of the load/store byte,word,long
    // macros.  For example: The load_unsigned_byte instruction generates
    // an xor reg,reg inst prior to generating the movb instruction.  This
    // class must skip the xor instruction.
    
    class NativeMovRegMem: public NativeInstruction {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
nativeMovRegMem_at() 関数で, 
ポインタ値(差し先は load/store 命令のマシン語)から NativeMovRegMem オブジェクトを生成できる
(生成というか, 実際にはキャストしているだけで何もしてないが...).

生成した NativeMovRegMem オブジェクトからは, 
NativeMovRegMem::offset() メソッドでアクセスする際のオフセット値が取得できる.
また, NativeMovRegMem::set_offset() や NativeMovRegMem::add_offset_in_bytes() でオフセット値を変更することもできる.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
      bool is_nop()                        { return ubyte_at(0) == nop_instruction_code; }
      bool is_dtrace_trap();
      inline bool is_call();
      inline bool is_illegal();
      inline bool is_return();
      inline bool is_jump();
      inline bool is_cond_jump();
      inline bool is_safepoint_poll();
      inline bool is_mov_literal64();
```

#### 参考(for your information): nativeMovRegMem_at()
See: [here](no2935WZD.html) for details
### 内部構造(Internal structure)
(スーパークラスである NativeInstruction クラスと同じく) 内部には 1つのポインタ値だけを持つ (See: NativeInstruction).




### 詳細(Details)
See: [here](../doxygen/classNativeMovRegMem.html) for details

---
## <a name="nonZ5tyRqA" id="nonZ5tyRqA">NativeMovRegMemPatching</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    class NativeMovRegMemPatching: public NativeMovRegMem {
```




### 詳細(Details)
See: [here](../doxygen/classNativeMovRegMemPatching.html) for details

---
## <a name="nojU6rWW2Z" id="nojU6rWW2Z">NativeLoadAddress</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    // An interface for accessing/manipulating native leal instruction of form:
    //        leal reg, [reg + offset]
    
    class NativeLoadAddress: public NativeMovRegMem {
```




### 詳細(Details)
See: [here](../doxygen/classNativeLoadAddress.html) for details

---
## <a name="no2MJfQH9i" id="no2MJfQH9i">NativeJump</a>

### 概要(Summary)
NativeInstruction クラスのサブクラスの1つ.
このクラスは, 32bit 相対オフセットのジャンプ命令を処理する.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    // jump rel32off
    
    class NativeJump: public NativeInstruction {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
nativeJump_at() 関数で, 
ポインタ値(差し先はジャンプ命令のマシン語)から NativeJump オブジェクトを生成できる
(生成というか, 実際にはキャストしているだけで何もしてないが...).

生成した NativeJump オブジェクトからは, 
NativeJump::jump_destination() メソッドでジャンプ先のアドレスが取得できる.
また, NativeJump::set_jump_destination() でオフセット値を変更することもできる.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
      bool is_nop()                        { return ubyte_at(0) == nop_instruction_code; }
      bool is_dtrace_trap();
      inline bool is_call();
      inline bool is_illegal();
      inline bool is_return();
      inline bool is_jump();
      inline bool is_cond_jump();
      inline bool is_safepoint_poll();
      inline bool is_mov_literal64();
```

#### 参考(for your information): nativeJump_at()
See: [here](no2935jjJ.html) for details
### 内部構造(Internal structure)
(スーパークラスである NativeInstruction クラスと同じく) 内部には 1つのポインタ値だけを持つ (See: NativeInstruction).





### 詳細(Details)
See: [here](../doxygen/classNativeJump.html) for details

---
## <a name="noW1wmKmKp" id="noW1wmKmKp">NativeGeneralJump</a>

### 概要(Summary)
NativeInstruction クラスのサブクラスの1つ.
このクラスは, ジャンプ命令一般を処理する.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    // Handles all kinds of jump on Intel. Long/far, conditional/unconditional
    class NativeGeneralJump: public NativeInstruction {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
nativeGeneralJump_at() 関数で, 
ポインタ値(差し先はジャンプ命令のマシン語)から NativeGeneralJump オブジェクトを生成できる
(生成というか, 実際にはキャストしているだけで何もしてないが...).

生成した NativeGeneralJump オブジェクトからは, 
NativeGeneralJump::jump_destination() メソッドでジャンプ先のアドレスが取得できる.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
      bool is_nop()                        { return ubyte_at(0) == nop_instruction_code; }
      bool is_dtrace_trap();
      inline bool is_call();
      inline bool is_illegal();
      inline bool is_return();
      inline bool is_jump();
      inline bool is_cond_jump();
      inline bool is_safepoint_poll();
      inline bool is_mov_literal64();
```

#### 参考(for your information): nativeGeneralJump_at()
See: [here](no2935wtP.html) for details
### 内部構造(Internal structure)
(スーパークラスである NativeInstruction クラスと同じく) 内部には 1つのポインタ値だけを持つ (See: NativeInstruction).




### 詳細(Details)
See: [here](../doxygen/classNativeGeneralJump.html) for details

---
## <a name="nohwRE9Se1" id="nohwRE9Se1">NativePopReg</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    class NativePopReg : public NativeInstruction {
```




### 詳細(Details)
See: [here](../doxygen/classNativePopReg.html) for details

---
## <a name="noCUiS_L9l" id="noCUiS_L9l">NativeIllegalInstruction</a>

### 概要(Summary)
NativeInstruction クラスのサブクラスの1つ.
ただし, このクラスは illegal instruction 関係の定数や関数を納めた名前空間といった方が近い
(このクラスは AllStatic ではないが, static なフィールド／メソッドしか持たない).


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    class NativeIllegalInstruction: public NativeInstruction {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
NativeIllegalInstruction::insert() で, 
指定したアドレスに illegal instruction を埋め込むことが出来る.

### 内部構造(Internal structure)
内部には以下の定数定義およびメソッド定義(のみ)を含む.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
      enum Intel_specific_constants {
        instruction_code            = 0x0B0F,    // Real byte order is: 0x0F, 0x0B
        instruction_size            =    2,
        instruction_offset          =    0,
        next_instruction_offset     =    2
      };
```


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
      // Insert illegal opcode as specific address
      static void insert(address code_pos);
```




### 詳細(Details)
See: [here](../doxygen/classNativeIllegalInstruction.html) for details

---
## <a name="now1oaSB6A" id="now1oaSB6A">NativeReturn</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

NativeInstruction クラスのサブクラスの1つ.
ただし, このクラスはリターン命令関係の定数を納めた名前空間といった方が近い
(このクラスは AllStatic ではないが, static な定義しか持たない).


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    // return instruction that does not pop values of the stack
    class NativeReturn: public NativeInstruction {
```

### 使われ方(Usage)
NativeInstruction::is_return() 内で(のみ)使用されている
(が, この関数自体が使われていないような...?? #TODO)

### 内部構造(Internal structure)
内部には以下の定数定義(のみ)を含む.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
      enum Intel_specific_constants {
        instruction_code            = 0xC3,
        instruction_size            =    1,
        instruction_offset          =    0,
        next_instruction_offset     =    1
      };
```




### 詳細(Details)
See: [here](../doxygen/classNativeReturn.html) for details

---
## <a name="no1O3n_53G" id="no1O3n_53G">NativeReturnX</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

NativeInstruction クラスのサブクラスの1つ.
ただし, このクラスはリターン命令関係の定数を納めた名前空間といった方が近い
(このクラスは AllStatic ではないが, static な定義しか持たない).


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    // return instruction that does pop values of the stack
    class NativeReturnX: public NativeInstruction {
```

### 使われ方(Usage)
NativeInstruction::is_return() 内で(のみ)使用されている
(が, この関数自体が使われていないような...?? #TODO)

### 内部構造(Internal structure)
内部には以下の定数定義(のみ)を含む.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
      enum Intel_specific_constants {
        instruction_code            = 0xC2,
        instruction_size            =    2,
        instruction_offset          =    0,
        next_instruction_offset     =    2
      };
```




### 詳細(Details)
See: [here](../doxygen/classNativeReturnX.html) for details

---
## <a name="no-335Jizv" id="no-335Jizv">NativeTstRegMem</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (assert 内でしか使用されない).

NativeInstruction クラスのサブクラスの1つ.
ただし, このクラスは Safepoint Polling 関係の定数を納めた名前空間といった方が近い
(このクラスは AllStatic ではないが, static な定義しか持たない).


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
    // Simple test vs memory
    class NativeTstRegMem: public NativeInstruction {
```

### 使われ方(Usage)
NativeInstruction::is_safepoint_poll() 内で(のみ)使用されている.
そして, この関数は SharedRuntime::get_poll_stub() 内の assert でのみ使用されている.

### 内部構造(Internal structure)
内部には以下の定数定義(のみ)を含む.


```
    ((cite: hotspot/src/cpu/x86/vm/nativeInst_x86.hpp))
      enum Intel_specific_constants {
        instruction_rex_prefix_mask = 0xF0,
        instruction_rex_prefix      = Assembler::REX,
        instruction_code_memXregl   = 0x85,
        modrm_mask                  = 0x38, // select reg from the ModRM byte
        modrm_reg                   = 0x00  // rax
      };
```




### 詳細(Details)
See: [here](../doxygen/classNativeTstRegMem.html) for details

---
