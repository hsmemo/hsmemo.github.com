---
layout: default
title: G1CollectorPolicy クラス関連のクラス (PauseSummary, MainBodySummary, Summary, G1CollectorPolicy, G1CollectorPolicy_BestRegionsFirst, 及びそれらの補助クラス(LineBuffer, G1YoungGenSizer, CountCSClosure, HRSortIndexIsOKClosure, NextNonCSElemFinder, KnownGarbageClosure, ParKnownGarbageHRClosure, ParKnownGarbageTask))
---
[Top](../index.html)

#### G1CollectorPolicy クラス関連のクラス (PauseSummary, MainBodySummary, Summary, G1CollectorPolicy, G1CollectorPolicy_BestRegionsFirst, 及びそれらの補助クラス(LineBuffer, G1YoungGenSizer, CountCSClosure, HRSortIndexIsOKClosure, NextNonCSElemFinder, KnownGarbageClosure, ParKnownGarbageHRClosure, ParKnownGarbageTask))

これらは, G1GC 用の CollectorPolicy クラス (See: [here](no3718kvd.html) for details).
以下のような決定を行う.

* collection set (= GC の処理対象とする HeapRegion) の選択
* GC を行う契機の決定


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp))
    // A G1CollectorPolicy makes policy decisions that determine the
    // characteristics of the collector.  Examples include:
    //   * choice of collection set.
    //   * when to collect.
```


### クラス一覧(class list)

  * [G1CollectorPolicy](#nowjl-dPQ5)
  * [G1CollectorPolicy_BestRegionsFirst](#noSq8ALrUH)
  * [PauseSummary](#noN7CL84ZB)
  * [MainBodySummary](#noZqL-jj2g)
  * [Summary](#no1tkpLieo)
  * [LineBuffer](#no0P6B14Y-)
  * [G1YoungGenSizer](#nowtlFsp5t)
  * [CountCSClosure](#noUy4c4wZw)
  * [HRSortIndexIsOKClosure](#nogsjod64a)
  * [NextNonCSElemFinder](#nooUkdm2pD)
  * [KnownGarbageClosure](#nodIIXv5Rv)
  * [ParKnownGarbageHRClosure](#noY3aVBwB5)
  * [ParKnownGarbageTask](#no9oI8G-kT)


---
## <a name="nowjl-dPQ5" id="nowjl-dPQ5">G1CollectorPolicy</a>

### 概要(Summary)
G1GC 用の CollectorPolicy クラスの基底クラス (See: CollectorPolicy).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp))
    class G1CollectorPolicy: public CollectorPolicy {
```




### 詳細(Details)
See: [here](../doxygen/classG1CollectorPolicy.html) for details

---
## <a name="noSq8ALrUH" id="noSq8ALrUH">G1CollectorPolicy_BestRegionsFirst</a>

