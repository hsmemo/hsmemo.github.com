---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/gc_implementation/ 以下 ： g1/
---
[Up](noBI-e1EXt.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/gc_implementation/ 以下 ： g1/

--- 

File Name                                                                    | Description
---------------------------------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/vm/gc_implementation/g1/bufferingOopClosure.hpp            |  BufferingOopClosure クラス関連のクラスの定義 ([BufferingOopClosure, BufferingOopsInGenClosure, BufferingOopsInHeapRegionClosure](no59mM3lIy.html))
hotspot/src/share/vm/gc_implementation/g1/collectionSetChooser.cpp           |  CollectionSetChooser クラス及びその補助クラスの定義 ([CSetChooserCache, CollectionSetChooser](no7hac-25W.html))
hotspot/src/share/vm/gc_implementation/g1/collectionSetChooser.hpp           |  同上
hotspot/src/share/vm/gc_implementation/g1/concurrentG1Refine.cpp             |  ConcurrentG1Refine クラスの定義 ([ConcurrentG1Refine](noIpfIfgUd.html))
hotspot/src/share/vm/gc_implementation/g1/concurrentG1Refine.hpp             |  同上
hotspot/src/share/vm/gc_implementation/g1/concurrentG1RefineThread.cpp       |  ConcurrentG1RefineThread クラスの定義 ([ConcurrentG1RefineThread](noUrp03Lsz.html))
hotspot/src/share/vm/gc_implementation/g1/concurrentG1RefineThread.hpp       |  同上
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp                 |  ConcurrentMark クラス関連のクラスの定義 ([G1CMIsAliveClosure, CMBitMapRO, CMBitMap, CMMarkStack, CMRegionStack, ForceOverflowSettings, ConcurrentMark, CMTask, G1PrintRegionLivenessInfoClosure, 及びそれらの補助クラス(NoteStartOfMarkHRClosure, CMMarkRootsClosure, CMConcurrentMarkingTask, CalcLiveObjectsClosure, G1ParFinalCountTask, G1NoteEndOfConcMarkClosure, G1ParNoteEndTask, G1ParScrubRemSetTask, G1CMKeepAliveClosure, G1CMDrainMarkingStackClosure, G1CMParKeepAliveAndDrainClosure, G1CMParDrainMarkingStackClosure, G1RefProcTaskExecutor, G1RefProcTaskProxy, G1RefEnqueueTaskProxy, CMRemarkTask, PrintReachableOopClosure, PrintReachableObjectClosure, PrintReachableRegionClosure, CMGlobalObjectClosure, CSMarkOopClosure, CSMarkBitMapClosure, CompleteMarkingInCSHRClosure, ClearMarksInHRClosure, CMBitMapClosure, CMObjectClosure, CMOopClosure)](noSix1I5mG.html))
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp                 |  同上
hotspot/src/share/vm/gc_implementation/g1/concurrentMarkThread.cpp           |  ConcurrentMarkThread クラスの定義 ([ConcurrentMarkThread, 及びその補助クラス(CMCheckpointRootsInitialClosure, CMCheckpointRootsFinalClosure, CMCleanUp)](noENNPqKjJ.html))
hotspot/src/share/vm/gc_implementation/g1/concurrentMarkThread.hpp           |  同上
hotspot/src/share/vm/gc_implementation/g1/concurrentMarkThread.inline.hpp    |  同上
hotspot/src/share/vm/gc_implementation/g1/dirtyCardQueue.cpp                 |  DirtyCardQueue クラス関連のクラスの定義 ([CardTableEntryClosure, DirtyCardQueue, DirtyCardQueueSet](no-MT18QAm.html))
hotspot/src/share/vm/gc_implementation/g1/dirtyCardQueue.hpp                 |  同上
hotspot/src/share/vm/gc_implementation/g1/g1AllocRegion.cpp                  |  G1AllocRegion クラス及びその補助クラスの定義 ([G1AllocRegion, ar_ext_msg](nokV9tGhUm.html))
hotspot/src/share/vm/gc_implementation/g1/g1AllocRegion.hpp                  |  同上
hotspot/src/share/vm/gc_implementation/g1/g1AllocRegion.inline.hpp           |  同上
hotspot/src/share/vm/gc_implementation/g1/g1BlockOffsetTable.cpp             |  G1BlockOffsetTable クラス関連のクラスの定義 ([G1BlockOffsetTable, G1BlockOffsetSharedArray, G1BlockOffsetArray, G1BlockOffsetArrayContigSpace](noqscaXW35.html))
hotspot/src/share/vm/gc_implementation/g1/g1BlockOffsetTable.hpp             |  同上
hotspot/src/share/vm/gc_implementation/g1/g1BlockOffsetTable.inline.hpp      |  同上
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp                |  G1CollectedHeap クラス関連のクラスの定義 ([YoungList, MutatorAllocRegion, G1CollectedHeap, GCLabBitMapClosure, GCLabBitMap, G1ParGCAllocBuffer, G1ParScanThreadState, 及びそれらの補助クラス(RefineCardTableEntryClosure, ClearLoggedCardTableEntryClosure, RedirtyLoggedCardTableEntryClosure, RedirtyLoggedCardTableEntryFastClosure, PostMCRemSetClearClosure, PostMCRemSetInvalidateClosure, RebuildRSOutOfRegionClosure, ParRebuildRSTask, SumUsedClosure, SumUsedRegionsClosure, IterateOopClosureRegionClosure, IterateObjectClosureRegionClosure, SpaceClosureRegionClosure, ResetClaimValuesClosure, CheckClaimValuesClosure, VerifyLivenessOopClosure, VerifyObjsInRegionClosure, PrintObjsInRegionClosure, VerifyRegionClosure, VerifyRootsClosure, G1ParVerifyTask, PrintRegionClosure, VerifyMarkedObjsClosure, FindGCAllocRegion, G1IsAliveClosure, G1KeepAliveClosure, UpdateRSetDeferred, RemoveSelfPointerClosure, G1ParEvacuateFollowersClosure, G1ParTask, SaveMarksClosure, G1ParCleanupCTTask, G1VerifyCardTableCleanup, NoYoungRegionsClosure, RegionResetter, VerifyRegionListsClosure)](noXKu5VdA8.html))
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp                |  同上
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.inline.hpp         |  同上
hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp              |  G1CollectorPolicy クラス関連のクラスの定義 ([PauseSummary, MainBodySummary, Summary, G1CollectorPolicy, G1CollectorPolicy_BestRegionsFirst, 及びそれらの補助クラス(LineBuffer, G1YoungGenSizer, CountCSClosure, HRSortIndexIsOKClosure, NextNonCSElemFinder, KnownGarbageClosure, ParKnownGarbageHRClosure, ParKnownGarbageTask)](nocVQCI7P9.html))
hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp              |  同上
hotspot/src/share/vm/gc_implementation/g1/g1MMUTracker.cpp                   |  G1MMUTracker クラス関連のクラスの定義 ([G1MMUTracker, G1MMUTrackerQueueElem, G1MMUTrackerQueue](noSgbBKFzq.html))
hotspot/src/share/vm/gc_implementation/g1/g1MMUTracker.hpp                   |  同上
hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.cpp                    |  G1MarkSweep クラスの定義 ([G1MarkSweep, 及びその補助クラス(G1PrepareCompactClosure, FindFirstRegionClosure, G1AdjustPointersClosure, G1SpaceCompactClosure)](no6IqrM9Ls.html))
hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.hpp                    |  同上
hotspot/src/share/vm/gc_implementation/g1/g1MonitoringSupport.cpp            |  G1MonitoringSupport クラスの定義 ([G1MonitoringSupport](noEwlj0hED.html))
hotspot/src/share/vm/gc_implementation/g1/g1MonitoringSupport.hpp            |  同上
hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp                  |  G1GC の処理で使用される Closure クラスの定義 ([OopsInHeapRegionClosure, G1ParClosureSuper, G1ParPushHeapRSClosure, G1ParScanClosure, G1ParScanPartialArrayClosure, G1ParCopyHelper, G1ParCopyClosure, FilterIntoCSClosure, FilterInHeapRegionAndIntoCSClosure, FilterAndMarkInHeapRegionAndIntoCSClosure, FilterOutOfRegionClosure](nooxN1eSbp.html))
hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.inline.hpp           |  同上
hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp                       |  G1RemSet クラス関連のクラスの定義 ([G1RemSet, CountNonCleanMemRegionClosure, UpdateRSOopClosure, UpdateRSetImmediate, UpdateRSOrPushRefOopClosure, 及びそれらの補助クラス(IntoCSOopClosure, VerifyRSCleanCardOopClosure, ScanRSClosure, RefineRecordRefsIntoCSCardTableEntryClosure, PrintRSClosure, CountRSSizeClosure, cleanUpIteratorsClosure, UpdateRSetCardTableEntryIntoCSetClosure, ScrubRSClosure, TriggerClosure, InvokeIfNotTriggeredClosure, Mux2Closure, HRRSStatsIter, PrintRSThreadVTimeClosure)](noxuSNnpRS.html))
hotspot/src/share/vm/gc_implementation/g1/g1RemSet.hpp                       |  同上
hotspot/src/share/vm/gc_implementation/g1/g1RemSet.inline.hpp                |  同上
hotspot/src/share/vm/gc_implementation/g1/g1SATBCardTableModRefBS.cpp        |  G1SATBCardTableModRefBS クラス関連のクラスの定義 ([G1SATBCardTableModRefBS, G1SATBCardTableLoggingModRefBS](nooNuPhaQX.html))
hotspot/src/share/vm/gc_implementation/g1/g1SATBCardTableModRefBS.hpp        |  同上
hotspot/src/share/vm/gc_implementation/g1/g1_globals.cpp                     |  G1GC 関連の JVM のコマンドラインオプションの定義 (及びデフォルト値の変更) (※1) (See: [here](no7882y_Y.html) for details)
hotspot/src/share/vm/gc_implementation/g1/g1_globals.hpp                     |  同上
hotspot/src/share/vm/gc_implementation/g1/g1_specialized_oop_closures.hpp    |  特定の OopClosure に特化した oop_oop_iterate() メソッドを定義するためのマクロの定義 (※2)
hotspot/src/share/vm/gc_implementation/g1/heapRegion.cpp                     |  HeapRegion クラス関連のクラスの定義 ([HeapRegionDCTOC, G1OffsetTableContigSpace, HeapRegion, HeapRegionClosure, 及びそれらの補助クラス(VerifyLiveClosure, NextCompactionHeapRegionClosure)](no6LZSs7fe.html))
hotspot/src/share/vm/gc_implementation/g1/heapRegion.hpp                     |  同上
hotspot/src/share/vm/gc_implementation/g1/heapRegion.inline.hpp              |  同上
hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.cpp               |  HeapRegionRemSet クラス関連のクラスの定義 ([HRRSCleanupTask, OtherRegionsTable, HeapRegionRemSet, HeapRegionRemSetIterator, CardClosure, 及びそれらの補助クラス(PerRegionTable, PosParPRT)](noZewkCaEu.html))
hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.hpp               |  同上
hotspot/src/share/vm/gc_implementation/g1/heapRegionSeq.cpp                  |  HeapRegionSeq クラスの定義 ([HeapRegionSeq, 及びその補助クラス(PrintHeapRegionClosure)](noTZs0mRgZ.html))
hotspot/src/share/vm/gc_implementation/g1/heapRegionSeq.hpp                  |  同上
hotspot/src/share/vm/gc_implementation/g1/heapRegionSeq.inline.hpp           |  同上
hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.cpp                  |  HeapRegionSet クラス関連のクラスの定義 ([HeapRegionSetBase, hrs_ext_msg, HeapRegionSet, HeapRegionLinkedList, HeapRegionLinkedListIterator](noX1_0ngoz.html))
hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp                  |  同上
hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.inline.hpp           |  同上
hotspot/src/share/vm/gc_implementation/g1/heapRegionSets.cpp                 |  HeapRegionSet クラスのサブクラス群の定義 ([FreeRegionList, MasterFreeRegionList, SecondaryFreeRegionList, HumongousRegionSet, MasterHumongousRegionSet](noPxM63HpP.html))
hotspot/src/share/vm/gc_implementation/g1/heapRegionSets.hpp                 |  同上
hotspot/src/share/vm/gc_implementation/g1/ptrQueue.cpp                       |  PtrQueue クラス関連のクラスの定義 ([PtrQueue, BufferNode, PtrQueueSet](no7RhjTnQ3.html))
hotspot/src/share/vm/gc_implementation/g1/ptrQueue.hpp                       |  同上
hotspot/src/share/vm/gc_implementation/g1/satbQueue.cpp                      |  SATBMarkQueueSet クラス関連のクラスの定義 ([ObjPtrQueue, SATBMarkQueueSet](nosFOwY4Yw.html))
hotspot/src/share/vm/gc_implementation/g1/satbQueue.hpp                      |  同上
hotspot/src/share/vm/gc_implementation/g1/sparsePRT.cpp                      |  SparsePRT クラス関連のクラスの定義 ([SparsePRTEntry, RSHashTable, RSHashTableIter, SparsePRT, SparsePRTIter, SparsePRTCleanupTask](noET9h2_gK.html))
hotspot/src/share/vm/gc_implementation/g1/sparsePRT.hpp                      |  同上
hotspot/src/share/vm/gc_implementation/g1/survRateGroup.cpp                  |  SurvRateGroup クラスの定義 ([SurvRateGroup](nodRGFdeXS.html))
hotspot/src/share/vm/gc_implementation/g1/survRateGroup.hpp                  |  同上
hotspot/src/share/vm/gc_implementation/g1/vm_operations_g1.cpp               |  G1GC で使用する VM_Operation クラスの定義 ([VM_G1OperationWithAllocRequest, VM_G1CollectFull, VM_G1CollectForAllocation, VM_G1IncCollectionPause, VM_CGC_Operation](noy1VBBk8v.html))
hotspot/src/share/vm/gc_implementation/g1/vm_operations_g1.hpp               |  同上

### 備考(Notes)
* ※1: hpp で補助用のマクロを定義し, cpp で実際にオプションを定義している.
* ※2: hotspot/src/share/vm/memory/specialized_oop_closures.hpp も参照.







