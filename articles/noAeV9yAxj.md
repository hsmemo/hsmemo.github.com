---
layout: default
title: Copy クラス 
---
[Top](../index.html)

#### Copy クラス 



---
## <a name="noAG3556QE" id="noAG3556QE">Copy</a>

### 概要(Summary)
高速なメモリコピー処理用のユーティリティ・クラス 
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

対象のアーキテクチャに最適化されたコピー関数を定義している.


```cpp
    ((cite: hotspot/src/share/vm/utilities/copy.hpp))
    class Copy : AllStatic {
```

なおこのクラスのコピー処理には4つの分類軸がある. 各メソッドの名前は以下の命名規則に従う.

(ただし, 現状では全組み合わせに対して網羅的にメソッドを定義しているわけではない模様).

  * `[ 'aligned_' | 'arrayof_' ]`
    
    align されているかどうか.

    (aligned は long 境界で aligned, arrayof は(現状では)HeapWord で aligned (long/double は 8-byte aligned) と想定. 
     指定がなければ mis-aligned と想定)

  * `('conjoint_' | 'disjoint_')`
    
    コピー元とコピー先にオーバーラップの恐れがあるかどうか

  * `('words' | 'bytes' | 'jshorts' | 'jints' | 'jlongs' | 'oops')`

    コピー処理の単位

  * `[ '_atomic' ]`
    
    各コピー単位毎にアトミックにコピーする必要があるかどうか


```cpp
    ((cite: hotspot/src/share/vm/utilities/copy.hpp))
      // Block copy methods have four attributes.  We don't define all possibilities.
      //   alignment: aligned to BytesPerLong
      //   arrayof:   arraycopy operation with both operands aligned on the same
      //              boundary as the first element of an array of the copy unit.
      //              This is currently a HeapWord boundary on all platforms, except
      //              for long and double arrays, which are aligned on an 8-byte
      //              boundary on all platforms.
      //              arraycopy operations are implicitly atomic on each array element.
      //   overlap:   disjoint or conjoint.
      //   copy unit: bytes or words (i.e., HeapWords) or oops (i.e., pointers).
      //   atomicity: atomic or non-atomic on the copy unit.
      //
      // Names are constructed thusly:
      //
      //     [ 'aligned_' | 'arrayof_' ]
      //     ('conjoint_' | 'disjoint_')
      //     ('words' | 'bytes' | 'jshorts' | 'jints' | 'jlongs' | 'oops')
      //     [ '_atomic' ]
      //
      // Except in the arrayof case, whatever the alignment is, we assume we can copy
      // whole alignment units.  E.g., if BytesPerLong is 2x word alignment, an odd
      // count may copy an extra word.  In the arrayof case, we are allowed to copy
      // only the number of copy units specified.
      //
      // All callees check count for 0.
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
処理はアーキテクチャに依存するので, ほとんどの部分は cpu/ 下の copy_${cpu} ファイルで定義されている.




### 詳細(Details)
See: [here](../doxygen/classCopy.html) for details

---
