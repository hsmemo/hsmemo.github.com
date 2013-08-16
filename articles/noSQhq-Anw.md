---
layout: default
title: HeapDumper クラス (HeapDumper, 及びその補助クラス(DumpWriter, DumperSupport, SymbolTableDumper, JNILocalsDumper, JNIGlobalsDumper, MonitorUsedDumper, StickyClassDumper, HeapObjectDumper, VM_HeapDumper))
---
[Top](../index.html)

#### HeapDumper クラス (HeapDumper, 及びその補助クラス(DumpWriter, DumperSupport, SymbolTableDumper, JNILocalsDumper, JNIGlobalsDumper, MonitorUsedDumper, StickyClassDumper, HeapObjectDumper, VM_HeapDumper))

これらは, 保守運用機能のためのクラス.
より具体的に言うと, Java ヒープの中身を (hprof フォーマットで) ファイルにダンプするためのクラス (See: [here](no2114JQO.html) for details).

```
    ((cite: hotspot/src/share/vm/services/heapDumper.hpp))
    // HeapDumper is used to dump the java heap to file in HPROF binary format:
```

#### 参考(for your information): hprof フォーマット

```
    ((cite: hotspot/src/share/vm/services/heapDumper.cpp))
    /*
     * HPROF binary format - description copied from:
     *   src/share/demo/jvmti/hprof/hprof_io.c
     *
     *
     *  header    "JAVA PROFILE 1.0.1" or "JAVA PROFILE 1.0.2"
     *            (0-terminated)
     *
     *  u4        size of identifiers. Identifiers are used to represent
     *            UTF8 strings, objects, stack traces, etc. They usually
     *            have the same size as host pointers. For example, on
     *            Solaris and Win32, the size is 4.
     * u4         high word
     * u4         low word    number of milliseconds since 0:00 GMT, 1/1/70
     * [record]*  a sequence of records.
     *
     *
     * Record format:
     *
     * u1         a TAG denoting the type of the record
     * u4         number of *microseconds* since the time stamp in the
     *            header. (wraps around in a little more than an hour)
     * u4         number of bytes *remaining* in the record. Note that
     *            this number excludes the tag and the length field itself.
     * [u1]*      BODY of the record (a sequence of bytes)
     *
     *
     * The following TAGs are supported:
     *
     * TAG           BODY       notes
     *----------------------------------------------------------
     * HPROF_UTF8               a UTF8-encoded name
     *
     *               id         name ID
     *               [u1]*      UTF8 characters (no trailing zero)
     *
     * HPROF_LOAD_CLASS         a newly loaded class
     *
     *                u4        class serial number (> 0)
     *                id        class object ID
     *                u4        stack trace serial number
     *                id        class name ID
     *
     * HPROF_UNLOAD_CLASS       an unloading class
     *
     *                u4        class serial_number
     *
     * HPROF_FRAME              a Java stack frame
     *
     *                id        stack frame ID
     *                id        method name ID
     *                id        method signature ID
     *                id        source file name ID
     *                u4        class serial number
     *                i4        line number. >0: normal
     *                                       -1: unknown
     *                                       -2: compiled method
     *                                       -3: native method
     *
     * HPROF_TRACE              a Java stack trace
     *
     *               u4         stack trace serial number
     *               u4         thread serial number
     *               u4         number of frames
     *               [id]*      stack frame IDs
     *
     *
     * HPROF_ALLOC_SITES        a set of heap allocation sites, obtained after GC
     *
     *               u2         flags 0x0001: incremental vs. complete
     *                                0x0002: sorted by allocation vs. live
     *                                0x0004: whether to force a GC
     *               u4         cutoff ratio
     *               u4         total live bytes
     *               u4         total live instances
     *               u8         total bytes allocated
     *               u8         total instances allocated
     *               u4         number of sites that follow
     *               [u1        is_array: 0:  normal object
     *                                    2:  object array
     *                                    4:  boolean array
     *                                    5:  char array
     *                                    6:  float array
     *                                    7:  double array
     *                                    8:  byte array
     *                                    9:  short array
     *                                    10: int array
     *                                    11: long array
     *                u4        class serial number (may be zero during startup)
     *                u4        stack trace serial number
     *                u4        number of bytes alive
     *                u4        number of instances alive
     *                u4        number of bytes allocated
     *                u4]*      number of instance allocated
     *
     * HPROF_START_THREAD       a newly started thread.
     *
     *               u4         thread serial number (> 0)
     *               id         thread object ID
     *               u4         stack trace serial number
     *               id         thread name ID
     *               id         thread group name ID
     *               id         thread group parent name ID
     *
     * HPROF_END_THREAD         a terminating thread.
     *
     *               u4         thread serial number
     *
     * HPROF_HEAP_SUMMARY       heap summary
     *
     *               u4         total live bytes
     *               u4         total live instances
     *               u8         total bytes allocated
     *               u8         total instances allocated
     *
     * HPROF_HEAP_DUMP          denote a heap dump
     *
     *               [heap dump sub-records]*
     *
     *                          There are four kinds of heap dump sub-records:
     *
     *               u1         sub-record type
     *
     *               HPROF_GC_ROOT_UNKNOWN         unknown root
     *
     *                          id         object ID
     *
     *               HPROF_GC_ROOT_THREAD_OBJ      thread object
     *
     *                          id         thread object ID  (may be 0 for a
     *                                     thread newly attached through JNI)
     *                          u4         thread sequence number
     *                          u4         stack trace sequence number
     *
     *               HPROF_GC_ROOT_JNI_GLOBAL      JNI global ref root
     *
     *                          id         object ID
     *                          id         JNI global ref ID
     *
     *               HPROF_GC_ROOT_JNI_LOCAL       JNI local ref
     *
     *                          id         object ID
     *                          u4         thread serial number
     *                          u4         frame # in stack trace (-1 for empty)
     *
     *               HPROF_GC_ROOT_JAVA_FRAME      Java stack frame
     *
     *                          id         object ID
     *                          u4         thread serial number
     *                          u4         frame # in stack trace (-1 for empty)
     *
     *               HPROF_GC_ROOT_NATIVE_STACK    Native stack
     *
     *                          id         object ID
     *                          u4         thread serial number
     *
     *               HPROF_GC_ROOT_STICKY_CLASS    System class
     *
     *                          id         object ID
     *
     *               HPROF_GC_ROOT_THREAD_BLOCK    Reference from thread block
     *
     *                          id         object ID
     *                          u4         thread serial number
     *
     *               HPROF_GC_ROOT_MONITOR_USED    Busy monitor
     *
     *                          id         object ID
     *
     *               HPROF_GC_CLASS_DUMP           dump of a class object
     *
     *                          id         class object ID
     *                          u4         stack trace serial number
     *                          id         super class object ID
     *                          id         class loader object ID
     *                          id         signers object ID
     *                          id         protection domain object ID
     *                          id         reserved
     *                          id         reserved
     *
     *                          u4         instance size (in bytes)
     *
     *                          u2         size of constant pool
     *                          [u2,       constant pool index,
     *                           ty,       type
     *                                     2:  object
     *                                     4:  boolean
     *                                     5:  char
     *                                     6:  float
     *                                     7:  double
     *                                     8:  byte
     *                                     9:  short
     *                                     10: int
     *                                     11: long
     *                           vl]*      and value
     *
     *                          u2         number of static fields
     *                          [id,       static field name,
     *                           ty,       type,
     *                           vl]*      and value
     *
     *                          u2         number of inst. fields (not inc. super)
     *                          [id,       instance field name,
     *                           ty]*      type
     *
     *               HPROF_GC_INSTANCE_DUMP        dump of a normal object
     *
     *                          id         object ID
     *                          u4         stack trace serial number
     *                          id         class object ID
     *                          u4         number of bytes that follow
     *                          [vl]*      instance field values (class, followed
     *                                     by super, super's super ...)
     *
     *               HPROF_GC_OBJ_ARRAY_DUMP       dump of an object array
     *
     *                          id         array object ID
     *                          u4         stack trace serial number
     *                          u4         number of elements
     *                          id         array class ID
     *                          [id]*      elements
     *
     *               HPROF_GC_PRIM_ARRAY_DUMP      dump of a primitive array
     *
     *                          id         array object ID
     *                          u4         stack trace serial number
     *                          u4         number of elements
     *                          u1         element type
     *                                     4:  boolean array
     *                                     5:  char array
     *                                     6:  float array
     *                                     7:  double array
     *                                     8:  byte array
     *                                     9:  short array
     *                                     10: int array
     *                                     11: long array
     *                          [u1]*      elements
     *
     * HPROF_CPU_SAMPLES        a set of sample traces of running threads
     *
     *                u4        total number of samples
     *                u4        # of traces
     *               [u4        # of samples
     *                u4]*      stack trace serial number
     *
     * HPROF_CONTROL_SETTINGS   the settings of on/off switches
     *
     *                u4        0x00000001: alloc traces on/off
     *                          0x00000002: cpu sampling on/off
     *                u2        stack trace depth
     *
     *
     * When the header is "JAVA PROFILE 1.0.2" a heap dump can optionally
     * be generated as a sequence of heap dump segments. This sequence is
     * terminated by an end record. The additional tags allowed by format
     * "JAVA PROFILE 1.0.2" are:
     *
     * HPROF_HEAP_DUMP_SEGMENT  denote a heap dump segment
     *
     *               [heap dump sub-records]*
     *               The same sub-record types allowed by HPROF_HEAP_DUMP
     *
     * HPROF_HEAP_DUMP_END      denotes the end of a heap dump
     *
     */
```



