---
layout: default
title: CodeCache クラス (CodeCache, 及びその補助クラス(CodeBlob_sizes))
---
[Top](../index.html)

#### CodeCache クラス (CodeCache, 及びその補助クラス(CodeBlob_sizes))

これらは, 実行時に生成されたマシン語コードを管理するためのクラス (See: [here](no7882z5r.html) for details).


### クラス一覧(class list)

  * [CodeCache](#nokT1mnmIv)
  * [CodeBlob_sizes](#noXHOGjHRP)


---
## <a name="nokT1mnmIv" id="nokT1mnmIv">CodeCache</a>

### 概要(Summary)
動的生成コードを格納するメモリ領域(CodeBlobオブジェクト)を管理するためのクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

生成された CodeBlob オブジェクトは全てこのクラスが管理している.


```cpp
    ((cite: hotspot/src/share/vm/code/codeCache.hpp))
    // The CodeCache implements the code cache for various pieces of generated
    // code, e.g., compiled java methods, runtime stubs, transition frames, etc.
    // The entries in the CodeCache are all CodeBlob's.
    
    // Implementation:
    //   - Each CodeBlob occupies one chunk of memory.
    //   - Like the offset table in oldspace the zone has at table for
    //     locating a method given a addess of an instruction.
    
    ...
    class CodeCache : AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classCodeCache.html) for details

---
## <a name="noXHOGjHRP" id="noXHOGjHRP">CodeBlob_sizes</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

(デバッグ用の機能である CodeCache::print() 内で使用される補助クラス)


```cpp
    ((cite: hotspot/src/share/vm/code/codeCache.cpp))
    // Helper class for printing in CodeCache
    
    class CodeBlob_sizes {
```

### 使われ方(Usage)
CodeCache::print() 内で各 CodeBlob の統計情報が CodeBlob_sizes に集められ. 最終的に CodeBlob_sizes::print() で出力される.

(正確には, CodeCache::print() 内では CodeBlob_sizes インスタンスを2つ使用し,
 生きているものと死んでいるものとで分けて統計情報は集められる.
 このため, 統計情報は2種類が表示される.)

#### 参考(for your information): CodeCache::print()
See: [here](no7882n3g.html) for details
#### 参考(for your information): CodeBlob_sizes::print()
See: [here](no4230oiY.html) for details



### 詳細(Details)
See: [here](../doxygen/classCodeBlob__sizes.html) for details

---
