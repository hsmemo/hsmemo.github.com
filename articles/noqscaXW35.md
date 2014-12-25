---
layout: default
title: G1BlockOffsetTable クラス関連のクラス (G1BlockOffsetTable, G1BlockOffsetSharedArray, G1BlockOffsetArray, G1BlockOffsetArrayContigSpace)
---
[Top](../index.html)

#### G1BlockOffsetTable クラス関連のクラス (G1BlockOffsetTable, G1BlockOffsetSharedArray, G1BlockOffsetArray, G1BlockOffsetArrayContigSpace)

これらは, G1GC 用の BlockOffsetTable クラス (See: [here](no3718kvd.html) for details).

### 概要(Summary)
コメントによると, 
これらは BlockOffsetTable クラス(およびそのサブクラス)とほぼ同じ, 
とのこと (See: BlockOffsetTable).

また, 
重複しているコードが多いので G1BlockOffsetTable の内容を BlockOffsetTable に merge して1つに統合したい, 
とも書かれている.

ただし, 
G1BlockOffsetTable の場合は block_start*() メソッドが non-const なので, 
統合後の BlockOffsetTable は現状の BlockOffsetTable よりも(コンパイラの最適化が効きにくいという意味で)
不利になるかもしれないのがネック, 
とのこと.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1BlockOffsetTable.hpp))
    // The CollectedHeap type requires subtypes to implement a method
    // "block_start".  For some subtypes, notably generational
    // systems using card-table-based write barriers, the efficiency of this
    // operation may be important.  Implementations of the "BlockOffsetArray"
    // class may be useful in providing such efficient implementations.
    //
    // While generally mirroring the structure of the BOT for GenCollectedHeap,
    // the following types are tailored more towards G1's uses; these should,
    // however, be merged back into a common BOT to avoid code duplication
    // and reduce maintenance overhead.
    //
    //    G1BlockOffsetTable (abstract)
    //    -- G1BlockOffsetArray                (uses G1BlockOffsetSharedArray)
    //       -- G1BlockOffsetArrayContigSpace
    //
    // A main impediment to the consolidation of this code might be the
    // effect of making some of the block_start*() calls non-const as
    // below. Whether that might adversely affect performance optimizations
    // that compilers might normally perform in the case of non-G1
    // collectors needs to be carefully investigated prior to any such
    // consolidation.
```

なお, これらのクラスは以下のような継承関係を持つ
(G1BlockOffsetTable や G1BlockOffsetArray は, 
 BlockOffsetTable や BlockOffsetArray と同じく, abstract class).

  * G1BlockOffsetTable (abstract class)
      * G1BlockOffsetArray (abstract class)
          * G1BlockOffsetArrayContigSpace


### クラス一覧(class list)

  * [G1BlockOffsetTable](#noFc31zDfx)
  * [G1BlockOffsetSharedArray](#no9GSFv-jG)
  * [G1BlockOffsetArray](#no5-6FUa1f)
  * [G1BlockOffsetArrayContigSpace](#noHMw7Nezy)


---
## <a name="noFc31zDfx" id="noFc31zDfx">G1BlockOffsetTable</a>

### 概要(Summary)
G1GC 用の BlockOffsetTable クラス (See: BlockOffsetTable).

(継承関係としては BlockOffsetTable クラスと何の関係もないが, 果たしている役割は類似)

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1BlockOffsetTable.hpp))
    class G1BlockOffsetTable VALUE_OBJ_CLASS_SPEC {
```




### 詳細(Details)
See: [here](../doxygen/classG1BlockOffsetTable.html) for details

---
## <a name="no9GSFv-jG" id="no9GSFv-jG">G1BlockOffsetSharedArray</a>

### 概要(Summary)
G1GC 用の BlockOffsetSharedArray クラス (See: BlockOffsetSharedArray).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1BlockOffsetTable.hpp))
    // This implementation of "G1BlockOffsetTable" divides the covered region
    // into "N"-word subregions (where "N" = 2^"LogN".  An array with an entry
    // for each such subregion indicates how far back one must go to find the
    // start of the chunk that includes the first word of the subregion.
    //
    // Each BlockOffsetArray is owned by a Space.  However, the actual array
    // may be shared by several BlockOffsetArrays; this is useful
    // when a single resizable area (such as a generation) is divided up into
    // several spaces in which contiguous allocation takes place,
    // such as, for example, in G1 or in the train generation.)
    
    // Here is the shared array type.
    
    class G1BlockOffsetSharedArray: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _bot_shared フィールドに(のみ)格納されている
(「各」と言っても1つしかいないが...).

(このフィールドの値が, G1BlockOffsetArray オブジェクトを作る際のコンストラクタ引数として用いられる)

#### 生成箇所(where its instances are created)
G1CollectedHeap::initialize() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classG1BlockOffsetSharedArray.html) for details

---
## <a name="no5-6FUa1f" id="no5-6FUa1f">G1BlockOffsetArray</a>

### 概要(Summary)
G1GC 用の BlockOffsetArray クラス (See: BlockOffsetArray).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1BlockOffsetTable.hpp))
    // And here is the G1BlockOffsetTable subtype that uses the array.
    
    class G1BlockOffsetArray: public G1BlockOffsetTable {
```




### 詳細(Details)
See: [here](../doxygen/classG1BlockOffsetArray.html) for details

---
## <a name="noHMw7Nezy" id="noHMw7Nezy">G1BlockOffsetArrayContigSpace</a>

### 概要(Summary)
G1GC 用の BlockOffsetArrayContigSpace クラス (See: BlockOffsetArrayContigSpace). 


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1BlockOffsetTable.hpp))
    // A subtype of BlockOffsetArray that takes advantage of the fact
    // that its underlying space is a ContiguousSpace, so that its "active"
    // region can be more efficiently tracked (than for a non-contiguous space).
    class G1BlockOffsetArrayContigSpace: public G1BlockOffsetArray {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1OffsetTableContigSpace オブジェクトの _offsets フィールドに(のみ)格納されている

#### 生成箇所(where its instances are created)
(G1OffsetTableContigSpace クラスの _offsets フィールドは, ポインタ型ではなく実体なので,
 G1OffsetTableContigSpace オブジェクトの生成時に一緒に生成される)




### 詳細(Details)
See: [here](../doxygen/classG1BlockOffsetArrayContigSpace.html) for details

---
