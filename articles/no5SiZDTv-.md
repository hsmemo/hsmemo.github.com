---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： oops/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： oops/

--- 

File Name                                                             | Description
--------------------------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/vm/oops/arrayKlass.cpp                              |  arrayKlass クラスの定義 ([arrayKlass](noI8lr5dVD.html))
hotspot/src/share/vm/oops/arrayKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/arrayKlassKlass.cpp	                      |  arrayKlassKlass クラスの定義 ([arrayKlassKlass](noFEiwh6Ph.html))
hotspot/src/share/vm/oops/arrayKlassKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/arrayOop.cpp		                      |  arrayOopDesc クラスの定義 ([arrayOopDesc](noK0Aem-lI.html)) (※1)
hotspot/src/share/vm/oops/arrayOop.hpp		                      |  同上
hotspot/src/share/vm/oops/compiledICHolderKlass.cpp                   |  compiledICHolderKlass クラスの定義 ([compiledICHolderKlass](noD3aPxnRt.html))
hotspot/src/share/vm/oops/compiledICHolderKlass.hpp                   |  同上
hotspot/src/share/vm/oops/compiledICHolderOop.cpp                     |  compiledICHolderOopDesc クラスの定義 ([compiledICHolderOopDesc](nomT3QvYge.html)) (※1)
hotspot/src/share/vm/oops/compiledICHolderOop.hpp                     |  同上
hotspot/src/share/vm/oops/constMethodKlass.cpp	                      |  constMethodKlass クラスの定義 ([constMethodKlass](nokPjqeORG.html))
hotspot/src/share/vm/oops/constMethodKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/constMethodOop.cpp	                      |  constMethodOop クラス関連のクラスの定義 ([CheckedExceptionElement, LocalVariableTableElement, constMethodOopDesc](nof4G7FEo8.html))
hotspot/src/share/vm/oops/constMethodOop.hpp	                      |  同上
hotspot/src/share/vm/oops/constantPoolKlass.cpp	                      |  constantPoolKlass クラスの定義 ([constantPoolKlass](noStmIC3KC.html))
hotspot/src/share/vm/oops/constantPoolKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/constantPoolOop.cpp	                      |  constantPoolOopDesc クラス関連のクラスの定義 ([CPSlot, constantPoolOopDesc, SymbolHashMapEntry, SymbolHashMapBucket, SymbolHashMap](no6yICYRHh.html))
hotspot/src/share/vm/oops/constantPoolOop.hpp	                      |  同上
hotspot/src/share/vm/oops/cpCacheKlass.cpp	                      |  constantPoolCacheKlass クラスの定義 ([constantPoolCacheKlass](noR1yWkyoI.html))
hotspot/src/share/vm/oops/cpCacheKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/cpCacheOop.cpp	                      |  constantPoolCacheOopDesc クラス関連のクラスの定義 ([ConstantPoolCacheEntry, constantPoolCacheOopDesc, 及びそれらの補助クラス(LocalOopClosure)](nob3d4g7L1.html))
hotspot/src/share/vm/oops/cpCacheOop.hpp	                      |  同上
hotspot/src/share/vm/oops/generateOopMap.cpp	                      |  GenerateOopMap クラス関連のクラスの定義 ([RetTableEntry, RetTable, CellTypeState, BasicBlock, GenerateOopMap, ResolveOopMapConflicts, GeneratePairingInfo, 及びそれらの補助クラス(ComputeCallStack, ComputeEntryStack, RelocCallback)](nofGSzXFYC.html))
hotspot/src/share/vm/oops/generateOopMap.hpp	                      |  同上
hotspot/src/share/vm/oops/instanceKlass.cpp	                      |  instanceKlass クラス関連のクラスの定義 ([FieldClosure, FieldPrinter, OopMapBlock, instanceKlass, JNIid, PreviousVersionNode, PreviousVersionInfo, PreviousVersionWalker, 及びそれらの補助クラス(nmethodBucket, VerifyFieldClosure)](no7KYSWR9-.html))
hotspot/src/share/vm/oops/instanceKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/instanceKlassKlass.cpp                      |  instanceKlassKlass クラスの定義 ([instanceKlassKlass, 及びその補助クラス(VerifyFieldClosure)](nozGHKtKeh.html))
hotspot/src/share/vm/oops/instanceKlassKlass.hpp                      |  同上
hotspot/src/share/vm/oops/instanceMirrorKlass.cpp                     |  instanceMirrorKlass クラスの定義 ([instanceMirrorKlass](nohe7A_jzb.html))
hotspot/src/share/vm/oops/instanceMirrorKlass.hpp                     |  同上
hotspot/src/share/vm/oops/instanceOop.cpp	                      |  instanceOopDesc クラスの定義 ([instanceOopDesc](noJCU5mbFR.html)) (※1)
hotspot/src/share/vm/oops/instanceOop.hpp	                      |  同上
hotspot/src/share/vm/oops/instanceRefKlass.cpp	                      |  instanceRefKlass クラスの定義 ([instanceRefKlass](nozR2kOP8x.html))
hotspot/src/share/vm/oops/instanceRefKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/klass.cpp		                      |  Klass クラス関連のクラスの定義 ([Klass_vtbl, Klass](noTHKPedbg.html))
hotspot/src/share/vm/oops/klass.hpp		                      |  同上
hotspot/src/share/vm/oops/klass.inline.hpp	                      |  同上
hotspot/src/share/vm/oops/klassKlass.cpp	                      |  klassKlass クラスの定義 ([klassKlass](no2DOxR7-K.html))
hotspot/src/share/vm/oops/klassKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/klassOop.cpp		                      |  klassOopDesc クラスの定義 ([klassOopDesc](noY7Uk_TlO.html)) (※1)
hotspot/src/share/vm/oops/klassOop.hpp		                      |  同上
hotspot/src/share/vm/oops/klassPS.hpp		                      |  Klass の "Parallel Scavenge" 及び "Parallel Compaction" 用のメソッドの型宣言 (※3)
hotspot/src/share/vm/oops/klassVtable.cpp	                      |  instanceKlass の vtable/itable 関連のクラスの定義 ([klassVtable, vtableEntry, itableOffsetEntry, itableMethodEntry, klassItable, 及びそれらの補助クラス(InterfaceVisiterClosure, CountInterfacesClosure, SetupItableClosure, VtableStats)](nok3cQYg2G.html))
hotspot/src/share/vm/oops/klassVtable.hpp	                      |  同上
hotspot/src/share/vm/oops/markOop.cpp		                      |  markOopDesc クラスの定義 ([markOopDesc](no_1nXJzmX.html))
hotspot/src/share/vm/oops/markOop.hpp		                      |  同上
hotspot/src/share/vm/oops/markOop.inline.hpp	                      |  同上
hotspot/src/share/vm/oops/methodDataKlass.cpp	                      |  methodDataKlass クラスの定義 ([methodDataKlass](no7OygL-Z0.html))
hotspot/src/share/vm/oops/methodDataKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/methodDataOop.cpp	                      |  methodDataOopDesc クラス関連のクラスの定義 ([DataLayout, ProfileData, BitData, CounterData, JumpData, ReceiverTypeData, VirtualCallData, RetData, BranchData, ArrayData, MultiBranchData, ArgInfoData, methodDataOopDesc](no2565EcHt.html))
hotspot/src/share/vm/oops/methodDataOop.hpp	                      |  同上
hotspot/src/share/vm/oops/methodKlass.cpp	                      |  methodKlass クラスの定義 ([methodKlass](noGJ-d0Y8y.html))
hotspot/src/share/vm/oops/methodKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/methodOop.cpp		                      |  methodOopDesc クラス関連のクラスの定義 ([methodOopDesc, CompressedLineNumberWriteStream, CompressedLineNumberReadStream, BreakpointInfo, 及びそれらの補助クラス(SignatureTypePrinter)](nog4446PhV.html))
hotspot/src/share/vm/oops/methodOop.hpp		                      |  同上
hotspot/src/share/vm/oops/objArrayKlass.cpp	                      |  objArrayKlass クラスの定義 ([objArrayKlass](noLWt_VM7T.html))
hotspot/src/share/vm/oops/objArrayKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/objArrayKlass.inline.hpp                    |  同上
hotspot/src/share/vm/oops/objArrayKlassKlass.cpp                      |  objArrayKlassKlass クラスの定義 ([objArrayKlassKlass](noKz6YzV86.html))
hotspot/src/share/vm/oops/objArrayKlassKlass.hpp                      |  同上
hotspot/src/share/vm/oops/objArrayOop.cpp	                      |  objArrayOopDesc クラスの定義 ([objArrayOopDesc](noPE7CamOr.html))
hotspot/src/share/vm/oops/objArrayOop.hpp	                      |  同上
hotspot/src/share/vm/oops/oop.cpp		                      |  oopDesc クラスの定義 ([oopDesc](noeDD7wSAk.html))
hotspot/src/share/vm/oops/oop.hpp		                      |  同上
hotspot/src/share/vm/oops/oop.inline.hpp	                      |  同上 (※2)
hotspot/src/share/vm/oops/oop.inline2.hpp	                      |  同上 (※2)
hotspot/src/share/vm/oops/oop.pcgc.inline.hpp	                      |  oopDesc の Parallel Compaction GC 用のメソッド定義
hotspot/src/share/vm/oops/oop.psgc.inline.hpp	                      |  oopDesc の Parallel Scavenge GC 用のメソッド定義
hotspot/src/share/vm/oops/oopsHierarchy.cpp	                      |  oop の定義 ([oop, instanceOop, methodOop, constMethodOop, methodDataOop, arrayOop, objArrayOop, typeArrayOop, constantPoolOop, constantPoolCacheOop, klassOop, markOop, compiledICHolderOop](nogixYkOS7.html))
hotspot/src/share/vm/oops/oopsHierarchy.hpp	                      |  同上
hotspot/src/share/vm/oops/symbol.cpp		                      |  Symbol クラスの定義 ([Symbol](nowAluQvw3.html))
hotspot/src/share/vm/oops/symbol.hpp		                      |  同上
hotspot/src/share/vm/oops/typeArrayKlass.cpp	                      |  typeArrayKlass クラスの定義 ([typeArrayKlass](noJDAZpAgo.html))
hotspot/src/share/vm/oops/typeArrayKlass.hpp	                      |  同上
hotspot/src/share/vm/oops/typeArrayKlassKlass.cpp                     |  typeArrayKlassKlass クラスの定義 ([typeArrayKlassKlass](nokl5sGzSy.html))
hotspot/src/share/vm/oops/typeArrayKlassKlass.hpp                     |  同上
hotspot/src/share/vm/oops/typeArrayOop.cpp	                      |  typeArrayOopDesc クラスの定義 ([typeArrayOopDesc](noOkLq3zse.html)) (※1)
hotspot/src/share/vm/oops/typeArrayOop.hpp	                      |  同上

### 備考(Notes)
* ※1:
  cpp の方は #include 以外は何も書いていない
* ※2:
  依存関係が再帰するのを避けるため, oop.hpp とは別ファイルとしているとのこと
* ※3:
  実装は hotspot/src/share/vm/oops/ の各Klass定義の中(*Klass.cppファイル)で行っている模様.






