---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/gc_implementation/ 以下 ： parallelScavenge/
---
[Up](noBI-e1EXt.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/gc_implementation/ 以下 ： parallelScavenge/

--- 

File Name                                                                                  | Description
------------------------------------------------------------------------------------------ | -----------------------------------------------------------------
hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningGenerations.cpp           |  AdjoiningGenerations クラスの定義 ([AdjoiningGenerations](nolnC02zdF.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningGenerations.hpp           |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningVirtualSpaces.cpp         |  AdjoiningVirtualSpaces クラスの定義 ([AdjoiningVirtualSpaces](noqD_eKfoe.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningVirtualSpaces.hpp         |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/asPSOldGen.cpp                     |  ASPSOldGen クラスの定義 ([ASPSOldGen](no3oXeR9y8.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/asPSOldGen.hpp                     |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/asPSYoungGen.cpp                   |  ASPSYoungGen クラスの定義 ([ASPSYoungGen](no3pd27T-B.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/asPSYoungGen.hpp                   |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/cardTableExtension.cpp             |  CardTableExtension クラスの定義 ([CardTableExtension, 及びその補助クラス(CheckForUnmarkedOops, CheckForUnmarkedObjects, CheckForPreciseMarks)](no2uCUvoLc.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/cardTableExtension.hpp             |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.cpp                  |  GCTaskManager クラス関連のクラスの定義 ([GCTask, GCTask::Kind, GCTaskQueue, SynchronizedGCTaskQueue, NotifyDoneClosure, GCTaskManager, NoopGCTask, BarrierGCTask, ReleasingBarrierGCTask, NotifyingBarrierGCTask, WaitForBarrierGCTask, MonitorSupply](noQgT7pkf7.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp                  |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskThread.cpp                   |  GCTaskThread クラス及びその補助クラスの定義 ([GCTaskThread, GCTaskTimeStamp](noJO2dsHz3.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskThread.hpp                   |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/generationSizer.hpp                |  GenerationSizer クラスの定義 ([GenerationSizer](noAKBFrt-C.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/objectStartArray.cpp               |  ObjectStartArray クラスの定義 ([ObjectStartArray](no8ofUb1Tj.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/objectStartArray.hpp               |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/parMarkBitMap.cpp                  |  ParMarkBitMap クラスの定義 ([ParMarkBitMap](no1xQmLikM.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/parMarkBitMap.hpp                  |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/parMarkBitMap.inline.hpp           |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.cpp           |  ParallelScavengeHeap クラス関連のクラスの定義 ([ParallelScavengeHeap, ParallelScavengeHeap::ParStrongRootsScope](nocI5UjrOw.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.hpp           |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.inline.hpp    |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.cpp                        |  Parallel Compaction 用の GCTask のサブクラスの定義 ([ThreadRootsMarkingTask, MarkFromRootsTask, RefProcTaskProxy, RefEnqueueTaskProxy, RefProcTaskExecutor, StealMarkingTask, StealRegionCompactionTask, UpdateDensePrefixTask, DrainStacksCompactionTask](noFikUCFeI.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.hpp                        |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psAdaptiveSizePolicy.cpp           |  PSAdaptiveSizePolicy クラスの定義 ([PSAdaptiveSizePolicy](noizKvqcuF.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psAdaptiveSizePolicy.hpp           |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psCompactionManager.cpp            |  ParCompactionManager クラスの定義 ([ParCompactionManager](no0cCD67lh.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psCompactionManager.hpp            |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psCompactionManager.inline.hpp     |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psGCAdaptivePolicyCounters.cpp     |  PSGCAdaptivePolicyCounters クラスの定義 ([PSGCAdaptivePolicyCounters](nol1o-XS6V.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psGCAdaptivePolicyCounters.hpp     |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psGenerationCounters.cpp           |  PSGenerationCounters クラスの定義 ([PSGenerationCounters](nosowYtl3v.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psGenerationCounters.hpp           |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.cpp                    |  PSMarkSweep クラスの定義 ([PSMarkSweep, 及びその補助クラス(PSAlwaysTrueClosure)](no1aPXCRTu.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.hpp                    |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweepDecorator.cpp           |  PSMarkSweepDecorator クラスの定義 ([PSMarkSweepDecorator](notHxsGq3B.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweepDecorator.hpp           |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psOldGen.cpp                       |  PSOldGen クラスの定義 ([PSOldGen, 及びその補助クラス(VerifyObjectStartArrayClosure)](nodSluwO6C.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psOldGen.hpp                       |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp              |  PSParallelCompact クラス関連のクラスの定義 ([SplitInfo, SpaceInfo, ParallelCompactData, ParallelCompactData::RegionData, ParMarkBitMapClosure, PSParallelCompact, PSParallelCompact::IsAliveClosure, PSParallelCompact::KeepAliveClosure, PSParallelCompact::FollowRootClosure, PSParallelCompact::FollowStackClosure, PSParallelCompact::AdjustPointerClosure, PSParallelCompact::VerifyUpdateClosure, PSParallelCompact::ResetObjectsClosure, PSParallelCompact::MarkAndPushClosure, MoveAndUpdateClosure, UpdateOnlyClosure, FillClosure, 及びそれらの補助クラス(PreGCValues, PSAlwaysTrueClosure, AdjusterTracker)](nopIVGGi4c.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp              |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPermGen.cpp                      |  PSPermGen クラスの定義 ([PSPermGen](noiz-XRVYv.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPermGen.hpp                      |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionLAB.cpp                 |  PSPromotionLAB クラス及びそのサブクラスの定義 ([PSPromotionLAB, PSYoungPromotionLAB, PSOldPromotionLAB](noXmVKMKHq.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionLAB.hpp                 |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionManager.cpp             |  PSPromotionManager クラスの定義 ([PSPromotionManager](noUfH_bnys.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionManager.hpp             |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionManager.inline.hpp      |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp                     |  PSScavenge クラス関連のクラスの定義 ([PSScavenge, PSScavengeRootsClosure, 及びそれらの補助クラス(PSIsAliveClosure, PSKeepAliveClosure, PSEvacuateFollowersClosure, PSPromotionFailedClosure, PSRefProcTaskProxy, PSRefEnqueueTaskProxy, PSRefProcTaskExecutor)](nooqLHTdGw.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.hpp                     |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.inline.hpp              |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psTasks.cpp                        |  Parallel Scavenge 用の GCTask のサブクラスの定義 ([ScavengeRootsTask, ThreadRootsTask, StealTask, SerialOldToYoungRootsTask, OldToYoungRootsTask](noyXf3dVUN.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psTasks.hpp                        |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psVirtualspace.cpp                 |  PSVirtualSpace クラス関連のクラスの定義 ([PSVirtualSpace, PSVirtualSpaceVerifier, PSVirtualSpaceHighToLow](nolkqrTJmA.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psVirtualspace.hpp                 |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/psYoungGen.cpp                     |  PSYoungGen クラスの定義 ([PSYoungGen](nos9IDO7nn.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/psYoungGen.hpp                     |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/vmPSOperations.cpp                 |  ParallelScavenge 用の VM_GC_Operation のサブクラスの定義 ([VM_ParallelGCFailedAllocation, VM_ParallelGCFailedPermanentAllocation, VM_ParallelGCSystemGC](noHijyuVQ-.html))
hotspot/src/share/vm/gc_implementation/parallelScavenge/vmPSOperations.hpp                 |  同上
hotspot/src/share/vm/gc_implementation/parallelScavenge/vmStructs_parallelgc.hpp           |  VMStructs クラス用のマクロ定義 (VM_STRUCTS_PARALLELGC, VM_TYPES_PARALLELGC)






