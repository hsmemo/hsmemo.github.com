---
layout: default
title: CompactingPermGenGen クラス (CompactingPermGenGen, 及びその補助クラス(AdjustSharedObjectClosure, RecursiveAdjustSharedObjectClosure, TraversePlaceholdersClosure, VerifyMarksClearedClosure))
---
[Top](../index.html)

#### CompactingPermGenGen クラス (CompactingPermGenGen, 及びその補助クラス(AdjustSharedObjectClosure, RecursiveAdjustSharedObjectClosure, TraversePlaceholdersClosure, VerifyMarksClearedClosure))

これらは, Java ヒープ中の「Perm 世代領域 (Perm Generation)」 を管理するためのクラス.

(なお Perm Generation とは, 主にクラスデータを格納するためのメモリ領域)

Perm Generation を管理するクラスは使用する GC アルゴリズムによって異なるが,
これらのクラスは GC アルゴリズムが ParallelScavenge でも CMS でもない場合に使用される 
(See: PSPermGen, CMSPermGenGen) (See: [here](no3718kvd.html) for details).

(なおコメントによると, Perm Generation は普通の Generation に少し似ているが違う点もあるので別扱いとしている, とのこと)
(ところで, このコメントは permGen.hpp に書かれている内容と全く同じ...??)


```cpp
    ((cite: hotspot/src/share/vm/memory/compactingPermGenGen.hpp))
    // All heaps contains a "permanent generation," containing permanent
    // (reflective) objects.  This is like a regular generation in some ways,
    // but unlike one in others, and so is split apart.
```



