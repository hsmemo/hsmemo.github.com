---
layout: default
title: PerfData クラス関連のクラス (PerfData, PerfLongSampleHelper, PerfLong, PerfLongConstant, PerfLongVariant, PerfLongCounter, PerfLongVariable, PerfByteArray, PerfString, PerfStringConstant, PerfStringVariable, PerfDataList, PerfDataManager, PerfTraceTime, PerfTraceTimedEvent)
---
[Top](../index.html)

#### PerfData クラス関連のクラス (PerfData, PerfLongSampleHelper, PerfLong, PerfLongConstant, PerfLongVariant, PerfLongCounter, PerfLongVariable, PerfByteArray, PerfString, PerfStringConstant, PerfStringVariable, PerfDataList, PerfDataManager, PerfTraceTime, PerfTraceTimedEvent)

これらは, 保守運用機能のためのクラス (PerfData 管理用のクラス. UsePerfData オプションが指定されている場合にのみ使用される)
(See: [here](no3420acA.html) for details)

### 概要(Summary)
PerfData クラス (とそのサブクラス) は jvmstat 機能 (PerfData 機能) が記録する統計情報を溜めておくためのクラス.

各 PerfData オブジェクトは, 
出力用の shared memory file ("hsperfdata" ファイル) 内にそれぞれ出力用の領域を持っており, 
StatSampler クラスによって定期的にその出力用領域に統計情報が書き出される 
(というか PerfData オブジェクト内の値がそのメモリ領域に反映される)
(See: PerfMemory, StatSampler).

(<= といっても, 値が変化しない場合 (= 定数の場合) は定期的に書き出す必要は無いし, 
現状では値が文字列の場合も共有メモリ上に直接確保されるので, 
書き出しが行われるのは PerfLongVariant (後述) の場合だけだが...)

なお, PerfData クラスの種別としては以下の3種類がある.

  * 定数(Constants)
    -  value is written to the PerfData memory once, on creation
  * 変数(Variables)
    -  value is modifiable, with no particular restrictions
  * カウンタ(Counters)
    -  value is monotonically changing (increasing or decreasing)

また, PerfData に入れられるデータの型としては以下の2つが用意されている.

 * Long
   - performance data holds a Java long type
 * ByteArray
   - performance data holds an array of Java bytes used for holding C++ char arrays.

このため, 具体的な PerfData クラスとしては (上の2つの掛け合わせとして) 以下のようなクラスが定義されている.

    - PerfData (Abstract)
        - PerfLong (Abstract)
            - PerfLongConstant        (alias: PerfConstant)
            - PerfLongVariant (Abstract)
                - PerfLongVariable    (alias: PerfVariable)
                - PerfLongCounter     (alias: PerfCounter)

        - PerfByteArray (Abstract)
            - PerfString (Abstract)
                - PerfStringVariable
                - PerfStringConstant

なお, 値の「単位」としては以下のようなものが定義されている (PerfData::Units 型も参照).

 * None
   - the data has no units of measure
 * Bytes
   - data is measured in bytes
 * Ticks
   - data is measured in clock ticks
 * Events
   - data is measured in events. For example, 
     the number of garbage collection events or the number of methods compiled.
 * String
   - data is not numerical. For example, the java command line options
 * Hertz
   - data is a frequency

また, 各 PerfData の名前空間(name space)によって, そのインターフェースの実装の安定性(stability)やサポートの有無が表現されている
(java.* で始まるものは stable, 等) (後述の enum CounterNS も参照).

 * java.*
   - stable, supported interface
 * com.sun.*
   - unstable, supported interface
 * sun.*
   - unstable, unsupported interface

なお, 現状では一度生成された PerfData オブジェクトは HotSpot の終了時まで消えることはない.
PerfData オブジェクトの生成には PerfDataManager クラスのファクトリメソッドを使用する.
生成された PerfData オブジェクトは全て PerfDataManager クラスが管理している.

PerfData の作成例/使用例:

    ...(以下のコメント内を参照)

