---
layout: default
title: Space クラス関連のクラス (SpaceMemRegionOopsIterClosure, Space, DirtyCardToOopClosure, CompactPoint, CompactibleSpace, ContiguousSpace, Filtering_DCTOC, ContiguousSpaceDCTOC, EdenSpace, ConcEdenSpace, OffsetTableContigSpace, TenuredSpace, ContigPermSpace, 及びそれらの補助クラス(VerifyOldOopClosure))
---
[Top](../index.html)

#### Space クラス関連のクラス (SpaceMemRegionOopsIterClosure, Space, DirtyCardToOopClosure, CompactPoint, CompactibleSpace, ContiguousSpace, Filtering_DCTOC, ContiguousSpaceDCTOC, EdenSpace, ConcEdenSpace, OffsetTableContigSpace, TenuredSpace, ContigPermSpace, 及びそれらの補助クラス(VerifyOldOopClosure))

これらは, メモリ管理用のクラス.
より具体的に言うと, Java ヒープ用のメモリ領域を管理するためのクラス (See: [here](no3718kvd.html) for details).

Java ヒープ用のメモリ領域を管理するクラスは使用する GC アルゴリズムによって異なるが, 
これらのクラスは GC アルゴリズムが ParallelScavenge 以外の場合に使用される 
(つまり, G1CollectedHeap および GenCollectedHeap 用) (See: ImmutableSpace, MutableSpace).

(<= なお, 正確には G1CollectedHeap もここで定義されているクラスをそのまま使っているわけではなく, 
独自に作ったサブクラスを使用している (See: HeapRegion))

(<= なので, ここで定義されているクラスを直接的に使っているのは GenCollectedHeap だけ)


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // A space is an abstraction for the "storage units" backing
    // up the generation abstraction. It includes specific
    // implementations for keeping track of free and used space,
    // for iterating over objects and free blocks, etc.
```

なお, これらのクラスは以下のような継承関係を持つ.


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // Here's the Space hierarchy:
    //
    // - Space               -- an asbtract base class describing a heap area
    //   - CompactibleSpace  -- a space supporting compaction
    //     - CompactibleFreeListSpace -- (used for CMS generation)
    //     - ContiguousSpace -- a compactible space in which all free space
    //                          is contiguous
    //       - EdenSpace     -- contiguous space used as nursery
    //         - ConcEdenSpace -- contiguous space with a 'soft end safe' allocation
    //       - OffsetTableContigSpace -- contiguous space with a block offset array
    //                          that allows "fast" block_start calls
    //         - TenuredSpace -- (used for TenuredGeneration)
    //         - ContigPermSpace -- an offset table contiguous space for perm gen
```