### クラス一覧(class list)

  * [CompactingPermGenGen](#noTD5zUIY8)
  * [AdjustSharedObjectClosure](#no8cpop1Ca)
  * [RecursiveAdjustSharedObjectClosure](#no8ZB4gYSI)
  * [TraversePlaceholdersClosure](#nor0hkzcht)
  * [VerifyMarksClearedClosure](#noHWHaGsW2)


---
## <a name="noTD5zUIY8" id="noTD5zUIY8">CompactingPermGenGen</a>

### 概要(Summary)
Perm 領域を表す Generation クラスの1つ. (See: [here](no3718kvd.html) for details).

なお, このクラスは CompactingPermGen クラスとセットで使用される
(このため, 「CompactingPermGen を Generation として扱うためのラッパークラス」とも言える. (See: CompactingPermGen)).

なお, CDS に関する扱い方は少しトリッキーなので注意, とのこと.


```cpp
    ((cite: hotspot/src/share/vm/memory/compactingPermGenGen.hpp))
    // This is the "generation" view of a CompactingPermGen.
    // NOTE: the shared spaces used for CDS are here handled in
    // a somewhat awkward and potentially buggy fashion, see CR 6801625.
    // This infelicity should be fixed, see CR 6897789.
    class CompactingPermGenGen: public OneContigSpaceCardGeneration {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CompactingPermGen オブジェクトの _gen フィールド内に(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/compactPermGen.hpp))
      // The "generation" view.
      OneContigSpaceCardGeneration* _gen;
```

#### 生成箇所(where its instances are created)
CompactingPermGen::CompactingPermGen() 内で(のみ)生成されている (See: [here](no2114tfN.html) and [here](no2114gVH.html) for details).





### 詳細(Details)
See: [here](../doxygen/classCompactingPermGenGen.html) for details

---
## <a name="no8cpop1Ca" id="no8cpop1Ca">AdjustSharedObjectClosure</a>

### 概要(Summary)
CompactingPermGenGen の GC 処理で使用される補助クラス.

コンパクション処理時に Perm 領域にある live object 内のポインタを新しいアドレスに修正するための Closure.

(なお, このクラスは JVMTI の RedefineClasses() が呼ばれていた場合に(のみ)使用される)

(なお, コメントでは "Recursively" と書いてあるが, 実際の処理は recursive になっていないような...
というか, recursive な処理は RecursiveAdjustSharedObjectClosure の担当のような気もするが... #TODO)


```cpp
    ((cite: hotspot/src/share/vm/memory/compactingPermGenGen.cpp))
    // An ObjectClosure helper: Recursively adjust all pointers in an object
    // and all objects by referenced it. Clear marks on objects in order to
    // prevent visiting any object twice. This helper is used when the
    // RedefineClasses() API has been called.
    
    class AdjustSharedObjectClosure : public ObjectClosure {
```

### 使われ方(Usage)
CompactingPermGenGen::pre_adjust_pointers() の中で(のみ)使用されている

(なお, この関数は G1GC の場合には呼び出されていないため, 現状では GenCollectedHeap の場合専用のクラスである模様)
(See: [here](no2114hPa.html) for details).




### 詳細(Details)
See: [here](../doxygen/classAdjustSharedObjectClosure.html) for details

---
## <a name="no8ZB4gYSI" id="no8ZB4gYSI">RecursiveAdjustSharedObjectClosure</a>

### 概要(Summary)
CompactingPermGenGen の GC 処理で使用される補助クラス.

コンパクション処理時に Perm 領域にある live object 内のポインタを再帰的に新しいアドレスに修正するための Closure.


```cpp
    ((cite: hotspot/src/share/vm/memory/compactingPermGenGen.cpp))
    // An OopClosure helper: Recursively adjust all pointers in an object
    // and all objects by referenced it. Clear marks on objects in order
    // to prevent visiting any object twice.
    
    class RecursiveAdjustSharedObjectClosure : public OopClosure {
```

### 使われ方(Usage)
CompactingPermGenGen::pre_adjust_pointers() の中で(のみ)使用されている

(なお, この関数は G1GC の場合には呼び出されていないため, 現状では GenCollectedHeap の場合専用のクラスである模様)
(See: [here](no2114hPa.html) for details).

(CompactingPermGenGen::pre_adjust_pointers() 内で直接使用されているほか,
 その中で呼び出される TraversePlaceholdersClosure::placeholders_do() の中でも使用されている.)




### 詳細(Details)
See: [here](../doxygen/classRecursiveAdjustSharedObjectClosure.html) for details

---
## <a name="nor0hkzcht" id="nor0hkzcht">TraversePlaceholdersClosure</a>

### 概要(Summary)
CompactingPermGenGen の GC 処理で使用される補助クラス.

コンパクション処理時に SystemDictionary の PlaceholderTable 内にあるポインタを再帰的に新しいアドレスに修正するための Closure.


```cpp
    ((cite: hotspot/src/share/vm/memory/compactingPermGenGen.cpp))
    // We need to go through all placeholders in the system dictionary and
    // try to resolve them into shared classes. Other threads might be in
    // the process of loading a shared class and have strong roots on
    // their stack to the class without having added the class to the
    // dictionary yet. This means the class will be marked during phase 1
    // but will not be unmarked during the application of the
    // RecursiveAdjustSharedObjectClosure to the SystemDictionary.
    class TraversePlaceholdersClosure {
```

### 使われ方(Usage)
CompactingPermGenGen::pre_adjust_pointers() の中で(のみ)使用されている

(なお, この関数は G1GC の場合には呼び出されていないため, 現状では GenCollectedHeap の場合専用のクラスである模様)
(See: [here](no2114hPa.html) for details).




### 詳細(Details)
See: [here](../doxygen/classTraversePlaceholdersClosure.html) for details

---
## <a name="noHWHaGsW2" id="noHWHaGsW2">VerifyMarksClearedClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

CDS 使用時に, GC 終了後には shared space 内のオブジェクトに mark が付いていないことをチェックする Closure.


```cpp
    ((cite: hotspot/src/share/vm/memory/compactingPermGenGen.cpp))
    #ifdef ASSERT
    class VerifyMarksClearedClosure : public ObjectClosure {
```

### 使われ方(Usage)
CompactingPermGenGen::post_compact() 内で(のみ)使用されている

(なお, この関数は G1GC の場合には呼び出されていないため, 現状では GenCollectedHeap の場合専用のクラスである模様)
(See: [here](no2114hPa.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVerifyMarksClearedClosure.html) for details

---