なお, "Always-on non-sampled counters" は UsePerfData フラグの値にかかわらず作成できる
(UsePerfData フラグがオフの場合は C ヒープ上に作成される).
それ以外のものは UsePerfData フラグがオンの場合にだけ作成／使用するように, とのこと
(将来的にはこのフラグは消えるかもしれないが).


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * Classes to support access to production performance data
     *
     * The PerfData class structure is provided for creation, access, and update
     * of performance data (a.k.a. instrumentation) in a specific memory region
     * which is possibly accessible as shared memory. Although not explicitly
     * prevented from doing so, developers should not use the values returned
     * by accessor methods to make algorithmic decisions as they are potentially
     * extracted from a shared memory region. Although any shared memory region
     * created is with appropriate access restrictions, allowing read-write access
     * only to the principal that created the JVM, it is believed that a the
     * shared memory region facilitates an easier attack path than attacks
     * launched through mechanisms such as /proc. For this reason, it is
     * recommended that data returned by PerfData accessor methods be used
     * cautiously.
     *
     * There are three variability classifications of performance data
     *   Constants  -  value is written to the PerfData memory once, on creation
     *   Variables  -  value is modifiable, with no particular restrictions
     *   Counters   -  value is monotonically changing (increasing or decreasing)
     *
     * The performance data items can also have various types. The class
     * hierarchy and the structure of the memory region are designed to
     * accommodate new types as they are needed. Types are specified in
     * terms of Java basic types, which accommodates client applications
     * written in the Java programming language. The class hierarchy is:
     *
     * - PerfData (Abstract)
     *     - PerfLong (Abstract)
     *         - PerfLongConstant        (alias: PerfConstant)
     *         - PerfLongVariant (Abstract)
     *             - PerfLongVariable    (alias: PerfVariable)
     *             - PerfLongCounter     (alias: PerfCounter)
     *
     *     - PerfByteArray (Abstract)
     *         - PerfString (Abstract)
     *             - PerfStringVariable
     *             - PerfStringConstant
     *
     *
     * As seen in the class hierarchy, the initially supported types are:
     *
     *    Long      - performance data holds a Java long type
     *    ByteArray - performance data holds an array of Java bytes
     *                used for holding C++ char arrays.
     *
     * The String type is derived from the ByteArray type.
     *
     * A PerfData subtype is not required to provide an implementation for
     * each variability classification. For example, the String type provides
     * Variable and Constant variablility classifications in the PerfStringVariable
     * and PerfStringConstant classes, but does not provide a counter type.
     *
     * Performance data are also described by a unit of measure. Units allow
     * client applications to make reasonable decisions on how to treat
     * performance data generically, preventing the need to hard-code the
     * specifics of a particular data item in client applications. The current
     * set of units are:
     *
     *   None        - the data has no units of measure
     *   Bytes       - data is measured in bytes
     *   Ticks       - data is measured in clock ticks
     *   Events      - data is measured in events. For example,
     *                 the number of garbage collection events or the
     *                 number of methods compiled.
     *   String      - data is not numerical. For example,
     *                 the java command line options
     *   Hertz       - data is a frequency
     *
     * The performance counters also provide a support attribute, indicating
     * the stability of the counter as a programmatic interface. The support
     * level is also implied by the name space in which the counter is created.
     * The counter name space support conventions follow the Java package, class,
     * and property support conventions:
     *
     *    java.*          - stable, supported interface
     *    com.sun.*       - unstable, supported interface
     *    sun.*           - unstable, unsupported interface
     *
     * In the above context, unstable is a measure of the interface support
     * level, not the implementation stability level.
     *
     * Currently, instances of PerfData subtypes are considered to have
     * a life time equal to that of the VM and are managed by the
     * PerfDataManager class. All constructors for the PerfData class and
     * its subtypes have protected constructors. Creation of PerfData
     * instances is performed by invoking various create methods on the
     * PerfDataManager class. Users should not attempt to delete these
     * instances as the PerfDataManager class expects to perform deletion
     * operations on exit of the VM.
     *
     * Examples:
     *
     * Creating performance counter that holds a monotonically increasing
     * long data value with units specified in U_Bytes in the "java.gc.*"
     * name space.
     *
     *   PerfLongCounter* foo_counter;
     *
     *   foo_counter = PerfDataManager::create_long_counter(JAVA_GC, "foo",
     *                                                       PerfData::U_Bytes,
     *                                                       optionalInitialValue,
     *                                                       CHECK);
     *   foo_counter->inc();
     *
     * Creating a performance counter that holds a variably change long
     * data value with untis specified in U_Bytes in the "com.sun.ci
     * name space.
     *
     *   PerfLongVariable* bar_varible;
     *   bar_variable = PerfDataManager::create_long_variable(COM_CI, "bar",
    .*                                                        PerfData::U_Bytes,
     *                                                        optionalInitialValue,
     *                                                        CHECK);
     *
     *   bar_variable->inc();
     *   bar_variable->set_value(0);
     *
     * Creating a performance counter that holds a constant string value in
     * the "sun.cls.*" name space.
     *
     *   PerfDataManager::create_string_constant(SUN_CLS, "foo", string, CHECK);
     *
     *   Although the create_string_constant() factory method returns a pointer
     *   to the PerfStringConstant object, it can safely be ignored. Developers
     *   are not encouraged to access the string constant's value via this
     *   pointer at this time due to security concerns.
     *
     * Creating a performance counter in an arbitrary name space that holds a
     * value that is sampled by the StatSampler periodic task.
     *
     *    PerfDataManager::create_counter("foo.sampled", PerfData::U_Events,
     *                                    &my_jlong, CHECK);
     *
     *    In this example, the PerfData pointer can be ignored as the caller
     *    is relying on the StatSampler PeriodicTask to sample the given
     *    address at a regular interval. The interval is defined by the
     *    PerfDataSamplingInterval global variable, and is applyied on
     *    a system wide basis, not on an per-counter basis.
     *
     * Creating a performance counter in an arbitrary name space that utilizes
     * a helper object to return a value to the StatSampler via the take_sample()
     * method.
     *
     *     class MyTimeSampler : public PerfLongSampleHelper {
     *       public:
     *         jlong take_sample() { return os::elapsed_counter(); }
     *     };
     *
     *     PerfDataManager::create_counter(SUN_RT, "helped",
     *                                     PerfData::U_Ticks,
     *                                     new MyTimeSampler(), CHECK);
     *
     *     In this example, a subtype of PerfLongSampleHelper is instantiated
     *     and its take_sample() method is overridden to perform whatever
     *     operation is necessary to generate the data sample. This method
     *     will be called by the StatSampler at a regular interval, defined
     *     by the PerfDataSamplingInterval global variable.
     *
     *     As before, PerfSampleHelper is an alias for PerfLongSampleHelper.
     *
     * For additional uses of PerfData subtypes, see the utility classes
     * PerfTraceTime and PerfTraceTimedEvent below.
     *
     * Always-on non-sampled counters can be created independent of
     * the UsePerfData flag. Counters will be created on the c-heap
     * if UsePerfData is false.
     *
     * Until further noice, all PerfData objects should be created and
     * manipulated within a guarded block. The guard variable is
     * UsePerfData, a product flag set to true by default. This flag may
     * be removed from the product in the future.
     *
     */
