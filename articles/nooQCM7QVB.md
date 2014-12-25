---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： prims/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： prims/

--- 

File Name                                                    | Description
------------------------------------------------------------ | -----------------------------------------------------------------
hotspot/src/share/vm/prims/evmCompat.cpp                     |  ExactVM 時代の obsolete な公開 API の定義 (※1)
hotspot/src/share/vm/prims/forte.cpp                         |  Forte クラス関連のクラスの定義 ([Forte, vframeStreamForte](no0t68B6Eq.html))
hotspot/src/share/vm/prims/forte.hpp                         |  同上
hotspot/src/share/vm/prims/jni.cpp                           |  JNI 関数(JNI で外部に公開する関数)の定義(※2), 及びそのための補助クラスの定義 ([DTraceReturnProbeMark_*, JNITraceWrapper, JNIHistogramElement, JNI_ArgumentPusher, JNI_ArgumentPusherVaArg, JNI_ArgumentPusherArray](noaYsePyKB.html))
hotspot/src/share/vm/prims/jni.h                             |  JNI の仕様で定められている構造体/データ型/定数値/関数等の宣言 (※3)
hotspot/src/share/vm/prims/jniCheck.cpp                      |  jniCheck クラス, および checked_jni_NativeInterface 大域変数, jni_functions_check() 関数の定義 ([jniCheck](nok3Wwe7y8.html))
hotspot/src/share/vm/prims/jniCheck.hpp                      |  同上
hotspot/src/share/vm/prims/jniFastGetField.cpp               |  JNI_FastGetField クラスの定義 ([JNI_FastGetField](noDJcVP7zC.html))
hotspot/src/share/vm/prims/jniFastGetField.hpp               |  同上
hotspot/src/share/vm/prims/jni_md.h                          |  jni.h の machine dependent な内容の宣言 (※4)
hotspot/src/share/vm/prims/jvm.cpp                           |  CVMI 関数 (= 標準的な JNI 関数以外で HotSpot が外部に公開する関数) の定義(※5), 及びそのための補助クラスの定義 ([JVMTraceWrapper, JVMHistogramElement, RegisterArrayForGC, KlassLink](no88zay6h_.html))
hotspot/src/share/vm/prims/jvm.h                             |  同上 (<= こちらは基本的な内容? #TODO)
hotspot/src/share/vm/prims/jvm_misc.hpp                      |  同上 (<= こちらはその他的な内容? #TODO)
hotspot/src/share/vm/prims/jvmti.xml                         |  JVMTI 関連のソースコードの自動生成に用いられる XML ファイル (See: [here](no2935l2f.html) for details)
hotspot/src/share/vm/prims/jvmti.xsl                         |  jvmti.html (JVMTI 仕様を記述した HTML ファイル) を生成するための XSLT ファイル (See: [here](no2935l2f.html) for details)
hotspot/src/share/vm/prims/jvmtiAgentThread.hpp              |  JvmtiAgentThread クラスの宣言 ([JvmtiAgentThread](nokueCRvR2.html)) (※6)
hotspot/src/share/vm/prims/jvmtiClassFileReconstituter.cpp   |  JvmtiClassFileReconstituter クラス関連のクラスの定義 ([JvmtiConstantPoolReconstituter, JvmtiClassFileReconstituter](noq85pi-RX.html))
hotspot/src/share/vm/prims/jvmtiClassFileReconstituter.hpp   |  同上
hotspot/src/share/vm/prims/jvmtiCodeBlobEvents.cpp           |  JvmtiCodeBlobEvents クラスの定義 ([JvmtiCodeBlobEvents, 及びその補助クラス(CodeBlobCollector)](norUqEGBUg.html))
hotspot/src/share/vm/prims/jvmtiCodeBlobEvents.hpp           |  同上
hotspot/src/share/vm/prims/jvmtiEnter.hpp                    |  (空のファイル) (※7)
hotspot/src/share/vm/prims/jvmtiEnter.xsl                    |  jvmtiEnter.cpp と jvmtiEnterTrace.cpp を生成するための XSLT ファイル (See: [here](no2935l2f.html) for details)
hotspot/src/share/vm/prims/jvmtiEnv.cpp                      |  JvmtiEnv クラス関連のクラスの定義 ([JvmtiEnv, VM_JNIFunctionTableCopier](noovsUr_Kc.html)) (※8)
hotspot/src/share/vm/prims/jvmtiEnv.xsl                      |  jvmtiEnvRecommended.cpp を生成するための XSLT ファイル (See: [here](no2935l2f.html) for details)
hotspot/src/share/vm/prims/jvmtiEnvBase.cpp                  |  JvmtiEnvBase クラス関連のクラスの定義 ([JvmtiEnvBase, JvmtiEnvIterator, VM_GetOwnedMonitorInfo, VM_GetObjectMonitorUsage, VM_GetCurrentContendedMonitor, VM_GetStackTrace, VM_GetMultipleStackTraces, VM_GetAllStackTraces, VM_GetThreadListStackTraces, VM_GetFrameCount, VM_GetFrameLocation, ResourceTracker, JvmtiMonitorClosure, 及びそれらの補助クラス(ThreadInsideIterationClosure)](noKuV-kHhl.html))
hotspot/src/share/vm/prims/jvmtiEnvBase.hpp                  |  同上
hotspot/src/share/vm/prims/jvmtiEnvFill.java                 |  jvmtiEnvRecommended.cpp を生成するためのプログラム (jvmtiEnvFill) のソースコード (See: [here](no2935l2f.html) for details)
hotspot/src/share/vm/prims/jvmtiEnvThreadState.cpp           |  JvmtiEnvThreadState クラス関連のクラスの定義 ([JvmtiFramePop, JvmtiFramePops, JvmtiEnvThreadState, 及びそれらの補助クラス(VM_GetCurrentLocation)](noI_TTNwwM.html))
hotspot/src/share/vm/prims/jvmtiEnvThreadState.hpp           |  同上
hotspot/src/share/vm/prims/jvmtiEventController.cpp          |  JvmtiEventController クラス関連のクラスの定義 ([JvmtiEventEnabled, JvmtiEnvThreadEventEnable, JvmtiThreadEventEnable, JvmtiEnvEventEnable, JvmtiEventController, 及びそれらの補助クラス(VM_EnterInterpOnlyMode, VM_ChangeSingleStep, JvmtiEventControllerPrivate)](noOm5BIEEa.html))
hotspot/src/share/vm/prims/jvmtiEventController.hpp          |  同上
hotspot/src/share/vm/prims/jvmtiEventController.inline.hpp   |  同上
hotspot/src/share/vm/prims/jvmtiExport.cpp                   |  JvmtiExport クラス関連のクラスの定義 ([JvmtiExport, JvmtiCodeBlobDesc, JvmtiEventCollector, JvmtiDynamicCodeEventCollector, JvmtiVMObjectAllocEventCollector, NoJvmtiVMObjectAllocMark, JvmtiGCMarker, JvmtiHideSingleStepping, 及びそれらの補助クラス(JvmtiJavaThreadEventTransition, JvmtiThreadEventTransition, JvmtiEventMark, JvmtiThreadEventMark, JvmtiClassEventMark, JvmtiMethodEventMark, JvmtiLocationEventMark, JvmtiExceptionEventMark, JvmtiClassFileLoadEventMark, JvmtiClassFileLoadHookPoster, JvmtiVMObjectAllocEventMark, JvmtiCompiledMethodLoadEventMark, JvmtiMonitorEventMark)](noq11YIn7C.html))
hotspot/src/share/vm/prims/jvmtiExport.hpp                   |  同上
hotspot/src/share/vm/prims/jvmtiExtensions.cpp               |  JvmtiExtensions クラスの定義 ([JvmtiExtensions](noi9GSnAOK.html))
hotspot/src/share/vm/prims/jvmtiExtensions.hpp               |  同上
hotspot/src/share/vm/prims/jvmtiGen.java                     |  XSLT 処理用プログラム (jvmtiGen) のソースコード (See: [here](no2935l2f.html) for details)
hotspot/src/share/vm/prims/jvmtiGetLoadedClasses.cpp         |  JvmtiGetLoadedClasses クラスの定義 ([JvmtiGetLoadedClasses, 及びその補助クラス(JvmtiGetLoadedClassesClosure)](noka9LcuHd.html))
hotspot/src/share/vm/prims/jvmtiGetLoadedClasses.hpp         |  同上
hotspot/src/share/vm/prims/jvmtiH.xsl                        |  jvmti.h (JVMTI 仕様で定められている構造体や関数の型宣言が書かれたファイル) を生成するための XSLT ファイル (See: [here](no2935l2f.html) for details)
hotspot/src/share/vm/prims/jvmtiHpp.xsl                      |  jvmtiEnv.hpp を生成するための XSLT ファイル (See: [here](no2935l2f.html) for details)
hotspot/src/share/vm/prims/jvmtiImpl.cpp                     |  各種の JVMTI の処理で使われるクラスの定義 ([GrowableElement, GrowableCache, JvmtiBreakpointCache, JvmtiBreakpoint, VM_ChangeBreakpoints, JvmtiBreakpoints, JvmtiCurrentBreakpoints, VM_GetOrSetLocal, VM_GetReceiver, JvmtiSuspendControl, JvmtiDeferredEvent, JvmtiDeferredEventQueue, JvmtiDeferredEventQueue::QueueNode](no-rV36ySs.html))
hotspot/src/share/vm/prims/jvmtiImpl.hpp                     |  同上
hotspot/src/share/vm/prims/jvmtiLib.xsl                      |  他の XSL ファイル(jvmti*.xslファイル)から xsl:import されるライブラリファイル (See: [here](no2935l2f.html) for details)
hotspot/src/share/vm/prims/jvmtiManageCapabilities.cpp       |  JvmtiManageCapabilities クラスの定義 ([JvmtiManageCapabilities](noqdxnFgRU.html))
hotspot/src/share/vm/prims/jvmtiManageCapabilities.hpp       |  同上
hotspot/src/share/vm/prims/jvmtiRawMonitor.cpp               |  JvmtiRawMonitor クラス関連のクラスの定義 ([JvmtiRawMonitor, JvmtiPendingMonitors](noY5YaVAhE.html))
hotspot/src/share/vm/prims/jvmtiRawMonitor.hpp               |  同上
hotspot/src/share/vm/prims/jvmtiRedefineClasses.cpp          |  VM_RedefineClasses クラスの定義 ([VM_RedefineClasses, 及びその補助クラス(TransferNativeFunctionRegistration)](not2WPwAOy.html))
hotspot/src/share/vm/prims/jvmtiRedefineClasses.hpp          |  同上
hotspot/src/share/vm/prims/jvmtiRedefineClassesTrace.hpp     |  RedefineClasse 機能に関するトレース出力用のマクロの定義 (RC_TRACE*(), RC_TIMER_START(), RC_TIMER_STOP()) (※9)
hotspot/src/share/vm/prims/jvmtiTagMap.cpp                   |     (#TODO) JvmtiTagMap クラスの定義 ([JvmtiTagMap, 及びその補助クラス(JvmtiTagHashmapEntry, JvmtiTagHashmap, JvmtiTagHashmapEntryClosure, CallbackWrapper, TwoOopCallbackWrapper, ClassFieldDescriptor, ClassFieldMap, JvmtiCachedClassFieldMap, ClassFieldMapCacheMark, VM_HeapIterateOperation, IterateOverHeapObjectClosure, IterateThroughHeapObjectClosure, TagObjectCollector, RestoreMarksClosure, ObjectMarker, ObjectMarkerController, HeapWalkContext, BasicHeapWalkContext, AdvancedHeapWalkContext, CallbackInvoker, SimpleRootsClosure, JNILocalRootsClosure, VM_HeapWalkOperation)](nocwA2NXcs.html))
hotspot/src/share/vm/prims/jvmtiTagMap.hpp                   |  同上
hotspot/src/share/vm/prims/jvmtiThreadState.cpp              |  JvmtiThreadState クラス関連のクラスの定義 ([JvmtiEnvThreadStateIterator, JvmtiThreadState, RedefineVerifyMark](noc2f_bwrZ.html))
hotspot/src/share/vm/prims/jvmtiThreadState.hpp              |  同上
hotspot/src/share/vm/prims/jvmtiThreadState.inline.hpp       |  同上
hotspot/src/share/vm/prims/jvmtiTrace.cpp                    |  JvmtiTrace クラスの定義 ([JvmtiTrace](no92dy_yRJ.html))
hotspot/src/share/vm/prims/jvmtiTrace.hpp                    |  同上
hotspot/src/share/vm/prims/jvmtiUtil.cpp                     |  JvmtiUtil クラス及び SafeResourceMark クラスの定義 ([JvmtiUtil, SafeResourceMark](notDtCtQN_.html))
hotspot/src/share/vm/prims/jvmtiUtil.hpp                     |  同上
hotspot/src/share/vm/prims/methodComparator.cpp              |  MethodComparator クラスおよびその補助クラスの定義 ([MethodComparator, BciMap](nopDju2H7p.html))
hotspot/src/share/vm/prims/methodComparator.hpp              |  同上
hotspot/src/share/vm/prims/methodHandleWalk.cpp              |  MethodHandleWalker クラス関連のクラスの定義 ([MethodHandleChain, MethodHandleWalker, ArgToken, MethodHandleCompiler, ConstantValue, 及びそれらの補助クラス(MethodHandlePrinter)](noxcKtELBW.html))
hotspot/src/share/vm/prims/methodHandleWalk.hpp              |  同上
hotspot/src/share/vm/prims/methodHandles.cpp                 |  MethodHandles クラス関連のクラスの定義 ([MethodHandles, MethodHandleEntry, MethodHandleEntry::Data, MethodHandlesAdapterGenerator](nobhBlO8ok.html))
hotspot/src/share/vm/prims/methodHandles.hpp                 |  同上
hotspot/src/share/vm/prims/nativeLookup.cpp                  |  NativeLookup クラスの定義 ([NativeLookup](noroXXDJyH.html))
hotspot/src/share/vm/prims/nativeLookup.hpp                  |  同上
hotspot/src/share/vm/prims/perf.cpp                          |  sun.misc.Perf クラス用のネイティブメソッドの定義
hotspot/src/share/vm/prims/privilegedStack.cpp               |  PrivilegedElement クラスの定義 ([PrivilegedElement](noyLVCRN6q.html))
hotspot/src/share/vm/prims/privilegedStack.hpp               |  同上
hotspot/src/share/vm/prims/unsafe.cpp                        |  sun.misc.Unsafe クラス用のネイティブメソッドの定義

### 備考(Notes)
* ※1: といっても, 実行された場合は ShouldNotReachHere() になるだけの実装. 単に古い JDK をリンクエラーにしないためだけの措置.

```cpp
    ((cite: hotspot/src/share/vm/prims/evmCompat.cpp))
    // This file contains definitions for functions that exist
    // in the ExactVM, but not in HotSpot. They are stubbed out
    // here to prevent linker errors when attempting to use HotSpot
    // with the ExactVM jdk.
```

* ※2: 定義されている JNI 関数は, jni_GetEnv() 等の jni_* という名前の関数(JNINativeInterface_ 経由で公開)や, JNI_CreateJavaVM() 等の JNI_* という名前の関数.
  またその他に, チェック無しの JNINativeInterface_ である jni_NativeInterface 変数や, 適切な JNINativeInterface_ を返す jni_functions() 関数, JNIInvokeInterface_ である jni_InvokeInterface 変数等が定義されている.

* ※3: 中身は JDK をインストールするとついてくる jni.h と全く同じもの

* ※4: 中身は JDK をインストールするとついてくる jni_md.h と全く同じもの

* ※5: なお, CVMI 関数は全て JVM_*() という名称になっている.

* ※6: 実装は hotspot/src/share/vm/prims/jvmtiImpl.cpp にある

* ※7: #include 以外は何も書いていない

* ※8: JvmtiEnv クラスの宣言は jvmtiEnv.hpp で行われている (See: [here](no2935l2f.html) for details).

* ※9: これらは TraceRedefineClasses オプションが指定された場合にのみ使用される.







