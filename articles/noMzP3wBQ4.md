---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： code/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： code/

--- 

File Name                                          | Description
-------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/vm/code/codeBlob.cpp             | CodeBlob クラス及びそのサブクラスの定義 ([CodeBlob, BufferBlob, AdapterBlob, MethodHandlesAdapterBlob, RuntimeStub, SingletonBlob, RicochetBlob, DeoptimizationBlob, UncommonTrapBlob, ExceptionBlob, SafepointBlob](nod9zY9D-X.html))
hotspot/src/share/vm/code/codeBlob.hpp             | 同上
hotspot/src/share/vm/code/codeCache.cpp            | CodeCache クラスの定義 ([CodeCache, 及びその補助クラス(CodeBlob_sizes)](no3_HYHOCS.html))
hotspot/src/share/vm/code/codeCache.hpp            | 同上
hotspot/src/share/vm/code/compiledIC.cpp           | CompiledIC, CompiledStaticCall クラス関連のクラスの定義 ([CompiledICInfo, CompiledIC, StaticCallInfo, CompiledStaticCall](nokU5y27YJ.html))
hotspot/src/share/vm/code/compiledIC.hpp           | 同上
hotspot/src/share/vm/code/compressedStream.cpp     | CompressedStream クラス関連のクラスの定義 ([CompressedStream, CompressedReadStream, CompressedWriteStream](noy2uEkC8T.html))
hotspot/src/share/vm/code/compressedStream.hpp     | 同上
hotspot/src/share/vm/code/debugInfo.cpp            | デバッグ情報関連のクラスの定義 ([ScopeValue, LocationValue, ObjectValue, ConstantIntValue, ConstantLongValue, ConstantDoubleValue, ConstantOopWriteValue, ConstantOopReadValue, MonitorValue, DebugInfoReadStream, DebugInfoWriteStream](noQmJF4KJf.html))
hotspot/src/share/vm/code/debugInfo.hpp            | 同上
hotspot/src/share/vm/code/debugInfoRec.cpp         | DebugInformationRecorder クラスの定義 ([DebugInformationRecorder, 及びその補助クラス(DIR_Chunk)](noll5bD3QG.html))
hotspot/src/share/vm/code/debugInfoRec.hpp         | 同上
hotspot/src/share/vm/code/dependencies.cpp         | Dependencies クラス関連のクラスの定義 ([Dependencies, Dependencies::DepStream, DepChange, DepChange::ContextStream, 及びそれらの補助クラス(ClassHierarchyWalker)](noqEslgUtE.html))
hotspot/src/share/vm/code/dependencies.hpp         | 同上
hotspot/src/share/vm/code/exceptionHandlerTable.cpp| ExceptionHandlerTable クラス関連のクラスの定義 ([HandlerTableEntry, ExceptionHandlerTable, ImplicitExceptionTable](nolaFex5Dh.html))
hotspot/src/share/vm/code/exceptionHandlerTable.hpp| 同上
hotspot/src/share/vm/code/icBuffer.cpp             | InlineCacheBuffer クラス関連のクラスの定義 ([ICStub, InlineCacheBuffer](noshE-y6pe.html))
hotspot/src/share/vm/code/icBuffer.hpp             | 同上
hotspot/src/share/vm/code/jvmticmlr.h              | JVMTI の CompiledMethodLoad callback 関数に渡されるデータ構造の定義 (jvmtiCMLRKind, jvmtiCompiledMethodLoadRecordHeader, PCStackInfo, jvmtiCompiledMethodLoadInlineRecord, jvmtiCompiledMethodLoadDummyRecord)
hotspot/src/share/vm/code/location.cpp             | Location クラスの定義 ([Location](novNSA7VDa.html))
hotspot/src/share/vm/code/location.hpp             | 同上
hotspot/src/share/vm/code/nmethod.cpp              | nmethod クラス関連のクラスの定義 ([ExceptionCache, PcDescCache, nmethod, nmethodLocker, 及びそれらの補助クラス(DetectScavengeRoot, VerifyOopsClosure, DebugScavengeRoot)](no14LXQg07.html))
hotspot/src/share/vm/code/nmethod.hpp              | 同上
hotspot/src/share/vm/code/oopRecorder.cpp          | OopRecorder クラス及びその補助クラスの定義 ([OopRecorder, OopRecorder::IndexCache](no5lsOrHOX.html))
hotspot/src/share/vm/code/oopRecorder.hpp          | 同上
hotspot/src/share/vm/code/pcDesc.cpp               | PcDesc クラスの定義 ([PcDesc](noH0oeOEYZ.html))
hotspot/src/share/vm/code/pcDesc.hpp               | 同上
hotspot/src/share/vm/code/relocInfo.cpp            | relocInfo クラス関連のクラスの定義 ([relocInfo, *_Relocation(FORWARD_DECLARE_EACH_CLASSで生成されるクラス群), RelocationHolder, RelocIterator, Relocation, DataRelocation, CallRelocation, oop_Relocation, virtual_call_Relocation, opt_virtual_call_Relocation, static_call_Relocation, static_stub_Relocation, runtime_call_Relocation, external_word_Relocation, internal_word_Relocation, section_word_Relocation, poll_Relocation, poll_return_Relocation, breakpoint_Relocation, PatchingRelocIterator](no2935-q0.html))
hotspot/src/share/vm/code/relocInfo.hpp            | 同上
hotspot/src/share/vm/code/scopeDesc.cpp            | ScopeDesc クラス関連のクラスの定義 ([SimpleScopeDesc, ScopeDesc](noJ3FtsV5g.html))
hotspot/src/share/vm/code/scopeDesc.hpp            | 同上
hotspot/src/share/vm/code/stubs.cpp                | Stub クラス関連のクラスの定義 ([Stub, StubInterface, StubQueue](noKr7qYgCo.html))
hotspot/src/share/vm/code/stubs.hpp                | 同上
hotspot/src/share/vm/code/vmreg.cpp                | VMReg クラス関連のクラスの定義 ([VMRegImpl, VMRegPair](no_4UQEKGO.html))
hotspot/src/share/vm/code/vmreg.hpp                | 同上
hotspot/src/share/vm/code/vtableStubs.cpp          | VtableStubs クラス関連のクラスの定義 ([VtableStub, VtableStubs](noC1s4zTwP.html))
hotspot/src/share/vm/code/vtableStubs.hpp          | 同上