### 概要(Summary)
G1CollectorPolicy クラスの具象サブクラス.
以下のようなポリシーを実装している.

  * いつ concurrent mark を開始するか?

    最後に concurrent mark したときから, ヒープ使用量がある量以上増加した時点.
    量は CMSTriggerRatio や MinHeapFreeRatio に応じて決定.
    (<= とコメントには書いてあるが CMSTriggerRatio は CMS でしか使わないような...?? InitiatingHeapOccupancyPercent の間違い??)

  * いつ evacuation を実行するか?

    確保した量が「一回の evacuation で開放できる平均的な量(これまでの実績値)」と等しくなれば実行.
    ただし (新たな HeapRegion の確保が起こったタイミングで実行するかどうかを決めるので)
    最低でも 1つは HeapRegion が一杯にならないと実行されることはない.

  * ヒープサイズをどのように動的に調整するか?

    望ましい allocation space (New 領域相当の領域のことだと思われる) の大きさに基づいて決定.
    なお, allocation space の大きさは, survival rate と desired future to size (?? #TODO) に応じて決定.

  * collection set をどのように選ぶか?

    (この部分のコメントの内容は正しいか?? #TODO)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp))
    // This encapsulates a particular strategy for a g1 Collector.
    //
    //      Start a concurrent mark when our heap size is n bytes
    //            greater then our heap size was at the last concurrent
    //            mark.  Where n is a function of the CMSTriggerRatio
    //            and the MinHeapFreeRatio.
    //
    //      Start a g1 collection pause when we have allocated the
    //            average number of bytes currently being freed in
    //            a collection, but only if it is at least one region
    //            full
    //
    //      Resize Heap based on desired
    //      allocation space, where desired allocation space is
    //      a function of survival rate and desired future to size.
    //
    //      Choose collection set by first picking all older regions
    //      which have a survival rate which beats our projected young
    //      survival rate.  Then fill out the number of needed regions
    //      with young regions.
    
    class G1CollectorPolicy_BestRegionsFirst: public G1CollectorPolicy {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている
(なお, 格納されているオブジェクトは同一).

* 各 G1CollectedHeap オブジェクトの SharedHeap::_collector_policy フィールド
* 各 G1CollectedHeap オブジェクトの _g1_policy フィールド

#### 生成箇所(where its instances are created)
Universe::initialize_heap() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classG1CollectorPolicy__BestRegionsFirst.html) for details

---
## <a name="noN7CL84ZB" id="noN7CL84ZB">PauseSummary</a>

### 概要(Summary)
G1CollectorPolicy クラス内で使用される補助クラス(の基底クラス).
これまでの GC による停止時間の履歴を記録する.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp))
    class PauseSummary: public CHeapObj {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り
(履歴を取っておくための NumberSeq を格納しているだけ)
(そして, メソッドはこれらのフィールドへの getter メソッド(アクセサメソッド)のみ).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp))
      define_num_seq(total)
        define_num_seq(other)
```

なお, define_num_seq() は以下のように定義されたマクロ.
NumberSeq 型の private フィールドと, それにアクセスするための public なアクセサメソッドを定義する
(コメントによると, あまりいい方法じゃないけどコピペが氾濫するよりはいい, とのこと).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp))
    // Yes, this is a bit unpleasant... but it saves replicating the same thing
    // over and over again and introducing subtle problems through small typos and
    // cutting and pasting mistakes. The macros below introduces a number
    // sequnce into the following two classes and the methods that access it.
    
    #define define_num_seq(name)                                                  \
    private:                                                                      \
      NumberSeq _all_##name##_times_ms;                                           \
    public:                                                                       \
      void record_##name##_time_ms(double ms) {                                   \
        _all_##name##_times_ms.add(ms);                                           \
      }                                                                           \
      NumberSeq* get_##name##_seq() {                                             \
        return &_all_##name##_times_ms;                                           \
      }
```




### 詳細(Details)
See: [here](../doxygen/classPauseSummary.html) for details

---
## <a name="noZqL-jj2g" id="noZqL-jj2g">MainBodySummary</a>

### 概要(Summary)
G1CollectorPolicy クラス内で使用される補助クラス(の基底クラス).
様々な処理時間の履歴を記録する.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp))
    class MainBodySummary: public CHeapObj {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り
(履歴を取っておくための NumberSeq を格納しているだけ)
(そして, メソッドはこれらのフィールドへの getter メソッド(アクセサメソッド)のみ).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp))
      define_num_seq(satb_drain) // optional
      define_num_seq(parallel) // parallel only
        define_num_seq(ext_root_scan)
        define_num_seq(mark_stack_scan)
        define_num_seq(update_rs)
        define_num_seq(scan_rs)
        define_num_seq(obj_copy)
        define_num_seq(termination) // parallel only
        define_num_seq(parallel_other) // parallel only
      define_num_seq(mark_closure)
      define_num_seq(clear_ct)  // parallel only
