---
layout: default
title: Assembler クラス関連のクラス (Argument, Address, AddressLiteral, RuntimeAddress, OopAddress, ExternalAddress, InternalAddress, ArrayAddress, Assembler, MacroAssembler, SkipIfEqual, 及びそれらの補助クラス(ControlWord, StatusWord, TagWord, FPU_Register, FPU_State, Flag_Register, IU_Register, IU_State, CPU_State))
---
[Top](../index.html)

#### Assembler クラス関連のクラス (Argument, Address, AddressLiteral, RuntimeAddress, OopAddress, ExternalAddress, InternalAddress, ArrayAddress, Assembler, MacroAssembler, SkipIfEqual, 及びそれらの補助クラス(ControlWord, StatusWord, TagWord, FPU_Register, FPU_State, Flag_Register, IU_Register, IU_State, CPU_State))

これらは, 動的コード生成を補佐するクラス.
より具体的に言うと, Assembler クラス (実行時に x86 用のマシン語を出力するクラス) 関連のプラットフォーム依存な部分を定義するクラス (See: [here](no7882z5r.html) for details).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    // Contains all the definitions needed for x86 assembly code generation.
```


### クラス一覧(class list)

  * [Assembler](#no-MNBoPID)
  * [MacroAssembler](#nosvers-nn)
  * [Argument](#noDOR2mHzp)
  * [Address](#noKfybH0Rs)
  * [AddressLiteral](#no3T2bRz5O)
  * [RuntimeAddress](#nonARVKeka)
  * [OopAddress](#nosFc8Xm5O)
  * [ExternalAddress](#nolhkebjt2)
  * [InternalAddress](#noF6SSmw5M)
  * [ArrayAddress](#no0SQLNybj)
  * [SkipIfEqual](#noo7FvClju)
  * [CPU_State](#noi4fmN5Wa)
  * [FPU_State](#nohOIWExAY)
  * [ControlWord](#noZ2jlJDr0)
  * [StatusWord](#noLofJcVTv)
  * [TagWord](#now-sMnJs6)
  * [FPU_Register](#noFHuSxeuK)
  * [IU_State](#no8hlXmm93)
  * [Flag_Register](#noQ-8_ZpHt)
  * [IU_Register](#no5DG_M0A2)


---
## <a name="no-MNBoPID" id="no-MNBoPID">Assembler</a>

### 概要(Summary)
x86 用のマシン語を実行時に生成するためのクラス (の基底クラス) (See: [here](no7882z5r.html) for details).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    // The Intel x86/Amd64 Assembler: Pure assembler doing NO optimizations on the instruction
    // level (e.g. mov rax, 0 is not translated into xor rax, rax!); i.e., what you write
    // is what you get. The Assembler is generating code into a CodeBuffer.
    
    class Assembler : public AbstractAssembler  {
```

### 内部構造(Internal structure)
内部には, 各命令のニーモニックに対応するメソッドが定義されている.

(これらを呼び出すと対応するマシン語が CodeBuffer 中に出力される)


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    ...
      void lea(Register dst, Address src);
    
      void mov(Register dst, Register src);
    
      void pusha();
      void popa();
    
    ...
      // Vanilla instructions in lexical order
    
      void adcl(Address dst, int32_t imm32);
    ...
    
      void addl(Address dst, int32_t imm32);
    ...
      void andl(Register dst, int32_t imm32);
    ...
      void call(Label& L, relocInfo::relocType rtype);
    ...
      void cmpl(Address dst, int32_t imm32);
    ...
      void cmpxchgq(Register reg, Address adr);
    ...
```




### 詳細(Details)
See: [here](../doxygen/classAssembler.html) for details

---
## <a name="nosvers-nn" id="nosvers-nn">MacroAssembler</a>

### 概要(Summary)
Assembler クラスの具象サブクラス.

Assembler クラスのメソッドに加えて, 
(複数の命令から構成されるような) 少し複雑なマシン語生成ルーチンが用意されている
(See: [here](no7882z5r.html) for details).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    // MacroAssembler extends Assembler by frequently used macros.
    //
    // Instructions for which a 'better' code sequence exists depending
    // on arguments should also go in here.
    
    class MacroAssembler: public Assembler {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 出力先となる CodeBuffer オブジェクトを用意する
2. MacroAssembler オブジェクトを生成する (用意した CodeBuffer オブジェクトをコンストラクタ引数として渡す)
3. MacroAssembler のメソッドを用いてコードを CodeBuffer 中に書き出していく.
4. 生成し終わったら Assembler::flush() を呼び出して完了.


```
    ((cite: hotspot/src/cpu/x86/vm/vtableStubs_x86_64.cpp))
    #define __ masm->
