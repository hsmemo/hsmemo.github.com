---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： interpreter/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： interpreter/

--- 

File Name                                                             | Description
--------------------------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/vm/interpreter/abstractInterpreter.hpp              |  AbstractInterpreter クラスおよび AbstractInterpreterGenerator クラスの定義 ([AbstractInterpreter, AbstractInterpreterGenerator](nogrGzFsk-.html))
hotspot/src/share/vm/interpreter/bytecode.cpp			      |  Bytecode クラス関連のクラスの定義 ([Bytecode, LookupswitchPair, Bytecode_lookupswitch, Bytecode_tableswitch, Bytecode_member_ref, Bytecode_invoke, Bytecode_field, Bytecode_checkcast, Bytecode_instanceof, Bytecode_new, Bytecode_multianewarray, Bytecode_anewarray, Bytecode_loadconstant](no6LMKI1_f.html))
hotspot/src/share/vm/interpreter/bytecode.hpp			      |  同上
hotspot/src/share/vm/interpreter/bytecodeHistogram.cpp		      |  BytecodeHistogram クラス関連のクラスの定義 ([BytecodeCounter, BytecodeHistogram, BytecodePairHistogram, 及びそれらの補助クラス(HistoEntry)](noyRrXp8Ha.html))
hotspot/src/share/vm/interpreter/bytecodeHistogram.hpp		      |  同上
hotspot/src/share/vm/interpreter/bytecodeInterpreter.cpp	      |  BytecodeInterpreter クラスの定義 ([BytecodeInterpreter](noNNHB5McV.html)) (※1)
hotspot/src/share/vm/interpreter/bytecodeInterpreter.hpp	      |  同上
hotspot/src/share/vm/interpreter/bytecodeInterpreter.inline.hpp	      |  同上
hotspot/src/share/vm/interpreter/bytecodeInterpreterWithChecks.xml    |  JVMTI 用に bytecodeInterpreterWithChecks.cpp を生成するためのXML/XSLTファイル (See: hotspot/make/${os}/makefiles/jvmti.make) (※3)
hotspot/src/share/vm/interpreter/bytecodeInterpreterWithChecks.xsl    |  同上
hotspot/src/share/vm/interpreter/bytecodeStream.cpp		      |  BytecodeStream クラス関連のクラスの定義 ([BaseBytecodeStream, RawBytecodeStream, BytecodeStream](norXbVk_e6.html))
hotspot/src/share/vm/interpreter/bytecodeStream.hpp		      |  同上
hotspot/src/share/vm/interpreter/bytecodeTracer.cpp		      |  BytecodeTracer クラス関連のクラスの定義 ([BytecodeTracer, BytecodeClosure, 及びそれらの補助クラス(BytecodePrinter)](noQn1XVNm9.html))
hotspot/src/share/vm/interpreter/bytecodeTracer.hpp		      |  同上
hotspot/src/share/vm/interpreter/bytecodes.cpp			      |  Bytecodes クラスの定義 ([Bytecodes](noH27yEtWY.html))
hotspot/src/share/vm/interpreter/bytecodes.hpp			      |  同上
hotspot/src/share/vm/interpreter/cppInterpreter.cpp		      |  CppInterpreter クラスの定義 ([CppInterpreter](nomcEebD7I.html)) (※1)
hotspot/src/share/vm/interpreter/cppInterpreter.hpp		      |  同上
hotspot/src/share/vm/interpreter/cppInterpreterGenerator.hpp	      |  CppInterpreterGenerator クラスの定義 ([CppInterpreterGenerator](no-DGUzaqt.html)) (※1)
hotspot/src/share/vm/interpreter/interpreter.cpp		      |  Interpreter クラス関連のクラスの定義 ([InterpreterCodelet, CodeletMark, Interpreter](noNIwNW8-r.html))
hotspot/src/share/vm/interpreter/interpreter.hpp		      |  同上
hotspot/src/share/vm/interpreter/interpreterGenerator.hpp	      |  InterpreterGenerator クラスの定義 ([InterpreterGenerator](normpdoQ4G.html))
hotspot/src/share/vm/interpreter/interpreterRuntime.cpp		      |  InterpreterRuntime クラス関連のクラスの定義 ([InterpreterRuntime, SignatureHandlerLibrary, 及びそれらの補助クラス(UnlockFlagSaver)](noWUOvGRuf.html))
hotspot/src/share/vm/interpreter/interpreterRuntime.hpp		      |  同上
hotspot/src/share/vm/interpreter/invocationCounter.cpp		      |  InvocationCounter クラスの定義 ([InvocationCounter](noAWg22n6P.html))
hotspot/src/share/vm/interpreter/invocationCounter.hpp		      |  同上
hotspot/src/share/vm/interpreter/linkResolver.cpp		      |  LinkResolver クラス関連のクラスの定義 ([LinkInfo, FieldAccessInfo, CallInfo, LinkResolver](no2eAG3NwW.html))
hotspot/src/share/vm/interpreter/linkResolver.hpp		      |  同上
hotspot/src/share/vm/interpreter/oopMapCache.cpp		      |  OopMapCache クラス関連のクラスの定義 ([OffsetClosure, InterpreterOopMap, OopMapCache, 及びそれらの補助クラス(OopMapCacheEntry, OopMapForCacheEntry, VerifyClosure, MaskFillerForNative)](nojjFYgcwR.html))
hotspot/src/share/vm/interpreter/oopMapCache.hpp		      |  同上
hotspot/src/share/vm/interpreter/rewriter.cpp			      |  Rewriter クラスの定義 ([Rewriter](noHQhpwzRf.html))
hotspot/src/share/vm/interpreter/rewriter.hpp			      |  同上
hotspot/src/share/vm/interpreter/templateInterpreter.cpp	      |  TemplateInterpreter クラス関連のクラスの定義 ([EntryPoint, DispatchTable, TemplateInterpreter](nolL3dNvMR.html)) (※2) (なお, cpp の方には TemplateInterpreterGenerator のメソッド定義も含まれている)
hotspot/src/share/vm/interpreter/templateInterpreter.hpp	      |  同上
hotspot/src/share/vm/interpreter/templateInterpreterGenerator.hpp     |  TemplateInterpreterGenerator クラスの宣言 ([TemplateInterpreterGenerator](noSBSGKGcw.html)) (※2)
hotspot/src/share/vm/interpreter/templateTable.cpp		      |  TemplateTable クラス関連のクラスの定義 ([Template, TemplateTable](noqIh7H8Vi.html)) (※2)
hotspot/src/share/vm/interpreter/templateTable.hpp		      |  同上

### 備考(Notes)
* ※1: #ifdef CC_INTERP でない時には中身は空になる
* ※2: #ifndef CC_INTERP でない時には中身は空になる
* ※3: ただし中身はほとんど空に近く, #define VM_JVMTI と #include "interpreter/bytecodeInterpreter.cpp" という2行が生成されるのみ.