```

### 備考(Notes)
PerfData オブジェクトの名前空間としては, 以下のようなものが用意されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /* jvmstat global and subsystem counter name space - enumeration value
     * serve as an index into the PerfDataManager::_name_space[] array
     * containing the corresponding name space string. Only the top level
     * subsystem name spaces are represented here.
     */
    enum CounterNS {
      // top level name spaces
      JAVA_NS,
      COM_NS,
      SUN_NS,
      // subsystem name spaces
      JAVA_GC,              // Garbage Collection name spaces
      COM_GC,
      SUN_GC,
      JAVA_CI,              // Compiler name spaces
      COM_CI,
      SUN_CI,
      JAVA_CLS,             // Class Loader name spaces
      COM_CLS,
      SUN_CLS,
      JAVA_RT,              // Runtime name spaces
      COM_RT,
      SUN_RT,
      JAVA_OS,              // Operating System name spaces
      COM_OS,
      SUN_OS,
      JAVA_THREADS,         // Threads System name spaces
      COM_THREADS,
      SUN_THREADS,
      JAVA_PROPERTY,        // Java Property name spaces
      COM_PROPERTY,
      SUN_PROPERTY,
      NULL_NS,
      COUNTERNS_LAST = NULL_NS
    };
```


### クラス一覧(class list)

  * [PerfData](#noKIT1KelE)
  * [PerfLong](#noYmXvqvDW)
  * [PerfLongConstant](#nodg9GAPHv)
  * [PerfLongVariant](#noGCyzcgPP)
  * [PerfLongSampleHelper](#nohh44gebX)
  * [PerfLongCounter](#noBT7tn-xX)
  * [PerfLongVariable](#noUR-lUIpI)
  * [PerfByteArray](#noBDFeanVq)
  * [PerfString](#noLWbOhb28)
  * [PerfStringConstant](#nogxOF-TgW)
  * [PerfStringVariable](#noFsUnFYS8)
  * [PerfDataList](#noQVCMWiED)
  * [PerfDataManager](#nogj8MEDng)
  * [PerfTraceTime](#noHcXFs940)
  * [PerfTraceTimedEvent](#nojVyOaWAy)


---
## <a name="noKIT1KelE" id="noKIT1KelE">PerfData</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス. UsePerfData オプションが指定されている場合にのみ使用される).

jvmstat 機能が記録する統計情報を溜めておくためのクラス (の基底クラス) (See: [here](no3420acA.html) for details).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    class PerfData : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classPerfData.html) for details

---
## <a name="noYmXvqvDW" id="noYmXvqvDW">PerfLong</a>

### 概要(Summary)
PerfData クラスのサブクラスの1つ.

このクラスは数値データ(long 値)を格納する場合用.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * PerfLong is the base class for the various Long PerfData subtypes.
     * it contains implementation details that are common among its derived
     * types.
     */
    class PerfLong : public PerfData {
```




### 詳細(Details)
See: [here](../doxygen/classPerfLong.html) for details

---
## <a name="nodg9GAPHv" id="nodg9GAPHv">PerfLongConstant</a>

### 概要(Summary)
PerfLong クラスの具象サブクラスの1つ.

このクラスは, 定数の数値データ(long 値)を格納する場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * The PerfLongConstant class, and its alias PerfConstant, implement
     * a PerfData subtype that holds a jlong data value that is set upon
     * creation of an instance of this class. This class provides no
     * methods for changing the data value stored in PerfData memory region.
     */
    class PerfLongConstant : public PerfLong {
```

なお PerfConstant という型も使われるが, これは PerfLongConstant の別名.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    typedef PerfLongConstant PerfConstant;
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
HotSpot の様々なクラスの中に格納されている (#TODO).

なお, PerfDataManager 内の以下のフィールドにも格納されている.

* PerfDataManager クラスの _all フィールド (static フィールド)
  
  (正確には, このフィールドは PerfDataList オブジェクトを格納するフィールド.
  この中に, 生成された全ての PerfLongConstant オブジェクトが格納されている)
  
  (なお, このフィールドへの登録は PerfDataManager::add_item() で行われる)

* PerfDataManager クラスの _constants フィールド (static フィールド)
  
  (正確には, このフィールドは PerfDataList オブジェクトを格納するフィールド.
  この中に, 生成された全ての PerfLongConstant オブジェクトが格納されている)

  (なお, このフィールドへの登録は PerfDataManager::add_item() で行われる)

#### 生成箇所(where its instances are created)
PerfDataManager::create_long_constant() というファクトリメソッドが用意されており, その中で(のみ)生成されている.

(なお, PerfDataManager::create_constant() というファクトリメソッドも用意されているが, 
 これは PerfDataManager::create_long_constant() のラッパー)




### 詳細(Details)
See: [here](../doxygen/classPerfLongConstant.html) for details

---
## <a name="noGCyzcgPP" id="noGCyzcgPP">PerfLongVariant</a>

### 概要(Summary)
PerfData クラスのサブクラスの1つ.

このクラスは変更可能な数値データ(long 値)を格納する場合用.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * The PerfLongVariant class, and its alias PerfVariant, implement
     * a PerfData subtype that holds a jlong data value that can be modified
     * in an unrestricted manner. This class provides the implementation details
     * for common functionality among its derived types.
     */
    class PerfLongVariant : public PerfLong {
```




### 詳細(Details)
See: [here](../doxygen/classPerfLongVariant.html) for details

---
## <a name="nohh44gebX" id="nohh44gebX">PerfLongSampleHelper</a>

### 概要(Summary)
PerfLongVariant クラス用の補助クラス(の基底クラス).

PerfLongVariant オブジェクトは, StatSampler によって定期的に値が shared memory に書き込まれる.
PerfLongSampleHelper クラスは, StatSampler が書き出すタイミングで値を動的に計算したい場合に使用される.

(例えばメモリの使用量情報を表す PerfLongVariant なんかの場合, 
常時最新の値を持っておくのは無駄なので, PerfLongSampleHelper を使って書き出しのタイミングで使用量を取得したりしている)

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * PerfLongSampleHelper, and its alias PerfSamplerHelper, is a base class
     * for helper classes that rely upon the StatSampler periodic task to
     * invoke the take_sample() method and write the value returned to its
     * appropriate location in the PerfData memory region.
     */
    class PerfLongSampleHelper : public CHeapObj {
```

なお PerfSampleHelper という型も使われるが, これは PerfLongSampleHelper の別名.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    typedef PerfLongSampleHelper PerfSampleHelper;
```

### 使われ方(Usage)
使用する際には, take_sample() メソッドをオーバーライドしたサブクラスを作ればいい
(このメソッドの返値が shared memory に書き込まれる).


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
        virtual jlong take_sample() = 0;
```




### 詳細(Details)
See: [here](../doxygen/classPerfLongSampleHelper.html) for details

---
## <a name="noBT7tn-xX" id="noBT7tn-xX">PerfLongCounter</a>

### 概要(Summary)
PerfLongVariant クラスの具象サブクラスの1つ.

このクラスは, 単調に増加／減少する数値データ(long 値)を格納する場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * The PerfLongCounter class, and its alias PerfCounter, implement
     * a PerfData subtype that holds a jlong data value that can (should)
     * be modified in a monotonic manner. The inc(jlong) and add(jlong)
     * methods can be passed negative values to implement a monotonically
     * decreasing value. However, we rely upon the programmer to honor
     * the notion that this counter always moves in the same direction -
     * either increasing or decreasing.
     */
    class PerfLongCounter : public PerfLongVariant {
```

なお PerfCounter という型も使われるが, これは PerfLongCounter の別名.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    typedef PerfLongCounter PerfCounter;
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
HotSpot の様々なクラスの中に格納されている (#TODO).

なお, PerfDataManager 内の以下のフィールドにも格納されている.

* PerfDataManager クラスの _all フィールド (static フィールド)
  
  (正確には, このフィールドは PerfDataList オブジェクトを格納するフィールド.
  この中に, 生成された全ての PerfLongCounter オブジェクトが格納されている)
  
  (なお, このフィールドへの登録は PerfDataManager::add_item() で行われる)

* PerfDataManager クラスの _sampled フィールド (static フィールド)
  
  (正確には, このフィールドは PerfDataList オブジェクトを格納するフィールド.
  この中に, StatSampler の処理対象となる全ての PerfLongCounter オブジェクトが格納されている)

  (なお, このフィールドへの登録は PerfDataManager::add_item() で行われる)

#### 生成箇所(where its instances are created)
以下のファクトリメソッドが用意されており, その中で(のみ)生成されている.

* PerfDataManager::create_long_counter(CounterNS ns, const char* name, PerfData::Units u, jlong ival, TRAPS)
* PerfDataManager::create_long_counter(CounterNS ns, const char* name, PerfData::Units u, jlong* sp, TRAPS)
* PerfDataManager::create_long_counter(CounterNS ns, const char* name, PerfData::Units u, PerfSampleHelper* sh, TRAPS)

(なお, PerfDataManager::create_counter() というファクトリメソッドも用意されているが, 
 これは PerfDataManager::create_long_counter() のラッパー)




### 詳細(Details)
See: [here](../doxygen/classPerfLongCounter.html) for details

---
## <a name="noUR-lUIpI" id="noUR-lUIpI">PerfLongVariable</a>

### 概要(Summary)
PerfLongVariant クラスの具象サブクラスの1つ.

このクラスは, 単調ではない変化の仕方をする数値データ(long 値)を格納する場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * The PerfLongVariable class, and its alias PerfVariable, implement
     * a PerfData subtype that holds a jlong data value that can
     * be modified in an unrestricted manner.
     */
    class PerfLongVariable : public PerfLongVariant {
```

なお PerfVariable という型も使われるが, これは PerfLongVariable の別名.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    typedef PerfLongVariable PerfVariable;
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
HotSpot の様々なクラスの中に格納されている (#TODO).

なお, PerfDataManager 内の以下のフィールドにも格納されている.

* PerfDataManager クラスの _all フィールド (static フィールド)
  
  (正確には, このフィールドは PerfDataList オブジェクトを格納するフィールド.
  この中に, 生成された全ての PerfLongVariable オブジェクトが格納されている)
  
  (なお, このフィールドへの登録は PerfDataManager::add_item() で行われる)

* PerfDataManager クラスの _sampled フィールド (static フィールド)
  
  (正確には, このフィールドは PerfDataList オブジェクトを格納するフィールド.
  この中に, StatSampler の処理対象となる全ての PerfLongVariable オブジェクトが格納されている)

  (なお, このフィールドへの登録は PerfDataManager::add_item() で行われる)

#### 生成箇所(where its instances are created)
以下のファクトリメソッドが用意されており, その中で(のみ)生成されている.

* PerfDataManager::create_long_variable(CounterNS ns, const char* name, PerfData::Units u, jlong ival, TRAPS)
* PerfDataManager::create_long_variable(CounterNS ns, const char* name, PerfData::Units u, jlong* sp, TRAPS)
* PerfDataManager::create_long_variable(CounterNS ns, const char* name, PerfData::Units u, PerfSampleHelper* sh, TRAPS)

(なお, PerfDataManager::create_variable() というファクトリメソッドも用意されているが, 
 これは PerfDataManager::create_long_variable() のラッパー)




### 詳細(Details)
See: [here](../doxygen/classPerfLongVariable.html) for details

---
## <a name="noBDFeanVq" id="noBDFeanVq">PerfByteArray</a>

### 概要(Summary)
PerfData クラスのサブクラスの1つ.

このクラスはバイト列を格納する場合用.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * The PerfByteArray provides a PerfData subtype that allows the creation
     * of a contiguous region of the PerfData memory region for storing a vector
     * of bytes. This class is currently intended to be a base class for
     * the PerfString class, and cannot be instantiated directly.
     */
    class PerfByteArray : public PerfData {
```




### 詳細(Details)
See: [here](../doxygen/classPerfByteArray.html) for details

---
## <a name="noLWbOhb28" id="noLWbOhb28">PerfString</a>

### 概要(Summary)
PerfByteArray クラスのサブクラスの1つ.

このクラスは文字列データ(string 値)を格納する場合用.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    class PerfString : public PerfByteArray {
```




### 詳細(Details)
See: [here](../doxygen/classPerfString.html) for details

---
## <a name="nogxOF-TgW" id="nogxOF-TgW">PerfStringConstant</a>

### 概要(Summary)
PerfString クラスの具象サブクラスの1つ.

このクラスは, 定数の文字列データ(string 値)を格納する場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * The PerfStringConstant class provides a PerfData sub class that
     * allows a null terminated string of single byte characters to be
     * stored in the PerfData memory region.
     */
    class PerfStringConstant : public PerfString {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* PerfDataManager クラスの _all フィールド (static フィールド)
  
  (正確には, このフィールドは PerfDataList オブジェクトを格納するフィールド.
  この中に, 生成された全ての PerfStringConstant オブジェクトが格納されている)
  
  (なお, このフィールドへの登録は PerfDataManager::add_item() で行われる)

* PerfDataManager クラスの _constants フィールド (static フィールド)
  
  (正確には, このフィールドは PerfDataList オブジェクトを格納するフィールド.
  この中に, 生成された全ての PerfStringConstant オブジェクトが格納されている)

  (なお, このフィールドへの登録は PerfDataManager::add_item() で行われる)

#### 生成箇所(where its instances are created)
PerfDataManager::create_string_constant() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classPerfStringConstant.html) for details

---
## <a name="noFsUnFYS8" id="noFsUnFYS8">PerfStringVariable</a>

### 概要(Summary)
PerfString クラスの具象サブクラスの1つ.

このクラスは, 変更可能な文字列データ(string 値)を格納する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * The PerfStringVariable class provides a PerfData sub class that
     * allows a null terminated string of single byte character data
     * to be stored in PerfData memory region. The string value can be reset
     * after initialization. If the string value is >= max_length, then
     * it will be truncated to max_length characters. The copied string
     * is always null terminated.
     */
    class PerfStringVariable : public PerfString {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
HotSpot の様々なクラスの中に格納されている (#TODO).

なお, PerfDataManager 内の以下のフィールドにも格納されている.

* PerfDataManager クラスの _all フィールド (static フィールド)
  
  (正確には, このフィールドは PerfDataList オブジェクトを格納するフィールド.
  この中に, 生成された全ての PerfStringConstant オブジェクトが格納されている)
  
  (なお, このフィールドへの登録は PerfDataManager::add_item() で行われる)

#### 生成箇所(where its instances are created)
PerfDataManager::create_string_variable() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classPerfStringVariable.html) for details

---
## <a name="noQVCMWiED" id="noQVCMWiED">PerfDataList</a>

### 概要(Summary)
PerfData オブジェクトを束ねておくためのコンテナクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * The PerfDataList class is a container class for managing lists
     * of PerfData items. The intention of this class is to allow for
     * alternative implementations for management of list of PerfData
     * items without impacting the code that uses the lists.
     *
     * The initial implementation is based upon GrowableArray. Searches
     * on GrowableArray types is linear in nature and this may become
     * a performance issue for creation of PerfData items, particularly
     * from Java code where a test for existence is implemented as a
     * search over all existing PerfData items.
     *
     * The abstraction is not complete. A more general container class
     * would provide an Iterator abstraction that could be used to
     * traverse the lists. This implementation still relys upon integer
     * iterators and the at(int index) method. However, the GrowableArray
     * is not directly visible outside this class and can be replaced by
     * some other implementation, as long as that implementation provides
     * a mechanism to iterate over the container by index.
     */
    class PerfDataList : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* PerfDataManager クラスの _all フィールド (static フィールド)
  
  この中に, 生成された全ての PerfData オブジェクトが格納されている.

* PerfDataManager クラスの _sampled フィールド (static フィールド)
  
  この中に, StatSampler の処理対象となる全ての PerfData オブジェクトが格納されている.

* PerfDataManager クラスの _constants フィールド (static フィールド)
  
  この中に, 値が固定の (= 種別が PerfData::V_Constant である) 全ての PerfData オブジェクトが格納されている.

* StatSampler クラスの _sampled フィールド (static フィールド)
  
  この中に, StatSampler の処理対象となる全ての PerfData オブジェクトが格納されている
  (内容は PerfDataManager::_sampled からコピーしたもの).

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* PerfDataManager::add_item()
* PerfDataList::clone()
  
そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

```
* 各種 PerfData の生成時
  
  PerfDataManager::create_long_constant(CounterNS ns, const char* name, PerfData::Units u, jlong val, TRAPS)
  -> PerfDataManager::add_item()

  PerfDataManager::create_long_variable(CounterNS ns, const char* name, PerfData::Units u, jlong ival, TRAPS)
  -> PerfDataManager::add_item()

  PerfDataManager::create_long_variable(CounterNS ns, const char* name, PerfData::Units u, jlong* sp, TRAPS)
  -> PerfDataManager::add_item()

  PerfDataManager::create_long_variable(CounterNS ns, const char* name, PerfData::Units u, PerfSampleHelper* sh, TRAPS)
  -> PerfDataManager::add_item()

  PerfDataManager::create_long_counter(CounterNS ns, const char* name, PerfData::Units u, jlong ival, TRAPS)
  -> PerfDataManager::add_item()

  PerfDataManager::create_long_counter(CounterNS ns, const char* name, PerfData::Units u, jlong* sp, TRAPS)
  -> PerfDataManager::add_item()

  PerfDataManager::create_long_counter(CounterNS ns, const char* name, PerfData::Units u, PerfSampleHelper* sh, TRAPS)
  -> PerfDataManager::add_item()

  PerfDataManager::create_string_constant(CounterNS ns, const char* name, const char* s, TRAPS)
  -> PerfDataManager::add_item()

  PerfDataManager::create_string_variable(CounterNS ns, const char* name, jint max_length, const char* s, TRAPS)
  -> PerfDataManager::add_item()

* StatSampler の開始時
  
  (HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
  -> Threads::create_vm()
     -> StatSampler::engage()
        -> StatSampler::initialize()
           -> PerfDataManager::sampled()
              -> PerfDataList::clone()

* ?? (使用箇所が見当たらない)
  
  ??
  -> PerfDataManager::all()
     -> PerfDataList::clone()

  ??
  -> PerfDataManager::constants()
     -> PerfDataList::clone()
```



### 詳細(Details)
See: [here](../doxygen/classPerfDataList.html) for details

---
## <a name="nogj8MEDng" id="nogj8MEDng">PerfDataManager</a>

### 概要(Summary)
PerfData を管理するための機能を納めた名前空間(AllStatic クラス).

以下のような機能を提供している.

* PerfData を生成するファクトリメソッド
* 生成された全ての PerfData オブジェクトを管理する機能


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * The PerfDataManager class is responsible for creating PerfData
     * subtypes via a set a factory methods and for managing lists
     * of the various PerfData types.
     */
    class PerfDataManager : AllStatic {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classPerfDataManager.html) for details

---
## <a name="noHcXFs940" id="noHcXFs940">PerfTraceTime</a>

### 概要(Summary)
処理時間の簡単な計測処理を行うためのユーティリティ・クラス(StackObjクラス).

処理時間の計測処理をソースコード上のスコープに合わせて行うことができる
(= コンストラクタが呼ばれてからデストラクタが呼ばれるまでの時間を計測する)
(TraceTime クラスに類似したクラス).

なお, 計測結果の格納に PerfData を用いている
(より正確に言うと, コンストラクタ引数で指定された PerfLongCounter に結果を格納する).

(なお, このクラス内で UsePerfData を確認しているので, 使用点では UsePerfData を確認する必要は無い)


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /*
     * this class will administer a PerfCounter used as a time accumulator
     * for a basic block much like the TraceTime class.
     *
     * Example:
     *
     *    static PerfCounter* my_time_counter = PerfDataManager::create_counter("my.time.counter", PerfData::U_Ticks, 0LL, CHECK);
     *
     *    {
     *      PerfTraceTime ptt(my_time_counter);
     *      // perform the operation you want to measure
     *    }
     *
     * Note: use of this class does not need to occur within a guarded
     * block. The UsePerfData guard is used with the implementation
     * of this class.
     */
    class PerfTraceTime : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* SystemDictionary::load_instance_class()
* VMThread::evaluate_operation()

### 内部構造(Internal structure)
処理時間の計測には elapsedTimer を用いている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
        elapsedTimer _t;
        PerfLongCounter* _timerp;
```




### 詳細(Details)
See: [here](../doxygen/classPerfTraceTime.html) for details

---
## <a name="nojVyOaWAy" id="nojVyOaWAy">PerfTraceTimedEvent</a>

### 概要(Summary)
特殊な PerfTraceTime クラス.

PerfTraceTime の機能 (= あるスコープ内の処理にかかった時間を計測する機能) に加えて, 
指定された PerfCounter の値を 1つインクリメントする機能を備える.

(このために PerfCounter を 2つ受け取る. 
 1つは PerfTraceTime の機能のために使用し, もう一つはコンストラクタ内で 1つインクリメントする)

(なお, PerfTraceTime と同じくこのクラス内で UsePerfData を確認しているので, 使用点では UsePerfData を確認する必要は無い)


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
    /* The PerfTraceTimedEvent class is responsible for counting the
     * occurrence of some event and measuring the the elapsed time of
     * the event in two separate PerfCounter instances.
     *
     * Example:
     *
     *    static PerfCounter* my_time_counter = PerfDataManager::create_counter("my.time.counter", PerfData::U_Ticks, CHECK);
     *    static PerfCounter* my_event_counter = PerfDataManager::create_counter("my.event.counter", PerfData::U_Events, CHECK);
     *
     *    {
     *      PerfTraceTimedEvent ptte(my_time_counter, my_event_counter);
     *      // perform the operation you want to count and measure
     *    }
     *
     * Note: use of this class does not need to occur within a guarded
     * block. The UsePerfData guard is used with the implementation
     * of this class.
     *
     */
    class PerfTraceTimedEvent : public PerfTraceTime {
```

### 使われ方(Usage)
CompileBroker::compiler_thread_loop() 内で(のみ)使用されている.

### 内部構造(Internal structure)
コンストラクタでは, 
PerfTraceTime() のコンストラクタを実行するだけでなく, 
PerfLongCounter::inc() でカウンタ値のインクリメントも行う.


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfData.hpp))
      public:
        inline PerfTraceTimedEvent(PerfLongCounter* timerp, PerfLongCounter* eventp): PerfTraceTime(timerp), _eventp(eventp) {
          if (!UsePerfData) return;
          _eventp->inc();
        }
    
        inline PerfTraceTimedEvent(PerfLongCounter* timerp, PerfLongCounter* eventp, int* recursion_counter): PerfTraceTime(timerp, recursion_counter), _eventp(eventp) {
          if (!UsePerfData) return;
          _eventp->inc();
        }
```




### 詳細(Details)
See: [here](../doxygen/classPerfTraceTimedEvent.html) for details

---
