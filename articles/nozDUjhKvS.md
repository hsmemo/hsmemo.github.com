---
layout: default
title: (#WIP) HotSpot の内部で使われるデータ構造 ： 詳細 ： memory/ 編
---
[Up](nolpd4szt5.html) [Top](../index.html)

#### (#WIP) HotSpot の内部で使われるデータ構造 ： 詳細 ： memory/ 編

--- 
#Under Construction


* [HotSpot を構成するクラスの基底クラス群, 及びそれらのメモリ管理用クラス (AllocatedObj(ALLOCATION_SUPER_CLASS_SPEC), CHeapObj, StackObj, _ValueObj, AllStatic, Chunk, Arena, ResourceObj, AllocStats, ReallocMark, 及びそれらの補助クラス(ChunkPool, ChunkPoolCleaner))](nosjRDBXzA.html)
* [BarrierSet クラス ](no_7YU6nOu.html)
* [BlockOffsetTable クラス関連のクラス (BlockOffsetTable, BlockOffsetSharedArray, BlockOffsetArray, BlockOffsetArrayNonContigSpace, BlockOffsetArrayContigSpace)](noZvNdRGYi.html)
* [CardTableModRefBS クラス関連のクラス (CardTableModRefBS, CardTableModRefBSForCTRS)](noFHryn0Ku.html)
* [CardTableRS クラス関連のクラス (CardTableRS, ClearNoncleanCardWrapper, 及びそれらの補助クラス(VerifyCleanCardClosure, VerifyCTSpaceClosure, VerifyCTGenClosure))](noAv2BI8fQ.html)
* [Class Data Sharing (CDS) の統計情報出力のための補助クラス (ClassifyObjectClosure, ClassifyInstanceKlassClosure, ClearAllocCountClosure)](nokL3GJEf3.html)
* [CollectorPolicy クラス関連のクラス (CollectorPolicy, ClearedAllSoftRefs, GenCollectorPolicy, TwoGenerationCollectorPolicy, MarkSweepPolicy)](now_5MkHd7.html)
* [CompactingPermGen クラス ](noTFajniew.html)
* [CompactingPermGenGen クラス (CompactingPermGenGen, 及びその補助クラス(AdjustSharedObjectClosure, RecursiveAdjustSharedObjectClosure, TraversePlaceholdersClosure, VerifyMarksClearedClosure))](noEPvINBhn.html)
* [DefNewGeneration クラス (DefNewGeneration, 及びその補助クラス(DefNewGeneration::IsAliveClosure, DefNewGeneration::KeepAliveClosure, DefNewGeneration::FastKeepAliveClosure, DefNewGeneration::EvacuateFollowersClosure, DefNewGeneration::FastEvacuateFollowersClosure,  RemoveForwardPointerClosure))](nomFbG9rH9.html)
* [Class Data Sharing (CDS) のダンプ出力用メソッド(GenCollectedHeap::preload_and_dump())の補助クラス (FingerprintMethodsClosure, StringHashCodeClosure, RemoveUnshareableInfoClosure, MoveSymbols, MarkObjectsOopClosure, MarkObjectsSkippingKlassesOopClosure, MarkCommonReadOnly, CommonSymbolsClosure, MarkStringValues, CheckRemainingObjects, MarkReadWriteObjects, MarkStringObjects, MoveMarkedObjects, MarkAndMoveOrderedReadOnly, MarkAndMoveOrderedReadWrite, ResolveForwardingClosure, SortMethodsClosure, ReinitializeTables, PatchOopsClosure, ClearSpaceClosure, WriteClosure, ResolveConstantPoolsClosure, PatchKlassVtables, PatchSymbolVtables, VM_PopulateDumpSharedSpace, LinkClassesClosure)](noi9m5MKFL.html)
* [FileMapInfo クラス ](noFtPcj8Rn.html)
* [GC_locker クラス関連のクラス (GC_locker, No_GC_Verifier, Pause_No_GC_Verifier, No_Safepoint_Verifier, Pause_No_Safepoint_Verifier, SkipGCALot, JRT_Leaf_Verifier, No_Alloc_Verifier)](no4-wW9-3x.html)
* [GenCollectedHeap クラス関連のクラス (GenCollectedHeap, GenCollectedHeap::GenClosure, 及びそれらの補助クラス(GenPrepareForVerifyClosure, GenGCPrologueClosure, GenGCEpilogueClosure, GenGCSaveTopsBeforeGCClosure, GenEnsureParsabilityClosure, GenTimeOfLastGCClosure))](noRLJGPwet.html)
* [GenMarkSweep クラス (GenMarkSweep, 及びその補助クラス(GenAdjustPointersClosure, GenCompactClosure))](no65WQPG61.html)
* [OopClosure の様々なサブクラス (OopsInGenClosure, ScanClosure, FastScanClosure, FilteringClosure, ScanWeakRefClosure, VerifyOopClosure)](no1RjdFgkK.html)
* [GenRemSet クラス ](noe5AOWcIW.html)
* [Generation クラス関連のクラス (Generation, CardGeneration, OneContigSpaceCardGeneration, 及びそれらの補助クラス(GenerationIsInReservedClosure, GenerationIsInClosure, GenerationBlockStartClosure, GenerationBlockSizeClosure, GenerationBlockIsObjClosure, GenerationOopIterateClosure, GenerationObjIterateClosure, GenerationSafeObjIterateClosure, AdjustPointersClosure))](noYh_WbceH.html)
* [GenerationSpec クラス関連のクラス (GenerationSpec, PermanentGenerationSpec)](no5v3uwSFN.html)
* [CodeHeap クラス及びその補助クラス (HeapBlock, FreeBlock, CodeHeap)](noh-LTrtBG.html)
* [HeapInspection クラス及びその補助クラス (KlassInfoEntry, KlassInfoClosure, KlassInfoBucket, KlassInfoTable, KlassInfoHisto, HeapInspection, 及びそれらの補助クラス(HistoClosure, RecordInstanceClosure, FindInstanceClosure))](noosMmbEKE.html)
* [Closure クラス関連のクラス (Closure, OopClosure, ObjectClosure, BoolObjectClosure, ObjectToOopClosure, UpwardsObjectClosure, ObjectClosureCareful, BlkClosure, BlkClosureCareful, SpaceClosure, CompactibleSpaceClosure, CodeBlobClosure, MarkingCodeBlobClosure, MarkingCodeBlobClosure::MarkScope, CodeBlobToOopClosure, MonitorClosure, VoidClosure, YieldClosure, SerializeOopClosure, SymbolClosure, RememberKlassesChecker)](noxdFDDXjm.html)
* [MemRegion クラス関連のクラス (MemRegion, MemRegionClosure, MemRegionClosureRO)](now2R1NnBJ.html)
* [ModRefBarrierSet クラス ](no8g6eGLgI.html)
* [oopFactory クラス ](noMJQ3M-Uv.html)
* [PermGen クラス ](nodiTBTDNm.html)
* [ReferencePolicy クラス関連のクラス (ReferencePolicy, NeverClearPolicy, AlwaysClearPolicy, LRUCurrentHeapPolicy, LRUMaxHeapPolicy)](noz-glFiSM.html)
* [ReferenceProcessor クラス関連のクラス (ReferenceProcessor, NoRefDiscovery, ReferenceProcessorSpanMutator, ReferenceProcessorMTDiscoveryMutator, ReferenceProcessorIsAliveMutator, ReferenceProcessorAtomicMutator, ReferenceProcessorMTProcMutator, AbstractRefProcTaskExecutor, AbstractRefProcTaskExecutor::ProcessTask, AbstractRefProcTaskExecutor::EnqueueTask, 及びそれらの補助クラス(DiscoveredList, AlwaysAliveClosure, CountHandleClosure, RefProcEnqueueTask, DiscoveredListIterator, RefProcPhase1Task, RefProcPhase2Task, RefProcPhase3Task))](no6c8mMCUW.html)
* [ResourceArea クラス関連のクラス (ResourceArea, ResourceMark, DeoptResourceMark)](nojwDN3-FN.html)
* [CompactingPermGenGen::initialize_oops() 用の補助クラス (ReadClosure) ](noihDtb1tt.html)
* [SharedHeap クラス関連のクラス (SharedHeap, SharedHeap::StrongRootsScope, 及びそれらの補助クラス(AssertIsPermClosure, AssertNonScavengableClosure, AlwaysTrueClosure, SkipAdjustingSharedStrings))](noysUrzWdK.html)
* [Space クラス関連のクラス (SpaceMemRegionOopsIterClosure, Space, DirtyCardToOopClosure, CompactPoint, CompactibleSpace, ContiguousSpace, Filtering_DCTOC, ContiguousSpaceDCTOC, EdenSpace, ConcEdenSpace, OffsetTableContigSpace, TenuredSpace, ContigPermSpace, 及びそれらの補助クラス(VerifyOldOopClosure))](noEwj5VK63.html)
* [SpecializationStats クラス ](noXUdVICpE.html)
* [TenuredGeneration クラス ](noF3DfmKtp.html)
* [ThreadLocalAllocBuffer クラス関連のクラス (ThreadLocalAllocBuffer, GlobalTLABStats)](no-MX5_-HV.html)
* [Universe クラス関連のクラス (CommonMethodOopCache, ActiveMethodOopsCache, LatestMethodOopCache, Universe, DeferredObjAllocEvent, 及びそれらの補助クラス(FixupMirrorClosure))](norWgXjc-w.html)
* [WaterMark クラス ](noenM3nIXR.html)





