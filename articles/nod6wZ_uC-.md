---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： memory/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： memory/

--- 

File Name                                                             | Description
--------------------------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/vm/memory/allocation.cpp                            |  HotSpot を構成するクラスの基底クラス群, 及びそれらのメモリ管理用クラスの定義 ([AllocatedObj(ALLOCATION_SUPER_CLASS_SPEC), CHeapObj, StackObj, _ValueObj, AllStatic, Chunk, Arena, ResourceObj, AllocStats, ReallocMark, 及びそれらの補助クラス(ChunkPool, ChunkPoolCleaner)](nosjRDBXzA.html))
hotspot/src/share/vm/memory/allocation.hpp                            |  同上
hotspot/src/share/vm/memory/allocation.inline.hpp                     |  同上
hotspot/src/share/vm/memory/barrierSet.cpp                            |  BarrierSet クラスの定義 ([BarrierSet](no_7YU6nOu.html))
hotspot/src/share/vm/memory/barrierSet.hpp                            |  同上
hotspot/src/share/vm/memory/barrierSet.inline.hpp                     |  同上
hotspot/src/share/vm/memory/blockOffsetTable.cpp                      |  BlockOffsetTable クラス関連のクラスの定義 ([BlockOffsetTable, BlockOffsetSharedArray, BlockOffsetArray, BlockOffsetArrayNonContigSpace, BlockOffsetArrayContigSpace](noZvNdRGYi.html))
hotspot/src/share/vm/memory/blockOffsetTable.hpp                      |  同上
hotspot/src/share/vm/memory/blockOffsetTable.inline.hpp               |  同上
hotspot/src/share/vm/memory/cardTableModRefBS.cpp                     |  CardTableModRefBS クラス関連のクラスの定義 ([CardTableModRefBS, CardTableModRefBSForCTRS](noFHryn0Ku.html))
hotspot/src/share/vm/memory/cardTableModRefBS.hpp                     |  同上
hotspot/src/share/vm/memory/cardTableRS.cpp                           |  CardTableRS クラス関連のクラスの定義 ([CardTableRS, ClearNoncleanCardWrapper, 及びそれらの補助クラス(VerifyCleanCardClosure, VerifyCTSpaceClosure, VerifyCTGenClosure)](noAv2BI8fQ.html))
hotspot/src/share/vm/memory/cardTableRS.hpp                           |  同上
hotspot/src/share/vm/memory/classify.cpp                              |  Class Data Sharing (CDS) の統計情報出力のための補助クラスの定義 ([ClassifyObjectClosure, ClassifyInstanceKlassClosure, ClearAllocCountClosure](nokL3GJEf3.html))
hotspot/src/share/vm/memory/classify.hpp                              |  同上
hotspot/src/share/vm/memory/collectorPolicy.cpp                       |  CollectorPolicy クラス関連のクラスの定義 ([CollectorPolicy, ClearedAllSoftRefs, GenCollectorPolicy, TwoGenerationCollectorPolicy, MarkSweepPolicy](now_5MkHd7.html))
hotspot/src/share/vm/memory/collectorPolicy.hpp                       |  同上
hotspot/src/share/vm/memory/compactPermGen.hpp                        |  CompactingPermGen クラスの定義 ([CompactingPermGen](noTFajniew.html))
hotspot/src/share/vm/memory/compactingPermGenGen.cpp                  |  CompactingPermGenGen クラスの定義 ([CompactingPermGenGen, 及びその補助クラス(AdjustSharedObjectClosure, RecursiveAdjustSharedObjectClosure, TraversePlaceholdersClosure, VerifyMarksClearedClosure)](noEPvINBhn.html))
hotspot/src/share/vm/memory/compactingPermGenGen.hpp                  |  同上
hotspot/src/share/vm/memory/defNewGeneration.cpp                      |  DefNewGeneration クラスの定義 ([DefNewGeneration, 及びその補助クラス(DefNewGeneration::IsAliveClosure, DefNewGeneration::KeepAliveClosure, DefNewGeneration::FastKeepAliveClosure, DefNewGeneration::EvacuateFollowersClosure, DefNewGeneration::FastEvacuateFollowersClosure,  RemoveForwardPointerClosure)](nomFbG9rH9.html))
hotspot/src/share/vm/memory/defNewGeneration.hpp                      |  同上
hotspot/src/share/vm/memory/defNewGeneration.inline.hpp               |  同上
hotspot/src/share/vm/memory/dump.cpp                                  |    (#TODO) Class Data Sharing (CDS) のダンプ出力用メソッド(GenCollectedHeap::preload_and_dump())の定義およびその補助クラスの定義 ([FingerprintMethodsClosure, StringHashCodeClosure, RemoveUnshareableInfoClosure, MoveSymbols, MarkObjectsOopClosure, MarkObjectsSkippingKlassesOopClosure, MarkCommonReadOnly, CommonSymbolsClosure, MarkStringValues, CheckRemainingObjects, MarkReadWriteObjects, MarkStringObjects, MoveMarkedObjects, MarkAndMoveOrderedReadOnly, MarkAndMoveOrderedReadWrite, ResolveForwardingClosure, SortMethodsClosure, ReinitializeTables, PatchOopsClosure, ClearSpaceClosure, WriteClosure, ResolveConstantPoolsClosure, PatchKlassVtables, PatchSymbolVtables, VM_PopulateDumpSharedSpace, LinkClassesClosure](noi9m5MKFL.html))
hotspot/src/share/vm/memory/filemap.cpp                               |  FileMapInfo クラスの定義 ([FileMapInfo](noFtPcj8Rn.html))
hotspot/src/share/vm/memory/filemap.hpp                               |  同上
hotspot/src/share/vm/memory/gcLocker.cpp                              |  GC_locker クラス関連のクラスの定義 ([GC_locker, No_GC_Verifier, Pause_No_GC_Verifier, No_Safepoint_Verifier, Pause_No_Safepoint_Verifier, SkipGCALot, JRT_Leaf_Verifier, No_Alloc_Verifier](no4-wW9-3x.html))
hotspot/src/share/vm/memory/gcLocker.hpp                              |  同上
hotspot/src/share/vm/memory/gcLocker.inline.hpp                       |  同上
hotspot/src/share/vm/memory/genCollectedHeap.cpp                      |  GenCollectedHeap クラス関連のクラスの定義 ([GenCollectedHeap, GenCollectedHeap::GenClosure, 及びそれらの補助クラス(GenPrepareForVerifyClosure, GenGCPrologueClosure, GenGCEpilogueClosure, GenGCSaveTopsBeforeGCClosure, GenEnsureParsabilityClosure, GenTimeOfLastGCClosure)](noRLJGPwet.html))
hotspot/src/share/vm/memory/genCollectedHeap.hpp                      |  同上
hotspot/src/share/vm/memory/genMarkSweep.cpp                          |  GenMarkSweep クラスの定義 ([GenMarkSweep, 及びその補助クラス(GenAdjustPointersClosure, GenCompactClosure)](no65WQPG61.html))
hotspot/src/share/vm/memory/genMarkSweep.hpp                          |  同上
hotspot/src/share/vm/memory/genOopClosures.hpp                        |  OopClosure の様々なサブクラスの定義 ([OopsInGenClosure, ScanClosure, FastScanClosure, FilteringClosure, ScanWeakRefClosure, VerifyOopClosure](no1RjdFgkK.html))
hotspot/src/share/vm/memory/genOopClosures.inline.hpp                 |  同上
hotspot/src/share/vm/memory/genRemSet.cpp                             |  GenRemSet クラスの定義 ([GenRemSet](noe5AOWcIW.html))
hotspot/src/share/vm/memory/genRemSet.hpp                             |  同上
hotspot/src/share/vm/memory/genRemSet.inline.hpp                      |  同上
hotspot/src/share/vm/memory/generation.cpp                            |  Generation クラス関連のクラスの定義 ([Generation, CardGeneration, OneContigSpaceCardGeneration, 及びそれらの補助クラス(GenerationIsInReservedClosure, GenerationIsInClosure, GenerationBlockStartClosure, GenerationBlockSizeClosure, GenerationBlockIsObjClosure, GenerationOopIterateClosure, GenerationObjIterateClosure, GenerationSafeObjIterateClosure, AdjustPointersClosure)](noYh_WbceH.html))
hotspot/src/share/vm/memory/generation.hpp                            |  同上
hotspot/src/share/vm/memory/generation.inline.hpp                     |  同上
hotspot/src/share/vm/memory/generationSpec.cpp                        |  GenerationSpec クラス関連のクラスの定義 ([GenerationSpec, PermanentGenerationSpec](no5v3uwSFN.html))
hotspot/src/share/vm/memory/generationSpec.hpp                        |  同上
hotspot/src/share/vm/memory/heap.cpp                                  |  CodeHeap クラス及びその補助クラスの定義 ([HeapBlock, FreeBlock, CodeHeap](noh-LTrtBG.html))
hotspot/src/share/vm/memory/heap.hpp                                  |  同上
hotspot/src/share/vm/memory/heapInspection.cpp                        |  HeapInspection クラス及びその補助クラスの定義 ([KlassInfoEntry, KlassInfoClosure, KlassInfoBucket, KlassInfoTable, KlassInfoHisto, HeapInspection, 及びそれらの補助クラス(HistoClosure, RecordInstanceClosure, FindInstanceClosure)](noosMmbEKE.html))
hotspot/src/share/vm/memory/heapInspection.hpp                        |  同上
hotspot/src/share/vm/memory/iterator.cpp                              |  Closure クラス関連のクラスの定義 ([Closure, OopClosure, ObjectClosure, BoolObjectClosure, ObjectToOopClosure, UpwardsObjectClosure, ObjectClosureCareful, BlkClosure, BlkClosureCareful, SpaceClosure, CompactibleSpaceClosure, CodeBlobClosure, MarkingCodeBlobClosure, MarkingCodeBlobClosure::MarkScope, CodeBlobToOopClosure, MonitorClosure, VoidClosure, YieldClosure, SerializeOopClosure, SymbolClosure, RememberKlassesChecker](noxdFDDXjm.html))
hotspot/src/share/vm/memory/iterator.hpp                              |  同上
hotspot/src/share/vm/memory/memRegion.cpp                             |  MemRegion クラス関連のクラスの定義 ([MemRegion, MemRegionClosure, MemRegionClosureRO](now2R1NnBJ.html))
hotspot/src/share/vm/memory/memRegion.hpp                             |  同上
hotspot/src/share/vm/memory/modRefBarrierSet.hpp                      |  ModRefBarrierSet クラスの定義 ([ModRefBarrierSet](no8g6eGLgI.html))
hotspot/src/share/vm/memory/oopFactory.cpp                            |  oopFactory クラスの定義 ([oopFactory](noMJQ3M-Uv.html))
hotspot/src/share/vm/memory/oopFactory.hpp                            |  同上
hotspot/src/share/vm/memory/permGen.cpp                               |  PermGen クラスの定義 ([PermGen](nodiTBTDNm.html)) (注: cpp には CompactingPermGen クラスのメソッド定義を一部含む)
hotspot/src/share/vm/memory/permGen.hpp                               |  同上
hotspot/src/share/vm/memory/referencePolicy.cpp                       |  ReferencePolicy クラス関連のクラスの定義 ([ReferencePolicy, NeverClearPolicy, AlwaysClearPolicy, LRUCurrentHeapPolicy, LRUMaxHeapPolicy](noz-glFiSM.html))
hotspot/src/share/vm/memory/referencePolicy.hpp                       |  同上
hotspot/src/share/vm/memory/referenceProcessor.cpp                    |  ReferenceProcessor クラス関連のクラスの定義 ([ReferenceProcessor, NoRefDiscovery, ReferenceProcessorSpanMutator, ReferenceProcessorMTDiscoveryMutator, ReferenceProcessorIsAliveMutator, ReferenceProcessorAtomicMutator, ReferenceProcessorMTProcMutator, AbstractRefProcTaskExecutor, AbstractRefProcTaskExecutor::ProcessTask, AbstractRefProcTaskExecutor::EnqueueTask, 及びそれらの補助クラス(DiscoveredList, AlwaysAliveClosure, CountHandleClosure, RefProcEnqueueTask, DiscoveredListIterator, RefProcPhase1Task, RefProcPhase2Task, RefProcPhase3Task)](no6c8mMCUW.html))
hotspot/src/share/vm/memory/referenceProcessor.hpp                    |  同上
hotspot/src/share/vm/memory/resourceArea.cpp                          |  ResourceArea クラス関連のクラスの定義 ([ResourceArea, ResourceMark, DeoptResourceMark](nojwDN3-FN.html))
hotspot/src/share/vm/memory/resourceArea.hpp                          |  同上
hotspot/src/share/vm/memory/restore.cpp                               |    (#TODO) CompactingPermGenGen クラスの restore 処理に関するメソッド(CompactingPermGenGen::initialize_oops())の定義およびその補助クラスの定義([ReadClosure](noihDtb1tt.html)) (<= UseSharedSpaces 有効時の処理を実装している模様. CDS のためのメモリダンプを読む処理??)
hotspot/src/share/vm/memory/serialize.cpp                             |    (#TODO) CompactingPermGenGen クラスの serializing 処理に関するメソッドの定義 (CompactingPermGenGen::serialize_bts(), CompactingPermGenGen::serialize_oops()) (<= CDS のためのメモリダンプを出す処理??)
hotspot/src/share/vm/memory/sharedHeap.cpp                            |  SharedHeap クラス関連のクラスの定義 ([SharedHeap, SharedHeap::StrongRootsScope, 及びそれらの補助クラス(AssertIsPermClosure, AssertNonScavengableClosure, AlwaysTrueClosure, SkipAdjustingSharedStrings)](noysUrzWdK.html))
hotspot/src/share/vm/memory/sharedHeap.hpp                            |  同上
hotspot/src/share/vm/memory/space.cpp                                 |  Space クラス関連のクラスの定義 ([SpaceMemRegionOopsIterClosure, Space, DirtyCardToOopClosure, CompactPoint, CompactibleSpace, ContiguousSpace, Filtering_DCTOC, ContiguousSpaceDCTOC, EdenSpace, ConcEdenSpace, OffsetTableContigSpace, TenuredSpace, ContigPermSpace, 及びそれらの補助クラス(VerifyOldOopClosure)](noEwj5VK63.html))
hotspot/src/share/vm/memory/space.hpp                                 |  同上
hotspot/src/share/vm/memory/space.inline.hpp                          |  同上
hotspot/src/share/vm/memory/specialized_oop_closures.cpp              |  特定の OopClosure に特化した oop_oop_iterate() メソッドを停止するためのマクロ集の定義、及びその補助クラスの定義 ([SpecializationStats](noXUdVICpE.html))
hotspot/src/share/vm/memory/specialized_oop_closures.hpp              |  同上
hotspot/src/share/vm/memory/tenuredGeneration.cpp                     |  TenuredGeneration クラスの定義 ([TenuredGeneration](noF3DfmKtp.html))
hotspot/src/share/vm/memory/tenuredGeneration.hpp                     |  同上
hotspot/src/share/vm/memory/threadLocalAllocBuffer.cpp                |  ThreadLocalAllocBuffer クラス関連のクラスの定義 ([ThreadLocalAllocBuffer, GlobalTLABStats](no-MX5_-HV.html))
hotspot/src/share/vm/memory/threadLocalAllocBuffer.hpp                |  同上
hotspot/src/share/vm/memory/threadLocalAllocBuffer.inline.hpp         |  同上
hotspot/src/share/vm/memory/universe.cpp                              |  Universe クラス関連のクラスの定義 ([CommonMethodOopCache, ActiveMethodOopsCache, LatestMethodOopCache, Universe, DeferredObjAllocEvent, 及びそれらの補助クラス(FixupMirrorClosure)](norWgXjc-w.html))
hotspot/src/share/vm/memory/universe.hpp                              |  同上
hotspot/src/share/vm/memory/universe.inline.hpp                       |  同上
hotspot/src/share/vm/memory/watermark.hpp                             |  WaterMark クラスの定義 ([WaterMark](noenM3nIXR.html))







