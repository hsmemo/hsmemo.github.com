---
layout: default
title: CodeHeap クラス及びその補助クラス (HeapBlock, FreeBlock, CodeHeap)
---
[Top](../index.html)

#### CodeHeap クラス及びその補助クラス (HeapBlock, FreeBlock, CodeHeap)

これらは, 実行時に動的生成されるマシン語コードを格納するためのメモリ領域を管理するクラス (See: [here](no7882z5r.html) and [here](no7882Ruk.html) for details).


### クラス一覧(class list)

  * [CodeHeap](#noIAdH9t3z)
  * [HeapBlock](#noD-nEJD6u)
  * [FreeBlock](#no0N-klR_k)


---
## <a name="noIAdH9t3z" id="noIAdH9t3z">CodeHeap</a>

### 概要(Summary)
実行時に動的生成されるマシン語コードのためのメモリ領域を管理するクラス 
(つまり, CodeCache 用のメモリ領域を管理するクラス) (See: CodeCache).


```
    ((cite: hotspot/src/share/vm/memory/heap.hpp))
    class CodeHeap : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
CodeCache クラスの _heap フィールド (static フィールド) に(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/code/codeCache.hpp))
    class CodeCache : AllStatic {
    ...
      // CodeHeap is malloc()'ed at startup and never deleted during shutdown,
      // so that the generated assembly code is always there when it's needed.
      // This may cause memory leak, but is necessary, for now. See 4423824,
      // 4422213 or 4436291 for details.
      static CodeHeap * _heap;
```

#### 生成箇所(where its instances are created)
起動時に(のみ)生成されている.


```
    ((cite: hotspot/src/share/vm/code/codeCache.cpp))
    CodeHeap * CodeCache::_heap = new CodeHeap();
```




### 詳細(Details)
See: [here](../doxygen/classCodeHeap.html) for details

---
## <a name="noD-nEJD6u" id="noD-nEJD6u">HeapBlock</a>

### 概要(Summary)
CodeHeap クラス内で使用される補助クラス.

CodeHeap が管理している各メモリ領域のメタ情報を記録しておくクラス
(現状で記録しているのは, 「その領域の領域長」および「その領域が使用中かどうか」).


```
    ((cite: hotspot/src/share/vm/memory/heap.hpp))
    class HeapBlock VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
CodeHeap が管理している各メモリ領域の先頭に埋め込まれている.
(より具体的には, CodeHeap から確保された各メモリ領域の先頭, 及び CodeHeap 内で管理されている各空き領域の先頭)

CodeHeap からメモリが確保される際には, 要求サイズより HeapBlock 分だけ大きい領域が確保され, 
先頭部分にその領域のメタ情報を表す HeapBlock が埋め込まれる.

(<= 要は malloc() が使用するメタ情報のようなもの)


```
    ((cite: hotspot/src/share/vm/memory/heap.cpp))
    void* CodeHeap::allocate(size_t size) {
      size_t length = number_of_segments(size + sizeof(HeapBlock));
    ...
      HeapBlock* block = search_freelist(length);
```

### 内部構造(Internal structure)
内部には以下の 2つのフィールドのみを持つ.
それぞれ「その領域の領域長(_length)」および「その領域が使用中かどうか(_used)」を表す.

(なお, 実際には padding も入れた分の大きさが HeapBlock オブジェクトの大きさになる)


```
    ((cite: hotspot/src/share/vm/memory/heap.hpp))
     public:
      struct Header {
        size_t  _length;                             // the length in segments
        bool    _used;                               // Used bit
      };
    
     protected:
      union {
        Header _header;
        int64_t _padding[ (sizeof(Header) + sizeof(int64_t)-1) / sizeof(int64_t) ];
                            // pad to 0 mod 8
      };
```




### 詳細(Details)
See: [here](../doxygen/classHeapBlock.html) for details

---
## <a name="no0N-klR_k" id="no0N-klR_k">FreeBlock</a>

### 概要(Summary)
CodeHeap クラス内で使用される補助クラス.

CodeHeap 内の空き領域を管理するためのフリーリスト.


```
    ((cite: hotspot/src/share/vm/memory/heap.hpp))
    class FreeBlock: public HeapBlock {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CodeHeap オブジェクトの _freelist フィールドに(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/memory/heap.hpp))
      FreeBlock*   _freelist;
```

### 内部構造(Internal structure)
HeapBlock のフィールドに加えて, リストを構成するための _link フィールドを持つ

(<= つまり, 各空き領域の先頭に HeapBlock が埋め込まれているので, 
さらにその直後の 1 word をリンクに使ってフリーリストを構成する).


```
    ((cite: hotspot/src/share/vm/memory/heap.hpp))
      FreeBlock* _link;
```




### 詳細(Details)
See: [here](../doxygen/classFreeBlock.html) for details

---
