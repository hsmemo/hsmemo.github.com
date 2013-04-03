---
layout: default
title: G1MonitoringSupport クラス 
---
[Top](../index.html)

#### G1MonitoringSupport クラス 



---
## <a name="noFUkW2aGX" id="noFUkW2aGX">G1MonitoringSupport</a>

保守運用機能のためのクラス (JMM 機能および PerfData 管理用のクラス).

G1GC 使用時に, Garbage Collection 処理に関する情報を格納しておくためのクラス.

(なお, G1GC ではメモリ領域に対して仮想的に young という区別を儲けている.
 そして old は young ではない region 全てを指す.
 こういう区別をしているのは, JMM の API が世代別を想定した作りになっているため
 (JMM の API では, young 領域を表す Space と old 領域を表す Space がある, ということを前提にしている))

(G1MonitoringSupport 内に格納されている値には, 
 (仮想的に作った) 各世代の "capacity" や "used" といった可変な値もあれば, 
 minimum capacity や maximum capacities といった固定な値もある.
 また, G1GC の concurrent および stop-the-world 方式の GC が起こった回数を計測したカウンタ値も格納されている.)


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MonitoringSupport.hpp))
    // Class for monitoring logical spaces in G1.
    // G1 defines a set of regions as a young
    // collection (analogous to a young generation).
    // The young collection is a logical generation
    // with no fixed chunk (see space.hpp) reflecting
    // the address space for the generation.  In addition
    // to the young collection there is its complement
    // the non-young collection that is simply the regions
    // not in the young collection.  The non-young collection
    // is treated here as a logical old generation only
    // because the monitoring tools expect a generational
    // heap.  The monitoring tools expect that a Space
    // (see space.hpp) exists that describe the
    // address space of young collection and non-young
    // collection and such a view is provided here.
    //
    // This class provides interfaces to access
    // the value of variables for the young collection
    // that include the "capacity" and "used" of the
    // young collection along with constant values
    // for the minimum and maximum capacities for
    // the logical spaces.  Similarly for the non-young
    // collection.
    //
    // Also provided are counters for G1 concurrent collections
    // and stop-the-world full heap collecitons.
    //
    // Below is a description of how "used" and "capactiy"
    // (or committed) is calculated for the logical spaces.
    //
    // 1) The used space calculation for a pool is not necessarily
    // independent of the others. We can easily get from G1 the overall
    // used space in the entire heap, the number of regions in the young
    // generation (includes both eden and survivors), and the number of
    // survivor regions. So, from that we calculate:
    //
    //  survivor_used = survivor_num * region_size
    //  eden_used     = young_region_num * region_size - survivor_used
    //  old_gen_used  = overall_used - eden_used - survivor_used
    //
    // Note that survivor_used and eden_used are upper bounds. To get the
    // actual value we would have to iterate over the regions and add up
    // ->used(). But that'd be expensive. So, we'll accept some lack of
    // accuracy for those two. But, we have to be careful when calculating
    // old_gen_used, in case we subtract from overall_used more then the
    // actual number and our result goes negative.
    //
    // 2) Calculating the used space is straightforward, as described
    // above. However, how do we calculate the committed space, given that
    // we allocate space for the eden, survivor, and old gen out of the
    // same pool of regions? One way to do this is to use the used value
    // as also the committed value for the eden and survivor spaces and
    // then calculate the old gen committed space as follows:
    //
    //  old_gen_committed = overall_committed - eden_committed - survivor_committed
    //
    // Maybe a better way to do that would be to calculate used for eden
    // and survivor as a sum of ->used() over their regions and then
    // calculate committed as region_num * region_size (i.e., what we use
    // to calculate the used space now). This is something to consider
    // in the future.
    //
    // 3) Another decision that is again not straightforward is what is
    // the max size that each memory pool can grow to. One way to do this
    // would be to use the committed size for the max for the eden and
    // survivors and calculate the old gen max as follows (basically, it's
    // a similar pattern to what we use for the committed space, as
    // described above):
    //
    //  old_gen_max = overall_max - eden_max - survivor_max
    //
    // Unfortunately, the above makes the max of each pool fluctuate over
    // time and, even though this is allowed according to the spec, it
    // broke several assumptions in the M&M framework (there were cases
    // where used would reach a value greater than max). So, for max we
    // use -1, which means "undefined" according to the spec.
    //
    // 4) Now, there is a very subtle issue with all the above. The
    // framework will call get_memory_usage() on the three pools
    // asynchronously. As a result, each call might get a different value
    // for, say, survivor_num which will yield inconsistent values for
    // eden_used, survivor_used, and old_gen_used (as survivor_num is used
    // in the calculation of all three). This would normally be
    // ok. However, it's possible that this might cause the sum of
    // eden_used, survivor_used, and old_gen_used to go over the max heap
    // size and this seems to sometimes cause JConsole (and maybe other
    // clients) to get confused. There's not a really an easy / clean
    // solution to this problem, due to the asynchrounous nature of the
    // framework.
    
    class G1MonitoringSupport : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _g1mm フィールドに(のみ)格納されている
(「各」と言っても1つしかいないが...).

#### 生成箇所(where its instances are created)
G1CollectedHeap::initialize() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
記録した情報は G1MemoryPoolSuper から(のみ)参照されている (See: G1MemoryPoolSuper).

(<= ただし PerfData のデータだけなら (PerfData なので) jcmd 等から見ようと思えば見ることはできるが...)

### 内部構造(Internal structure)
内部には以下のパフォーマンスカウンタを保持しており, これらにアクセスするためのメソッドを提供している
(See: CollectorCounters, GenerationCounters, HSpaceCounters).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MonitoringSupport.hpp))
      // jstat performance counters
      //  incremental collections both fully and partially young
      CollectorCounters*   _incremental_collection_counters;
      //  full stop-the-world collections
      CollectorCounters*   _full_collection_counters;
      //  young collection set counters.  The _eden_counters,
      // _from_counters, and _to_counters are associated with
      // this "generational" counter.
      GenerationCounters*  _young_collection_counters;
      //  non-young collection set counters. The _old_space_counters
      // below are associated with this "generational" counter.
      GenerationCounters*  _non_young_collection_counters;
      // Counters for the capacity and used for
      //   the whole heap
      HSpaceCounters*      _old_space_counters;
      //   the young collection
      HSpaceCounters*      _eden_counters;
      //   the survivor collection (only one, _to_counters, is actively used)
      HSpaceCounters*      _from_counters;
      HSpaceCounters*      _to_counters;
```




### 詳細(Details)
See: [here](../doxygen/classG1MonitoringSupport.html) for details

---