### クラス一覧(class list)

  * [HeapDumper](#noMFdAQvoz)
  * [DumpWriter](#nocCAPM4rT)
  * [DumperSupport](#nol5l0X8Go)
  * [SymbolTableDumper](#nozsFaiFKu)
  * [JNILocalsDumper](#nozxpoak_z)
  * [JNIGlobalsDumper](#noCGzan_UZ)
  * [MonitorUsedDumper](#noR-2LRo7D)
  * [StickyClassDumper](#nodkfy8VlC)
  * [HeapObjectDumper](#noVE8lQxhp)
  * [VM_HeapDumper](#norViZ-nV9)


---
## <a name="noMFdAQvoz" id="noMFdAQvoz">HeapDumper</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する serviceability 機能からのみ使用される).

Java ヒープの中身を (hprof フォーマットで) ファイルにダンプするための一時オブジェクト(StackObjクラス)
(See: [here](no2114JQO.html) for details).


```
    ((cite: hotspot/src/share/vm/services/heapDumper.hpp))
    class HeapDumper : public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
HeapDumper 型の局所変数を作り, HeapDumper::dump() メソッドを呼び出すだけ.


```
    ((cite: hotspot/src/share/vm/services/heapDumper.hpp))
    //  { HeapDumper dumper(true /* full GC before heap dump */);
    //    if (dumper.dump("/export/java.hprof")) {
    //      ResourceMark rm;
    //      tty->print_cr("Dump failed: %s", dumper.error_as_C_string());
    //    } else {
    //      // dump succeeded
    //    }
    //  }
```

あるいは, インスタンスを作らずに HeapDumper::dump_heap() を直接呼ぶという手もある模様
(この場合は内部でインスタンスが作られた後 dump() が呼び出される).

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* jmm_DumpHeap0()

  Platform MXBean である sun.management.HotSpotDiagnostic クラス用の補助関数.
  sun.management.HotSpotDiagnostic.dumpHeap() から(のみ)呼び出される.

* CollectedHeap::pre_full_gc_dump()

  保守運用機能用の関数.
  GC 実行前に呼び出され, 関連するコマンドラインオプションの値に応じてヒープの情報を出力する.
  HeapDumpBeforeFullGC オプションが指定されていると, HeapDumper が呼び出される.

* CollectedHeap::post_full_gc_dump()

  保守運用機能用の関数.
  GC 実行後に呼び出され, 関連するコマンドラインオプションの値に応じてヒープの情報を出力する.
  HeapDumpAfterFullGC オプションが指定されていると, HeapDumper が呼び出される.

* dump_heap()

  Attach API の "dumpheap" コマンドの処理を行う関数.




### 詳細(Details)
See: [here](../doxygen/classHeapDumper.html) for details

---
## <a name="nocCAPM4rT" id="nocCAPM4rT">DumpWriter</a>

### 概要(Summary)
HeapDumper クラス内で使用される補助クラス.

ダンプの出力先ファイルを管理する (See: [here](no2114JQO.html) for details).

```
    ((cite: hotspot/src/share/vm/services/heapDumper.cpp))
    // Supports I/O operations on a dump file
    
    class DumpWriter : public StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classDumpWriter.html) for details

---
## <a name="nol5l0X8Go" id="nol5l0X8Go">DumperSupport</a>

### 概要(Summary)
HeapDumper クラス内で使用される補助クラス.

ダンプ用のユーティリティ関数を納めた名前空間(AllStatic クラス) (See: [here](no2114JQO.html) for details).

```
    ((cite: hotspot/src/share/vm/services/heapDumper.cpp))
    // Support class with a collection of functions used when dumping the heap
    
    class DumperSupport : AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classDumperSupport.html) for details

---
## <a name="nozsFaiFKu" id="nozsFaiFKu">SymbolTableDumper</a>

### 概要(Summary)
HeapDumper クラス内で使用される補助クラス.

SymbolTable 内の情報を元に HPROF_UTF8 record を出力する (See: [here](no2114JQO.html) for details).

```
    ((cite: hotspot/src/share/vm/services/heapDumper.cpp))
    // Support class used to generate HPROF_UTF8 records from the entries in the
    // SymbolTable.
    
    class SymbolTableDumper : public SymbolClosure {
```




### 詳細(Details)
See: [here](../doxygen/classSymbolTableDumper.html) for details

---
## <a name="nozxpoak_z" id="nozxpoak_z">JNILocalsDumper</a>

### 概要(Summary)
HeapDumper クラス内で使用される補助クラス.

HPROF_GC_ROOT_JNI_LOCAL レコードの出力を行う (See: [here](no2114JQO.html) for details).

```
    ((cite: hotspot/src/share/vm/services/heapDumper.cpp))
    // Support class used to generate HPROF_GC_ROOT_JNI_LOCAL records
    
    class JNILocalsDumper : public OopClosure {
```




### 詳細(Details)
See: [here](../doxygen/classJNILocalsDumper.html) for details

---
## <a name="noCGzan_UZ" id="noCGzan_UZ">JNIGlobalsDumper</a>

### 概要(Summary)
HeapDumper クラス内で使用される補助クラス.

HPROF_GC_ROOT_JNI_GLOBAL レコードの出力を行う (See: [here](no2114JQO.html) for details).

```
    ((cite: hotspot/src/share/vm/services/heapDumper.cpp))
    // Support class used to generate HPROF_GC_ROOT_JNI_GLOBAL records
    
    class JNIGlobalsDumper : public OopClosure {
```




### 詳細(Details)
See: [here](../doxygen/classJNIGlobalsDumper.html) for details

---
## <a name="noR-2LRo7D" id="noR-2LRo7D">MonitorUsedDumper</a>

### 概要(Summary)
HeapDumper クラス内で使用される補助クラス.

HPROF_GC_ROOT_MONITOR_USED レコードの出力を行う (See: [here](no2114JQO.html) for details).

```
    ((cite: hotspot/src/share/vm/services/heapDumper.cpp))
    // Support class used to generate HPROF_GC_ROOT_MONITOR_USED records
    
    class MonitorUsedDumper : public OopClosure {
```




### 詳細(Details)
See: [here](../doxygen/classMonitorUsedDumper.html) for details

---
## <a name="nodkfy8VlC" id="nodkfy8VlC">StickyClassDumper</a>

### 概要(Summary)
HeapDumper クラス内で使用される補助クラス.

HPROF_GC_ROOT_STICKY_CLASS レコードの出力を行う (See: [here](no2114JQO.html) for details).

```
    ((cite: hotspot/src/share/vm/services/heapDumper.cpp))
    // Support class used to generate HPROF_GC_ROOT_STICKY_CLASS records
    
    class StickyClassDumper : public OopClosure {
```




### 詳細(Details)
See: [here](../doxygen/classStickyClassDumper.html) for details

---
## <a name="noVE8lQxhp" id="noVE8lQxhp">HeapObjectDumper</a>

### 概要(Summary)
HeapDumper クラス内で使用される補助クラス.

HPROF_GC_INSTANCE_DUMP レコードや HPROF_GC_OBJ_ARRAY_DUMPレコード, 
HPROF_GC_PRIM_ARRAY_DUMP レコードの出力を行う (See: [here](no2114JQO.html) for details).


```
    ((cite: hotspot/src/share/vm/services/heapDumper.cpp))
    // Support class using when iterating over the heap.
    
    class HeapObjectDumper : public ObjectClosure {
```




### 詳細(Details)
See: [here](../doxygen/classHeapObjectDumper.html) for details

---
## <a name="norViZ-nV9" id="norViZ-nV9">VM_HeapDumper</a>

### 概要(Summary)
HeapDumper クラス内で使用される補助クラス.

スレッドを停止させた状態でダンプ処理を行うための VM_GC_Operation クラス.
実際のダンプ処理はこのクラス内で行われる
(See: [here](no2114JQO.html) for details).


```
    ((cite: hotspot/src/share/vm/services/heapDumper.cpp))
    // The VM operation that performs the heap dump
    class VM_HeapDumper : public VM_GC_Operation {
```




### 詳細(Details)
See: [here](../doxygen/classVM__HeapDumper.html) for details

---