### クラス一覧(class list)

  * [Space](#noeb9cq9Z9)
  * [CompactibleSpace](#nompeS3kVU)
  * [ContiguousSpace](#nosfPJ2U5x)
  * [EdenSpace](#noIZyKziSy)
  * [ConcEdenSpace](#no8FHK1G5-)
  * [OffsetTableContigSpace](#noYm5xLT_L)
  * [TenuredSpace](#noovz6n1yP)
  * [ContigPermSpace](#noV6M8RJCt)
  * [SpaceMemRegionOopsIterClosure](#noH5pa7CM-)
  * [DirtyCardToOopClosure](#noxdeJT0ko)
  * [CompactPoint](#nouUxf0P_t)
  * [Filtering_DCTOC](#noKGN98dil)
  * [ContiguousSpaceDCTOC](#noaYncFxCR)
  * [VerifyOldOopClosure](#noExTSXlGp)


---
## <a name="noeb9cq9Z9" id="noeb9cq9Z9">Space</a>

### 概要(Summary)
Java ヒープ用のメモリ領域を管理するためのクラス (の基底クラス) (See: [here](no3718kvd.html) for details).

Java ヒープ用のメモリ領域を管理するクラスは使用する GC アルゴリズムによって異なるが,
このクラスは GC アルゴリズムが ParallelScavenge 以外の場合に使用される
(つまり, G1CollectedHeap および GenCollectedHeap 用) (See: ImmutableSpace).

以下のような機能を提供する.

* 対象の領域内からメモリを確保する機能
* 対象の領域の使用量／空き容量を取得する機能
* 対象の領域内のオブジェクトに対して iterate 処理を行う機能
* etc

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

なお, このクラスにおいては以下のような不変条件がある. 
(とコメントには書いてあるが, top は ContiguousSpace のサブクラスでないと存在しないような... 
なお, bottom は領域の下端, top は使用済みの範囲の終端, end は領域の上端, を指す)

* bottom と end は page 境界で align されている
* "`bottom() <= top() <= end()`" という不等式が成り立つ


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // A Space describes a heap area. Class Space is an abstract
    // base class.
    //
    // Space supports allocation, size computation and GC support is provided.
    //
    // Invariant: bottom() and end() are on page_size boundaries and
    // bottom() <= top() <= end()
    // top() is inclusive and end() is exclusive.
    
    class Space: public CHeapObj {
```


#### 内部構造(Internal structure)
内部には以下のようなフィールドを含む.

_bottom は領域の下端のアドレスを, _end は領域の上端のアドレスを格納する.


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
      HeapWord* _bottom;
      HeapWord* _end;
```




### 詳細(Details)
See: [here](../doxygen/classSpace.html) for details

---
## <a name="nompeS3kVU" id="nompeS3kVU">CompactibleSpace</a>

### 概要(Summary)
GC によるコンパクション処理をサポートした Space クラス (の基底クラス) (See: [here](no3718kvd.html) for details).

コメントによると, 大抵は連続領域になるがそうである必要性はなく, 
例えば通常時はフリーリスト管理で Full GC 時のみコンパクションを行うという実装も考えられる, とのこと.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // A space that supports compaction operations.  This is usually, but not
    // necessarily, a space that is normally contiguous.  But, for example, a
    // free-list-based space whose normal collection is a mark-sweep without
    // compaction could still support compaction in full GC's.
    
    class CompactibleSpace: public Space {
```

### 内部構造(Internal structure)
(スーパークラスである Space クラスのメソッドに加えて) 
以下のようなコンパクション処理を補佐するメソッドが定義されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
      // MarkSweep support phase2
    
      // Start the process of compaction of the current space: compute
      // post-compaction addresses, and insert forwarding pointers.  The fields
      // "cp->gen" and "cp->compaction_space" are the generation and space into
      // which we are currently compacting.  This call updates "cp" as necessary,
      // and leaves the "compaction_top" of the final value of
      // "cp->compaction_space" up-to-date.  Offset tables may be updated in
      // this phase as if the final copy had occurred; if so, "cp->threshold"
      // indicates when the next such action should be taken.
      virtual void prepare_for_compaction(CompactPoint* cp);
      // MarkSweep support phase3
      virtual void adjust_pointers();
      // MarkSweep support phase4
      virtual void compact();
```




### 詳細(Details)
See: [here](../doxygen/classCompactibleSpace.html) for details

---
## <a name="nosfPJ2U5x" id="nosfPJ2U5x">ContiguousSpace</a>

### 概要(Summary)
CompactibleSpace クラスの具象サブクラスの1つ.

このクラスは, 空き領域が連続領域になるように管理する場合用.
連続領域にすることで高速なメモリ確保操作やコンパクション操作が実現できている

(なお, 不連続に管理する場合の具象サブクラスは CompactibleFreeListSpace).


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // A space in which the free area is contiguous.  It therefore supports
    // faster allocation, and compaction.
    class ContiguousSpace: public CompactibleSpace {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 DefNewGeneration オブジェクトの _from_space フィールド
  
  (DefNewGeneration 使用時の From 領域を表す)

* 各 DefNewGeneration オブジェクトの _to_space フィールド
  
  (DefNewGeneration 使用時の To 領域を表す)


```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.cpp))
    DefNewGeneration::DefNewGeneration(ReservedSpace rs,
                                       size_t initial_size,
                                       int level,
                                       const char* policy)
    ...
    {
    ...
      _from_space = new ContiguousSpace();
      _to_space   = new ContiguousSpace();
```

#### 生成箇所(where its instances are created)
DefNewGeneration::DefNewGeneration() 内で(のみ)生成されている.

### 内部構造(Internal structure)
内部では, 現在使用済みの範囲の終端を _top フィールドが指している 

(_bottom から _top までが使用済みの範囲, _top から _end までが未使用の範囲).


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
      HeapWord* _top;
```

### 備考(Notes)
このクラスは G1CollectedHeap および GenCollectedHeap 用のクラスだが (See: Space), 
ParallelScavengeHeap 使用時についても類似した MutableSpace というクラスが使用されている (See: MutableSpace).




### 詳細(Details)
See: [here](../doxygen/classContiguousSpace.html) for details

---
## <a name="noIZyKziSy" id="noIZyKziSy">EdenSpace</a>

### 概要(Summary)
DefNewGeneration クラス用の補助クラス (See: DefNewGeneration).

DefNewGeneration 使用時の Eden 領域を表すためのクラスの 1つ.
このクラスは, GC アルゴリズムが CMS (incremental mode) ではない場合用 (See: ConcEdenSpace).

(<= もう少し正確に言うと, 
"DefNewGeneration 使用時 && (CMS 非使用時 || CMSIncrementalMode オプション非指定時)", ためのクラス.
つまり "Serial GC && Serial Old GC 時" か "Serial GC && CMS GC (非 incremental mode) 時" のためのクラス)


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // Class EdenSpace describes eden-space in new generation.
    ...
    class EdenSpace : public ContiguousSpace {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 DefNewGeneration オブジェクトの _eden_space フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
DefNewGeneration::DefNewGeneration() 内で(のみ)生成されている.

(なお EdenSpace が生成されるかどうかは, GC アルゴリズム及び CMSIncrementalMode オプションの値によって決まる.
GC アルゴリズムが CMS でかつ CMSIncrementalMode オプションが true の場合には
(CollectorPolicy::has_soft_ended_eden() が true を返すため) ConcEdenSpace が生成される.
そうでない場合には EdenSpace が生成される)


```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.cpp))
    DefNewGeneration::DefNewGeneration(ReservedSpace rs,
                                       size_t initial_size,
                                       int level,
                                       const char* policy)
    ...
    {
    ...
      if (GenCollectedHeap::heap()->collector_policy()->has_soft_ended_eden()) {
        _eden_space = new ConcEdenSpace(this);
      } else {
        _eden_space = new EdenSpace(this);
      }
```

#### 参考(for your information): CollectorPolicy::has_soft_ended_eden()
See: [here](no7882x5H.html) for details
#### 参考(for your information): ConcurrentMarkSweepPolicy::has_soft_ended_eden()
See: [here](no7882-DO.html) for details



### 詳細(Details)
See: [here](../doxygen/classEdenSpace.html) for details

---
## <a name="no8FHK1G5-" id="no8FHK1G5-">ConcEdenSpace</a>

### 概要(Summary)
DefNewGeneration クラス用の補助クラス (See: DefNewGeneration).

DefNewGeneration 使用時の Eden 領域を表すためのクラスの 1つ.
このクラスは, GC アルゴリズムが CMS (incremental mode) の場合用 (See: EdenSpace).

(<= もう少し正確に言うと, 
"DefNewGeneration 使用時 && CMS 使用時 && CMSIncrementalMode オプション指定時", ためのクラス.
つまり "Serial GC && CMS (incremental mode) 時" のためのクラス)


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // Class ConcEdenSpace extends EdenSpace for the sake of safe
    // allocation while soft-end is being modified concurrently
    
    class ConcEdenSpace : public EdenSpace {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 DefNewGeneration オブジェクトの _eden_space フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
DefNewGeneration::DefNewGeneration() 内で(のみ)生成されている.

(なお EdenSpace が生成されるかどうかは, GC アルゴリズム及び CMSIncrementalMode オプションの値によって決まる.
GC アルゴリズムが CMS でかつ CMSIncrementalMode オプションが true の場合には
(CollectorPolicy::has_soft_ended_eden() が true を返すため) ConcEdenSpace が生成される.
そうでない場合には EdenSpace が生成される)


```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.cpp))
    DefNewGeneration::DefNewGeneration(ReservedSpace rs,
                                       size_t initial_size,
                                       int level,
                                       const char* policy)
    ...
    {
    ...
      if (GenCollectedHeap::heap()->collector_policy()->has_soft_ended_eden()) {
        _eden_space = new ConcEdenSpace(this);
      } else {
        _eden_space = new EdenSpace(this);
      }
```

#### 参考(for your information): CollectorPolicy::has_soft_ended_eden()
See: [here](no7882x5H.html) for details
#### 参考(for your information): ConcurrentMarkSweepPolicy::has_soft_ended_eden()
See: [here](no7882-DO.html) for details
### 備考(Notes)
CMSIncrementalMode オプションは "-Xincgc" というオプション名でも指定可能 
(-Xincgc は CMSIncrementalMode のエイリアス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/arguments.cpp))
        // -Xincgc: i-CMS
        } else if (match_option(option, "-Xincgc", &tail)) {
          FLAG_SET_CMDLINE(bool, UseConcMarkSweepGC, true);
          FLAG_SET_CMDLINE(bool, CMSIncrementalMode, true);
```

### 内部構造(Internal structure)
内部的には, concurrent に soft-end を変更されても動作するように変更されている.




### 詳細(Details)
See: [here](../doxygen/classConcEdenSpace.html) for details

---
## <a name="noYm5xLT_L" id="noYm5xLT_L">OffsetTableContigSpace</a>

### 概要(Summary)
BlockOffsetArray を用いることで block_start() メソッドを効率的にサポートした ContiguousSpace クラス (See: BlockOffsetArray).

(なお, コメントでは abstract base class と書かれているが, ソース中でインスタンスが生成されている箇所は存在する.
もう少し具体的に言うと, 
CDS 機能使用時に CompactingPermGenGen 内の ro_space 及び rw_space の管理に用いられている)


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // A ContigSpace that Supports an efficient "block_start" operation via
    // a BlockOffsetArray (whose BlockOffsetSharedArray may be shared with
    // other spaces.)  This is the abstract base class for old generation
    // (tenured, perm) spaces.
    
    class OffsetTableContigSpace: public ContiguousSpace {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 CompactingPermGenGen オブジェクトの _ro_space フィールド
* 各 CompactingPermGenGen オブジェクトの _rw_space フィールド

#### 生成箇所(where its instances are created)
CompactingPermGenGen::CompactingPermGenGen() 内で(のみ)生成されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/compactingPermGenGen.cpp))
    CompactingPermGenGen::CompactingPermGenGen(ReservedSpace rs,
                                               ReservedSpace shared_rs,
                                               size_t initial_byte_size,
                                               int level, GenRemSet* remset,
                                               ContiguousSpace* space,
                                               PermanentGenerationSpec* spec_) :
    ...
      // Allocate shared spaces
      if (spec()->enable_shared_spaces()) {
    ...
        _ro_space = new OffsetTableContigSpace(_ro_bts,
                      MemRegion(readonly_bottom, readonly_end));
    ...
        _rw_space = new OffsetTableContigSpace(_rw_bts,
                      MemRegion(readwrite_bottom, readwrite_end));
```




### 詳細(Details)
See: [here](../doxygen/classOffsetTableContigSpace.html) for details

---
## <a name="noovz6n1yP" id="noovz6n1yP">TenuredSpace</a>

### 概要(Summary)
TenuredGeneration クラス用の補助クラス (See: TenuredGeneration).

TenuredGeneration 使用時の Old Generation を表すためのクラス
(= TenuredGeneration クラス内で実際のメモリ領域の管理を担当するクラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // Class TenuredSpace is used by TenuredGeneration
    
    class TenuredSpace: public OffsetTableContigSpace {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 TenuredGeneration オブジェクトの _the_space フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created) 
TenuredGeneration::TenuredGeneration() 内で(のみ)生成されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/tenuredGeneration.cpp))
    TenuredGeneration::TenuredGeneration(ReservedSpace rs,
                                         size_t initial_byte_size, int level,
                                         GenRemSet* remset) :
    ...
    {
    ...
      _the_space  = new TenuredSpace(_bts, MemRegion(bottom, end));
```




### 詳細(Details)
See: [here](../doxygen/classTenuredSpace.html) for details

---
## <a name="noV6M8RJCt" id="noV6M8RJCt">ContigPermSpace</a>

### 概要(Summary)
CompactingPermGenGen クラス用の補助クラス (See: CompactingPermGenGen).

CompactingPermGenGen 使用時の Perm Generation を表すためのクラス.
(= CompactingPermGenGen クラス内で実際のメモリ領域の管理を担当するクラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // Class ContigPermSpace is used by CompactingPermGen
    
    class ContigPermSpace: public OffsetTableContigSpace {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CompactingPermGenGen オブジェクトの _the_space フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created) 
CompactingPermGenGen::CompactingPermGenGen() 内で(のみ)生成されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/compactingPermGenGen.cpp))
    CompactingPermGenGen::CompactingPermGenGen(ReservedSpace rs,
                                               ReservedSpace shared_rs,
                                               size_t initial_byte_size,
                                               int level, GenRemSet* remset,
                                               ContiguousSpace* space,
                                               PermanentGenerationSpec* spec_) :
    ...
      // Allocate the unshared (default) space.
      _the_space = new ContigPermSpace(_bts,
                   MemRegion(unshared_bottom, heap_word_size(initial_byte_size)));
```




### 詳細(Details)
See: [here](../doxygen/classContigPermSpace.html) for details

---
## <a name="noH5pa7CM-" id="noH5pa7CM-">SpaceMemRegionOopsIterClosure</a>

### 概要(Summary)
Space クラス(とそのサブクラス)の処理で使用される補助クラス.

他の OopClosure と組み合わせて使用される Closure クラス (より正確には OopClosure クラス).
「指定した OopClosure の効果をあるアドレス範囲のみに限定したい」という場合に使用される.


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // An oop closure that is circumscribed by a filtering memory region.
    class SpaceMemRegionOopsIterClosure: public OopClosure {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* ContiguousSpace::oop_iterate()
* CompactibleFreeListSpace::oop_iterate()

### 内部構造(Internal structure)
コンストラクタ引数として, OopClosure オブジェクト及びその適用範囲を示す MemRegion を受け取る.


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
      SpaceMemRegionOopsIterClosure(OopClosure* cl, MemRegion mr):
```

実行する処理自体は, コンストラクタ引数で渡された OopClosure を呼び出すだけ.

ただし, 処理対象のアドレスが (同じくコンストラクタ引数で渡された) MemRegion 内に入っていなければ呼び出さない.

#### 参考(for your information): SpaceMemRegionOopsIterClosure::do_oop()

```cpp
    ((cite: hotspot/src/share/vm/memory/space.cpp))
    void SpaceMemRegionOopsIterClosure::do_oop(oop* p)       { SpaceMemRegionOopsIterClosure::do_oop_work(p); }
    void SpaceMemRegionOopsIterClosure::do_oop(narrowOop* p) { SpaceMemRegionOopsIterClosure::do_oop_work(p); }
```

#### 参考(for your information): SpaceMemRegionOopsIterClosure::do_oop_work()
See: [here](no788297S.html) for details



### 詳細(Details)
See: [here](../doxygen/classSpaceMemRegionOopsIterClosure.html) for details

---
## <a name="noxdeJT0ko" id="noxdeJT0ko">DirtyCardToOopClosure</a>

### 概要(Summary)
write barrier によって dirty 化されているメモリ範囲に対して何らかの処理を行う Closure クラス.

他の OopClosure と組み合わせて使用される Closure クラス (より正確には MemRegionClosureRO クラス).
「指定した OopClosure の効果を dirty 化されている箇所のポインタフィールドのみに限定したい」という場合に使用される
(なお, write barrier は多少不正確なので, 指定された MemRegion の外まで処理対象がはみ出す場合もある).

なお, このクラスが想定しているメモリ領域は連続的でもないし境界もない.
それ以外のメモリ領域については, 適切なサブクラスを用意している 
(連続的な領域の場合は ContiguousDCTOC, 境界付きの場合は Filtering_DCTOC, etc).


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // A MemRegionClosure (ResourceObj) whose "do_MemRegion" function applies an
    // OopClosure to (the addresses of) all the ref-containing fields that could
    // be modified by virtue of the given MemRegion being dirty. (Note that
    // because of the imprecise nature of the write barrier, this may iterate
    // over oops beyond the region.)
    // This base type for dirty card to oop closures handles memory regions
    // in non-contiguous spaces with no boundaries, and should be sub-classed
    // to support other space types. See ContiguousDCTOC for a sub-class
    // that works with ContiguousSpaces.
    
    class DirtyCardToOopClosure: public MemRegionClosureRO {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
現状では, 常に ClearNoncleanCardWrapper とセットで使用されている.
実際に処理範囲を dirty 化されている範囲に絞る作業は ClearNoncleanCardWrapper が行っている 
(See: ClearNoncleanCardWrapper).

#### 生成箇所(where its instances are created)
Space::new_dcto_cl() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

(なお, DirtyCardToOopClosure のサブクラス用に Space::new_dcto_cl() がオーバーライドされるケースもある.
 それらのオーバーライドされたメソッドも同じパスでのみ呼び出されている.
 (See: ContiguousSpaceDCTOC, FreeListSpace_DCTOC))

<div class="flow-abst"><pre>
* OneContigSpaceCardGeneration 用の処理

  OneContigSpaceCardGeneration::younger_refs_iterate()
  -&gt; Generation::younger_refs_in_space_iterate()
     -&gt; CardTableRS::younger_refs_in_space_iterate()
        -&gt; CardTableModRefBS::non_clean_card_iterate_possibly_parallel()
           -&gt; Space::new_dcto_cl()

* CompactingPermGenGen 用の処理

  CompactingPermGenGen::younger_refs_iterate()
  -&gt; CardTableRS::younger_refs_in_space_iterate()
     -&gt; (同上)

* ConcurrentMarkSweepGeneration 用の処理 (CMS 用)

  ConcurrentMarkSweepGeneration::younger_refs_iterate()
  -&gt; Generation::younger_refs_in_space_iterate()
     -&gt; (同上)

* ParNew 用の処理

  CardTableRS::younger_refs_in_space_iterate()
  -&gt; CardTableModRefBS::non_clean_card_iterate_possibly_parallel()
     -&gt; CardTableModRefBS::non_clean_card_iterate_parallel_work()
        -&gt; CardTableModRefBS::process_stride()
           -&gt; Space::new_dcto_cl()
</pre></div>

#### 使用箇所(where its instances are used)
ClearNoncleanCardWrapper::do_MemRegion() 内で(のみ)使用されている.

### 内部構造(Internal structure)
DirtyCardToOopClosure::do_MemRegion() が呼ばれると, 
最終的には DirtyCardToOopClosure::walk_mem_region() で処理が行われる.
このため, サブクラスを作成する場合は walk_mem_region() メソッドだけをオーバーライドすればいい.

#### 参考(for your information): DirtyCardToOopClosure::do_MemRegion()
See: [here](no34207rK.html) for details



### 詳細(Details)
See: [here](../doxygen/classDirtyCardToOopClosure.html) for details

---
## <a name="nouUxf0P_t" id="nouUxf0P_t">CompactPoint</a>

### 概要(Summary)
Generation::prepare_for_compaction() 内で使用される補助クラス(StackObjクラス).

Garbage Collection におけるコンパクション処理時に, コンパクション先領域の情報を管理するために使われる.


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // A structure to represent a point at which objects are being copied
    // during compaction.
    class CompactPoint : public StackObj {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* GenMarkSweep::mark_sweep_phase2() 内

  (正確には, GenMarkSweep::mark_sweep_phase2() の場合は,
   GenMarkSweep::mark_sweep_phase2() 内で直接生成されているほか,
   そこから呼び出される GenCollectedHeap::prepare_for_compaction() 内でも生成されている)

* G1MarkSweep::mark_sweep_phase2() 内

#### 使用箇所(where its instances are used)
Generation::prepare_for_compaction() 内(およびそこから呼び出される補助関数内)で(のみ)使用されている.

### 内部構造(Internal structure)
単なる構造体のようなクラス.
内部には以下の3つの public フィールドのみを持つ (そしてメソッドはない).


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    public:
      Generation* gen;
      CompactibleSpace* space;
      HeapWord* threshold;
```




### 詳細(Details)
See: [here](../doxygen/classCompactPoint.html) for details

---
## <a name="noKGN98dil" id="noKGN98dil">Filtering_DCTOC</a>

### 概要(Summary)
境界を指定可能な DirtyCardToOopClosure クラスの基底クラス.
「指定した OopClosure を dirty 化された領域でかつ指定のアドレス以下の範囲のみに適用したい」という場合に使用される


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // A dirty card to oop closure that does filtering.
    // It knows how to filter out objects that are outside of the _boundary.
    class Filtering_DCTOC : public DirtyCardToOopClosure {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
      virtual void walk_mem_region_with_cl(MemRegion mr,
                                           HeapWord* bottom, HeapWord* top,
                                           OopClosure* cl) = 0;
      virtual void walk_mem_region_with_cl(MemRegion mr,
                                           HeapWord* bottom, HeapWord* top,
                                           FilteringClosure* cl) = 0;
```

### 内部構造(Internal structure)
DirtyCardToOopClosure::walk_mem_region() がオーバーライドされており, 
指定したアドレス以上の範囲には OopClosure を適用しないように変更されている
(なお, 現在の実装では内部的に FilteringClosure を用いることで実現されている)
(See: DirtyCardToOopClosure).

なお, Filtering_DCTOC::walk_mem_region() が呼ばれると, 
最終的には Filtering_DCTOC::walk_mem_region_with_cl() で処理が行われる.
このため, サブクラスを作成する場合は walk_mem_region_with_cl() メソッドだけをオーバーライドすればいい.

#### 参考(for your information): Filtering_DCTOC::walk_mem_region()




### 詳細(Details)
See: [here](../doxygen/classFiltering__DCTOC.html) for details

---
## <a name="noaYncFxCR" id="noaYncFxCR">ContiguousSpaceDCTOC</a>

### 概要(Summary)
Filtering_DCTOC クラスの具象サブクラスの1つ.

このクラスは ContiguousSpace 用 (= メモリ領域が連続している場合用).


```cpp
    ((cite: hotspot/src/share/vm/memory/space.hpp))
    // A dirty card to oop closure for contiguous spaces
    // (ContiguousSpace and sub-classes).
    // It is a FilteringClosure, as defined above, and it knows:
    //
    // 1. That the actual top of any area in a memory region
    //    contained by the space is bounded by the end of the contiguous
    //    region of the space.
    // 2. That the space is really made up of objects and not just
    //    blocks.
    
    class ContiguousSpaceDCTOC : public Filtering_DCTOC {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ContiguousSpace::new_dcto_cl() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

(なお, このパスはオーバーライド元である Space::new_dcto_cl() の呼び出しパスと同一.
 (See: DirtyCardToOopClosure))

<div class="flow-abst"><pre>
* OneContigSpaceCardGeneration 用の処理
  OneContigSpaceCardGeneration::younger_refs_iterate()
  -&gt; Generation::younger_refs_in_space_iterate()
     -&gt; CardTableRS::younger_refs_in_space_iterate()
        -&gt; CardTableModRefBS::non_clean_card_iterate_possibly_parallel()
           -&gt; Space::new_dcto_cl()

* CompactingPermGenGen 用の処理
  CompactingPermGenGen::younger_refs_iterate()
  -&gt; CardTableRS::younger_refs_in_space_iterate()
     -&gt; (同上)

* ConcurrentMarkSweepGeneration 用の処理 (CMS 用)
  ConcurrentMarkSweepGeneration::younger_refs_iterate()
  -&gt; Generation::younger_refs_in_space_iterate()
     -&gt; (同上)

* ParNew 用の処理
  CardTableRS::younger_refs_in_space_iterate()
  -&gt; CardTableModRefBS::non_clean_card_iterate_possibly_parallel()
     -&gt; CardTableModRefBS::non_clean_card_iterate_parallel_work()
        -&gt; CardTableModRefBS::process_stride()
           -&gt; Space::new_dcto_cl()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classContiguousSpaceDCTOC.html) for details

---
## <a name="noExTSXlGp" id="noExTSXlGp">VerifyOldOopClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/space.cpp))
    class VerifyOldOopClosure : public OopClosure {
```

### 使われ方(Usage)
OffsetTableContigSpace::verify() 内で(のみ)使用されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/space.cpp))
    void OffsetTableContigSpace::verify(bool allow_dirty) const {
    ...
      VerifyOldOopClosure blk;      // Does this do anything?
      blk._allow_dirty = allow_dirty;
    ...
          blk._the_obj = oop(p);
          oop(p)->oop_iterate(&blk);
```




### 詳細(Details)
See: [here](../doxygen/classVerifyOldOopClosure.html) for details

---
