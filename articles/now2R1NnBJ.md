---
layout: default
title: MemRegion クラス関連のクラス (MemRegion, MemRegionClosure, MemRegionClosureRO)
---
[Top](../index.html)

#### MemRegion クラス関連のクラス (MemRegion, MemRegionClosure, MemRegionClosureRO)

これらは, メモリ管理用のユーティリティ・クラス (See: [here](no3718kvd.html) for details).


### クラス一覧(class list)

  * [MemRegion](#no9mPvmCw3)
  * [MemRegionClosure](#nodmhU2liL)
  * [MemRegionClosureRO](#no4r61xvor)


---
## <a name="no9mPvmCw3" id="no9mPvmCw3">MemRegion</a>

### 概要(Summary)
メモリ管理用のユーティリティ・クラス.
仮想メモリ空間内の「連続したメモリ領域」を表す.

メモリ中の特定の範囲(連続領域)を表したい場面で汎用的に用いられている.

(なお, このオブジェクト自体はものすごく小さいので VALUE_OBJ クラスになっている)

```
    ((cite: hotspot/src/share/vm/memory/memRegion.hpp))
    // A very simple data structure representing a contigous region
    // region of address space.
    
    // Note that MemRegions are passed by value, not by reference.
    // The intent is that they remain very small and contain no
    // objects.
    
    class MemRegion VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている. (#TODO)

### 内部構造(Internal structure)
内部には2つのフィールド(のみ)を保持する (領域の開始アドレスと領域長).

```
    ((cite: hotspot/src/share/vm/memory/memRegion.hpp))
      HeapWord* _start;
      size_t    _word_size;
```

メソッドとしては,
指定のポインタが領域内に入っているかどうかを判定する contains() メソッドなどを備える.

```
    ((cite: hotspot/src/share/vm/memory/memRegion.hpp))
      bool contains(const void* addr) const {
        return addr >= (void*)_start && addr < (void*)end();
      }
```

領域同士の集合演算を行うメソッド等も用意されている.

```
    ((cite: hotspot/src/share/vm/memory/memRegion.hpp))
      MemRegion intersection(const MemRegion mr2) const;
      // regions must overlap or be adjacent
      MemRegion _union(const MemRegion mr2) const;
      // minus will fail a guarantee if mr2 is interior to this,
      // since there's no way to return 2 disjoint regions.
      MemRegion minus(const MemRegion mr2) const;
```




### 詳細(Details)
See: [here](../doxygen/classMemRegion.html) for details

---
## <a name="nodmhU2liL" id="nodmhU2liL">MemRegionClosure</a>

### 概要(Summary)
MemRegion に対して何らかの処理を行う Closure クラスの基底クラス.

(なお, このクラスは Closure クラスのサブクラスではなく, StackObj のサブクラスになっている)


```
    ((cite: hotspot/src/share/vm/memory/memRegion.hpp))
    // For iteration over MemRegion's.
    
    class MemRegionClosure : public StackObj {
```

### 使われ方(Usage)
MemRegion を処理する do_MemRegion() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```
    ((cite: hotspot/src/share/vm/memory/memRegion.hpp))
      virtual void do_MemRegion(MemRegion mr) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classMemRegionClosure.html) for details

---
## <a name="no4r61xvor" id="no4r61xvor">MemRegionClosureRO</a>

### 概要(Summary)
MemRegionClosure の類似品.

スタック上に確保される MemRegionClosure と異なり, こちらのクラスは ResourceArea 内に確保される.


```
    ((cite: hotspot/src/share/vm/memory/memRegion.hpp))
    // A ResourceObj version of MemRegionClosure
    
    class MemRegionClosureRO: public MemRegionClosure {
```

### 内部構造(Internal structure)
new が ResourceObj として確保するように変更されている (そして delete は何もしない). 
この点以外に MemRegionClosure との違いはない. 

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```
    ((cite: hotspot/src/share/vm/memory/memRegion.hpp))
      void* operator new(size_t size, ResourceObj::allocation_type type) {
            return ResourceObj::operator new(size, type);
      }
      void* operator new(size_t size, Arena *arena) {
            return ResourceObj::operator new(size, arena);
      }
      void* operator new(size_t size) {
            return ResourceObj::operator new(size);
      }
    
      void  operator delete(void* p) {} // nothing to do
```




### 詳細(Details)
See: [here](../doxygen/classMemRegionClosureRO.html) for details

---
