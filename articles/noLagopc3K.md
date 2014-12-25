---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： utilities/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： utilities/

--- 

File Name                                                         | Description
----------------------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/vm/utilities/accessFlags.cpp                    |  AccessFlags クラスの定義 ([AccessFlags](no9RrSoNwq.html))
hotspot/src/share/vm/utilities/accessFlags.hpp			  |  同上
hotspot/src/share/vm/utilities/array.cpp			  |  ResourceArray クラス, CHeapArray クラス, およびそれらのサブクラスを定義するためのマクロ群の定義(define_generic_array(), define_array(), define_stack(), define_resource_list(), define_resource_pointer_list(), define_c_heap_list(), define_c_heap_pointer_list()) ([ResourceArray, CHeapArray](noZABXxUG2.html))
hotspot/src/share/vm/utilities/array.hpp			  |  同上
hotspot/src/share/vm/utilities/bitMap.cpp			  |  BitMap クラス関連のクラスの定義 ([BitMap, BitMap2D, BitMapClosure](noHtQ2lRjs.html))
hotspot/src/share/vm/utilities/bitMap.hpp			  |  同上
hotspot/src/share/vm/utilities/bitMap.inline.hpp		  |  同上
hotspot/src/share/vm/utilities/constantTag.cpp			  |  constantTag クラスの定義 ([constantTag](no4OPcgUEw.html))
hotspot/src/share/vm/utilities/constantTag.hpp			  |  同上
hotspot/src/share/vm/utilities/copy.cpp				  |  Copy クラスの定義 ([Copy](noAeV9yAxj.html))
hotspot/src/share/vm/utilities/copy.hpp				  |  同上
hotspot/src/share/vm/utilities/debug.cpp			  |  デバッグ用の関数/マクロおよびそれらの補助クラスの定義 ([FormatBuffer, Command, LookForRefInGenClosure, LookForRefInObjectClosure, FindClassObjectClosure](noOKTpHyYV.html)) (See: [here](no7882AEy.html) for details)
hotspot/src/share/vm/utilities/debug.hpp			  |  同上
hotspot/src/share/vm/utilities/decoder.cpp			  |  Decoder クラスの定義 ([Decoder](noWDhWK7vC.html))
hotspot/src/share/vm/utilities/decoder.hpp			  |  同上
hotspot/src/share/vm/utilities/defaultStream.hpp		  |  defaultStream クラスの宣言 (なお, 宣言したメソッドの定義は hotspot/src/share/vm/utilities/ostream.cpp で行っている) ([defaultStream](noswNLSGaQ.html))
hotspot/src/share/vm/utilities/dtrace.hpp			  |  HotSpot 内に DTrace のプローブポイントを埋め込むためのマクロ群(HS_DTRACE_PROBE*)の定義 (See: [here](no28916ldL.html) for details)
hotspot/src/share/vm/utilities/elfFile.cpp			  |  ElfFile クラスの定義 ([ElfFile](noFtACyy5H.html))
hotspot/src/share/vm/utilities/elfFile.hpp			  |  同上
hotspot/src/share/vm/utilities/elfStringTable.cpp		  |  ElfStringTable クラスの定義 ([ElfStringTable](noK0SD8q1c.html))
hotspot/src/share/vm/utilities/elfStringTable.hpp		  |  同上
hotspot/src/share/vm/utilities/elfSymbolTable.cpp		  |  ElfSymbolTable クラスの定義 ([ElfSymbolTable](nomy_KFBJ3.html))
hotspot/src/share/vm/utilities/elfSymbolTable.hpp		  |  同上
hotspot/src/share/vm/utilities/errorReporter.cpp		  |  ErrorReporter クラスの定義 ([ErrorReporter](nojv-2cPi8.html))
hotspot/src/share/vm/utilities/errorReporter.hpp		  |  同上
hotspot/src/share/vm/utilities/events.cpp			  |  Events クラス関連のクラスの定義 ([Events, EventMark, 及びそれらの補助クラス(Event, EventBuffer)](noxTtK1H6P.html))
hotspot/src/share/vm/utilities/events.hpp			  |  同上
hotspot/src/share/vm/utilities/exceptions.cpp			  |  VM 内部での例外生成/例外ハンドリングに関するクラス/関数/マクロの定義 ([ThreadShadow, Exceptions, ExceptionMark](noVQOY4-Bi.html))
hotspot/src/share/vm/utilities/exceptions.hpp			  |  同上
hotspot/src/share/vm/utilities/globalDefinitions.cpp              |  HotSpot 内で広く利用される定数や型宣言,ユーティリティ・クラス等の定義 ([HeapWord, Padded, Padded01, JavaValue](noD-c2GLBI.html))
hotspot/src/share/vm/utilities/globalDefinitions.hpp              |  同上
hotspot/src/share/vm/utilities/globalDefinitions_gcc.hpp          |  同上 (そのうちのコンパイラ依存の内容. gcc 用)
hotspot/src/share/vm/utilities/globalDefinitions_sparcWorks.hpp   |  同上 (そのうちのコンパイラ依存の内容. SparcWorks 用)
hotspot/src/share/vm/utilities/globalDefinitions_visCPP.hpp       |  同上 (そのうちのコンパイラ依存の内容. Visual C++ 用)
hotspot/src/share/vm/utilities/growableArray.cpp		  |  GrowableArray クラス関連のクラスの定義 ([GenericGrowableArray, GrowableArray](noHAQ8mj1b.html))
hotspot/src/share/vm/utilities/growableArray.hpp		  |  同上
hotspot/src/share/vm/utilities/hashtable.cpp			  |  HashTable クラス関連のクラスの定義 ([BasicHashtableEntry, HashtableEntry, HashtableBucket, BasicHashtable, Hashtable, TwoOopHashtable](noXK-YaL-S.html))
hotspot/src/share/vm/utilities/hashtable.hpp			  |  同上
hotspot/src/share/vm/utilities/hashtable.inline.hpp		  |  同上
hotspot/src/share/vm/utilities/histogram.cpp			  |  Histogram クラス関連のクラスの定義 ([HistogramElement, Histogram](noi163tmlK.html))
hotspot/src/share/vm/utilities/histogram.hpp			  |  同上
hotspot/src/share/vm/utilities/intHisto.cpp			  |  IntHistogram クラスの定義 ([IntHistogram](noLrXEqp7E.html))
hotspot/src/share/vm/utilities/intHisto.hpp			  |  同上
hotspot/src/share/vm/utilities/macros.hpp			  |  特定のコンパイル条件下でのみ使用するコードのためのマクロの定義 (COMPILER1_PRESENT(), CC_INTERP_ONLY(), etc)
hotspot/src/share/vm/utilities/numberSeq.cpp			  |  NumberSeq クラス関連のクラスの定義 ([AbsSeq, NumberSeq, TruncatedSeq](no-gSP771g.html))
hotspot/src/share/vm/utilities/numberSeq.hpp			  |  同上
hotspot/src/share/vm/utilities/ostream.cpp			  |  outputStream クラス関連のクラスの定義 ([outputStream, ttyLocker, ttyUnlocker, stringStream, fileStream, fdStream, staticBufferStream, bufferedStream, networkStream](noX6JV6NzV.html))
hotspot/src/share/vm/utilities/ostream.hpp			  |  同上
hotspot/src/share/vm/utilities/preserveException.cpp		  |  PreserveExceptionMark クラス関連のクラスの定義 ([PreserveExceptionMark, CautiouslyPreserveExceptionMark, WeakPreserveExceptionMark](noHvGcNOW7.html))
hotspot/src/share/vm/utilities/preserveException.hpp		  |  同上
hotspot/src/share/vm/utilities/sizes.cpp			  |  ByteSize クラスおよび WordSize クラスの定義 ([ByteSize, WordSize](noDoSV-PFG.html)) (なお, cpp の方は空ファイル)
hotspot/src/share/vm/utilities/sizes.hpp			  |  同上
hotspot/src/share/vm/utilities/stack.hpp			  |  Stack クラス関連のクラスの定義 ([StackBase, Stack, ResourceStack, StackIterator](noRZLXzyje.html))
hotspot/src/share/vm/utilities/stack.inline.hpp			  |  同上
hotspot/src/share/vm/utilities/taskqueue.cpp			  |  TaskQueue クラス関連のクラスの定義 ([TaskQueueStats, TaskQueueSuper, Age, GenericTaskQueue, OverflowTaskQueue, TaskQueueSetSuper, GenericTaskQueueSet, TerminatorTerminator, ParallelTaskTerminator, StarTask, ObjArrayTask](no3026s50.html))
hotspot/src/share/vm/utilities/taskqueue.hpp			  |  同上
hotspot/src/share/vm/utilities/top.hpp				  |  (空のファイル)  (※1)
hotspot/src/share/vm/utilities/utf8.cpp				  |  UTF8 クラスおよび UNICODE クラスの定義 ([UTF8, UNICODE](nocsNHC2w7.html))
hotspot/src/share/vm/utilities/utf8.hpp				  |  同上
hotspot/src/share/vm/utilities/vmError.cpp			  |  VMError クラスの定義 ([VMError, 及びその補助クラス(VM_ReportJavaOutOfMemory)](nofBaz8QYI.html))
hotspot/src/share/vm/utilities/vmError.hpp			  |  同上
hotspot/src/share/vm/utilities/workgroup.cpp			  |  WorkGang クラス関連のクラスの定義 ([AbstractGangTask, AbstractGangTaskWOopQueues, AbstractWorkGang, WorkData, WorkGang, GangWorker, FlexibleWorkGang, WorkGangBarrierSync, SubTasksDone, SequentialSubTasksDone, FreeIdSet](noBaH-n84s.html))
hotspot/src/share/vm/utilities/workgroup.hpp			  |  同上
hotspot/src/share/vm/utilities/xmlstream.cpp                      |  xmlStream クラス関連のクラスの定義 ([xmlTextStream, xmlStream](no0wuO-KFY.html))
hotspot/src/share/vm/utilities/xmlstream.hpp			  |  同上
hotspot/src/share/vm/utilities/yieldingWorkgroup.cpp		  |  YieldingFlexibleWorkGang クラス関連のクラスの定義 ([YieldingFlexibleGangWorker, FlexibleGangTask, YieldingFlexibleGangTask, YieldingFlexibleWorkGang](nooE5ojgIq.html))
hotspot/src/share/vm/utilities/yieldingWorkgroup.hpp		  |  同上

### 備考(Notes)
* ※1:
  #include 以外は何も書いていない

```cpp
    ((cite: hotspot/src/share/vm/utilities/top.hpp))
    // THIS FILE IS INTESIONALLY LEFT EMPTY
    // IT IS USED TO MINIMIZE THE NUMBER OF DEPENDENCIES IN includeDB
```