```


```
    ((cite: hotspot/src/cpu/x86/vm/vtableStubs_x86_64.cpp))
      CodeBuffer cb(s->entry_point(), amd64_code_length);
      MacroAssembler* masm = new MacroAssembler(&cb);
    
    ...
        Label L;
        // check offset vs vtable length
        __ cmpl(Address(rax, instanceKlass::vtable_length_offset() * wordSize),
                vtable_index * vtableEntry::size());
        __ jcc(Assembler::greater, L);
        __ movl(rbx, vtable_index);
        __ call_VM(noreg,
                   CAST_FROM_FN_PTR(address, bad_compiled_vtable_index), j_rarg0, rbx);
        __ bind(L);
    ...
    
      __ flush();
```

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(基本的にはコード生成を行う箇所, あるいはコード生成を行うクラスのコンストラクタ).

* CompactingPermGenGen::generate_vtable_methods()
* InlineCacheBuffer::assemble_ic_buffer_code()
* SignatureHandlerGenerator::SignatureHandlerGenerator()
* JNI_FastGetField::generate_fast_get_int_field0()
* JNI_FastGetField::generate_fast_get_long_field()  (32bit の場合のみ)
* JNI_FastGetField::generate_fast_get_float_field0()
* OptoRuntime::generate_exception_blob()
* SharedRuntime::generate_deopt_blob()
* SharedRuntime::generate_uncommon_trap_blob()
* SharedRuntime::generate_handler_blob()
* SharedRuntime::generate_resolve_blob()
* StubGenerator::generate_throw_exception()
* VtableStubs::create_vtable_stub()
* VtableStubs::create_itable_stub()
* os::register_code_area()  (64bit の場合のみ)
* SharedRuntime::generate_ricochet_blob()
* StubCodeGenerator::StubCodeGenerator()
* SharkCompiler::compile_method()

### 内部構造(Internal structure)
内部には, 以下のようなメソッドが定義されている.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    ...
      void null_check(Register reg, int offset = -1);
    ...
      // Stack frame creation/removal
      void enter();
      void leave();
    ...
      // Support for VM calls
      //
      // It is imperative that all calls into the VM are handled via the call_VM macros.
      // They make sure that the stack linkage is setup correctly. call_VM's correspond
      // to ENTRY/ENTRY_X entry points while call_VM_leaf's correspond to LEAF entry points.
    
    
      void call_VM(Register oop_result,
                   address entry_point,
                   bool check_exceptions = true);
    ...
      // Stores
      void store_check(Register obj);                // store check for obj - register is destroyed afterwards
    ...
      void load_klass(Register dst, Register src);
      void store_klass(Register dst, Register src);
```

