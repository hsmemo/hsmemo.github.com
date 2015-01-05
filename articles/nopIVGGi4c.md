---
layout: default
title: PSParallelCompact クラス関連のクラス (SplitInfo, SpaceInfo, ParallelCompactData, ParallelCompactData::RegionData, ParMarkBitMapClosure, PSParallelCompact, PSParallelCompact::IsAliveClosure, PSParallelCompact::KeepAliveClosure, PSParallelCompact::FollowRootClosure, PSParallelCompact::FollowStackClosure, PSParallelCompact::AdjustPointerClosure, PSParallelCompact::VerifyUpdateClosure, PSParallelCompact::ResetObjectsClosure, PSParallelCompact::MarkAndPushClosure, MoveAndUpdateClosure, UpdateOnlyClosure, FillClosure, 及びそれらの補助クラス(PreGCValues, PSAlwaysTrueClosure, AdjusterTracker))
---
[Top](../index.html)

#### PSParallelCompact クラス関連のクラス (SplitInfo, SpaceInfo, ParallelCompactData, ParallelCompactData::RegionData, ParMarkBitMapClosure, PSParallelCompact, PSParallelCompact::IsAliveClosure, PSParallelCompact::KeepAliveClosure, PSParallelCompact::FollowRootClosure, PSParallelCompact::FollowStackClosure, PSParallelCompact::AdjustPointerClosure, PSParallelCompact::VerifyUpdateClosure, PSParallelCompact::ResetObjectsClosure, PSParallelCompact::MarkAndPushClosure, MoveAndUpdateClosure, UpdateOnlyClosure, FillClosure, 及びそれらの補助クラス(PreGCValues, PSAlwaysTrueClosure, AdjusterTracker))

これらは, ParallelScavengeHeap の Parallel Compaction 処理
(UseParallelOldGC オプションが指定されている場合の Major GC 処理)で使用される補助クラス (See: [here](no28916Gft.html) for details).


