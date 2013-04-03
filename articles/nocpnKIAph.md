---
layout: default
title: HotSpot の内部で使われるデータ構造 ： 詳細 ： gc_implementation/g1/ 編
---
[Up](nohFVVK2so.html) [Top](../index.html)

#### HotSpot の内部で使われるデータ構造 ： 詳細 ： gc_implementation/g1/ 編

--- 

* [BufferingOopClosure クラス関連のクラス (BufferingOopClosure, BufferingOopsInGenClosure, BufferingOopsInHeapRegionClosure)](no59mM3lIy.html)
* [CollectionSetChooser クラス及びその補助クラス (CSetChooserCache, CollectionSetChooser)](no7hac-25W.html)
* [ConcurrentG1Refine クラス ](noIpfIfgUd.html)
* [ConcurrentG1RefineThread クラス ](noUrp03Lsz.html)
* [ConcurrentMark クラス関連のクラス (G1CMIsAliveClosure, CMBitMapRO, CMBitMap, CMMarkStack, CMRegionStack, ForceOverflowSettings, ConcurrentMark, CMTask, G1PrintRegionLivenessInfoClosure, 及びそれらの補助クラス(NoteStartOfMarkHRClosure, CMMarkRootsClosure, CMConcurrentMarkingTask, CalcLiveObjectsClosure, G1ParFinalCountTask, G1NoteEndOfConcMarkClosure, G1ParNoteEndTask, G1ParScrubRemSetTask, G1CMKeepAliveClosure, G1CMDrainMarkingStackClosure, G1CMParKeepAliveAndDrainClosure, G1CMParDrainMarkingStackClosure, G1RefProcTaskExecutor, G1RefProcTaskProxy, G1RefEnqueueTaskProxy, CMRemarkTask, PrintReachableOopClosure, PrintReachableObjectClosure, PrintReachableRegionClosure, CMGlobalObjectClosure, CSMarkOopClosure, CSMarkBitMapClosure, CompleteMarkingInCSHRClosure, ClearMarksInHRClosure, CMBitMapClosure, CMObjectClosure, CMOopClosure))](noSix1I5mG.html)
* [ConcurrentMarkThread クラス (ConcurrentMarkThread, 及びその補助クラス(CMCheckpointRootsInitialClosure, CMCheckpointRootsFinalClosure, CMCleanUp))](noENNPqKjJ.html)
* [DirtyCardQueue クラス関連のクラス (CardTableEntryClosure, DirtyCardQueue, DirtyCardQueueSet)](no-MT18QAm.html)
* [G1AllocRegion クラス及びその補助クラス (G1AllocRegion, ar_ext_msg)](nokV9tGhUm.html)
* [G1BlockOffsetTable クラス関連のクラス (G1BlockOffsetTable, G1BlockOffsetSharedArray, G1BlockOffsetArray, G1BlockOffsetArrayContigSpace)](noqscaXW35.html)
* [G1CollectedHeap クラス関連のクラス (YoungList, MutatorAllocRegion, G1CollectedHeap, GCLabBitMapClosure, GCLabBitMap, G1ParGCAllocBuffer, G1ParScanThreadState, 及びそれらの補助クラス(RefineCardTableEntryClosure, ClearLoggedCardTableEntryClosure, RedirtyLoggedCardTableEntryClosure, RedirtyLoggedCardTableEntryFastClosure, PostMCRemSetClearClosure, PostMCRemSetInvalidateClosure, RebuildRSOutOfRegionClosure, ParRebuildRSTask, SumUsedClosure, SumUsedRegionsClosure, IterateOopClosureRegionClosure, IterateObjectClosureRegionClosure, SpaceClosureRegionClosure, ResetClaimValuesClosure, CheckClaimValuesClosure, VerifyLivenessOopClosure, VerifyObjsInRegionClosure, PrintObjsInRegionClosure, VerifyRegionClosure, VerifyRootsClosure, G1ParVerifyTask, PrintRegionClosure, VerifyMarkedObjsClosure, FindGCAllocRegion, G1IsAliveClosure, G1KeepAliveClosure, UpdateRSetDeferred, RemoveSelfPointerClosure, G1ParEvacuateFollowersClosure, G1ParTask, SaveMarksClosure, G1ParCleanupCTTask, G1VerifyCardTableCleanup, NoYoungRegionsClosure, RegionResetter, VerifyRegionListsClosure))](noXKu5VdA8.html)
* [G1CollectorPolicy クラス関連のクラス (PauseSummary, MainBodySummary, Summary, G1CollectorPolicy, G1CollectorPolicy_BestRegionsFirst, 及びそれらの補助クラス(LineBuffer, G1YoungGenSizer, CountCSClosure, HRSortIndexIsOKClosure, NextNonCSElemFinder, KnownGarbageClosure, ParKnownGarbageHRClosure, ParKnownGarbageTask))](nocVQCI7P9.html)
* [G1MMUTracker クラス関連のクラス (G1MMUTracker, G1MMUTrackerQueueElem, G1MMUTrackerQueue)](noSgbBKFzq.html)
* [G1MarkSweep クラス (G1MarkSweep, 及びその補助クラス(G1PrepareCompactClosure, FindFirstRegionClosure, G1AdjustPointersClosure, G1SpaceCompactClosure))](no6IqrM9Ls.html)
* [G1MonitoringSupport クラス ](noEwlj0hED.html)
* [G1GC の処理で使用される Closure クラス (OopsInHeapRegionClosure, G1ParClosureSuper, G1ParPushHeapRSClosure, G1ParScanClosure, G1ParScanPartialArrayClosure, G1ParCopyHelper, G1ParCopyClosure, FilterIntoCSClosure, FilterInHeapRegionAndIntoCSClosure, FilterAndMarkInHeapRegionAndIntoCSClosure, FilterOutOfRegionClosure)](nooxN1eSbp.html)
* [G1RemSet クラス関連のクラス (G1RemSet, CountNonCleanMemRegionClosure, UpdateRSOopClosure, UpdateRSetImmediate, UpdateRSOrPushRefOopClosure, 及びそれらの補助クラス(IntoCSOopClosure, VerifyRSCleanCardOopClosure, ScanRSClosure, RefineRecordRefsIntoCSCardTableEntryClosure, PrintRSClosure, CountRSSizeClosure, cleanUpIteratorsClosure, UpdateRSetCardTableEntryIntoCSetClosure, ScrubRSClosure, TriggerClosure, InvokeIfNotTriggeredClosure, Mux2Closure, HRRSStatsIter, PrintRSThreadVTimeClosure))](noxuSNnpRS.html)
* [G1SATBCardTableModRefBS クラス関連のクラス (G1SATBCardTableModRefBS, G1SATBCardTableLoggingModRefBS)](nooNuPhaQX.html)
* [HeapRegion クラス関連のクラス (HeapRegionDCTOC, G1OffsetTableContigSpace, HeapRegion, HeapRegionClosure, 及びそれらの補助クラス(VerifyLiveClosure, NextCompactionHeapRegionClosure))](no6LZSs7fe.html)
* [HeapRegionRemSet クラス関連のクラス (HRRSCleanupTask, OtherRegionsTable, HeapRegionRemSet, HeapRegionRemSetIterator, CardClosure, 及びそれらの補助クラス(PerRegionTable, PosParPRT))](noZewkCaEu.html)
* [HeapRegionSeq クラス (HeapRegionSeq, 及びその補助クラス(PrintHeapRegionClosure))](noTZs0mRgZ.html)
* [HeapRegionSet クラス関連のクラス (HeapRegionSetBase, hrs_ext_msg, HeapRegionSet, HeapRegionLinkedList, HeapRegionLinkedListIterator)](noX1_0ngoz.html)
* [HeapRegionSet クラスのサブクラス群 (FreeRegionList, MasterFreeRegionList, SecondaryFreeRegionList, HumongousRegionSet, MasterHumongousRegionSet)](noPxM63HpP.html)
* [PtrQueue クラス関連のクラス (PtrQueue, BufferNode, PtrQueueSet)](no7RhjTnQ3.html)
* [SATBMarkQueueSet クラス関連のクラス (ObjPtrQueue, SATBMarkQueueSet)](nosFOwY4Yw.html)
* [SparsePRT クラス関連のクラス (SparsePRTEntry, RSHashTable, RSHashTableIter, SparsePRT, SparsePRTIter, SparsePRTCleanupTask)](noET9h2_gK.html)
* [SurvRateGroup クラス ](nodRGFdeXS.html)
* [G1GC で使用する VM_Operation クラス (VM_G1OperationWithAllocRequest, VM_G1CollectFull, VM_G1CollectForAllocation, VM_G1IncCollectionPause, VM_CGC_Operation)](noy1VBBk8v.html)





