---
layout: default
title: AdjoiningVirtualSpaces クラス 
---
[Top](../index.html)

#### AdjoiningVirtualSpaces クラス 



---
## <a name="no5zurzdqF" id="no5zurzdqF">AdjoiningVirtualSpaces</a>

### 概要(Summary)
AdjoiningGenerations クラス内で使用される補助クラス (というか ASPSYoungGen と ASPSOldGen のための補助クラス).

AdjoiningGenerations が管理する generation オブジェクト (ASPSYoungGen, ASPSOldGen) について, 
それらの間にまたがった領域長変更を可能にするためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningVirtualSpaces.hpp))
    class AdjoiningVirtualSpaces {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
AdjoiningGenerations クラスの _virtual_spaces インスタンスフィールドに(のみ)格納されている.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningGenerations.hpp))
    class AdjoiningGenerations : public CHeapObj {
    ...
      // The spaces used by the two generations.
      AdjoiningVirtualSpaces _virtual_spaces;
```

#### 使用箇所(where its instances are used)
AdjoiningGenerations のコンストラクタ内で ASPSYoungGen オブジェクトと ASPSOldGen オブジェクトを生成する際に使用される.

#### 参考(for your information): AdjoiningGenerations::AdjoiningGenerations()
See: [here](no344AYS.html) for details
### 内部構造(Internal structure)
AdjoiningVirtualSpaces 自体は連続した1つのメモリ領域を表す.
このメモリ領域内には, 2つの PSVirtualSpace を格納することができる.

それぞれの PSVirtualSpace はメモリ領域の両端に配置され, どちらも中央方向に向かって伸び縮みさせることができる
(このため中央部分 (= 2つの PSVirtualSpace の境界) が可変になっており, 2つの領域間にまたがった領域長変更が行える).

(実際の使用時には L 側に ASPSOldGen, H 側に ASPSYoungGen を配置する. 
ASPSOldGen は (先頭から埋めていくため後方は空になっており) 後方の境界を移動しても問題ない.
また ASPSYoungGen は (Minor GC 時に空になるのでこのタイミングでなら) 前方の境界を移動しても問題ない)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningVirtualSpaces.hpp))
    // Contains two virtual spaces that each can individually span
    // most of the reserved region but committed parts of which
    // cannot overlap.
    //
    //      +-------+ <--- high_boundary for H
    //      |       |
    //      |   H   |
    //      |       |
    //      |       |
    //      |       |
    //      --------- <--- low for H
    //      |       |
    //      ========= <--- low_boundary for H, high_boundary for L
    //      |       |
    //      |       |
    //      |       |
    //      --------- <--- high for L
    //      |       |
    //      |   L   |
    //      |       |
    //      |       |
    //      |       |
    //      +-------+ <--- low_boundary for L
    //
    // Each virtual space in the AdjoiningVirtualSpaces grows and shrink
    // within its reserved region (between the low_boundary and the
    // boundary) independently.  If L want to grow above its high_boundary,
    // then the high_boundary of L and the low_boundary of H must be
    // moved up consistently.  AdjoiningVirtualSpaces provide the
    // interfaces for moving the this boundary.
```




### 詳細(Details)
See: [here](../doxygen/classAdjoiningVirtualSpaces.html) for details

---