```

なお, define_num_seq() は以下のように定義されたマクロ.
NumberSeq 型の private フィールドと, それにアクセスするための public なアクセサメソッドを定義する
(コメントによると, あまりいい方法じゃないけどコピペが氾濫するよりはいい, とのこと).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp))
    // Yes, this is a bit unpleasant... but it saves replicating the same thing
    // over and over again and introducing subtle problems through small typos and
    // cutting and pasting mistakes. The macros below introduces a number
    // sequnce into the following two classes and the methods that access it.
    
    #define define_num_seq(name)                                                  \
    private:                                                                      \
      NumberSeq _all_##name##_times_ms;                                           \
    public:                                                                       \
      void record_##name##_time_ms(double ms) {                                   \
        _all_##name##_times_ms.add(ms);                                           \
      }                                                                           \
      NumberSeq* get_##name##_seq() {                                             \
        return &_all_##name##_times_ms;                                           \
      }
```




### 詳細(Details)
See: [here](../doxygen/classMainBodySummary.html) for details

---
## <a name="no1tkpLieo" id="no1tkpLieo">Summary</a>

### 概要(Summary)
G1CollectorPolicy クラス内で使用される補助クラス.

PauseSummary クラスおよび MainBodySummary クラスの具象サブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp))
    class Summary: public PauseSummary,
                   public MainBodySummary {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectorPolicy オブジェクトの _summary フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
G1CollectorPolicy::G1CollectorPolicy() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classSummary.html) for details

---
## <a name="no0P6B14Y-" id="no0P6B14Y-">LineBuffer</a>

### 概要(Summary)
G1CollectorPolicy クラス内で使用される補助クラス.

