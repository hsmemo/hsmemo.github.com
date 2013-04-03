---
layout: default
title: VMReg クラス関連のクラス (VMRegImpl, VMRegPair)
---
[Top](../index.html)

#### VMReg クラス関連のクラス (VMRegImpl, VMRegPair)

これらは, データの格納場所となりうる場所(レジスタ or スタック)を統一的に扱うためのユーティリティ・クラス.


### クラス一覧(class list)

  * [VMRegImpl (VMReg)](#no2ItTe1lO)
  * [VMRegPair](#noI8rpuzrh)


---
## <a name="no2ItTe1lO" id="no2ItTe1lO">VMRegImpl (VMReg)</a>

### 概要(Summary)
データの格納場所となりうる場所(レジスタ or スタック)を統一的に扱うためのクラス.

それぞれの格納場所に対して一意な番号(整数値)を割り当て, 格納場所<=>整数の変換を行う.

(なお, "Reg" という名前だが, レジスタだけでなくスタック上のスロットも含めて統一的に扱っている)


```
    ((cite: hotspot/src/share/vm/code/vmreg.hpp))
    class VMRegImpl {
```

なお, VMRegImpl は 32bit の領域を表す. このため, 64bit のレジスタやスタックスロットは 2個の VMReg で表現される.

### 内部構造(Internal structure)
実際の値の対応付けは, 各 cpu/ ディレクトリ下で定義されている.
VMRegImpl::set_regName() が対応付けを行う関数であり, 対応付けの結果は regName[] という配列に格納されている.

sparc 版でも x86 版でも以下のような番号付けになっている模様.

  * 最初は汎用レジスタ. `[0, ..., ConcreteRegisterImpl::max_gpr-1]`
  * 汎用レジスタが終わったら, その後ろに浮動小数レジスタ. `[ConcreteRegisterImpl::max_gpr, ..., ConcreteRegisterImpl::max_fpr-1]`
  * SIMD レジスタがある場合はこの後ろに付ける?(x86ではそうっぽい). `[ConcreteRegisterImpl::max_fpr, ..., ConcreteRegisterImpl::max_xmm]`
  * (ここまでの総数が `VMRegImpl::register_count (= ConcreteRegisterImpl::number_of_registers)`)
  * 全レジスタが終わった次の数が `VMRegImpl::stack0`.
  * `VMRegImpl::stack0` より上の数字は, スタック上のスロット("offsets from the current stack pointer")を示す

#### 参考(for your information): VMRegImpl::set_regName() (sparc の場合)
See: [here](no4230JTy.html) for details
#### 参考(for your information): VMRegImpl::set_regName() (x86 の場合)
See: [here](no42307cB.html) for details
#### 参考(for your information): VMRegImpl::stack0
stack0 の値は, 以下のように 全レジスタ数+1 と設定されている。


```
    ((cite: hotspot/src/share/vm/code/vmreg.cpp))
    // First VMReg value that could refer to a stack slot
    VMReg VMRegImpl::stack0 = (VMReg)(intptr_t)((ConcreteRegisterImpl::number_of_registers + 1) & ~1);
```


```
    ((cite: hotspot/src/cpu/x86/vm/sharedRuntime_x86_64.cpp))
    // VMRegImpl::stack0 refers to the first slot 0(sp).
    // and VMRegImpl::stack0+1 refers to the memory word 4-byes higher.  Register
    // up to RegisterImpl::number_of_registers) are the 64-bit
    // integer registers.
```

### 備考(Notes)
なお VMReg という型も使われるが, これは VMRegImpl* のこと.


```
    ((cite: hotspot/src/share/vm/code/vmreg.hpp))
    typedef VMRegImpl* VMReg;
```

### 備考(Notes)
VMRegImpl::stack0 以下の数字の部分は VM/compiled code 共通だが, 
VMRegImpl::stack0 よりも上の部分は compiled code では warped している可能性がある模様.

(compile 時に frame の大きさが分からない場合は warped させる必要があるとのこと.
 大きさがきちんと分からないから, 番号を隙間なく詰めるのが難しい(途中で warp する番号が出る)ということ?? #TODO)


```
    ((cite: hotspot/src/share/vm/code/vmreg.hpp))
    //------------------------------VMReg------------------------------------------
    // The VM uses 'unwarped' stack slots; the compiler uses 'warped' stack slots.
    // Register numbers below VMRegImpl::stack0 are the same for both.  Register
    // numbers above stack0 are either warped (in the compiler) or unwarped
    // (in the VM).  Unwarped numbers represent stack indices, offsets from
    // the current stack pointer.  Warped numbers are required during compilation
    // when we do not yet know how big the frame will be.
```




### 詳細(Details)
See: [here](../doxygen/classVMRegImpl (VMReg).html) for details

---
## <a name="noI8rpuzrh" id="noI8rpuzrh">VMRegPair</a>

### 概要(Summary)
その名の通り VMReg 2つからなるクラス. 64bit 長の格納領域を表すために使われる (VMReg 1つが 32bit 長なので).

calling convention を表現するために使われている模様. (その他の用途でも使われている?)

値が32bitである場合については, 片方は不正(Badという定数値を入れる), という値で表現できる.
また, void(等?)の場合用に, 両方不正値(両方Bad)という状態も可能な模様.


```
    ((cite: hotspot/src/share/vm/code/vmreg.hpp))
    //---------------------------VMRegPair-------------------------------------------
    // Pairs of 32-bit registers for arguments.
    // SharedRuntime::java_calling_convention will overwrite the structs with
    // the calling convention's registers.  VMRegImpl::Bad is returned for any
    // unused 32-bit register.  This happens for the unused high half of Int
    // arguments, or for 32-bit pointers or for longs in the 32-bit sparc build
    // (which are passed to natives in low 32-bits of e.g. O0/O1 and the high
    // 32-bits of O0/O1 are set to VMRegImpl::Bad).  Longs in one register & doubles
    // always return a high and a low register, as do 64-bit pointers.
    //
    class VMRegPair {
```

### 使われ方(Usage)
calling convention 関連の使用箇所としては,
SharedRuntime::java_calling_convention() や
SharedRuntime::c_calling_convention() で
calling convention に合わせて VMRegPair を作成している.

(作成された VMRegPair オブジェクトは, SharedRuntime::gen_i2c_adapter() や
SharedRuntime::gen_c2i_adapter(),
SharedRuntime::generate_native_wrapper() で使用されている)

(その他の使用箇所は?? #TODO)

#### 生成箇所(where its instances are created)

```
    ((cite: hotspot/src/cpu/x86/vm/sharedRuntime_x86_64.cpp))
    int SharedRuntime::java_calling_convention(const BasicType *sig_bt,
                                               VMRegPair *regs,
                                               int total_args_passed,
                                               int is_outgoing) {
    ...
        case T_BOOLEAN:
        case T_CHAR:
        case T_BYTE:
        case T_SHORT:
        case T_INT:
          if (int_args < Argument::n_int_register_parameters_j) {
            regs[i].set1(INT_ArgReg[int_args++]->as_VMReg());
    ...
        case T_OBJECT:
        case T_ARRAY:
        case T_ADDRESS:
          if (int_args < Argument::n_int_register_parameters_j) {
            regs[i].set2(INT_ArgReg[int_args++]->as_VMReg());
    ...
```


```
    ((cite: hotspot/src/cpu/x86/vm/sharedRuntime_x86_64.cpp))
    int SharedRuntime::c_calling_convention(const BasicType *sig_bt,
                                             VMRegPair *regs,
                                             int total_args_passed) {
    ...
          case T_BOOLEAN:
          case T_CHAR:
          case T_BYTE:
          case T_SHORT:
          case T_INT:
            if (int_args < Argument::n_int_register_parameters_c) {
              regs[i].set1(INT_ArgReg[int_args++]->as_VMReg());
    ...
          case T_OBJECT:
          case T_ARRAY:
          case T_ADDRESS:
            if (int_args < Argument::n_int_register_parameters_c) {
              regs[i].set2(INT_ArgReg[int_args++]->as_VMReg());
    ...
```

#### 使用箇所(where its instances are used)

```
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    AdapterHandlerEntry* AdapterHandlerLibrary::get_adapter(methodHandle method) {
    ...
        // Get a description of the compiled java calling convention and the largest used (VMReg) stack slot usage
        int comp_args_on_stack = SharedRuntime::java_calling_convention(sig_bt, regs, total_args_passed, false);
    ...
          entry = SharedRuntime::generate_i2c2i_adapters(&_masm,
                                                         total_args_passed,
                                                         comp_args_on_stack,
                                                         sig_bt,
                                                         regs,
                                                         fingerprint);
```


```
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    nmethod *AdapterHandlerLibrary::create_native_wrapper(methodHandle method, int compile_id) {
    ...
          comp_args_on_stack = SharedRuntime::java_calling_convention(sig_bt, regs, total_args_passed, false);
    
          // Generate the compiled-to-native wrapper code
          nm = SharedRuntime::generate_native_wrapper(&_masm,
                                                      method,
                                                      compile_id,
                                                      total_args_passed,
                                                      comp_args_on_stack,
                                                      sig_bt,regs,
                                                      ret_type);
```




### 詳細(Details)
See: [here](../doxygen/classVMRegPair.html) for details

---
