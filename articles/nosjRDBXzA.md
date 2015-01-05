---
layout: default
title: HotSpot を構成するクラスの基底クラス群, 及びそれらのメモリ管理用クラス (AllocatedObj(ALLOCATION_SUPER_CLASS_SPEC), CHeapObj, StackObj, _ValueObj, AllStatic, Chunk, Arena, ResourceObj, AllocStats, ReallocMark, 及びそれらの補助クラス(ChunkPool, ChunkPoolCleaner))
---
[Top](../index.html)

#### HotSpot を構成するクラスの基底クラス群, 及びそれらのメモリ管理用クラス (AllocatedObj(ALLOCATION_SUPER_CLASS_SPEC), CHeapObj, StackObj, _ValueObj, AllStatic, Chunk, Arena, ResourceObj, AllocStats, ReallocMark, 及びそれらの補助クラス(ChunkPool, ChunkPoolCleaner))

これらは, HotSpot 内で定義されている (ほとんど) 全てのクラスの基底クラス群 (See: [here](no28916gIW.html) and [here](no28916iKk.html) for details).


### クラス一覧(class list)

  * [CHeapObj](#nobizgwF4d)
  * [StackObj](#nohe1OHtVW)
  * [_ValueObj](#no4h-GzEua)
  * [AllStatic](#noCDH6oVpH)
  * [Chunk](#no78L9U_Ie)
  * [Arena](#noLozBoofi)
  * [ResourceObj](#noRAo4SXgE)
  * [AllocatedObj (ALLOCATION_SUPER_CLASS_SPEC)](#noAftP8xQu)
  * [AllocStats](#noGfeaEvJ_)
  * [ReallocMark](#noAqhCKZrb)
  * [ChunkPool](#noTRxcJKfZ)
  * [ChunkPoolCleaner](#noxRam0h09)


---
## <a name="nobizgwF4d" id="nobizgwF4d">CHeapObj</a>

### 概要(Summary)
C ヒープ上に確保されるオブジェクト用の基底クラス (See: [here](no28916gIW.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.hpp))
    class CHeapObj ALLOCATION_SUPER_CLASS_SPEC {
```



### 詳細(Details)
See: [here](../doxygen/classCHeapObj.html) for details

---
## <a name="nohe1OHtVW" id="nohe1OHtVW">StackObj</a>

### 概要(Summary)
スタック上に確保されるオブジェクト用の基底クラス (See: [here](no28916gIW.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.hpp))
    // Base class for objects allocated on the stack only.
    // Calling new or delete will result in fatal error.
    
    class StackObj ALLOCATION_SUPER_CLASS_SPEC {
```



### 詳細(Details)
See: [here](../doxygen/classStackObj.html) for details

---
## <a name="no4h-GzEua" id="no4h-GzEua">_ValueObj</a>

### 概要(Summary)
常に値渡しで使われるオブジェクト用の基底クラス (See: [here](no28916gIW.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.hpp))
    // Base class for objects used as value objects.
    // Calling new or delete will result in fatal error.
    ...
    //
    class _ValueObj {
```



### 詳細(Details)
See: [here](../doxygen/class__ValueObj.html) for details

---
## <a name="noCDH6oVpH" id="noCDH6oVpH">AllStatic</a>

### 概要(Summary)
静的にメモリを確保するオブジェクト用の基底クラス (See: [here](no28916gIW.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.hpp))
    // Base class for classes that constitute name spaces.
    
    class AllStatic {
```



### 詳細(Details)
See: [here](../doxygen/classAllStatic.html) for details

---
## <a name="no78L9U_Ie" id="no78L9U_Ie">Chunk</a>

### 概要(Summary)
Arena クラス内で使用される補助クラス (See: [here](no28916iKk.html) for details).

開放されたメモリ領域を管理し, 次回の確保要求に備える.


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.hpp))
    //------------------------------Chunk------------------------------------------
    // Linked list of raw memory chunks
    class Chunk: public CHeapObj {
```



### 詳細(Details)
See: [here](../doxygen/classChunk.html) for details

---
## <a name="noLozBoofi" id="noLozBoofi">Arena</a>

### 概要(Summary)
C ヒープ内でのメモリ確保を効率的に行うためのクラス (See: [here](no28916iKk.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.hpp))
    //------------------------------Arena------------------------------------------
    // Fast allocation of memory
    class Arena: public CHeapObj {
```



### 詳細(Details)
See: [here](../doxygen/classArena.html) for details

---
## <a name="noRAo4SXgE" id="noRAo4SXgE">ResourceObj</a>

### 概要(Summary)
ResourceArea に確保されるオブジェクト(= スレッドローカルで一時的なオブジェクト)用の基底クラス (See: [here](no28916gIW.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.hpp))
    //----------------------------------------------------------------------
    // Base class for objects allocated in the resource area per default.
    // Optionally, objects may be allocated on the C heap with
    // new(ResourceObj::C_HEAP) Foo(...) or in an Arena with new (&arena)
    // ResourceObj's can be allocated within other objects, but don't use
    // new or delete (allocation_type is unknown).  If new is used to allocate,
    // use delete to deallocate.
    class ResourceObj ALLOCATION_SUPER_CLASS_SPEC {
```




### 詳細(Details)
See: [here](../doxygen/classResourceObj.html) for details

---
## <a name="noAftP8xQu" id="noAftP8xQu">AllocatedObj (ALLOCATION_SUPER_CLASS_SPEC)</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

デバッグ時における (ほとんど) 全てのクラスの基底クラス.
`#ifndef PRODUCT` 時には (ほとんど) 全てのクラスがこのクラスのサブクラスになる
(より正確に言うと, `#ifndef PRODUCT` 時にはこのクラスが CHeapObj, StackObj, ResourceObj のスーパークラスになる) (See: [here](no28916gIW.html) for details).

内部にそのオブジェクトの情報を出力する print() メソッド及び print_value() メソッドを定義しているため, 
デバッグ時には (ほとんど) 全てのクラスで情報出力が行えるようになる.

なお, 実際の使用箇所では ALLOCATION_SUPER_CLASS_SPEC という名前で使用される.
デバッグ時にはこれが AllocatedObj の別名になる.
デバッグ時以外には ALLOCATION_SUPER_CLASS_SPEC は空文字列になるため, 
CHeapObj, StackObj, ResourceObj はスーパークラスを持たないクラスになる.


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.hpp))
    // Base class for objects allocated in the C-heap.
    
    // In non product mode we introduce a super class for all allocation classes
    // that supports printing.
    // We avoid the superclass in product mode since some C++ compilers add
    // a word overhead for empty super classes.
    
    #ifdef PRODUCT
    #define ALLOCATION_SUPER_CLASS_SPEC
    #else
    #define ALLOCATION_SUPER_CLASS_SPEC : public AllocatedObj
    class AllocatedObj {
```




### 詳細(Details)
See: [here](../doxygen/classAllocatedObj (ALLOCATION__SUPER__CLASS__SPEC).html) for details

---
## <a name="noGfeaEvJ_" id="noGfeaEvJ_">AllocStats</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

C ヒープ内でのメモリ確保処理に関する統計情報を表示するためのクラス.

(例えば, malloc() や free() の呼び出し回数, 確保した量(バイト数), 開放した量(バイト数), etc)


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.hpp))
    // for statistics
    #ifndef PRODUCT
    class AllocStats : StackObj {
```

### 使われ方(Usage)
コンストラクタ内で, その時点での各指標の値(os::num_mallocs, os::num_frees, etc)が記録される.

AllocStats::print() が呼ばれると, 記録していた値とその時点での値の差分を表示する.


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.cpp))
    AllocStats::AllocStats() {
      start_mallocs      = os::num_mallocs;
      start_frees        = os::num_frees;
      start_malloc_bytes = os::alloc_bytes;
      start_mfree_bytes  = os::free_bytes;
      start_res_bytes    = Arena::_bytes_allocated;
    }
    
    julong  AllocStats::num_mallocs() { return os::num_mallocs - start_mallocs; }
    julong  AllocStats::alloc_bytes() { return os::alloc_bytes - start_malloc_bytes; }
    julong  AllocStats::num_frees()   { return os::num_frees - start_frees; }
    julong  AllocStats::free_bytes()  { return os::free_bytes - start_mfree_bytes; }
    julong  AllocStats::resource_bytes() { return Arena::_bytes_allocated - start_res_bytes; }
    void    AllocStats::print() {
      tty->print_cr(UINT64_FORMAT " mallocs (" UINT64_FORMAT "MB), "
                    UINT64_FORMAT" frees (" UINT64_FORMAT "MB), " UINT64_FORMAT "MB resrc",
                    num_mallocs(), alloc_bytes()/M, num_frees(), free_bytes()/M, resource_bytes()/M);
    }
```

#### インスタンスの格納場所(where its instances are stored)
インスタンスは alloc_stats という大域変数に格納されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.cpp))
    AllocStats alloc_stats;
```

#### 使用箇所(where its instances are used)
print_statistics() 関数の中で呼び出されている.

(ただし, PrintMallocStatistics オプションが指定されている場合にのみ, 出力が行われる)


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.cpp))
    void print_statistics() {
    ...
      if (PrintMallocStatistics) {
        tty->print("allocation stats: ");
        alloc_stats.print();
        tty->cr();
      }
```



### 詳細(Details)
See: [here](../doxygen/classAllocStats.html) for details

---
## <a name="noAqhCKZrb" id="noAqhCKZrb">ReallocMark</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

ResourceArea 内に確保した配列を REALLOC_RESOURCE_ARRAY() で伸ばす際に, 
その配列が ResourceMark によって既に解放されていないかどうかをチェックする.


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.hpp))
    //------------------------------ReallocMark---------------------------------
    // Code which uses REALLOC_RESOURCE_ARRAY should check an associated
    // ReallocMark, which is declared in the same scope as the reallocated
    // pointer.  Any operation that could __potentially__ cause a reallocation
    // should check the ReallocMark.
    class ReallocMark: public StackObj {
```

### 内部構造(Internal structure)
コンストラクタが呼び出された時点と ReallocMark::check() が呼び出された時点とで
ResourceArea の nest レベル (以下の nesting) をチェックし, もしレベルが異なっていれば fatal() で異常終了させる
(なお開発時用の機能であるため, #ifdef ASSERT 時以外はコードは全て空になる).


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.cpp))
    ReallocMark::ReallocMark() {
    #ifdef ASSERT
      Thread *thread = ThreadLocalStorage::get_thread_slow();
      _nesting = thread->resource_area()->nesting();
    #endif
    }
    
    void ReallocMark::check() {
    #ifdef ASSERT
      if (_nesting != Thread::current()->resource_area()->nesting()) {
        fatal("allocation bug: array could grow within nested ResourceMark");
      }
    #endif
    }
```


なお, ResourceArea::nesting() は ResourceMark によって増減される値
(ResourceMark のコンストラクタが呼ばれるとインクリメントされ, 
 ResourceMark のデストラクタが呼ばれるとデクリメントされる.)


```cpp
    ((cite: hotspot/src/share/vm/memory/resourceArea.hpp))
      ResourceMark( ResourceArea *r ) :
    ...
        debug_only(_area->_nesting++;)
    ...
      }
    ...
    
      ~ResourceMark() {
    ...
        debug_only(_area->_nesting--;)
    ...
      }
```



### 詳細(Details)
See: [here](../doxygen/classReallocMark.html) for details

---
## <a name="noTRxcJKfZ" id="noTRxcJKfZ">ChunkPool</a>

### 概要(Summary)
Chunk クラス内で使用される補助クラス (See: [here](no28916iKk.html) for details).

未使用の Chunk を管理するためのフリーリスト.

(なお, thread の初期化が終わる前から使用されるため, 内部で mutex は使わないように, とのこと.)


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.cpp))
    //--------------------------------------------------------------------------------------
    // ChunkPool implementation
    
    // MT-safe pool of chunks to reduce malloc/free thrashing
    // NB: not using Mutex because pools are used before Threads are initialized
    class ChunkPool {
```



### 詳細(Details)
See: [here](../doxygen/classChunkPool.html) for details

---
## <a name="noxRam0h09" id="noxRam0h09">ChunkPoolCleaner</a>

### 概要(Summary)
Chunk クラス用の補助クラス (See: [here](no28916iKk.html) for details).

ChunkPool 内の Chunk 数が多くなりすぎないように, 
定期間隔で ChunkPool 内の Chunk を解放するためのクラス(PeriodicTaskクラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.cpp))
    //--------------------------------------------------------------------------------------
    // ChunkPoolCleaner implementation
    //
    
    class ChunkPoolCleaner : public PeriodicTask {
```

### 内部構造(Internal structure)
定期間隔で ChunkPool::clean() を呼び出しているだけ.


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.cpp))
       void task() {
         ChunkPool::clean();
       }
```


なお, 現在の処理間隔は 5 秒.


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.cpp))
      enum { CleaningInterval = 5000 };      // cleaning interval in ms
```

呼び出される ChunkPool::clean() の中では, 各 ChunkPool の要素を最大 5 個を残して解放する.


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.cpp))
      static void clean() {
        enum { BlocksToKeep = 5 };
         _small_pool->free_all_but(BlocksToKeep);
         _medium_pool->free_all_but(BlocksToKeep);
         _large_pool->free_all_but(BlocksToKeep);
      }
```

ChunkPool::free_all_but() の処理は以下の通り
(ChunkPool 内に指定された個数以上に Chunk が入っていれば, 指定個数まで解放する).


```cpp
    ((cite: hotspot/src/share/vm/memory/allocation.cpp))
      // Prune the pool
      void free_all_but(size_t n) {
        // if we have more than n chunks, free all of them
        ThreadCritical tc;
        if (_num_chunks > n) {
          // free chunks at end of queue, for better locality
          Chunk* cur = _first;
          for (size_t i = 0; i < (n - 1) && cur != NULL; i++) cur = cur->next();
    
          if (cur != NULL) {
            Chunk* next = cur->next();
            cur->set_next(NULL);
            cur = next;
    
            // Free all remaining chunks
            while(cur != NULL) {
              next = cur->next();
              os::free(cur);
              _num_chunks--;
              cur = next;
            }
          }
        }
      }
```




### 詳細(Details)
See: [here](../doxygen/classChunkPoolCleaner.html) for details

---