ログ出力処理を簡単に行うための補助クラス(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
    // Help class for avoiding interleaved logging
    class LineBuffer: public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. LineBuffer オブジェクトを作成する (なお, コンストラクタ引数で indent_level を指定できる模様).
2. LineBuffer::append() で出力したい文字列を追加していく.
3. 最後に LineBuffer::append_and_print_cr() で出力する


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
    void G1CollectorPolicy::print_par_stats(int level,
                                            const char* str,
                                            double* data) {
    ...
      LineBuffer buf(level);
      buf.append("[%s (ms):", str);
    ...
      buf.append_and_print_cr("");
```

#### 使用箇所(where its instances are used)
情報出力用(デバッグ用??)の関数内で(のみ)使用されている模様.

### 内部構造(Internal structure)
内部的には 1024 byte の char 配列を用意している (このためこれ以上は出力できない).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
      static const int BUFFER_LEN = 1024;
    ...
      char _buffer[BUFFER_LEN];
```

また, LineBuffer::append_and_print_cr() での出力は gclog_or_tty->print_cr() によって行われる.

また #ifndef PRODUCT の場合には, 
デストラクタ内でバッファ長が指定したところまで届いているかどうか
(append() や append_and_print_cr() の呼び忘れが無いかどうか)のチェックを行ってくれる模様.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
    #ifndef PRODUCT
      ~LineBuffer() {
        assert(_cur == _indent_level * INDENT_CHARS, "pending data in buffer - append_and_print_cr() not called?");
      }
    #endif
```

#### 参考(for your information): LineBuffer::append_and_print_cr()
See: [here](no3420Gfm.html) for details



### 詳細(Details)
See: [here](../doxygen/classLineBuffer.html) for details

---
## <a name="nowtlFsp5t" id="nowtlFsp5t">G1YoungGenSizer</a>

### 概要(Summary)
G1CollectorPolicy クラス内で使用される補助クラス.

G1Gen オプションが指定されている場合 (かつ UseAdaptiveSizePolicy オプションが指定されていなかった場合) に,
New 領域相当の HeapRegion の個数を決定する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
    // The easiest way to deal with the parsing of the NewSize /
    // MaxNewSize / etc. parameteres is to re-use the code in the
    // TwoGenerationCollectorPolicy class. This is similar to what
    // ParallelScavenge does with its GenerationSizer class (see
    // ParallelScavengeHeap::initialize()). We might change this in the
    // future, but it's a good start.
    class G1YoungGenSizer : public TwoGenerationCollectorPolicy {
```

### 使われ方(Usage)
G1CollectorPolicy::init() 内で(のみ)使用されている.

(なお, このクラスは CHeapObj クラスだが局所変数としてのみ生成されている).

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている.

なお, 実際に使用されるのは initial_young_region_num() メソッドのみ
(これは _initial_gen0_size の値を返すだけの関数. 
 この値は NewSize オプションなどから計算される模様 #TODO).
その他のメソッドは定義されてはいるが使用箇所が見当たらない...


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
      size_t min_young_region_num() {
        return size_to_region_num(_min_gen0_size);
      }
      size_t initial_young_region_num() {
        return size_to_region_num(_initial_gen0_size);
      }
      size_t max_young_region_num() {
        return size_to_region_num(_max_gen0_size);
      }
```




### 詳細(Details)
See: [here](../doxygen/classG1YoungGenSizer.html) for details

---
## <a name="noUy4c4wZw" id="noUy4c4wZw">CountCSClosure</a>

### 概要(Summary)
G1CollectorPolicy クラス内で使用される補助クラス.

collection set に選ばれた HeapRegion を辿り, それらの使用量 (HeapRegion::used()) の合計値を計算する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
    class CountCSClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectorPolicy::count_CS_bytes_used() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classCountCSClosure.html) for details

---
## <a name="nogsjod64a" id="nogsjod64a">HRSortIndexIsOKClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

G1CollectorPolicy_BestRegionsFirst クラス内で使用される補助クラス.

各 HeapRegion オブジェクトの HeapRegion::sort_index() に格納されている優先順位が, 
実際に CollectionSetChooser が記録している優先順位と合致しているかどうかをチェックする.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
    #ifndef PRODUCT
    class HRSortIndexIsOKClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectorPolicy_BestRegionsFirst::assertMarkedBytesDataOK() 内で(のみ)使用されている
(なお, この関数自体も #ifndef PRODUCT 時にしか定義されない).




### 詳細(Details)
See: [here](../doxygen/classHRSortIndexIsOKClosure.html) for details

---
## <a name="nooUkdm2pD" id="nooUkdm2pD">NextNonCSElemFinder</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

名前の通り, collection set に入っていない HeapRegion を1つ見つける Closure のようだが...


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
    class NextNonCSElemFinder: public HeapRegionClosure {
```




### 詳細(Details)
See: [here](../doxygen/classNextNonCSElemFinder.html) for details

---
## <a name="nodIIXv5Rv" id="nodIIXv5Rv">KnownGarbageClosure</a>

### 概要(Summary)
G1CollectorPolicy_BestRegionsFirst クラス内で使用される補助クラス.

既に mark 済みで, かつ humongous でも young でもない HeapRegion を CollectionSetChooser に登録する
(ParKnownGarbageHRClosure に似ているが, こちらは処理を single thread で行う際に使われる模様).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
    class KnownGarbageClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectorPolicy_BestRegionsFirst::record_concurrent_mark_cleanup_end() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classKnownGarbageClosure.html) for details

---
## <a name="noY3aVBwB5" id="noY3aVBwB5">ParKnownGarbageHRClosure</a>

### 概要(Summary)
ParKnownGarbageTask クラス内で使用される補助クラス.

既に mark 済みで, かつ humongous でも young でもない HeapRegion を CollectionSetChooser に登録する
(KnownGarbageClosure に似ているが, こちらは処理をマルチスレッドで行う際に使われる模様).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
    class ParKnownGarbageHRClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
ParKnownGarbageTask::work() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classParKnownGarbageHRClosure.html) for details

---
## <a name="no9oI8G-kT" id="no9oI8G-kT">ParKnownGarbageTask</a>

### 概要(Summary)
G1CollectorPolicy_BestRegionsFirst クラス内で使用される補助クラス.

ConcurrentMark::cleanup() の処理 (の一部) を parallel に行うための AbstractGangTask.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
    class ParKnownGarbageTask: public AbstractGangTask {
```

### 使われ方(Usage)
G1CollectorPolicy_BestRegionsFirst::record_concurrent_mark_cleanup_end() 内で(のみ)使用されている.

### 内部構造(Internal structure)
実際の処理は ParKnownGarbageHRClosure が行っている (See: ParKnownGarbageHRClosure).




### 詳細(Details)
See: [here](../doxygen/classParKnownGarbageTask.html) for details

---