### 備考(Notes)
コード中では "__" という記法で使用されるケースが多い (多くの場合, "__" は "_masm->" や "_masm." の #define).




### 詳細(Details)
See: [here](../doxygen/classMacroAssembler.html) for details

---
## <a name="noDOR2mHzp" id="noDOR2mHzp">Argument</a>

### 概要(Summary)
x86 の calling convention に関する定数値を納めた名前空間
(このクラスは AllStatic ではないが, static な定義しか持たない).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    // Calling convention
    class Argument VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている
(これらは calling convention が関連する箇所).

* SignatureHandlerGenerator::SignatureHandlerGenerator() 内
* InterpreterRuntime::SignatureHandlerGenerator::pass_*() 内
* AbstractInterpreterGenerator::generate_slow_signature_handler() 内
* SharedRuntime::java_calling_convention() 内
* SharedRuntime::c_calling_convention() 内
* SharedRuntime::generate_dtrace_nmethod() 内

### 内部構造(Internal structure)
内部には以下の定数定義(のみ)を含む.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
      enum {
    #ifdef _LP64
    #ifdef _WIN64
        n_int_register_parameters_c   = 4, // rcx, rdx, r8, r9 (c_rarg0, c_rarg1, ...)
        n_float_register_parameters_c = 4,  // xmm0 - xmm3 (c_farg0, c_farg1, ... )
    #else
        n_int_register_parameters_c   = 6, // rdi, rsi, rdx, rcx, r8, r9 (c_rarg0, c_rarg1, ...)
        n_float_register_parameters_c = 8,  // xmm0 - xmm7 (c_farg0, c_farg1, ... )
    #endif // _WIN64
        n_int_register_parameters_j   = 6, // j_rarg0, j_rarg1, ...
        n_float_register_parameters_j = 8  // j_farg0, j_farg1, ...
    #else
        n_register_parameters = 0   // 0 registers used to pass arguments
    #endif // _LP64
      };
```




### 詳細(Details)
See: [here](../doxygen/classArgument.html) for details

---
## <a name="noKfybH0Rs" id="noKfybH0Rs">Address</a>

### 概要(Summary)
Assembler クラス内で使用される補助クラス.

x86 のアドレッシングモードによるメモリアドレス指定を表すクラス.

(主に load/store 命令のオペランドを扱う際などに使用される)


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    // Address is an abstraction used to represent a memory location
    // using any of the amd64 addressing modes with one object.
    //
    // Note: A register location is represented via a Register, not
    //       via an address for efficiency & simplicity reasons.
```


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    class Address VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り
(_rspec 以外は, それぞれ x86 のアドレッシングモードの要素に対応).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
      Register         _base;
      Register         _index;
      ScaleFactor      _scale;
      int              _disp;
      RelocationHolder _rspec;
```

なお, ScaleFactor 型は以下のように定義された enum 値.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
      enum ScaleFactor {
        no_scale = -1,
        times_1  =  0,
        times_2  =  1,
        times_4  =  2,
        times_8  =  3,
        times_ptr = LP64_ONLY(times_8) NOT_LP64(times_4)
      };
```




### 詳細(Details)
See: [here](../doxygen/classAddress.html) for details

---
## <a name="no3T2bRz5O" id="no3T2bRz5O">AddressLiteral</a>

### 概要(Summary)
Assembler クラス内で使用される補助クラス.

メモリアドレスを表す即値を表現するクラス.

(主に load/store 命令のオペランドを扱う際などに使用される)

(なおコメントによると, 
 「昔は Address クラスがこの役割も担当していたが
  32bit と 64bit で少し取り扱いが違うので別クラスにした」 
 とのこと)


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    //
    // AddressLiteral has been split out from Address because operands of this type
    // need to be treated specially on 32bit vs. 64bit platforms. By splitting it out
    // the few instructions that need to deal with address literals are unique and the
    // MacroAssembler does not have to implement every instruction in the Assembler
    // in order to search for address literals that may need special handling depending
    // on the instruction and the platform. As small step on the way to merging i486/amd64
    // directories.
    //
    class AddressLiteral VALUE_OBJ_CLASS_SPEC {
```

なお, このクラスは abstract class ではない
(See: 
 ArrayAddress, LIR_Assembler::call(), ArrayCopyStub::emit_code(), LIR_Assembler::as_Address(), 
 LIR_Assembler::return_op(), LIR_Assembler::safepoint_poll(), LIR_Assembler::ic_call(), 
 MacroAssembler::get_thread() (solaris x86 の場合, 及び windows x86 の場合))

### 使われ方(Usage)
#### 使用方法の概要(how to use)
Address オブジェクトが示すアドレッシングモードと異なり, 
AddressLiteral オブジェクトが示す即値は, 命令によってはオペランドのフィールドに収まらないことがある
(例えば 64bit 即値の場合, 即値フィールドが 32bit しかない命令だと入らない).

そのため, RIP 相対で 32bit 以内に収まるかを調べるための
Assembler::reachable() メソッドが用意されている.

(即値フィールドが 32bit の命令の場合, 
 まず Assembler::reachable() でチェックし, 
 32bit に収まらない場合はいったん lea でレジスタにセットしてから使用する, というパターンが多い
 (例: MacroAssembler::cmp64(), 等)).

#### 参考(for your information): Assembler::reachable() (x86 32bit の場合)
32bit の場合は, 常に true をリターンするだけ.

```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
      bool reachable(AddressLiteral adr) NOT_LP64({ return true;});
```

#### 参考(for your information): Assembler::reachable() (x86 64bit の場合)
See: [here](no2935-xo.html) for details

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* RelocationHolder _rspec
  
  このアドレス即値に関する再配置情報

  (例えば, Safepoint Polling 用のメモリページのアドレスであれば relocInfo::poll_type, 等.
  AddressLiteral クラスの各サブクラスの説明も参照)
  (See: [here](no2935-q0.html) for details).

* bool _is_lval

  このアドレスが指す中身ではなく, このアドレス値自体を使いたい場合には true になる
  (参考: MacroAssembler::movptr(), MacroAssembler::pushptr(), etc).
  
  なお, _is_lval フィールドが true になった AddressLiteral オブジェクトを取得するには
  AddressLiteral::addr() メソッドを使えば良い
  (その他のフィールドは receiver と同じで, _is_lval フィールドだけが true になった AddressLiteral オブジェクトが返される.
   _is_lval フィールドが true の AddressLiteral オブジェクトを作るには, これ以外の方法はない).

* address          _target
  
  この AddressLiteral オブジェクトが表すアドレス即値.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
      RelocationHolder _rspec;
      // Typically we use AddressLiterals we want to use their rval
      // However in some situations we want the lval (effect address) of the item.
      // We provide a special factory for making those lvals.
      bool _is_lval;
    
      // If the target is far we'll need to load the ea of this to
      // a register to reach it. Otherwise if near we can do rip
      // relative addressing.
    
      address          _target;
```

#### 参考(for your information): AddressLiteral::addr()
See: [here](no2935kdc.html) for details



### 詳細(Details)
See: [here](../doxygen/classAddressLiteral.html) for details

---
## <a name="nonARVKeka" id="nonARVKeka">RuntimeAddress</a>

### 概要(Summary)
AddressLiteral クラスのサブクラスの1つ.
このクラスは, ランタイムクラスが保持しているメソッド等のアドレス用.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    // Convience classes
    class RuntimeAddress: public AddressLiteral {
```

### 内部構造(Internal structure)
内部的には, スーパークラスである AddressLiteral とほぼ同じ.

(定義されているのはコンストラクタのみ. 再配置情報として relocInfo::runtime_call_type を指定する点が異なる)


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
      RuntimeAddress(address target) : AddressLiteral(target, relocInfo::runtime_call_type) {}
```




### 詳細(Details)
See: [here](../doxygen/classRuntimeAddress.html) for details

---
## <a name="nosFc8Xm5O" id="nosFc8Xm5O">OopAddress</a>

### 概要(Summary)
AddressLiteral クラスのサブクラスの1つ.
このクラスは, 何らかの oop を指すアドレス用.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    class OopAddress: public AddressLiteral {
```

### 内部構造(Internal structure)
内部的には, スーパークラスである AddressLiteral とほぼ同じ

(定義されているのはコンストラクタのみ. 再配置情報として relocInfo::oop_type を指定する点が異なる).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
      OopAddress(address target) : AddressLiteral(target, relocInfo::oop_type){}
```




### 詳細(Details)
See: [here](../doxygen/classOopAddress.html) for details

---
## <a name="nolhkebjt2" id="nolhkebjt2">ExternalAddress</a>

### 概要(Summary)
AddressLiteral クラスのサブクラスの1つ.
このクラスは, この CodeBlob 外にある何らかの固定アドレス用.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    class ExternalAddress: public AddressLiteral {
```

### 内部構造(Internal structure)
内部的には, スーパークラスである AddressLiteral とほぼ同じ

(定義されているのはコンストラクタ(とその補助関数)のみ. 
 再配置情報として ExternalAddress::reloc_for_target() の返値を指定する点が異なる).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
      ExternalAddress(address target) : AddressLiteral(target, reloc_for_target(target)) {}
```

#### 参考(for your information): ExternalAddress::reloc_for_target()
See: [here](no2935eJK.html) for details



### 詳細(Details)
See: [here](../doxygen/classExternalAddress.html) for details

---
## <a name="noF6SSmw5M" id="noF6SSmw5M">InternalAddress</a>

### 概要(Summary)
AddressLiteral クラスのサブクラスの1つ.
このクラスは, この CodeBlob 内にある何らかのアドレス用.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    class InternalAddress: public AddressLiteral {
```

### 内部構造(Internal structure)
内部的には, スーパークラスである AddressLiteral とほぼ同じ

(定義されているのはコンストラクタのみ. 再配置情報として relocInfo::internal_word_type を指定する点が異なる).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
      InternalAddress(address target) : AddressLiteral(target, relocInfo::internal_word_type) {}
```




### 詳細(Details)
See: [here](../doxygen/classInternalAddress.html) for details

---
## <a name="no0SQLNybj" id="no0SQLNybj">ArrayAddress</a>

### 概要(Summary)
Assembler クラス内で使用される補助クラス.

配列内を指すアドレスを表現するためのクラス.

(主に load/store 命令のオペランドを扱う際などに使用される)

(x86 の複雑なアドレッシングモードのおかげで「配列アクセスを1命令で行える」という特性を活かすためのクラス.
 ただし, 64bit では (即値フィールドが 32bit しかなくて) そうもいかないので基本的に 32bit 専用.
 (See: Address::make_array(), MacroAssembler::jump(), etc))


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    // x86 can do array addressing as a single operation since disp can be an absolute
    // address amd64 can't. We create a class that expresses the concept but does extra
    // magic on amd64 to get the final result
    
    class ArrayAddress VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
所望の配列要素を表す ArrayAddress オブジェクトを作った後, 
MacroAssembler::as_Address() で対応する Address オブジェクトを生成する.

### 内部構造(Internal structure)
定義されているフィールドは以下の通り

(そして, メソッドはこれらのフィールドへの getter メソッド(アクセサメソッド)のみ).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
      AddressLiteral _base;
      Address        _index;
```




### 詳細(Details)
See: [here](../doxygen/classArrayAddress.html) for details

---
## <a name="noo7FvClju" id="noo7FvClju">SkipIfEqual</a>

### 概要(Summary)
Assembler クラス用のユーティリティ・クラス.
Assembler クラスによるコード生成を容易化する.

「ある条件が成り立つ場合にだけ実行するコード」を生成する際に, それをソースコード上のスコープに合わせて行うことができる
(このクラスは StackObj のサブクラスではないが, 使い方としては実質 StackObj クラスのようなもの).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
    /**
     * class SkipIfEqual:
     *
     * Instantiating this class will result in assembly code being output that will
     * jump around any code emitted between the creation of the instance and it's
     * automatic destruction at the end of a scope block, depending on the value of
     * the flag passed to the constructor, which will be checked at run-time.
     */
    class SkipIfEqual {
```

### 使われ方(Usage)
#### 使用例(usage examples)
(「DTraceMethodProbes オプションがセットされている場合にのみ 
SharedRuntime::dtrace_method_entry() を呼び出す」というコードを生成する)


```
    ((cite: hotspot/src/cpu/x86/vm/interp_masm_x86_64.cpp))
      {
        SkipIfEqual skip(this, &DTraceMethodProbes, false);
        get_method(c_rarg1);
        call_VM_leaf(CAST_FROM_FN_PTR(address, SharedRuntime::dtrace_method_entry),
                     r15_thread, c_rarg1);
      }
```

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* InterpreterMacroAssembler::notify_method_entry()
* InterpreterMacroAssembler::notify_method_exit()
* SharedRuntime::generate_native_wrapper()
* TemplateTable::_new()

### 内部構造(Internal structure)
コンストラクタで cmp8 と jcc を生成し, デストラクタで jcc の飛び先を bind() するだけ.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
    SkipIfEqual::SkipIfEqual(
        MacroAssembler* masm, const bool* flag_addr, bool value) {
      _masm = masm;
      _masm->cmp8(ExternalAddress((address)flag_addr), value);
      _masm->jcc(Assembler::equal, _label);
    }
    
    SkipIfEqual::~SkipIfEqual() {
      _masm->bind(_label);
    }
```




### 詳細(Details)
See: [here](../doxygen/classSkipIfEqual.html) for details

---
## <a name="noi4fmN5Wa" id="noi4fmN5Wa">CPU_State</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: VerifyFPU).

整数ユニット(IU)と浮動小数点ユニット(FPU)の現在の状態を表すクラス. 各レジスタの値などを保持している.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
    class CPU_State {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
MacroAssembler::push_CPU_state() が生成するコードにより, 
スタック上に CPU_State オブジェクトが生成される
(その時点の RSP の値を見れば, その CPU_State オブジェクトのアドレスが取得できる).

使い終わったら, MacroAssembler::pop_CPU_state() が生成するコードで破棄できる.

(なお, CPU_State オブジェクト自体は使わないが, 
単にレジスタの待避復帰処理のために
MacroAssembler::push_CPU_state() と MacroAssembler::pop_CPU_state() を使用している箇所もある
(See: RegisterSaver::save_live_registers(), RegisterSaver::restore_live_registers(), patch_callers_callsite())).

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* _print_CPU_state() 内
  
  なお, この関数は, 現在は以下のパスで(のみ)呼び出されている
  (が, MacroAssembler::print_CPU_state() 自体の使用箇所が見当たらない...).
  
```
MacroAssembler::print_CPU_state() が生成するコード
-> _print_CPU_state()
```

* _verify_FPU() 内 
  
  VerifyFPU オプションが指定されている場合にのみ, 
  MacroAssembler::verify_FPU() が生成するコード内で呼び出される.
  
```
MacroAssembler::verify_FPU() が生成するコード
-> _verify_FPU()  (VerifyFPU オプションが指定されている場合にのみ呼び出す)
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
      FPU_State _fpu_state;
      IU_State  _iu_state;
```




### 詳細(Details)
See: [here](../doxygen/classCPU__State.html) for details

---
## <a name="nohOIWExAY" id="nohOIWExAY">FPU_State</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: VerifyFPU).

CPU_State クラス内で使用される補助クラス.
浮動小数点ユニット(FPU)の現在の状態を表す (各レジスタの値などを保持している).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
    class FPU_State {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
      ControlWord  _control_word;
      StatusWord   _status_word;
      TagWord      _tag_word;
      int32_t      _error_offset;
      int32_t      _error_selector;
      int32_t      _data_offset;
      int32_t      _data_selector;
      int8_t       _register[register_size * number_of_registers];
```




### 詳細(Details)
See: [here](../doxygen/classFPU__State.html) for details

---
## <a name="noZ2jlJDr0" id="noZ2jlJDr0">ControlWord</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: VerifyFPU).

FPU_State 内で使用される補助クラス.
x87 の Control Word の現在の値を表す.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
    class ControlWord {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
      int32_t _value;
```




### 詳細(Details)
See: [here](../doxygen/classControlWord.html) for details

---
## <a name="noLofJcVTv" id="noLofJcVTv">StatusWord</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: VerifyFPU).

FPU_State 内で使用される補助クラス.
x87 の Status Word の現在の値を表す.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
    class StatusWord {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
      int32_t _value;
```




### 詳細(Details)
See: [here](../doxygen/classStatusWord.html) for details

---
## <a name="now-sMnJs6" id="now-sMnJs6">TagWord</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: VerifyFPU).

FPU_State クラス内で使用される補助クラス.
x87 の Tag Word の現在の値を表す.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
    class TagWord {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
      int32_t _value;
```




### 詳細(Details)
See: [here](../doxygen/classTagWord.html) for details

---
## <a name="noFHuSxeuK" id="noFHuSxeuK">FPU_Register</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: VerifyFPU).

FPU_State クラス内で使用される補助クラス.
各浮動小数点レジスタの現在の値を表す.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
    class FPU_Register {
```

### 使われ方(Usage)
FPU_State::st() の返値として(のみ)生成される.
FPU_State::print() 内でのみ使用されている.

### 内部構造(Internal structure)
定義されているフィールドは以下の通り
(それぞれ, 80bit のレジスタの, 仮数部(64bit)の上位32bits/下位32bitsと指数部16bitsに対応).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
      int32_t _m0;
      int32_t _m1;
      int16_t _ex;
```




### 詳細(Details)
See: [here](../doxygen/classFPU__Register.html) for details

---
## <a name="no8hlXmm93" id="no8hlXmm93">IU_State</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: VerifyFPU).

CPU_State クラス内で使用される補助クラス.
整数ユニット(IU)の現在の状態を表す (各レジスタの値などを保持している).


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
    class IU_State {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
      Flag_Register _eflags;
      IU_Register   _rdi;
      IU_Register   _rsi;
      IU_Register   _rbp;
      IU_Register   _rsp;
      IU_Register   _rbx;
      IU_Register   _rdx;
      IU_Register   _rcx;
      IU_Register   _rax;
```




### 詳細(Details)
See: [here](../doxygen/classIU__State.html) for details

---
## <a name="noQ-8_ZpHt" id="noQ-8_ZpHt">Flag_Register</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: VerifyFPU).

IU_State クラス内で使用される補助クラス.
フラグレジスタの現在の値を表す.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
    class Flag_Register {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
      int32_t _value;
```




### 詳細(Details)
See: [here](../doxygen/classFlag__Register.html) for details

---
## <a name="no5DG_M0A2" id="no5DG_M0A2">IU_Register</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: VerifyFPU).

IU_State クラス内で使用される補助クラス.
各汎用レジスタの現在の値を表す.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
    class IU_Register {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.cpp))
      int32_t _value;
```




### 詳細(Details)
See: [here](../doxygen/classIU__Register.html) for details

---
