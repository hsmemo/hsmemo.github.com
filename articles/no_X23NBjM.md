---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： services/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： services/

--- 

File Name                                                             | Description
--------------------------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/vm/services/attachListener.cpp                      |  AttachListener クラス関連のクラスの定義 ([AttachListener, AttachOperation](nok8M_8_GC.html))
hotspot/src/share/vm/services/attachListener.hpp                      |  同上
hotspot/src/share/vm/services/classLoadingService.cpp                 |  ClassLoadingService クラス関連のクラスの定義 ([ClassLoadingService, LoadedClassesEnumerator](novfZ8L0Qi.html))
hotspot/src/share/vm/services/classLoadingService.hpp                 |  同上
hotspot/src/share/vm/services/dtraceAttacher.cpp                      |  DTrace クラスの定義 ([DTrace, 及びその補助クラス(VM_DeoptimizeTheWorld)](noa4pW6mhZ.html))
hotspot/src/share/vm/services/dtraceAttacher.hpp                      |  同上
hotspot/src/share/vm/services/g1MemoryPool.cpp                        |  G1CollectedHeap 用の MemoryPool クラスの定義 ([G1MemoryPoolSuper, G1EdenPool, G1SurvivorPool, G1OldGenPool](nohMFm1PXC.html))
hotspot/src/share/vm/services/g1MemoryPool.hpp                        |  同上
hotspot/src/share/vm/services/gcNotifier.cpp                          |  GCNotifier クラス及びその補助クラスの定義 ([GCNotificationRequest, GCNotifier](nokXe6h2uE.html))
hotspot/src/share/vm/services/gcNotifier.hpp                          |  同上
hotspot/src/share/vm/services/heapDumper.cpp                          |  HeapDumper クラスの定義 ([HeapDumper, 及びその補助クラス(DumpWriter, DumperSupport, SymbolTableDumper, JNILocalsDumper, JNIGlobalsDumper, MonitorUsedDumper, StickyClassDumper, HeapObjectDumper, VM_HeapDumper)](noSQhq-Anw.html))
hotspot/src/share/vm/services/heapDumper.hpp                          |  同上
hotspot/src/share/vm/services/jmm.h                                   |  JMM (HotSpot Monitoring and Management Interface) 関連の定数/構造体の定義  (See: [here](no2114S_x.html) for details)
hotspot/src/share/vm/services/lowMemoryDetector.cpp                   |  LowMemoryDetector クラス関連のクラスの定義 ([ThresholdSupport, SensorInfo, LowMemoryDetector, LowMemoryDetectorDisabler](no-twyc8kx.html))
hotspot/src/share/vm/services/lowMemoryDetector.hpp                   |  同上
hotspot/src/share/vm/services/management.cpp                          |  JMM 関連の雑多なクラスの定義 ([Management, TraceVmCreationTime, VmThreadCountClosure, ThreadTimesClosure](noPUQCQRX6.html)) 及び JMM のエントリポイントとなる関数(jmm_*)の定義
hotspot/src/share/vm/services/management.hpp                          |  同上
hotspot/src/share/vm/services/memoryManager.cpp                       |  MemoryManager クラス関連のクラスの定義 ([MemoryManager, CodeCacheMemoryManager, GCStatInfo, GCMemoryManager, CopyMemoryManager, MSCMemoryManager, ParNewMemoryManager, CMSMemoryManager, PSScavengeMemoryManager, PSMarkSweepMemoryManager, G1YoungGenMemoryManager, G1OldGenMemoryManager](no6WcwJzZ8.html))
hotspot/src/share/vm/services/memoryManager.hpp                       |  同上
hotspot/src/share/vm/services/memoryPool.cpp                          |  MemoryPool クラス関連のクラスの定義 ([MemoryPool, CollectedMemoryPool, ContiguousSpacePool, SurvivorContiguousSpacePool, CompactibleFreeListSpacePool, GenerationPool, CodeHeapPool](norgoAqItH.html))
hotspot/src/share/vm/services/memoryPool.hpp                          |  同上
hotspot/src/share/vm/services/memoryService.cpp                       |  MemoryService クラス関連のクラスの定義 ([MemoryService, TraceMemoryManagerStats, 及びそれらの補助クラス(GcThreadCountClosure)](noG1syKBnp.html))
hotspot/src/share/vm/services/memoryService.hpp                       |  同上
hotspot/src/share/vm/services/memoryUsage.hpp                         |  MemoryUsage クラスの定義 ([MemoryUsage](noQQuccgaO.html))
hotspot/src/share/vm/services/psMemoryPool.cpp                        |  ParallelScavengeHeap 用の MemoryPool クラスの定義 ([PSGenerationPool, EdenMutableSpacePool, SurvivorMutableSpacePool](no-5dY4MxD.html))
hotspot/src/share/vm/services/psMemoryPool.hpp                        |  同上
hotspot/src/share/vm/services/runtimeService.cpp                      |  RuntimeService クラスの定義 ([RuntimeService](no0ZFloaHd.html))
hotspot/src/share/vm/services/runtimeService.hpp                      |  同上
hotspot/src/share/vm/services/serviceUtil.hpp                         |  ServiceUtil クラスの定義 ([ServiceUtil](no86HZ3Vo8.html))
hotspot/src/share/vm/services/threadService.cpp                       |  ThreadService クラス関連のクラスの定義 ([ThreadService, ThreadStatistics, ThreadSnapshot, ThreadStackTrace, StackFrameInfo, ThreadConcurrentLocks, ConcurrentLocksDump, ThreadDumpResult, DeadlockCycle, ThreadsListEnumerator, JavaThreadStatusChanger, JavaThreadInObjectWaitState, JavaThreadParkedState, JavaThreadBlockedOnMonitorEnterState, JavaThreadSleepState, 及びそれらの補助クラス(InflatedMonitorsClosure)](nosUynfb8I.html))
hotspot/src/share/vm/services/threadService.hpp                       |  同上






