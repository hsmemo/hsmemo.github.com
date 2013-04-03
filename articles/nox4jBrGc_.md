---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： runtime/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： runtime/

--- 

File Name                                                             | Description
--------------------------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/vm/runtime/advancedThresholdPolicy.cpp              |  AdvancedThresholdPolicy クラスの定義 ([AdvancedThresholdPolicy](noIZrqGU8B.html))
hotspot/src/share/vm/runtime/advancedThresholdPolicy.hpp              |  同上
hotspot/src/share/vm/runtime/aprofiler.cpp                            |  AllocationProfiler クラスの定義 ([AllocationProfiler, 及びその補助クラス(AllocProfClosure, AllocProfResetClosure)](noD7Aila06.html))
hotspot/src/share/vm/runtime/aprofiler.hpp                            |  同上
hotspot/src/share/vm/runtime/arguments.cpp                            |  コマンドラインオプションを処理するためのクラスの定義 ([SystemProperty, AgentLibrary, AgentLibraryList, Arguments, 及びそれらの補助クラス(SysClassPath)](noAt7DeGPz.html))
hotspot/src/share/vm/runtime/arguments.hpp                            |  同上
hotspot/src/share/vm/runtime/atomic.cpp                               |  Atomic クラスの定義 ([Atomic](nocWKoyNdo.html))
hotspot/src/share/vm/runtime/atomic.hpp                               |  同上
hotspot/src/share/vm/runtime/basicLock.cpp                            |  BasicLock クラスおよび BasicObjectLock クラスの定義 ([BasicLock, BasicObjectLock](noBMKPZJux.html))
hotspot/src/share/vm/runtime/basicLock.hpp                            |  同上
hotspot/src/share/vm/runtime/biasedLocking.cpp                        |  BiasedLocking クラス関連のクラスの定義 ([BiasedLockingCounters, BiasedLocking, 及びそれらの補助クラス(VM_EnableBiasedLocking, EnableBiasedLockingTask, VM_RevokeBias, VM_BulkRevokeBias)](no1JRHVHuU.html))
hotspot/src/share/vm/runtime/biasedLocking.hpp                        |  同上
hotspot/src/share/vm/runtime/compilationPolicy.cpp                    |  CompilationPolicy クラス関連のクラスの定義 ([CompilationPolicy, NonTieredCompPolicy, SimpleCompPolicy, StackWalkCompPolicy, 及びそれらの補助クラス(CounterDecay)](noMGMnPDsK.html))
hotspot/src/share/vm/runtime/compilationPolicy.hpp                    |  同上
hotspot/src/share/vm/runtime/deoptimization.cpp                       |  Deoptimization クラス関連のクラスの定義 ([Deoptimization, Deoptimization::UnrollBlock, DeoptimizationMarker, 及びそれらの補助クラス(FieldReassigner)](noU79VidNA.html))
hotspot/src/share/vm/runtime/deoptimization.hpp                       |  同上
hotspot/src/share/vm/runtime/dtraceJSDT.cpp                           |  DTraceJSDT クラス関連のクラスの定義 ([DTraceJSDT, RegisteredProbes](nosHEMBpLW.html))
hotspot/src/share/vm/runtime/dtraceJSDT.hpp                           |  同上
hotspot/src/share/vm/runtime/extendedPC.hpp                           |  ExtendedPC クラスの定義 ([ExtendedPC](noq4U9itf-.html))
hotspot/src/share/vm/runtime/fieldDescriptor.cpp                      |  fieldDescriptor クラスの定義 ([fieldDescriptor](nojfJmDZh1.html))
hotspot/src/share/vm/runtime/fieldDescriptor.hpp                      |  同上
hotspot/src/share/vm/runtime/fieldType.cpp                            |  FieldType クラス関連のクラスの定義 ([FieldArrayInfo, FieldType](nocXDdMA24.html))
hotspot/src/share/vm/runtime/fieldType.hpp                            |  同上
hotspot/src/share/vm/runtime/fprofiler.cpp                            |  FlatProfiler クラス関連のクラスの定義 ([ThreadProfilerMark, IntervalData, ThreadProfiler, FlatProfiler, 及びそれらの補助クラス(PCRecorder, tick_counter, ProfilerNode, interpretedNode, compiledNode, stubNode, adapterNode, runtimeStubNode, unknown_compiledNode, vmNode, FlatProfilerTask)](nou3XNMGoo.html))
hotspot/src/share/vm/runtime/fprofiler.hpp                            |  同上
hotspot/src/share/vm/runtime/frame.cpp                                |  frame クラス関連のクラスの定義, 及び RegisterMap クラスのメソッドの定義 (※1) ([frame, frame::CheckValueClosure, frame::CheckOopClosure, frame::ZapDeadClosure, FrameValue, FrameValues, StackFrameStream, 及びそれらの補助クラス(InterpreterFrameClosure, InterpretedArgumentOopFinder, EntryFrameOopFinder, CompiledArgumentOopFinder)](no0OYYFeCj.html))
hotspot/src/share/vm/runtime/frame.hpp                                |  同上
hotspot/src/share/vm/runtime/frame.inline.hpp                         |  同上
hotspot/src/share/vm/runtime/globals.cpp                              |  コマンドラインオプション関連のクラスの定義, および HotSpot のコマンドラインオプションの定義 ([FlagSetting, CounterSetting, IntFlagSetting, DoubleFlagSetting, CommandLineFlags, CommandLineFlagsEx](nomJ5AL1-A.html)) (See: [here](no7882y_Y.html) for details)
hotspot/src/share/vm/runtime/globals.hpp                              |  同上
hotspot/src/share/vm/runtime/globals_extension.hpp                    |  同上
hotspot/src/share/vm/runtime/handles.cpp                              |  Handle クラス関連のクラスの定義 ([Handle, KlassHandle,  instanceHandle, methodHandle, constMethodHandle, methodDataHandle, arrayHandle, constantPoolHandle, constantPoolCacheHandle, objArrayHandle, typeArrayHandle,  instanceKlassHandle, methodKlassHandle, constMethodKlassHandle, klassKlassHandle, arrayKlassKlassHandle, objArrayKlassKlassHandle, typeArrayKlassKlassHandle, arrayKlassHandle, typeArrayKlassHandle, objArrayKlassHandle, constantPoolKlassHandle, constantPoolCacheKlassHandle,  HandleArea, HandleMark, NoHandleMark, ResetNoHandleMark](noxGayud0o.html))
hotspot/src/share/vm/runtime/handles.hpp                              |  同上
hotspot/src/share/vm/runtime/handles.inline.hpp                       |  同上
hotspot/src/share/vm/runtime/icache.cpp                               |  AbstractICache クラス関連のクラスの定義 ([AbstractICache, ICacheStubGenerator](noKL5uaVPv.html))
hotspot/src/share/vm/runtime/icache.hpp                               |  同上
hotspot/src/share/vm/runtime/init.cpp                                 |  HotSpot の初期化/終了時用関数の定義 (init_globals(), vm_init_globals(), exit_globals(),  is_init_completed(), set_init_completed())
hotspot/src/share/vm/runtime/init.hpp                                 |  同上
hotspot/src/share/vm/runtime/interfaceSupport.cpp                     |  InterfaceSupport 関連のクラス, およびVMへのエントリーポイントを示すマクロ群の定義 ([HandleMarkCleaner, InterfaceSupport, ThreadStateTransition, ThreadInVMfromJava, ThreadInVMfromUnknown, ThreadInVMfromNative, ThreadToNativeFromVM, ThreadBlockInVM, ThreadInVMfromJavaNoAsyncException, VMEntryWrapper, VMNativeEntryWrapper, RuntimeHistogramElement](noFtgIidMd.html))
hotspot/src/share/vm/runtime/interfaceSupport.hpp                     |  同上
hotspot/src/share/vm/runtime/java.cpp                                 |  JDK_Version クラスの定義, 及び終了時処理用のクラス/関数の定義 (before_exit(), vm_exit()) ([JDK_Version, ExitProc](noV-DtZh9-.html))
hotspot/src/share/vm/runtime/java.hpp                                 |  同上
hotspot/src/share/vm/runtime/javaCalls.cpp                            |  JavaCalls クラス関連のクラスの定義 ([JavaCallWrapper, JavaCallArguments, JavaCalls, 及びそれらの補助クラス(SignatureChekker)](no4A501dEe.html))
hotspot/src/share/vm/runtime/javaCalls.hpp                            |  同上
hotspot/src/share/vm/runtime/javaFrameAnchor.hpp                      |  JavaFrameAnchor クラスの定義 ([JavaFrameAnchor](no85946UfQ.html))
hotspot/src/share/vm/runtime/jfieldIDWorkaround.hpp                   |  jfieldIDWorkaround クラスの定義 ([jfieldIDWorkaround](noGWSHGS0a.html))
hotspot/src/share/vm/runtime/jniHandles.cpp                           |  JNIHandles クラス関連のクラスの定義 ([JNIHandles, JNIHandleBlock, 及びそれらの補助クラス(AlwaysAliveClosure, CountHandleClosure, VerifyHandleClosure)](noGITxrX0d.html))
hotspot/src/share/vm/runtime/jniHandles.hpp                           |  同上
hotspot/src/share/vm/runtime/jniPeriodicChecker.cpp                   |  JniPeriodicChecker クラス関連のクラスの定義 ([JniPeriodicChecker, 及びその補助クラス(JniPeriodicCheckerTask)](noj_6cTE8l.html))
hotspot/src/share/vm/runtime/jniPeriodicChecker.hpp                   |  同上
hotspot/src/share/vm/runtime/memprofiler.cpp                          |  MemProfiler クラスの定義 ([MemProfiler, 及びその補助クラス(MemProfilerTask)](nofvSVV6bl.html))
hotspot/src/share/vm/runtime/memprofiler.hpp                          |  同上
hotspot/src/share/vm/runtime/monitorChunk.cpp                         |  MonitorChunk クラスの定義 ([MonitorChunk](noF_xc9sVV.html))
hotspot/src/share/vm/runtime/monitorChunk.hpp                         |  同上
hotspot/src/share/vm/runtime/mutex.cpp                                |  Monitor クラスおよび Mutex クラスの定義 ([Monitor, Mutex](noetuCUkWu.html))
hotspot/src/share/vm/runtime/mutex.hpp                                |  同上
hotspot/src/share/vm/runtime/mutexLocker.cpp                          |  MutexLocker クラス関連のクラスの定義 ([MutexLocker, MutexLockerEx, MonitorLockerEx, GCMutexLocker, MutexUnlocker, MutexUnlockerEx, VerifyMutexLocker](no_dVTspXC.html))
hotspot/src/share/vm/runtime/mutexLocker.hpp                          |  同上
hotspot/src/share/vm/runtime/objectMonitor.cpp                        |  ObjectMonitor クラス関連のクラスの定義 ([ObjectWaiter, ObjectMonitor](noBIzmHjmm.html))
hotspot/src/share/vm/runtime/objectMonitor.hpp                        |  同上
hotspot/src/share/vm/runtime/objectMonitor.inline.hpp                 |  同上
hotspot/src/share/vm/runtime/orderAccess.cpp                          |  OrderAccess クラスの定義 ([OrderAccess](noqGF673Mc.html))
hotspot/src/share/vm/runtime/orderAccess.hpp                          |  同上
hotspot/src/share/vm/runtime/os.cpp                                   |  os クラスの定義 ([os](noaQj9VA38.html))
hotspot/src/share/vm/runtime/os.hpp                                   |  同上
hotspot/src/share/vm/runtime/osThread.cpp                             |  OSThread クラス関連のクラスの定義 (※2) ([OSThread, OSThreadWaitState, OSThreadContendState](norXAmbelO.html))
hotspot/src/share/vm/runtime/osThread.hpp                             |  同上
hotspot/src/share/vm/runtime/park.cpp                                 |  Parker クラスおよび ParkEvent クラスの定義 ([Parker, ParkEvent](noIpkvMBri.html))
hotspot/src/share/vm/runtime/park.hpp                                 |  同上
hotspot/src/share/vm/runtime/perfData.cpp                             |  PerfData クラス関連のクラスの定義 ([PerfData, PerfLongSampleHelper, PerfLong, PerfLongConstant, PerfLongVariant, PerfLongCounter, PerfLongVariable, PerfByteArray, PerfString, PerfStringConstant, PerfStringVariable, PerfDataList, PerfDataManager, PerfTraceTime, PerfTraceTimedEvent](noOFNhRgni.html))
hotspot/src/share/vm/runtime/perfData.hpp                             |  同上
hotspot/src/share/vm/runtime/perfMemory.cpp                           |  PerfMemory クラスの定義 ([PerfMemory](noAv9taKrY.html))
hotspot/src/share/vm/runtime/perfMemory.hpp                           |  同上
hotspot/src/share/vm/runtime/prefetch.hpp                             |  Prefetch クラスの定義 ([Prefetch](nolqawQHLE.html))
hotspot/src/share/vm/runtime/reflection.cpp                           |  Reflection クラスの定義 ([Reflection](nolC3LIO6Q.html))
hotspot/src/share/vm/runtime/reflection.hpp                           |  同上
hotspot/src/share/vm/runtime/reflectionCompat.hpp                     |  Reflection クラス用のマクロ定義 (※3)
hotspot/src/share/vm/runtime/reflectionUtils.cpp                      |  Reflection クラス関連のクラスの定義 ([KlassStream, MethodStream, FieldStream, FilteredField, FilteredFieldsMap, FilteredFieldStream](noB-Rh4Kew.html))
hotspot/src/share/vm/runtime/reflectionUtils.hpp                      |  同上
hotspot/src/share/vm/runtime/registerMap.hpp                          |  RegisterMap クラスの定義 (※1) ([RegisterMap](noSNzilF5q.html))
hotspot/src/share/vm/runtime/relocator.cpp                            |  Relocator クラス関連のクラスの定義 ([RelocatorListener, Relocator, 及びそれらの補助クラス(ChangeItem, ChangeWiden, ChangeJumpWiden, ChangeSwitchPad)](no_JLQ8hqv.html))
hotspot/src/share/vm/runtime/relocator.hpp                            |  同上
hotspot/src/share/vm/runtime/rframe.cpp                               |  RFrame クラス関連のクラスの定義 ([RFrame, CompiledRFrame, InterpretedRFrame, DeoptimizedRFrame](noKEilSnQO.html))
hotspot/src/share/vm/runtime/rframe.hpp                               |  同上
hotspot/src/share/vm/runtime/safepoint.cpp                            |  Safepoint クラス関連のクラスの定義 ([SafepointSynchronize, ThreadSafepointState](nokXNL1Etn.html))
hotspot/src/share/vm/runtime/safepoint.hpp                            |  同上
hotspot/src/share/vm/runtime/serviceThread.cpp                        |  ServiceThread クラスの定義 ([ServiceThread](noq0K1-Ofz.html))
hotspot/src/share/vm/runtime/serviceThread.hpp                        |  同上
hotspot/src/share/vm/runtime/sharedRuntime.cpp                        |  SharedRuntime クラス関連のクラスの定義  ([SharedRuntime, AdapterHandlerEntry, AdapterHandlerLibrary, 及びそれらの補助クラス(MethodArityHistogram, AdapterFingerPrint, AdapterHandlerTable, AdapterHandlerTableIterator)](no0uVSE4lu.html))
hotspot/src/share/vm/runtime/sharedRuntime.hpp                        |  同上
hotspot/src/share/vm/runtime/sharedRuntimeTrans.cpp                   |  SharedRuntime クラスの数学関連のメソッド、およびその補助関数の定義 (※4)
hotspot/src/share/vm/runtime/sharedRuntimeTrig.cpp                    |  SharedRuntime クラスの三角関数関連のメソッド、およびその補助関数の定義 (※4)
hotspot/src/share/vm/runtime/signature.cpp                            |  SignatureIterator クラス関連のクラスの定義 ([SignatureIterator, SignatureTypeNames, SignatureInfo, ArgumentSizeComputer, ArgumentCount, ResultTypeFinder, Fingerprinter, NativeSignatureIterator, SignatureStream, SignatureVerifier](nok0uluWHI.html))
hotspot/src/share/vm/runtime/signature.hpp                            |  同上
hotspot/src/share/vm/runtime/simpleThresholdPolicy.cpp                |  SimpleThresholdPolicy クラスの定義 ([SimpleThresholdPolicy](noCrFSBo2X.html))
hotspot/src/share/vm/runtime/simpleThresholdPolicy.hpp                |  同上
hotspot/src/share/vm/runtime/simpleThresholdPolicy.inline.hpp         |  同上
hotspot/src/share/vm/runtime/stackValue.cpp                           |  StackValue クラスの定義 ([StackValue](noIyzzHkPu.html))
hotspot/src/share/vm/runtime/stackValue.hpp                           |  同上
hotspot/src/share/vm/runtime/stackValueCollection.cpp                 |  StackValueCollection クラスの定義 ([StackValueCollection](nofn04j4Qb.html))
hotspot/src/share/vm/runtime/stackValueCollection.hpp                 |  同上
hotspot/src/share/vm/runtime/statSampler.cpp                          |  StatSampler クラスの定義 ([StatSampler, 及びその補助クラス(StatSamplerTask, HighResTimeSampler)](noygDxFlii.html))
hotspot/src/share/vm/runtime/statSampler.hpp                          |  同上
hotspot/src/share/vm/runtime/stubCodeGenerator.cpp                    |  StubCodeGenerator クラス関連のクラスの定義 ([StubCodeDesc, StubCodeGenerator, StubCodeMark](no8lVJhGE9.html))
hotspot/src/share/vm/runtime/stubCodeGenerator.hpp                    |  同上
hotspot/src/share/vm/runtime/stubRoutines.cpp                         |  StubRoutines クラスの定義 ([StubRoutines](noWUM0z5IQ.html))
hotspot/src/share/vm/runtime/stubRoutines.hpp                         |  同上
hotspot/src/share/vm/runtime/sweeper.cpp                              |  NMethodSweeper クラスの定義 ([NMethodSweeper, 及びその補助クラス(SweeperRecord, MarkActivationClosure, NMethodMarker)](noOfyfR8Cz.html))
hotspot/src/share/vm/runtime/sweeper.hpp                              |  同上
hotspot/src/share/vm/runtime/synchronizer.cpp                         |  ObjectSynchronizer クラスおよび ObjectLocker クラスの定義 ([ObjectSynchronizer, ObjectLocker, 及びそれらの補助クラス(ReleaseJavaMonitorsClosure)](noL3z0U0-A.html))
hotspot/src/share/vm/runtime/synchronizer.hpp                         |  同上
hotspot/src/share/vm/runtime/task.cpp                                 |  PeriodicTask クラスの定義 ([PeriodicTask](noabromtf6.html))
hotspot/src/share/vm/runtime/task.hpp                                 |  同上
hotspot/src/share/vm/runtime/thread.cpp                               |  Thread クラス関連のクラスの定義 ([Thread, NamedThread, WorkerThread, WatcherThread, JavaThread, CompilerThread, Threads, ThreadClosure, SignalHandlerMark, 及びそれらの補助クラス(TraceSuspendDebugBits, RememberProcessedThread)](no07LblyQD.html))
hotspot/src/share/vm/runtime/thread.hpp                               |  同上
hotspot/src/share/vm/runtime/threadCritical.hpp                       |  ThreadCritical クラスの定義 ([ThreadCritical](noyfXAAg0P.html))
hotspot/src/share/vm/runtime/threadLocalStorage.cpp                   |  ThreadLocalStorage クラスの定義 ([ThreadLocalStorage](noR_PxFGp9.html))
hotspot/src/share/vm/runtime/threadLocalStorage.hpp                   |  同上
hotspot/src/share/vm/runtime/timer.cpp                                |  処理時間計測用のユーティリティ・クラスの定義 ([elapsedTimer, TimeStamp, TraceTime, TraceCPUTime](nof9FlAy0H.html))
hotspot/src/share/vm/runtime/timer.hpp                                |  同上
hotspot/src/share/vm/runtime/unhandledOops.cpp                        |  UnhandledOops クラスおよびその補助クラスの定義 ([UnhandledOopEntry, UnhandledOops](noOhJxqpEq.html))
hotspot/src/share/vm/runtime/unhandledOops.hpp                        |  同上
hotspot/src/share/vm/runtime/vframe.cpp                               |  vframe クラス関連のクラスの定義 ([vframe, javaVFrame, interpretedVFrame, externalVFrame, entryVFrame, MonitorInfo, vframeStreamCommon, vframeStream](nomwEP34td.html))
hotspot/src/share/vm/runtime/vframe.hpp                               |  同上
hotspot/src/share/vm/runtime/vframeArray.cpp                          |  vframeArray クラス関連のクラスの定義 ([vframeArrayElement, vframeArray](noj6qSln0N.html))
hotspot/src/share/vm/runtime/vframeArray.hpp                          |  同上
hotspot/src/share/vm/runtime/vframe_hp.cpp                            |  compiledVFrame クラス関連のクラスの定義 ([compiledVFrame, jvmtiDeferredLocalVariableSet, jvmtiDeferredLocalVariable](no3Lj6udVS.html))
hotspot/src/share/vm/runtime/vframe_hp.hpp                            |  同上
hotspot/src/share/vm/runtime/virtualspace.cpp                         |  VirtualSpace クラス関連のクラスの定義 ([ReservedSpace, ReservedHeapSpace, ReservedCodeSpace, VirtualSpace](nokKhCIS4Z.html))
hotspot/src/share/vm/runtime/virtualspace.hpp                         |  同上
hotspot/src/share/vm/runtime/vmStructs.cpp                            |  VMStructs クラスの定義 ([VMStructs](noNecEKiR2.html))
hotspot/src/share/vm/runtime/vmStructs.hpp                            |  同上
hotspot/src/share/vm/runtime/vmThread.cpp                             |  VMThread クラス関連のクラスの定義 ([VMOperationQueue, VMThread, 及びそれらの補助クラス(VM_Dummy)](no_Ul4ZEtW.html))
hotspot/src/share/vm/runtime/vmThread.hpp                             |  同上
hotspot/src/share/vm/runtime/vm_operations.cpp                        |  VM_Operation クラスとその(基本的な)サブクラスの定義 ([VM_Operation, VM_ThreadStop, VM_ForceSafepoint, VM_ForceAsyncSafepoint, VM_Deoptimize, VM_DeoptimizeFrame, VM_HandleFullCodeCache, VM_DeoptimizeAll, VM_ZombieAll, VM_UnlinkSymbols, VM_Verify, VM_PrintThreads, VM_PrintJNI, VM_FindDeadlocks, VM_ThreadDump, VM_Exit](noZcemTpcs.html))
hotspot/src/share/vm/runtime/vm_operations.hpp                        |  同上
hotspot/src/share/vm/runtime/vm_version.cpp                           |  Abstract_VM_Version クラスの定義 ([Abstract_VM_Version](nom2UP9bKP.html))
hotspot/src/share/vm/runtime/vm_version.hpp                           |  同上

### 備考(Notes)
* ※1: RegisterMap クラスは, クラス定義は runtime/registerMap.hpp にあるが, メソッドは runtime/frame.cpp 及び cpu/ 下の frame_*.cpp で定義されている.

* ※2: なお, 実際の定義は os 依存部にある. ここでは cpp 内ではコンストラクタとデストラクタ(と情報の出力関数)が定義されているだけ.

* ※3: SUPPORT_OLD_REFLECTION を #define しているのみ.

* ※4: ファイル名の Trans は 超越関数 ("TRANScendental function"), Trig は 三角関数 ("TRIGonometric function") のことだと思われる.







