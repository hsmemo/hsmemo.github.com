---
layout: default
title: GCStats クラス関連のクラス (GCStats, CMSGCStats)
---
[Top](../index.html)

#### GCStats クラス関連のクラス (GCStats, CMSGCStats)

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, それまでの GC 時における昇格(promotion)量の平均値を記録しておくクラス (See: [here](no3718kvd.html) for details).

(この情報は, GC 時に昇格量を予想して promotion 失敗が起きにくくするために使用される)

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcStats.hpp))
      // Avg amount promoted; used for avoiding promotion undo
```

(といっても, これらのクラスは処理のほとんど(というか全て?)を AdaptivePaddedNoZeroDevAverage に丸投げしているだけなので,
実質的には AdaptivePaddedNoZeroDevAverage のラッパーといった感じだが...)


### クラス一覧(class list)

  * [GCStats](#no9FiIVgV4)
  * [CMSGCStats](#noInaNQIxV)


---
## <a name="no9FiIVgV4" id="no9FiIVgV4">GCStats</a>

### 概要(Summary)
GC 時における昇格(promotion)量の平均値を記録しておくクラス.


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcStats.hpp))
    class GCStats : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 Generation オブジェクトの _gc_stats フィールド
* 各 PSAdaptiveSizePolicy オブジェクトの _gc_stats フィールド


```
    ((cite: hotspot/src/share/vm/memory/generation.hpp))
    class Generation: public CHeapObj {
    ...
      // Statistics for garbage collection
      GCStats* _gc_stats;
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psAdaptiveSizePolicy.hpp))
    class PSAdaptiveSizePolicy : public AdaptiveSizePolicy {
    ...
      // Statistical data gathered for GC
      GCStats _gc_stats;
```

(ただし, Generation::_gc_stats フィールドについては,
GCStats クラスではなくそのサブクラスのオブジェクトが格納されることもある (See: CMSGCStats))

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TenuredGeneration::TenuredGeneration()


```
    ((cite: hotspot/src/share/vm/memory/tenuredGeneration.cpp))
    TenuredGeneration::TenuredGeneration(ReservedSpace rs,
                                         size_t initial_byte_size, int level,
                                         GenRemSet* remset) :
    ...
    {
    ...
      _gc_stats = new GCStats();
```

* (PSAdaptiveSizePolicy クラスの _gc_stats フィールドは, ポインタ型ではなく実体なので,
  PSAdaptiveSizePolicy オブジェクトの生成時に一緒に生成される)

### 内部構造(Internal structure)
(中身は AdaptivePaddedNoZeroDevAverage のラッパーのような感じ)

内部には AdaptivePaddedNoZeroDevAverage を格納するフィールドがあるだけ.

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcStats.hpp))
      // Avg amount promoted; used for avoiding promotion undo
      // This class does not update deviations if the sample is zero.
      AdaptivePaddedNoZeroDevAverage*   _avg_promoted;
```

メソッドも, めぼしいものは AdaptivePaddedNoZeroDevAverage 型のフィールドへのアクセサだけ.

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcStats.hpp))
      AdaptivePaddedNoZeroDevAverage*  avg_promoted() const { return _avg_promoted; }
    
      // Average in bytes
      size_t average_promoted_in_bytes() const {
        return (size_t)_avg_promoted->average();
      }
    
      // Padded average in bytes
      size_t padded_average_promoted_in_bytes() const {
        return (size_t)_avg_promoted->padded_average();
      }
```

単なる AdaptivePaddedNoZeroDevAverage と違うのは, コンストラクタ引数の値が決まっている点くらい??

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcStats.cpp))
    GCStats::GCStats() {
        _avg_promoted       = new AdaptivePaddedNoZeroDevAverage(
                                                      AdaptiveSizePolicyWeight,
                                                      PromotedPadding);
    }
```




### 詳細(Details)
See: [here](../doxygen/classGCStats.html) for details

---
## <a name="noInaNQIxV" id="noInaNQIxV">CMSGCStats</a>

### 概要(Summary)
CMS 用の GCStats クラス.

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcStats.hpp))
    class CMSGCStats : public GCStats {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Generation オブジェクトの _gc_stats フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
ConcurrentMarkSweepGeneration::ConcurrentMarkSweepGeneration() 内で(のみ)生成されている.


```
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/concurrentMarkSweepGeneration.cpp))
    ConcurrentMarkSweepGeneration::ConcurrentMarkSweepGeneration(
         ReservedSpace rs, size_t initial_byte_size, int level,
         CardTableRS* ct, bool use_adaptive_freelists,
         FreeBlockDictionary::DictionaryChoice dictionaryChoice) :
      CardGeneration(rs, initial_byte_size, level, ct),
      _dilatation_factor(((double)MinChunkSize)/((double)(CollectedHeap::min_fill_size()))),
      _debug_collection_type(Concurrent_collection_type)
    {
    ...
      _gc_stats = new CMSGCStats();
```

### 内部構造(Internal structure)
スーパークラスである GCStats クラスと同じく, 実質的には AdaptivePaddedNoZeroDevAverage に処理を丸投げしているだけ.

GCStats (や単なる AdaptivePaddedNoZeroDevAverage) と違うのは, コンストラクタ引数の値くらい??

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcStats.cpp))
    CMSGCStats::CMSGCStats() {
        _avg_promoted       = new AdaptivePaddedNoZeroDevAverage(
                                                      CMSExpAvgFactor,
                                                      PromotedPadding);
    }
```




### 詳細(Details)
See: [here](../doxygen/classCMSGCStats.html) for details

---