### クラス一覧(class list)

  * [PSParallelCompact](#no2s8ZaRjP)
  * [SpaceInfo](#noSFqokTJM)
  * [SplitInfo](#noow4xd1aI)
  * [ParallelCompactData](#novqDME3-8)
  * [ParallelCompactData::RegionData](#noVIC0aNkP)
  * [ParMarkBitMapClosure](#noFldkc98F)
  * [PSParallelCompact::IsAliveClosure](#nobwcCVyFj)
  * [PSParallelCompact::KeepAliveClosure](#noH3RNEl_g)
  * [PSParallelCompact::FollowRootClosure](#noQD6FqAws)
  * [PSParallelCompact::FollowStackClosure](#noU-8CBQkX)
  * [PSParallelCompact::AdjustPointerClosure](#noSMdpPbEp)
  * [PSParallelCompact::VerifyUpdateClosure](#noBkPFayyF)
  * [PSParallelCompact::ResetObjectsClosure](#no-Emwgj2t)
  * [PSParallelCompact::MarkAndPushClosure](#no_pVJkxmW)
  * [MoveAndUpdateClosure](#noIlotqQu2)
  * [UpdateOnlyClosure](#noa3UxJAvy)
  * [FillClosure](#noCz5MZq2N)
  * [PreGCValues](#nor065ohi0)
  * [PSAlwaysTrueClosure](#no31UqR9Ja)
  * [AdjusterTracker](#nowr52OY10)


---
## <a name="no2s8ZaRjP" id="no2s8ZaRjP">PSParallelCompact</a>

### 概要(Summary)
ParallelScavengeHeap の Parallel Compaction 処理
(UseParallelOldGC オプションが指定されている場合の Major GC 処理)
を行うクラス (より正確には, そのための機能を納めた名前空間(AllStatic クラス)) (See: [here](no28916Gft.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    // The UseParallelOldGC collector is a stop-the-world garbage collector that
    // does parts of the collection using parallel threads.  The collection includes
    // the tenured generation and the young generation.  The permanent generation is
    // collected at the same time as the other two generations but the permanent
    // generation is collect by a single GC thread.  The permanent generation is
    // collected serially because of the requirement that during the processing of a
    // klass AAA, any objects reference by AAA must already have been processed.
    // This requirement is enforced by a left (lower address) to right (higher
    // address) sliding compaction.
    //
    // There are four phases of the collection.
    //
    //      - marking phase
    //      - summary phase
    //      - compacting phase
    //      - clean up phase
    //
    // Roughly speaking these phases correspond, respectively, to
    //      - mark all the live objects
    //      - calculate the destination of each object at the end of the collection
    //      - move the objects to their destination
    //      - update some references and reinitialize some variables
    //
    // These three phases are invoked in PSParallelCompact::invoke_no_policy().  The
    // marking phase is implemented in PSParallelCompact::marking_phase() and does a
    // complete marking of the heap.  The summary phase is implemented in
    // PSParallelCompact::summary_phase().  The move and update phase is implemented
    // in PSParallelCompact::compact().
    //
    // A space that is being collected is divided into regions and with each region
    // is associated an object of type ParallelCompactData.  Each region is of a
    // fixed size and typically will contain more than 1 object and may have parts
    // of objects at the front and back of the region.
    //
    // region            -----+---------------------+----------
    // objects covered   [ AAA  )[ BBB )[ CCC   )[ DDD     )
    //
    // The marking phase does a complete marking of all live objects in the heap.
    // The marking also compiles the size of the data for all live objects covered
    // by the region.  This size includes the part of any live object spanning onto
    // the region (part of AAA if it is live) from the front, all live objects
    // contained in the region (BBB and/or CCC if they are live), and the part of
    // any live objects covered by the region that extends off the region (part of
    // DDD if it is live).  The marking phase uses multiple GC threads and marking
    // is done in a bit array of type ParMarkBitMap.  The marking of the bit map is
    // done atomically as is the accumulation of the size of the live objects
    // covered by a region.
    //
    // The summary phase calculates the total live data to the left of each region
    // XXX.  Based on that total and the bottom of the space, it can calculate the
    // starting location of the live data in XXX.  The summary phase calculates for
    // each region XXX quantites such as
    //
    //      - the amount of live data at the beginning of a region from an object
    //        entering the region.
    //      - the location of the first live data on the region
    //      - a count of the number of regions receiving live data from XXX.
    //
    // See ParallelCompactData for precise details.  The summary phase also
    // calculates the dense prefix for the compaction.  The dense prefix is a
    // portion at the beginning of the space that is not moved.  The objects in the
    // dense prefix do need to have their object references updated.  See method
    // summarize_dense_prefix().
    //
    // The summary phase is done using 1 GC thread.
    //
    // The compaction phase moves objects to their new location and updates all
    // references in the object.
    //
    // A current exception is that objects that cross a region boundary are moved
    // but do not have their references updated.  References are not updated because
    // it cannot easily be determined if the klass pointer KKK for the object AAA
    // has been updated.  KKK likely resides in a region to the left of the region
    // containing AAA.  These AAA's have there references updated at the end in a
    // clean up phase.  See the method PSParallelCompact::update_deferred_objects().
    // An alternate strategy is being investigated for this deferral of updating.
    //
    // Compaction is done on a region basis.  A region that is ready to be filled is
    // put on a ready list and GC threads take region off the list and fill them.  A
    // region is ready to be filled if it empty of live objects.  Such a region may
    // have been initially empty (only contained dead objects) or may have had all
    // its live objects copied out already.  A region that compacts into itself is
    // also ready for filling.  The ready list is initially filled with empty
    // regions and regions compacting into themselves.  There is always at least 1
    // region that can be put on the ready list.  The regions are atomically added
    // and removed from the ready list.
    
    class PSParallelCompact : AllStatic {
```

### 使われ方(Usage)
UseParallelOldGC オプションが指定されている場合,
Major GC 処理はこのクラスの PSParallelCompact::invoke() メソッドが呼び出されることで実行される (See: [here](no28916Gft.html) for details).

なお UseParallelOldGC オプションが指定されていない場合は,
このクラスの代わりに PSMarkSweep クラスが使用される
(See: PSMarkSweep).




### 詳細(Details)
See: [here](../doxygen/classPSParallelCompact.html) for details

---
## <a name="noSFqokTJM" id="noSFqokTJM">SpaceInfo</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス
(つまり, Parallel Compaction 処理中で使用される補助クラス).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class SpaceInfo
    {
```

各 MutableSpace (Perm, Old, Eden, From, To) に関して,
コンパクション後の情報(コンパクション後の使用量, etc)を管理するためのクラス.

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PSParallelCompact オブジェクトの _space_info フィールドに(のみ)格納されている
(正確には, このフィールドは SpaceInfo の配列を格納するフィールド.
この中に, 使用される全ての SpaceInfo オブジェクトが格納されている)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class PSParallelCompact : AllStatic {
    ...
      typedef enum {
        perm_space_id, old_space_id, eden_space_id,
        from_space_id, to_space_id, last_space_id
      } SpaceId;
    ...
      static SpaceInfo            _space_info[last_space_id];
```

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.
(そして, メソッドはこれらのフィールドへのアクセサメソッドのみ).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      MutableSpace*     _space;
      HeapWord*         _new_top;
      HeapWord*         _min_dense_prefix;
      HeapWord*         _dense_prefix;
      ObjectStartArray* _start_array;
      SplitInfo         _split_info;
```




### 詳細(Details)
See: [here](../doxygen/classSpaceInfo.html) for details

---
## <a name="noow4xd1aI" id="noow4xd1aI">SplitInfo</a>

### 概要(Summary)
SpaceInfo クラス内で使用される補助クラス.

通常は, コンパクション処理によって 1つの Space 内の live オブジェクトは 1つの Space へと移動される
(例えば, Eden 内のオブジェクトは全て Old に移動される, 等).
しかし, ヒープの空き容量が逼迫している場合には, 
Eden の live オブジェクトのコンパクション先が Old と Eden に２カ所に分かれたりすることがある.
SplitInfo クラスは, そういうケースでコンパクション先をうまく分けるために使用されるクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    // The SplitInfo class holds the information needed to 'split' a source region
    // so that the live data can be copied to two destination *spaces*.  Normally,
    // all the live data in a region is copied to a single destination space (e.g.,
    // everything live in a region in eden is copied entirely into the old gen).
    // However, when the heap is nearly full, all the live data in eden may not fit
    // into the old gen.  Copying only some of the regions from eden to old gen
    // requires finding a region that does not contain a partial object (i.e., no
    // live object crosses the region boundary) somewhere near the last object that
    // does fit into the old gen.  Since it's not always possible to find such a
    // region, splitting is necessary for predictable behavior.
    //
    // A region is always split at the end of the partial object.  This avoids
    // additional tests when calculating the new location of a pointer, which is a
    // very hot code path.  The partial object and everything to its left will be
    // copied to another space (call it dest_space_1).  The live data to the right
    // of the partial object will be copied either within the space itself, or to a
    // different destination space (distinct from dest_space_1).
    //
    // Split points are identified during the summary phase, when region
    // destinations are computed:  data about the split, including the
    // partial_object_size, is recorded in a SplitInfo record and the
    // partial_object_size field in the summary data is set to zero.  The zeroing is
    // possible (and necessary) since the partial object will move to a different
    // destination space than anything to its right, thus the partial object should
    // not affect the locations of any objects to its right.
    //
    // The recorded data is used during the compaction phase, but only rarely:  when
    // the partial object on the split region will be copied across a destination
    // region boundary.  This test is made once each time a region is filled, and is
    // a simple address comparison, so the overhead is negligible (see
    // PSParallelCompact::first_src_addr()).
    //
    // Notes:
    //
    // Only regions with partial objects are split; a region without a partial
    // object does not need any extra bookkeeping.
    //
    // At most one region is split per space, so the amount of data required is
    // constant.
    //
    // A region is split only when the destination space would overflow.  Once that
    // happens, the destination space is abandoned and no other data (even from
    // other source spaces) is targeted to that destination space.  Abandoning the
    // destination space may leave a somewhat large unused area at the end, if a
    // large object caused the overflow.
    //
    // Future work:
    //
    // More bookkeeping would be required to continue to use the destination space.
    // The most general solution would allow data from regions in two different
    // source spaces to be "joined" in a single destination region.  At the very
    // least, additional code would be required in next_src_region() to detect the
    // join and skip to an out-of-order source region.  If the join region was also
    // the last destination region to which a split region was copied (the most
    // likely case), then additional work would be needed to get fill_region() to
    // stop iteration and switch to a new source region at the right point.  Basic
    // idea would be to use a fake value for the top of the source space.  It is
    // doable, if a bit tricky.
    //
    // A simpler (but less general) solution would fill the remainder of the
    // destination region with a dummy object and continue filling the next
    // destination region.
    
    class SplitInfo
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
SpaceInfo オブジェクトの _split_info フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class SpaceInfo
    {
    ...
      SplitInfo         _split_info;
```




### 詳細(Details)
See: [here](../doxygen/classSplitInfo.html) for details

---
## <a name="novqDME3-8" id="novqDME3-8">ParallelCompactData</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス
(つまり, Parallel Compaction 処理中で使用される補助クラス).

コンパクション処理で使用される情報(移動先アドレス等)を格納しておくクラス
(Parallel Compaction 処理では, 最初に marking を行う.
次に, marking 結果に基づいて移動先のアドレス等を計算して ParallelCompactData に格納する.
そして, 最後に ParallelCompactData の情報に基づいてオブジェクトの移動及びポインタの書き換えを行う.
(See: [here](no28916Gft.html) for details))


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class ParallelCompactData
    {
```

なお, 情報は "region" 単位で記録される
("region" は Parallel Compaction における処理の単位.
処理対象のヒープ領域は "region" という固定長の部分領域に分割され, この単位で複数スレッドにより並列処理される.
(See: [here](no28916Gft.html) for details)).

(例えば, コンパクション先のアドレスは各オブジェクト毎に記録されるのではなく,
各 Region 単位で「その Region の先頭がどこに移動するか」が記録される, 等)

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PSParallelCompact オブジェクトの _summary_data フィールドに(のみ)格納されている


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class PSParallelCompact : AllStatic {
    ...
      static ParallelCompactData  _summary_data;
```

アクセサメソッドは PSParallelCompact::summary_data()


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      static ParallelCompactData& summary_data() { return _summary_data; }
```

### 内部構造(Internal structure)
内部的には, region 数と同数の ParallelCompactData::RegionData オブジェクトを保持しており,
その中にそれぞれの region の情報が格納されていく.
ParallelCompactData 自体には情報はほとんど記録されない
(ParallelCompactData 自体は, 使用される全ての ParallelCompactData::RegionData オブジェクトを管理するためのクラスといった感じ.
 (See: ParallelCompactData::RegionData)).




### 詳細(Details)
See: [here](../doxygen/classParallelCompactData.html) for details

---
## <a name="noVIC0aNkP" id="noVIC0aNkP">ParallelCompactData::RegionData</a>

### 概要(Summary)
ParallelCompactData クラス内で使用される補助クラス.

コンパクション処理で使用される情報(移動先アドレス等)を格納しておくクラス
(ParallelCompactData が管理する情報のほとんどは, 実際にはこのクラスに格納されている).
Region 1つにつき 1つの ParallelCompactData::RegionData オブジェクトが対応する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      class RegionData
      {
```

例えば, 各 region に対して以下のような情報を格納している.

* 前の region からはみ出してその region の先頭部分にまでを占めている live オブジェクトの量
* その region 内での最初の live オブジェクトの位置
* この region 内のオブジェクトがコンパクション後にコピーされる先の region の数
* etc

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ParallelCompactData オブジェクトの _region_data フィールドに(のみ)格納されている.
(正確には, このフィールドは ParallelCompactData::RegionData の配列を格納するフィールド.
この中に, 使用される全ての ParallelCompactData::RegionData オブジェクトが格納されている)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class ParallelCompactData
    {
    ...
      PSVirtualSpace* _region_vspace;
      RegionData*     _region_data;
      size_t          _region_count;
```

なお, 関連するフィールドに _region_vspace と _region_count がある.

* _region_vspace : _region_data 用のメモリ領域を管理する PSVirtualSpace.
* _region_count : _region_data 内に納められている ParallelCompactData オブジェクトの個数

#### 生成箇所(where its instances are created)
ParallelCompactData::initialize_region_data() 内で(のみ)生成されている.
この関数は, 現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
ParallelScavengeHeap::initialize()
-&gt; PSParallelCompact::initialize()
   -&gt; ParallelCompactData::initialize()
      -&gt; ParallelCompactData::initialize_region_data()
</pre></div>

#### 参考(for your information): ParallelScavengeHeap::initialize()
See: [here](no344Yjc.html) for details
#### 参考(for your information): PSParallelCompact::initialize()
See: [here](no3172wN1.html) for details
#### 参考(for your information): ParallelCompactData::initialize()
See: [here](no3172iXE.html) for details
#### 参考(for your information): ParallelCompactData::initialize_region_data()
See: [here](no3172vhK.html) for details
### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.

* _destination :
  その region の先頭がコンパクションにより移動する先のアドレス.

* _source_region :
  コンパクションによってその region にコピーされてくるオブジェクトの異動元 region 数.

* _partial_obj_addr :
  (オブジェクトが前の region 内に収まらず, この region まではみ出てきた場合用)
  はみ出てきたオブジェクトの開始アドレス.

* _partial_obj_size :
  (オブジェクトが前の region 内に収まらず, この region まではみ出てきた場合用)
  はみ出てきたオブジェクトがこの region 内で占めているサイズ.

* _dc_and_los :
  destination count 情報と live obj size 情報.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
        // Constants for manipulating the _dc_and_los field, which holds both the
        // destination count and live obj size.  The live obj size lives at the
        // least significant end so no masking is necessary when adding.
    ...
        HeapWord*            _destination;
        size_t               _source_region;
        HeapWord*            _partial_obj_addr;
        region_sz_t          _partial_obj_size;
        region_sz_t volatile _dc_and_los;
    #ifdef ASSERT
        // These enable optimizations that are only partially implemented.  Use
        // debug builds to prevent the code fragments from breaking.
        HeapWord*            _data_location;
        HeapWord*            _highest_ref;
    #endif  // #ifdef ASSERT
    
    #ifdef ASSERT
       public:
        uint            _pushed;   // 0 until region is pushed onto a worker's stack
       private:
    #endif
```




### 詳細(Details)
See: [here](../doxygen/classParallelCompactData_1_1RegionData.html) for details

---
## <a name="noFldkc98F" id="noFldkc98F">ParMarkBitMapClosure</a>

### 概要(Summary)
ParMarkBitMap に対して何らかの処理を行う Closure クラスの基底クラス.

(なお, このクラスは Closure クラスのサブクラスではなく, StackObj のサブクラスになっている)

ParMarkBitMap::iterate() の引数として使用する (その際には do_addr() メソッドが呼び出される).

なお, このクラスは初期化時に処理対象の heap word 数を指定される. 処理量がその量に達すると 'full' 状態になる.
(といっても, この仕組みは現状ではたった一つのサブクラスでしか利用されていないらしいが...
なお, このロジック自体は ParMarkBitMap クラスに実装している. 
というのは, そうしないと is_full() メソッドを virtual call にしないといけなくなり live object 1つ1つについて virtual call 呼び出しが起こってしまうので.)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    // Abstract closure for use with ParMarkBitMap::iterate(), which will invoke the
    // do_addr() method.
    //
    // The closure is initialized with the number of heap words to process
    // (words_remaining()), and becomes 'full' when it reaches 0.  The do_addr()
    // methods in subclasses should update the total as words are processed.  Since
    // only one subclass actually uses this mechanism to terminate iteration, the
    // default initial value is > 0.  The implementation is here and not in the
    // single subclass that uses it to avoid making is_full() virtual, and thus
    // adding a virtual call per live object.
    
    class ParMarkBitMapClosure: public StackObj {
```

### 使われ方(Usage)
ParMarkBitMap 中のデータを処理する do_addr() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      virtual IterationStatus do_addr(HeapWord* addr, size_t words) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classParMarkBitMapClosure.html) for details

---
## <a name="nobwcCVyFj" id="nobwcCVyFj">PSParallelCompact::IsAliveClosure</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス(Closureクラス)
(つまり, Parallel Compaction 処理中で使用される補助クラス).

PSParallelCompact::IsAliveClosure::do_object_b() メソッドが呼ばれると,
処理対象のオブジェクトが生きているかどうかを返す.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      class IsAliveClosure: public BoolObjectClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PSParallelCompact クラスの _is_alive_closure フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class PSParallelCompact : AllStatic {
    ...
      static IsAliveClosure       _is_alive_closure;
```

アクセサメソッドは PSParallelCompact::is_alive_closure()

#### 使用箇所(where its instances are used)
PSParallelCompact::invoke_no_policy() 内で(のみ)使用されている
(より正確には, そこから呼び出される
PSParallelCompact::marking_phase(), PSParallelCompact::follow_weak_klass_links(),
及び PSParallelCompact::follow_mdo_weak_refs() 内で使用される補助クラス).




### 詳細(Details)
See: [here](../doxygen/classPSParallelCompact_1_1IsAliveClosure.html) for details

---
## <a name="noH3RNEl_g" id="noH3RNEl_g">PSParallelCompact::KeepAliveClosure</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス(Closureクラス)
(つまり, Parallel Compaction 処理中で使用される補助クラス).

参照オブジェクト(java.lang.ref オブジェクト)に対する処理に用いられる Closure クラス.
まだマークが付いていないオブジェクトに対して,
マークを付け, そこから辿れるものを marking stack にプッシュする.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      class KeepAliveClosure: public OopClosure {
```

### 使われ方(Usage)
#### 使用箇所(where its instances are used)
PSParallelCompact::invoke_no_policy() 内で(のみ)使用されている
(より正確には, そこから呼び出される
PSParallelCompact::follow_weak_klass_links() 内で使用される補助クラス).




### 詳細(Details)
See: [here](../doxygen/classPSParallelCompact_1_1KeepAliveClosure.html) for details

---
## <a name="noQD6FqAws" id="noQD6FqAws">PSParallelCompact::FollowRootClosure</a>

### 概要(Summary)
?? (使われていないクラス)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      // Current unused
      class FollowRootClosure: public OopsInGenClosure {
```

### 使われ方(Usage)
(現状では使われていない)




### 詳細(Details)
See: [here](../doxygen/classPSParallelCompact_1_1FollowRootClosure.html) for details

---
## <a name="noU-8CBQkX" id="noU-8CBQkX">PSParallelCompact::FollowStackClosure</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス(Closureクラス)
(つまり, Parallel Compaction 処理中で使用される補助クラス).

Mark Sweep Compact 処理の phase 1 で使われる Closure クラス.
marking stack に溜まっているポインタに対して, 
そこから辿れるもの全てに再帰的に処理を行う.

(現状では, 参照オブジェクト(java.lang.ref オブジェクト)に対する処理にしか用いられていないが...)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      class FollowStackClosure: public VoidClosure {
```

### 使われ方(Usage)
#### 使用箇所(where its instances are used)
PSParallelCompact::marking_phase() 内で(のみ)使用されている
(より正確には, PSParallelCompact::marking_phase() 内と,
そこから呼び出される RefProcTaskProxy::do_it() 内で使用される補助クラス).




### 詳細(Details)
See: [here](../doxygen/classPSParallelCompact_1_1FollowStackClosure.html) for details

---
## <a name="noSMdpPbEp" id="noSMdpPbEp">PSParallelCompact::AdjustPointerClosure</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス(Closureクラス)
(つまり, Parallel Compaction 処理中で使用される補助クラス).

Mark Sweep Compact 処理の phase 3 で使われる Closure クラス.
ポインタの値をコンパクション先の新しいアドレスへと書き換える処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      class AdjustPointerClosure: public OopsInGenClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PSParallelCompact クラスの _adjust_root_pointer_closure フィールド,
及び _adjust_pointer_closure フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class PSParallelCompact : AllStatic {
    ...
      static AdjustPointerClosure _adjust_root_pointer_closure;
      static AdjustPointerClosure _adjust_pointer_closure;
```

それぞれ, アクセサメソッドは
PSParallelCompact::adjust_root_pointer_closure()
及び PSParallelCompact::adjust_pointer_closure().


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      static OopClosure* adjust_pointer_closure()      { return (OopClosure*)&_adjust_pointer_closure; }
      static OopClosure* adjust_root_pointer_closure() { return (OopClosure*)&_adjust_root_pointer_closure; }
```

#### 使用箇所(where its instances are used)
PSParallelCompact::adjust_roots() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPSParallelCompact_1_1AdjustPointerClosure.html) for details

---
## <a name="noBkPFayyF" id="noBkPFayyF">PSParallelCompact::VerifyUpdateClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか使用されない).

ポインタの修正処理が正しく行われているかどうかをチェックする.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      // Closure for verifying update of pointers.  Does not
      // have any side effects.
      class VerifyUpdateClosure: public ParMarkBitMapClosure {
```

### 使われ方(Usage)
PSParallelCompact::move_and_update() 内で(のみ)使用されている (ただし, #ifdef ASSERT 時にしか使用されない).

#### 参考(for your information): PSParallelCompact::move_and_update()
See: [here](no2114ZW1.html) for details



### 詳細(Details)
See: [here](../doxygen/classPSParallelCompact_1_1VerifyUpdateClosure.html) for details

---
## <a name="no-Emwgj2t" id="no-Emwgj2t">PSParallelCompact::ResetObjectsClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか使用されない).

デバッグ用途で変更したオブジェクトの mark フィールドを初期状態の値に修復する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      // Closure for updating objects altered for debug checking
      class ResetObjectsClosure: public ParMarkBitMapClosure {
```

### 使われ方(Usage)
PSParallelCompact::move_and_update() 内で(のみ)使用されている (ただし, #ifdef ASSERT 時にしか使用されない).

#### 参考(for your information): PSParallelCompact::move_and_update()
See: [here](no2114ZW1.html) for details



### 詳細(Details)
See: [here](../doxygen/classPSParallelCompact_1_1ResetObjectsClosure.html) for details

---
## <a name="no_pVJkxmW" id="no_pVJkxmW">PSParallelCompact::MarkAndPushClosure</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス(Closureクラス)
(つまり, Parallel Compaction 処理中で使用される補助クラス).

Mark Sweep Compact 処理の phase 1 で使われる Closure クラス.
まだマークが付いていないオブジェクトに対して, 
マークを付け, そこから辿れるものを marking stack にプッシュする.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      class MarkAndPushClosure: public OopClosure {
```

### 使われ方(Usage)
PSParallelCompact::marking_phase() 内で(のみ)使用されている
(より正確には, PSParallelCompact::marking_phase() 内と,
そこから呼び出される ThreadRootsMarkingTask::do_it(), MarkFromRootsTask::do_it(),
RefProcTaskProxy::do_it(), StealMarkingTask::do_it() 内で使用される補助クラス).




### 詳細(Details)
See: [here](../doxygen/classPSParallelCompact_1_1MarkAndPushClosure.html) for details

---
## <a name="noIlotqQu2" id="noIlotqQu2">MoveAndUpdateClosure</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス(Closureクラス)
(つまり, Parallel Compaction 処理中で使用される補助クラス).

Mark Sweep Compact 処理の phase 4 で使われる Closure クラス.
各 live object を新しいアドレスに移動させ, それらの中にあるポインタを新しいアドレスに修正する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class MoveAndUpdateClosure: public ParMarkBitMapClosure {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
* コンストラクタの時点で, コンパクション先のアドレス(destination)及びその領域の長さ(words)を指定する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
      inline MoveAndUpdateClosure(ParMarkBitMap* bitmap, ParCompactionManager* cm,
                                  ObjectStartArray* start_array,
                                  HeapWord* destination, size_t words);
```

* 実際の処理は MoveAndUpdateClosure::do_addr() で行われる.

* コンパクション先の領域にどれだけの空き領域が残っているかは, MoveAndUpdateClosure::words_remaining() で取得可能.
  また, 知りたいのがコンパクション先の領域の空き領域が 0 かどうかだけなら,  MoveAndUpdateClosure::is_full() も使える.

* オブジェクトを実際にコピーしたら,MoveAndUpdateClosure::update_state() を呼んで, 残り空き容量の情報等を更新しておく
  (指定分だけ MoveAndUpdateClosure::words_remaining() の値が減る).

#### 使用箇所(where its instances are used)
PSParallelCompact::move_and_update() 及び PSParallelCompact::fill_region() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classMoveAndUpdateClosure.html) for details

---
## <a name="noa3UxJAvy" id="noa3UxJAvy">UpdateOnlyClosure</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス(Closureクラス)
(つまり, Parallel Compaction 処理中で使用される補助クラス).

Mark Sweep Compact 処理の phase 4 で使われる Closure クラス.
このクラスは dense prefix 部分を処理するためのもので, 
各 live object 中にあるポインタを新しいアドレスに修正する
(dense prefix 部分なのでオブジェクトの移動は必要ない).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class UpdateOnlyClosure: public ParMarkBitMapClosure {
```

### 使われ方(Usage)
PSParallelCompact::update_and_deadwood_in_dense_prefix() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classUpdateOnlyClosure.html) for details

---
## <a name="noCz5MZq2N" id="noCz5MZq2N">FillClosure</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス(Closureクラス)
(つまり, Parallel Compaction 処理中で使用される補助クラス).

Mark Sweep Compact 処理の phase 4 で使われる Closure クラス.
このクラスは dense prefix 部分を処理するためのもので, 
dead オブジェクトをダミーオブジェクトで上書きする処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class FillClosure: public ParMarkBitMapClosure
    {
```

### 使われ方(Usage)
PSParallelCompact::update_and_deadwood_in_dense_prefix() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classFillClosure.html) for details

---
## <a name="nor065ohi0" id="nor065ohi0">PreGCValues</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス
(つまり, Parallel Compaction 処理中で使用される補助クラス).

GC 実行前の Java ヒープの使用量を記録しておくためのクラス
(この情報は, GC 後にトレース出力を出したり Perm 領域の領域長を変更する際に参照される).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp))
    // Simple class for storing info about the heap at the start of GC, to be used
    // after GC for comparison/printing.
    class PreGCValues {
```

### 使われ方(Usage)
PSParallelCompact::invoke_no_policy() 内で(のみ)使用されている.

### 内部構造(Internal structure)
内部には4つのフィールド(のみ)を保持する.
(そして, メソッドはこれらのフィールドへの getter メソッド(アクセサメソッド)のみ)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp))
      size_t _heap_used;
      size_t _young_gen_used;
      size_t _old_gen_used;
      size_t _perm_gen_used;
```

フィールドの値は, コンストラクタ内あるいは明示的に PreGCValues::fill() が呼ばれた際に
ParallelScavengeHeap オブジェクトから取得している.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp))
      PreGCValues() { }
      PreGCValues(ParallelScavengeHeap* heap) { fill(heap); }
    
      void fill(ParallelScavengeHeap* heap) {
        _heap_used      = heap->used();
        _young_gen_used = heap->young_gen()->used_in_bytes();
        _old_gen_used   = heap->old_gen()->used_in_bytes();
        _perm_gen_used  = heap->perm_gen()->used_in_bytes();
      };
```




### 詳細(Details)
See: [here](../doxygen/classPreGCValues.html) for details

---
## <a name="no31UqR9Ja" id="no31UqR9Ja">PSAlwaysTrueClosure</a>

### 概要(Summary)
PSParallelCompact クラス内で使用される補助クラス(Closureクラス)
(つまり, Parallel Compaction 処理中で使用される補助クラス).

Mark Sweep Compact 処理の phase 3 で使われる Closure クラス.

PSParallelCompact 用の AlwaysTrueClosure クラス
(つまり, JNI の Weak Global Handle を辿る処理で使用される Closure.
 名前の通り, どんな場合でも常に true を返す. (See: AlwaysTrueClosure))


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp))
    // This should be moved to the shared markSweep code!
    class PSAlwaysTrueClosure: public BoolObjectClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
always_true という大域変数に(のみ)格納されている.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp))
    static PSAlwaysTrueClosure always_true;
```

#### 使用箇所(where its instances are used)
PSParallelCompact::adjust_roots() 内で(のみ)使用されている.

(なお, phase1 の中で ReferenceProcessor::process_discovered_references() が呼び出された段階で,
 死んでいる Weak Global Reference については NULL になっている.
 このため, PSAlwaysTrueClosure のような常に true を返すだけの Closure でも,
 生きている Weak Global Reference だけを全て辿ることができる.)




### 詳細(Details)
See: [here](../doxygen/classPSAlwaysTrueClosure.html) for details

---
## <a name="nowr52OY10" id="nowr52OY10">AdjusterTracker</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない). 
(なお, 正確に言うと「#ifdef VALIDATE_MARK_SWEEP 時にしか定義されない」. ただし, VALIDATE_MARK_SWEEP マクロ定数は #ifdef ASSERT 時にだけ #define されるので同義)

oopDesc::adjust_pointers() でのポインタの修正処理が正しく行われているかどうかを検証する.

(というか hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp の AdjusterTracker とほぼ同じ)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp))
    #ifdef VALIDATE_MARK_SWEEP
    ...
    class AdjusterTracker: public OopClosure {
```

### 使われ方(Usage)
#### 使用箇所(where its instances are used)
PSParallelCompact::track_interior_pointers() 内で(のみ)使用されている.

(なお, このクラスは (#ifdef VALIDATE_MARK_SWEEP 時であることに加えて)
ValidateMarkSweep オプションが指定されている場合にしか使用されない)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp))
    void PSParallelCompact::track_interior_pointers(oop obj) {
      if (ValidateMarkSweep) {
        _adjusted_pointers->clear();
        _pointer_tracking = true;
    
        AdjusterTracker checker;
        obj->oop_iterate(&checker);
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classAdjusterTracker.html) for details

---
