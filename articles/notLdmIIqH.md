---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/cpu/ 以下 ： x86/
---
[Up](noSDaFQh2w.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/cpu/ 以下 ： x86/

--- 

File Name                                                      | Description
-------------------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/cpu/x86/vm/assembler_x86.cpp                       |  Assembler クラス関連のクラスの定義 ([Argument, Address, AddressLiteral, RuntimeAddress, OopAddress, ExternalAddress, InternalAddress, ArrayAddress, Assembler, MacroAssembler, SkipIfEqual, 及びそれらの補助クラス(ControlWord, StatusWord, TagWord, FPU_Register, FPU_State, Flag_Register, IU_Register, IU_State, CPU_State)](noo_yMa7yM.html))
hotspot/src/cpu/x86/vm/assembler_x86.hpp                       |  同上
hotspot/src/cpu/x86/vm/assembler_x86.inline.hpp                |  同上
hotspot/src/cpu/x86/vm/bytecodeInterpreter_x86.cpp             |  BytecodeInterpreter クラスのプラットフォーム依存な部分の定義 (※1)
hotspot/src/cpu/x86/vm/bytecodeInterpreter_x86.hpp             |  同上
hotspot/src/cpu/x86/vm/bytecodeInterpreter_x86.inline.hpp      |  同上
hotspot/src/cpu/x86/vm/bytecodes_x86.cpp                       |  Bytecodes クラスのプラットフォーム依存な部分の定義 (Bytecodes::pd_initialize(), Bytecodes::pd_base_code_for()) (※2) (※3)
hotspot/src/cpu/x86/vm/bytecodes_x86.hpp                       |  同上
hotspot/src/cpu/x86/vm/bytes_x86.hpp                           |  Bytes クラスの定義 ([Bytes](noqIvHWGWb.html))
hotspot/src/cpu/x86/vm/c1_CodeStubs_x86.cpp                    |
hotspot/src/cpu/x86/vm/c1_Defs_x86.hpp                         |
hotspot/src/cpu/x86/vm/c1_FpuStackSim_x86.cpp                  |
hotspot/src/cpu/x86/vm/c1_FpuStackSim_x86.hpp                  |
hotspot/src/cpu/x86/vm/c1_FrameMap_x86.cpp                     |
hotspot/src/cpu/x86/vm/c1_FrameMap_x86.hpp                     |
hotspot/src/cpu/x86/vm/c1_LIRAssembler_x86.cpp                 |
hotspot/src/cpu/x86/vm/c1_LIRAssembler_x86.hpp                 |
hotspot/src/cpu/x86/vm/c1_LIRGenerator_x86.cpp                 |
hotspot/src/cpu/x86/vm/c1_LinearScan_x86.cpp                   |
hotspot/src/cpu/x86/vm/c1_LinearScan_x86.hpp                   |
hotspot/src/cpu/x86/vm/c1_MacroAssembler_x86.cpp               |
hotspot/src/cpu/x86/vm/c1_MacroAssembler_x86.hpp               |
hotspot/src/cpu/x86/vm/c1_Runtime1_x86.cpp                     |
hotspot/src/cpu/x86/vm/c1_globals_x86.hpp                      |  C1 関連の JVM のコマンドラインオプションのうちプラットフォーム依存なものの定義 (及びデフォルト値の変更) (See: [here](no7882y_Y.html) for details)
hotspot/src/cpu/x86/vm/c2_globals_x86.hpp                      |  C2 関連の JVM のコマンドラインオプションのうちプラットフォーム依存なものの定義 (及びデフォルト値の変更) (See: [here](no7882y_Y.html) for details)
hotspot/src/cpu/x86/vm/c2_init_x86.cpp                         |  Compile クラスのプラットフォーム依存な部分の定義 (Compile::pd_compiler2_init())
hotspot/src/cpu/x86/vm/codeBuffer_x86.hpp                      |  CodeBuffer クラスのプラットフォーム依存な部分の定義 (CodeBuffer::pd_initialize(), CodeBuffer::flush_bundle()) (※4)
hotspot/src/cpu/x86/vm/copy_x86.hpp                            |  Copy クラスのプラットフォーム依存な部分の定義
hotspot/src/cpu/x86/vm/cppInterpreterGenerator_x86.hpp         |  CppInterpreterGenerator クラスのプラットフォーム依存な部分の宣言 (なお, 宣言したメソッドの定義は hotspot/src/cpu/x86/vm/cppInterpreter_x86.cpp で行っている)
hotspot/src/cpu/x86/vm/cppInterpreter_x86.cpp                  |  CppInterpreter, CppInterpreterGenerator, AbstractInterpreter, AbstractInterpreterGenerator, BytecodeInterpreter, 及び InterpreterGenerator クラスのプラットフォーム依存な部分の定義 (※5)
hotspot/src/cpu/x86/vm/cppInterpreter_x86.hpp                  |  CppInterpreter クラスのプラットフォーム依存なフィールドの宣言 (CppInterpreter::InterpreterCodeSize)
hotspot/src/cpu/x86/vm/debug_x86.cpp                           |  プラットフォーム依存なデバッグ用の関数の定義(※6) (See: hotspot/src/share/vm/utilities/debug.hpp)
hotspot/src/cpu/x86/vm/depChecker_x86.cpp                      |  ??(空ファイル) (※7)
hotspot/src/cpu/x86/vm/depChecker_x86.hpp                      |  同上
hotspot/src/cpu/x86/vm/disassembler_x86.hpp                    |  Disassembler クラスのプラットフォーム依存な部分の定義 (pd_instruction_alignment(), pd_cpu_opts()) (※8)
hotspot/src/cpu/x86/vm/dump_x86_32.cpp                         |  Class Data Sharing (CDS) のダンプ出力用メソッド(GenCollectedHeap::preload_and_dump()) の補助関数のうちプラットフォーム依存なものの定義 (CompactingPermGenGen::generate_vtable_methods()) (x86 32bit 用)  (<= 具体的に何をするものなのかは不明 #TODO)
hotspot/src/cpu/x86/vm/dump_x86_64.cpp                         |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/frame_x86.cpp                           |  frame クラスのプラットフォーム依存な部分の定義
hotspot/src/cpu/x86/vm/frame_x86.hpp                           |  同上
hotspot/src/cpu/x86/vm/frame_x86.inline.hpp                    |  同上
hotspot/src/cpu/x86/vm/globalDefinitions_x86.hpp               |  HotSpot 内で広く利用される定数や型宣言,ユーティリティ・クラス等のうちプラットフォーム依存なものの定義 (See: hotspot/src/share/vm/utilities/globalDefinitions.hpp) (定義されているのは StackAlignmentInBytes という定数のみ)
hotspot/src/cpu/x86/vm/globals_x86.hpp                         |  JVM のコマンドラインオプションのうちプラットフォーム依存なものの定義 (及びデフォルト値の変更) (See: [here](no7882y_Y.html) for details)
hotspot/src/cpu/x86/vm/icBuffer_x86.cpp                        |  InlineCacheBuffer クラスのプラットフォーム依存な部分の定義
hotspot/src/cpu/x86/vm/icache_x86.cpp                          |  ICache クラスの定義, および ICacheStubGenerator クラスのプラットフォーム依存な部分の定義 (ICacheStubGenerator::generate_icache_flush()) ([ICache](noLFMG6GL9.html))
hotspot/src/cpu/x86/vm/icache_x86.hpp                          |  同上
hotspot/src/cpu/x86/vm/interp_masm_x86_32.cpp                  |  InterpreterMacroAssembler クラスの定義 (x86 32bit 用) ([InterpreterMacroAssembler](noODqGbz42.html))
hotspot/src/cpu/x86/vm/interp_masm_x86_32.hpp                  |  同上 (x86 32bit 用)
hotspot/src/cpu/x86/vm/interp_masm_x86_64.cpp                  |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/interp_masm_x86_64.hpp                  |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/interpreterGenerator_x86.hpp            |  InterpreterGenerator クラスのプラットフォーム依存な部分の宣言 (なお, 宣言したメソッドの定義は hotspot/src/cpu/x86/vm/cppInterpreter_x86.cpp, hotspot/src/cpu/x86/vm/templateInterpreter_x86_32(or64).cpp, 及び hotspot/src/cpu/x86/vm/interpreter_x86_32(or64).cpp で行っている)
hotspot/src/cpu/x86/vm/interpreterRT_x86.hpp                   |  InterpreterRuntime クラスのプラットフォーム依存な部分の定義(InterpreterRuntime::slow_signature_handler()), 及びその補助クラスの定義 ([InterpreterRuntime::SignatureHandlerGenerator, SlowSignatureHandler](nofTaBu4as.html))
hotspot/src/cpu/x86/vm/interpreterRT_x86_32.cpp                |  同上
hotspot/src/cpu/x86/vm/interpreterRT_x86_64.cpp                |  同上
hotspot/src/cpu/x86/vm/interpreter_x86.hpp                     |  Interpreter クラスのプラットフォーム依存な部分の定義
hotspot/src/cpu/x86/vm/interpreter_x86_32.cpp                  |  AbstractInterpreterGenerator, InterpreterGenerator, 及び Deoptimization クラスのプラットフォーム依存な部分の定義
hotspot/src/cpu/x86/vm/interpreter_x86_64.cpp                  |  同上
hotspot/src/cpu/x86/vm/javaFrameAnchor_x86.hpp                 |  JavaFrameAnchor クラスのプラットフォーム依存な部分の定義
hotspot/src/cpu/x86/vm/jniFastGetField_x86_32.cpp              |  JNI_FastGetField クラスのプラットフォーム依存な部分の定義 (x86 32bit 用)
hotspot/src/cpu/x86/vm/jniFastGetField_x86_64.cpp              |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/jniTypes_x86.hpp                        |  JNITypes クラスの定義 ([JNITypes](noPpUAbTya.html))
hotspot/src/cpu/x86/vm/jni_x86.h                               |  JNI 関連のマクロや型宣言のうちプラットフォーム依存なものの定義 (JNIEXPORT, JNIIMPORT, JNICALL, jint, jlong, jbyte)
hotspot/src/cpu/x86/vm/methodHandles_x86.cpp                   |  MethodHandles クラスのプラットフォーム依存な部分の定義, 及びその補助クラスの定義 ([MethodHandles::RicochetFrame](noisXLUWg7.html))
hotspot/src/cpu/x86/vm/methodHandles_x86.hpp                   |  同上
hotspot/src/cpu/x86/vm/nativeInst_x86.cpp                      |  NativeInstruction クラス関連のクラスの定義 ([NativeInstruction, NativeCall, NativeMovConstReg, NativeMovConstRegPatching, NativeMovRegMem, NativeMovRegMemPatching, NativeLoadAddress, NativeJump, NativeGeneralJump, NativePopReg, NativeIllegalInstruction, NativeReturn, NativeReturnX, NativeTstRegMem](no_H2Kx7Kd.html))
hotspot/src/cpu/x86/vm/nativeInst_x86.hpp                      |  同上
hotspot/src/cpu/x86/vm/registerMap_x86.hpp                     |  RegisterMap クラスのプラットフォーム依存な部分の定義 (RegisterMap::pd_location(), RegisterMap::pd_clear(), RegisterMap::pd_initialize(), RegisterMap::pd_initialize_from() のみ (※9))
hotspot/src/cpu/x86/vm/register_definitions_x86.cpp            |  CPU のレジスタに関する定義 (※10)
hotspot/src/cpu/x86/vm/register_x86.cpp                        |  RegisterImpl クラス関連のクラスの定義 ([RegisterImpl, FloatRegisterImpl, XMMRegisterImpl, ConcreteRegisterImpl](nojt9dWrY3.html))
hotspot/src/cpu/x86/vm/register_x86.hpp                        |  同上
hotspot/src/cpu/x86/vm/relocInfo_x86.cpp                       |  relocInfo 関連のクラスのプラットフォーム依存な部分の定義
hotspot/src/cpu/x86/vm/relocInfo_x86.hpp                       |  同上
hotspot/src/cpu/x86/vm/runtime_x86_32.cpp                      |  OptoRuntime クラスのプラットフォーム依存な部分の定義 (OptoRuntime::generate_exception_blob()) (※11)
hotspot/src/cpu/x86/vm/runtime_x86_64.cpp                      |  同上
hotspot/src/cpu/x86/vm/sharedRuntime_x86_32.cpp                |  SharedRuntime クラスのプラットフォーム依存な部分の定義, 及びその補助クラスの定義 (x86 32bit 用) ([SimpleRuntimeFrame, RegisterSaver](noNzVJ9cNB.html)) (※12)
hotspot/src/cpu/x86/vm/sharedRuntime_x86_64.cpp                |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/stubGenerator_x86_32.cpp                |  StubGenerator クラスの定義 (x86 32bit 用) ([StubGenerator](nodHW9-t8Q.html))
hotspot/src/cpu/x86/vm/stubGenerator_x86_64.cpp                |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/stubRoutines_x86_32.cpp                 |  StubRoutines クラスのプラットフォーム依存な部分の定義, 及びその補助クラスの定義 (x86 32bit 用) ([StubRoutines::x86](nofrYkCi3d.html))
hotspot/src/cpu/x86/vm/stubRoutines_x86_32.hpp                 |  同上 (x86 32bit 用)
hotspot/src/cpu/x86/vm/stubRoutines_x86_64.cpp                 |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/stubRoutines_x86_64.hpp                 |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/templateInterpreterGenerator_x86.hpp    |  TemplateInterpreterGenerator クラスのプラットフォーム依存な部分の宣言 (なお, 宣言したメソッドの定義は hotspot/src/cpu/x86/vm/templateInterpreter_x86_32(or64).cpp で行っている)
hotspot/src/cpu/x86/vm/templateInterpreter_x86.hpp             |  TemplateInterpreter クラスのプラットフォーム依存なフィールドの宣言 (TemplateInterpreter::InterpreterCodeSize)
hotspot/src/cpu/x86/vm/templateInterpreter_x86_32.cpp          |  TemplateInterpreterGenerator, AbstractInterpreter, AbstractInterpreterGenerator, InterpreterGenerator クラスのプラットフォーム依存な部分の定義 (x86 32bit 用)
hotspot/src/cpu/x86/vm/templateInterpreter_x86_64.cpp          |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/templateTable_x86_32.cpp                |  TemplateTable クラスのプラットフォーム依存な部分の定義 (x86 32bit 用)
hotspot/src/cpu/x86/vm/templateTable_x86_32.hpp                |  同上 (x86 32bit 用)
hotspot/src/cpu/x86/vm/templateTable_x86_64.cpp                |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/templateTable_x86_64.hpp                |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/vmStructs_x86.hpp                       |  VMStructs クラス用のプラットフォーム依存なマクロの定義 (VM_STRUCTS_CPU, VM_TYPES_CPU, VM_INT_CONSTANTS_CPU, VM_LONG_CONSTANTS_CPU)
hotspot/src/cpu/x86/vm/vm_version_x86.cpp                      |  VM_Version クラスの定義 ([VM_Version, 及びその補助クラス(VM_Version_StubGenerator)](nozOgHlzuU.html))
hotspot/src/cpu/x86/vm/vm_version_x86.hpp                      |  同上
hotspot/src/cpu/x86/vm/vmreg_x86.cpp                           |  VMRegImpl(VMReg) 関連のプラットフォーム依存な部分の定義 (※13)
hotspot/src/cpu/x86/vm/vmreg_x86.hpp                           |  同上
hotspot/src/cpu/x86/vm/vmreg_x86.inline.hpp                    |  同上
hotspot/src/cpu/x86/vm/vtableStubs_x86_32.cpp                  |  VtableStub クラスのプラットフォーム依存な部分の定義 (x86 32bit 用)
hotspot/src/cpu/x86/vm/vtableStubs_x86_64.cpp                  |  同上 (x86 64bit 用)
hotspot/src/cpu/x86/vm/x86_32.ad                               |  AD ファイル (x86 32bit 用) (See: [here](noVxQtU9lk.html) for details)
hotspot/src/cpu/x86/vm/x86_64.ad                               |  同上 (x86 64bit 用) (See: [here](noVxQtU9lk.html) for details)

### 備考(Notes)
* ※1: 
  cpp の方は #include 以外は何も書いていない. また inline.hpp には型キャストや四則演算などが C で書かれている (C だから share/ 以下でもいいような気もするが...#TODO)

* ※2: 
  どちらも特に何もしないメソッドとして定義されているだけ.

* ※3: 
  hpp の方は #include 以外は何も書いていない.

* ※4:
  どちらも何もしないメソッドとして実装されているだけ.

* ※5: #ifdef CC_INTERP でない時には中身は空になる

* ※6:
  定義されている関数は pd_ps() のみ. これは "print stack" とのことだが, 何もしない関数として実装されているだけ.

* ※7:
  hotspot/src/share/vm/compiler/disassembler.cpp から #include されているので Disassembler 関係だと思われるが, #include 以外は何も書いていない. (#TODO)

* ※8:
  これらは, Disassembler の実装のために libbfd に食わせるオプション等を指定する関数.

* ※9:
  どれも特に何もしない関数として定義されているだけ.

* ※10:
  HotSpot 内で使用するレジスタを RegisterImpl オブジェクトとして宣言しているだけ.

* ※11:
  ただし x86 64bit 版の方は, 内部的に SimpleRuntimeFrame クラスを使用する関係で, 
  現在は sharedRuntime_x86_64.cpp で OptoRuntime::generate_exception_blob() が定義されており, 
  このファイルは空ファイルになっている.

* ※12:
  ただし, SimpleRuntimeFrame クラスは x86_64 版のファイルでのみ定義されている.

* ※13:
  主に VMRegImpl と RegisterImpl の対応付けの設定が行われている (See: RegisterImpl).






