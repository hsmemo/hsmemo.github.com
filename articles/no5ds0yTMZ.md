---
layout: default
title: MutableNUMASpace およびその補助クラス (MutableNUMASpace, MutableNUMASpace::LGRPSpace)
---
[Top](../index.html)

#### MutableNUMASpace およびその補助クラス (MutableNUMASpace, MutableNUMASpace::LGRPSpace)

これらは, メモリ管理用のクラス.
より具体的に言うと, Java ヒープ用のメモリ領域を管理するためのクラス (See: [here](no3718kvd.html) and [here](no28916ddy.html) for details).


### クラス一覧(class list)

  * [MutableNUMASpace](#noYr9f334a)
  * [MutableNUMASpace::LGRPSpace](#noUjRpD_0U)


---
## <a name="noYr9f334a" id="noYr9f334a">MutableNUMASpace</a>

### 概要(Summary)
NUMA 上での配置を考慮した MutableSpace クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/mutableNUMASpace.hpp))
    class MutableNUMASpace : public MutableSpace {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PSYoungGen クラスの _eden_space フィールドに(のみ)格納されている.

(ただしこのフィールドには, MutableNUMASpace オブジェクトではなく,
スーパークラスである MutableSpace オブジェクトが格納されることもある.
その場合にはこのクラスはどこからも使用されない.)

#### 生成箇所(where its instances are created)
PSYoungGen::initialize_work() 内で(のみ)生成されている.

(なお MutableNUMASpace オブジェクトが生成されるかどうかは UseNUMA オプションの値によって決まる.
UseNUMA オプションが指定されている場合は MutableNUMASpace が生成され, そうでない場合は MutableSpace が生成される)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psYoungGen.cpp))
    void PSYoungGen::initialize_work() {
    ...
      if (UseNUMA) {
        _eden_space = new MutableNUMASpace(virtual_space()->alignment());
      } else {
        _eden_space = new MutableSpace(virtual_space()->alignment());
      }
```

### 内部構造(Internal structure)
インターフェース的には MutableSpace クラスと完全に同じであり, 内部実装だけが異なる.

* 内部では, メモリ領域を複数の "chunk" に分けて管理しており,
  各 chunk が NUMA 上での "locality group" に対応する.

  (ところで, "locality group" って Solaris 以外でもメジャーな用語なんだろうか??)

* 各スレッドのメモリ確保処理はそのスレッドに対応する "home locality group" の chunk から行われ,
  どれかの chunk が一杯になると Minor GC が実行される.

* chunk のサイズは, adaptive size policy によって動的に調整される.
  これは, fragmentation によって eden 領域のメモリ効率が悪化するのを防ぐ効果がある.

  fragmentation はスレッド間での確保量に偏りがあると発生する
  (アプリケーション固有の事情での偏りや, OS のスレッドスケジューリングの偏りなどが考えられる).
  chunk のサイズを適切に変更するため, 各 GC 間でのスレッド毎のメモリ確保量を記録しており,
  その確保パターンを反映させてサイズ変更を行う.
  また, 以下のクラスやコマンドラインオプションによってもチューニングできる.

  * AdaptiveWeightedAverage クラス (See: AdaptiveWeightedAverage)
  * NUMASpaceResizeRate コマンドラインオプション

* なお, chunk には remote memory page が含まれることもある (local memory が足りなかった場合など).
  こういった場合のために, "page-scanner" が導入されている.

  "page-scanner" は GC 直後にページを確認し, remote memory があれば解放することで,
  その後に再確保された際にはより上手くいくように仕向ける.
  (この方式は, 負荷が高くてたくさんのプロセスがメモリを奪い合うような状況で特に有効だと判明している)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/mutableNUMASpace.hpp))
    /*
     *    The NUMA-aware allocator (MutableNUMASpace) is basically a modification
     * of MutableSpace which preserves interfaces but implements different
     * functionality. The space is split into chunks for each locality group
     * (resizing for adaptive size policy is also supported). For each thread
     * allocations are performed in the chunk corresponding to the home locality
     * group of the thread. Whenever any chunk fills-in the young generation
     * collection occurs.
     *   The chunks can be also be adaptively resized. The idea behind the adaptive
     * sizing is to reduce the loss of the space in the eden due to fragmentation.
     * The main cause of fragmentation is uneven allocation rates of threads.
     * The allocation rate difference between locality groups may be caused either by
     * application specifics or by uneven LWP distribution by the OS. Besides,
     * application can have less threads then the number of locality groups.
     * In order to resize the chunk we measure the allocation rate of the
     * application between collections. After that we reshape the chunks to reflect
     * the allocation rate pattern. The AdaptiveWeightedAverage exponentially
     * decaying average is used to smooth the measurements. The NUMASpaceResizeRate
     * parameter is used to control the adaptation speed by restricting the number of
     * bytes that can be moved during the adaptation phase.
     *   Chunks may contain pages from a wrong locality group. The page-scanner has
     * been introduced to address the problem. Remote pages typically appear due to
     * the memory shortage in the target locality group. Besides Solaris would
     * allocate a large page from the remote locality group even if there are small
     * local pages available. The page-scanner scans the pages right after the
     * collection and frees remote pages in hope that subsequent reallocation would
     * be more successful. This approach proved to be useful on systems with high
     * load where multiple processes are competing for the memory.
     */
```



### 詳細(Details)
See: [here](../doxygen/classMutableNUMASpace.html) for details

---
## <a name="noUjRpD_0U" id="noUjRpD_0U">MutableNUMASpace::LGRPSpace</a>

### 概要(Summary)
MutableNUMASpace クラス内で使用される補助クラス.

それぞれの Locality Group を表すクラス.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/mutableNUMASpace.hpp))
      class LGRPSpace : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classMutableNUMASpace_1_1LGRPSpace.html) for details

---
