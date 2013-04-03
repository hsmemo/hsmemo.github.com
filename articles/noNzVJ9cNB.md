---
layout: default
title: SharedRuntime クラス用の補助クラス (SimpleRuntimeFrame, RegisterSaver)
---
[Top](../index.html)

#### SharedRuntime クラス用の補助クラス (SimpleRuntimeFrame, RegisterSaver)

これらのクラスは, SharedRuntime クラス用の補助クラス (See: SharedRuntime).


### クラス一覧(class list)

  * [SimpleRuntimeFrame](#no1HvLgpip)
  * [RegisterSaver](#noFd3C5C_U)


---
## <a name="no1HvLgpip" id="no1HvLgpip">SimpleRuntimeFrame</a>

### 概要(Summary)
x86 64bit 版にのみ存在するクラス (このクラスは x86 32bit では定義されていない).

スタックフレーム中のレイアウトに関する定数値を納めた名前空間
(このクラスは AllStatic ではないが, static な定義しか持たない).


```
    ((cite: hotspot/src/cpu/x86/vm/sharedRuntime_x86_64.cpp))
    class SimpleRuntimeFrame {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* SharedRuntime::generate_uncommon_trap_blob() (x86 64bit の場合)
* OptoRuntime::generate_exception_blob() (x86 64bit の場合)

### 内部構造(Internal structure)
内部には以下の定数定義(のみ)を含む.


```
    ((cite: hotspot/src/cpu/x86/vm/sharedRuntime_x86_64.cpp))
      // Most of the runtime stubs have this simple frame layout.
      // This class exists to make the layout shared in one place.
      // Offsets are for compiler stack slots, which are jints.
      enum layout {
        // The frame sender code expects that rbp will be in the "natural" place and
        // will override any oopMap setting for it. We must therefore force the layout
        // so that it agrees with the frame sender code.
        rbp_off = frame::arg_reg_save_area_bytes/BytesPerInt,
        rbp_off2,
        return_off, return_off2,
        framesize
      };
```




### 詳細(Details)
See: [here](../doxygen/classSimpleRuntimeFrame.html) for details

---
## <a name="noFd3C5C_U" id="noFd3C5C_U">RegisterSaver</a>

### 概要(Summary)
SharedRuntime クラス内で使用される補助クラス.

callee saved register の退避復帰用のコードを生成するためのクラス
(より正確には, そのための機能を納めた名前空間. 
 このクラスは AllStatic ではないが, static なフィールド／メソッドしか持たない).

(なお, 32bit か 64bit かによってクラス定義が別になっている.)


```
    ((cite: hotspot/src/cpu/x86/vm/sharedRuntime_x86_32.cpp))
    class RegisterSaver {
```


```
    ((cite: hotspot/src/cpu/x86/vm/sharedRuntime_x86_64.cpp))
    class RegisterSaver {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* SharedRuntime::generate_deopt_blob() (x86 32/64bit の場合)
* SharedRuntime::generate_handler_blob() (x86 32/64bit の場合)
* SharedRuntime::generate_resolve_blob() (x86 32/64bit の場合)

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている.

(save_live_registers() はレジスタの待避コードの生成, 
 restore_live_registers() はレジスタの復帰コードの生成, 
 restore_result_registers() は返値の復帰コードの生成(これは deopt 処理時用)を行う.
 その他のメソッドは待避したレジスタにアクセスする際に使われる.)

* 32bit の場合:

```
    ((cite: hotspot/src/cpu/x86/vm/sharedRuntime_x86_32.cpp))
      static OopMap* save_live_registers(MacroAssembler* masm, int additional_frame_words,
                                         int* total_frame_words, bool verify_fpu = true);
      static void restore_live_registers(MacroAssembler* masm);
    
      static int rax_offset() { return rax_off; }
      static int rbx_offset() { return rbx_off; }
    
      // Offsets into the register save area
      // Used by deoptimization when it is managing result register
      // values on its own
    
      static int raxOffset(void) { return rax_off; }
      static int rdxOffset(void) { return rdx_off; }
      static int rbxOffset(void) { return rbx_off; }
      static int xmm0Offset(void) { return xmm0_off; }
      // This really returns a slot in the fp save area, which one is not important
      static int fpResultOffset(void) { return st0_off; }
    
      // During deoptimization only the result register need to be restored
      // all the other values have already been extracted.
    
      static void restore_result_registers(MacroAssembler* masm);
```

* 64bit の場合:

```
    ((cite: hotspot/src/cpu/x86/vm/sharedRuntime_x86_64.cpp))
      static OopMap* save_live_registers(MacroAssembler* masm, int additional_frame_words, int* total_frame_words);
      static void restore_live_registers(MacroAssembler* masm);
    
      // Offsets into the register save area
      // Used by deoptimization when it is managing result register
      // values on its own
    
      static int rax_offset_in_bytes(void)    { return BytesPerInt * rax_off; }
      static int rdx_offset_in_bytes(void)    { return BytesPerInt * rdx_off; }
      static int rbx_offset_in_bytes(void)    { return BytesPerInt * rbx_off; }
      static int xmm0_offset_in_bytes(void)   { return BytesPerInt * xmm0_off; }
      static int return_offset_in_bytes(void) { return BytesPerInt * return_off; }
    
      // During deoptimization only the result registers need to be restored,
      // all the other values have already been extracted.
      static void restore_result_registers(MacroAssembler* masm);
```


### 備考(Notes)
このクラスによる退避復帰が行われるのは (deopt 処理等の) ごく一部のケース.
通常の退避復帰処理は Assembler::push(), Assembler::pop() 等で行われる.




### 詳細(Details)
See: [here](../doxygen/classRegisterSaver.html) for details

---
