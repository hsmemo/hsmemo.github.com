---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/gc_implementation/ 以下 ： shared/
---
[Up](noBI-e1EXt.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/gc_implementation/ 以下 ： shared/

--- 

File Name                                                                      | Description
------------------------------------------------------------------------------ | -----------------------------------------------------------------
hotspot/src/share/vm/gc_implementation/shared/adaptiveSizePolicy.cpp           |  AdaptiveSizePolicy クラス関連のクラスの定義 ([AdaptiveSizePolicy, AdaptiveSizePolicyOutput](no2_bkpmU4.html))
hotspot/src/share/vm/gc_implementation/shared/adaptiveSizePolicy.hpp           |  同上
hotspot/src/share/vm/gc_implementation/shared/ageTable.cpp                     |  ageTable クラスの定義 ([ageTable](noWbC_gstA.html))
hotspot/src/share/vm/gc_implementation/shared/ageTable.hpp                     |  同上
hotspot/src/share/vm/gc_implementation/shared/allocationStats.cpp              |  AllocationStats クラスの定義 ([AllocationStats](noesnEyjCz.html))
hotspot/src/share/vm/gc_implementation/shared/allocationStats.hpp              |  同上
hotspot/src/share/vm/gc_implementation/shared/cSpaceCounters.cpp               |  CSpaceCounters 及びその補助クラスの定義 ([CSpaceCounters, ContiguousSpaceUsedHelper](nokrAulcN3.html))
hotspot/src/share/vm/gc_implementation/shared/cSpaceCounters.hpp               |  同上
hotspot/src/share/vm/gc_implementation/shared/collectorCounters.cpp            |  CollectorCounters クラス関連のクラスの定義 ([CollectorCounters, TraceCollectorStats](nozQN0aiV3.html))
hotspot/src/share/vm/gc_implementation/shared/collectorCounters.hpp            |  同上
hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.cpp           |  ConcurrentGCThread クラス関連のクラスの定義 ([SuspendibleThreadSet, ConcurrentGCThread, SurrogateLockerThread](nobX3iklI0.html))
hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.hpp           |  同上
hotspot/src/share/vm/gc_implementation/shared/gSpaceCounters.cpp               |  GSpaceCounters 及びその補助クラス ([GSpaceCounters, GenerationUsedHelper](no2fgDYo46.html))
hotspot/src/share/vm/gc_implementation/shared/gSpaceCounters.hpp               |  同上
hotspot/src/share/vm/gc_implementation/shared/gcAdaptivePolicyCounters.cpp     |  GCAdaptivePolicyCounters クラスの定義 ([GCAdaptivePolicyCounters](noQPZicMu1.html))
hotspot/src/share/vm/gc_implementation/shared/gcAdaptivePolicyCounters.hpp     |  同上
hotspot/src/share/vm/gc_implementation/shared/gcPolicyCounters.cpp             |  GCPolicyCounters クラスの定義 ([GCPolicyCounters](no32E3BdCt.html))
hotspot/src/share/vm/gc_implementation/shared/gcPolicyCounters.hpp             |  同上
hotspot/src/share/vm/gc_implementation/shared/gcStats.cpp                      |  GCStats クラス関連のクラスの定義 ([GCStats, CMSGCStats](noxCj_R65V.html))
hotspot/src/share/vm/gc_implementation/shared/gcStats.hpp                      |  同上
hotspot/src/share/vm/gc_implementation/shared/gcUtil.cpp                       |  GC に関係した諸々のユーティリティ・クラスの定義 ([AdaptiveWeightedAverage, AdaptivePaddedAverage, AdaptivePaddedNoZeroDevAverage, LinearLeastSquareFit, GCPauseTimer](nobmiqSEMa.html))
hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp                       |  同上
hotspot/src/share/vm/gc_implementation/shared/generationCounters.cpp           |  GenerationCounters クラスの定義 ([GenerationCounters](noxFVfzYwv.html))
hotspot/src/share/vm/gc_implementation/shared/generationCounters.hpp           |  同上
hotspot/src/share/vm/gc_implementation/shared/hSpaceCounters.cpp               |  HSpaceCounters クラスの定義 ([HSpaceCounters](nonpotWO53.html))
hotspot/src/share/vm/gc_implementation/shared/hSpaceCounters.hpp               |  同上
hotspot/src/share/vm/gc_implementation/shared/immutableSpace.cpp               |  ImmutableSpace クラスの定義 ([ImmutableSpace](nobqCneHr4.html))
hotspot/src/share/vm/gc_implementation/shared/immutableSpace.hpp               |  同上
hotspot/src/share/vm/gc_implementation/shared/isGCActiveMark.hpp               |  IsGCActiveMark クラスの定義 ([IsGCActiveMark](no6rX1OB8I.html))
hotspot/src/share/vm/gc_implementation/shared/liveRange.hpp                    |  LiveRange クラスの定義 ([LiveRange](node9K2BmJ.html))
hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp                    |  MarkSweep クラス関連のクラスの定義 ([MarkSweep, MarkSweep::FollowRootClosure, MarkSweep::MarkAndPushClosure, MarkSweep::FollowStackClosure, MarkSweep::AdjustPointerClosure, MarkSweep::IsAliveClosure, MarkSweep::KeepAliveClosure, PreservedMark, 及びそれらの補助クラス(AdjusterTracker)](noIQOJi7xk.html))
hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp                    |  同上
hotspot/src/share/vm/gc_implementation/shared/markSweep.inline.hpp             |  同上
hotspot/src/share/vm/gc_implementation/shared/mutableNUMASpace.cpp             |  MutableNUMASpace およびその補助クラスの定義 ([MutableNUMASpace, MutableNUMASpace::LGRPSpace](no5ds0yTMZ.html))
hotspot/src/share/vm/gc_implementation/shared/mutableNUMASpace.hpp             |  同上
hotspot/src/share/vm/gc_implementation/shared/mutableSpace.cpp                 |  MutableSpace クラスの定義 ([MutableSpace](noM-xuT3DI.html))
hotspot/src/share/vm/gc_implementation/shared/mutableSpace.hpp                 |  同上
hotspot/src/share/vm/gc_implementation/shared/spaceCounters.cpp                |  SpaceCounters 及びその補助クラスの定義 ([SpaceCounters, MutableSpaceUsedHelper](noD79Ispje.html))
hotspot/src/share/vm/gc_implementation/shared/spaceCounters.hpp                |  同上
hotspot/src/share/vm/gc_implementation/shared/spaceDecorator.cpp               |  SpaceDecorator 及び SpaceMangler クラス関連のクラスの定義 ([SpaceDecorator, SpaceMangler, GenSpaceMangler, MutableSpaceMangler](no-5sfyT-L.html))
hotspot/src/share/vm/gc_implementation/shared/spaceDecorator.hpp               |  同上
hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.cpp               |  VM_GC_Operation クラス関連のクラスの定義 ([VM_GC_Operation, VM_GC_HeapInspection, VM_GenCollectForAllocation, VM_GenCollectFull, VM_GenCollectForPermanentAllocation, SvcGCMarker](noR8KDvgef.html))
hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.hpp               |  同上







