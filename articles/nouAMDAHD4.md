---
layout: default
title: IndexSet クラス関連のクラス (IndexSet, IndexSet::BitBlock, IndexSetIterator)
---
[Top](../index.html)

#### IndexSet クラス関連のクラス (IndexSet, IndexSet::BitBlock, IndexSetIterator)

これらは, C2 JIT Compiler 用のユーティリティ・クラス.

### 概要(Summary)
これらのクラスは, 「整数値の集合」を表すためのユーティリティ・クラス
(なお, 内部に格納する整数値のことを "index" と呼んでいる).

なお, 現在はレジスタ割り当て処理で使用されている.


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
    // This file defines the IndexSet class, a set of sparse integer indices.
    // This data structure is used by the compiler in its liveness analysis and
    // during register allocation.
```

集合なのでビットマップで表現できるが (そして実際に内部は bitvector だが),
ナイーブに用意するとメモリの消費量が大きくなるため, 
集合内の要素が sparse だと想定して (ページテーブルのような) 階層構造にしている.

具体的には 32bit の空間を下位 8bit とそれ以上に分け, 
ポインタ配列から 2**8 bit (= 256 bit = 32 byte) のビットマップを間接参照させている
(この 2**8 bit の配列を BitBlock というオブジェクトとして管理している).

(なお, ポインタ配列の長さは, コンストラクタに渡される「最大数」によって決まる.
「指定された最大数までの index を表現できる bit 数 / 8 bit」 のポインタ配列が確保される)


### クラス一覧(class list)

  * [IndexSet](#nojZ3stCgu)
  * [IndexSet::BitBlock](#no9Aad2kZQ)
  * [IndexSetIterator](#nokKJnp-LZ)


---
## <a name="nojZ3stCgu" id="nojZ3stCgu">IndexSet</a>

### 概要(Summary)
整数値の集合を表すためのユーティリティ・クラス.
より正確に言うと, uint 値 (ここでは "index" と呼んでいる) の集合(set)を表す.


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
    //-------------------------------- class IndexSet ----------------------------
    // An IndexSet is a piece-wise bitvector.  At the top level, we have an array
    // of pointers to bitvector chunks called BitBlocks.  Each BitBlock has a fixed
    // size and is allocated from a shared free list.  The bits which are set in
    // each BitBlock correspond to the elements of the set.
    
    class IndexSet : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 PhaseLive オブジェクトの _defs フィールド
  
  (正確には, このフィールドは IndexSet の配列を格納するフィールド)

* 各 PhaseLive オブジェクトの _free_IndexSet フィールド
  
  (正確には, このフィールドは IndexSet の線形リストを格納するフィールド(フリーリスト).
  IndexSet オブジェクトは _next フィールドで次の IndexSet オブジェクトを指せる構造になっている.
  
* 各 PhaseLive オブジェクトの _live フィールド

  (正確には, このフィールドは IndexSet の配列を格納するフィールド)

* 各 PhaseIFG オブジェクトの _adjs フィールド
  
  (正確には, このフィールドは IndexSet の配列を格納するフィールド)

* 各 PhaseConservativeCoalesce オブジェクトの _ulr フィールド

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* PhaseLive::getfreeset()

* PhaseLive::compute()
  
  (PhaseLive::_defs の確保用)
  
* PhaseLive::compute()
  
  (PhaseLive::_live の確保用)

* PhaseIFG::init()
  
  (PhaseIFG::_adjs の確保用)

* (PhaseConservativeCoalesce クラスの _ulr フィールドは, ポインタ型ではなく実体なので,
  PhaseConservativeCoalesce オブジェクトの生成時に一緒に生成される)

* PhaseChaitin::stretch_base_pointer_live_ranges() 内 (局所変数として生成)

* PhaseChaitin::build_ifg_physical() (local var) 内 (局所変数として生成)

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* ...(#TODO)

* BitBlock **_blocks
  
  内部に格納している BitBlock. 
  要素数が少なければ _preallocated_block_list のエイリアス, 
  そうでなければ Arena 上に確保された BitBlock 配列を指す.

* BitBlock  *_preallocated_block_list[preallocated_block_list_size]
  
  予め確保済みの BitBlock 配列. 
  BitBlock 数がこれ以下に収まるならこの配列が使用される (_blocks フィールド参照).


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
      // When we allocate an IndexSet, it starts off with an array of top level block
      // pointers of a set length.  This size is intended to be large enough for the
      // majority of IndexSets.  In the cases when this size is not large enough,
      // a separately allocated array is used.
    
      // The length of the preallocated top level block array
      enum { preallocated_block_list_size = 16 };
```

* ...(#TODO)


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
      // The number of elements in the set
      uint      _count;
    
      // Our top level array of bitvector segments
      BitBlock **_blocks;
    
      BitBlock  *_preallocated_block_list[preallocated_block_list_size];
    
      // The number of top level array entries in use
      uint       _max_blocks;
    
      // Our assertions need to know the maximum number allowed in the set
    #ifdef ASSERT
      uint       _max_elements;
    #endif
    
      // The next IndexSet on the free list (not used at same time as count)
      IndexSet *_next;
```





### 詳細(Details)
See: [here](../doxygen/classIndexSet.html) for details

---
## <a name="no9Aad2kZQ" id="no9Aad2kZQ">IndexSet::BitBlock</a>

### 概要(Summary)
IndexSet クラス用の補助クラス.
実際の情報を格納しておくためのビットマップ.


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
      //------------------------------ class BitBlock ----------------------------
      // The BitBlock class is a segment of a bitvector set.
    
      class BitBlock : public ResourceObj {
```


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
        // All of BitBlocks fields and methods are declared private.  We limit
        // access to IndexSet and IndexSetIterator.
```


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
      // Elements of a IndexSet get decomposed into three fields.  The highest order
      // bits are the block index, which tell which high level block holds the element.
      // Within that block, the word index indicates which word holds the element.
      // Finally, the bit index determines which single bit within that word indicates
      // membership of the element in the set.
```

以下のような操作が可能.


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
        // Operations.  A BitBlock supports four simple operations,
        // clear(), member(), insert(), and remove().  These methods do
        // not assume that the block index has been masked out.
```

なお, 未使用な BitBlock はフリーリストで管理されている.


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
      // All IndexSets share an arena from which they allocate BitBlocks.  Unused
      // BitBlocks are placed on a free list.
```


### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 Compile オブジェクトの _indexSet_free_block_list フィールド
  
  (このフィールドは BitBlock の線形リストを格納するフィールド(フリーリスト).
  BitBlock オブジェクトは _data._next フィールドで次の BitBlock オブジェクトを指せる構造になっている.
  このフィールドの線形リストに全ての未使用な BitBlock オブジェクトがつながれている.)

* IndexSet クラスの _empty_block フィールド (static フィールド)
  
  ダミーの BitBlock オブジェクト. 常に空.

* 各 IndexSet オブジェクトの _blocks フィールド
  
  その IndexSet が使用する BitBlock.
  
  (正確には, このフィールドは BitBlock のポインタの配列を格納するフィールド.
  この中に, その IndexSet 内で使用される全ての BitBlock オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
IndexSet::populate_free_list()  内で(のみ)生成されている.

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* _data._words
  
  2**8 bit (= 256 bit) の大きさを持つビットマップ. 現在は 8 個の uint32 値からなる配列として実現されている.
  (なお union 型なので _data._next と排他利用)

* _data._next
  
  線形リストを構成するためのポインタ. 次の IndexSet::BitBlock を指す.
  (なお union 型なので _data._words と排他利用)


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
        // A BitBlock is composed of some number of 32 bit words.  When a BitBlock
        // is not in use by any IndexSet, it is stored on a free list.  The next field
        // is used by IndexSet to mainting this free list.
    
        union {
          uint32 _words[words_per_block];
          BitBlock *_next;
        } _data;
```

なお, 関連する定数は以下のように定義されている.


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
      // The lengths of the index bitfields
      enum { bit_index_length = 5,
             word_index_length = 3,
             block_index_length = 8 // not used
      };
    
      // Derived constants used for manipulating the index bitfields
      enum {
             bit_index_offset = 0, // not used
             word_index_offset = bit_index_length,
             block_index_offset = bit_index_length + word_index_length,
    
             bits_per_word = 1 << bit_index_length,
             words_per_block = 1 << word_index_length,
             bits_per_block = bits_per_word * words_per_block,
    
             bit_index_mask = right_n_bits(bit_index_length),
             word_index_mask = right_n_bits(word_index_length)
      };
```




### 詳細(Details)
See: [here](../doxygen/classIndexSet_1_1BitBlock.html) for details

---
## <a name="nokKJnp-LZ" id="nokKJnp-LZ">IndexSetIterator</a>

### 概要(Summary)
IndexSet 内の整数値をたどるためのイテレータクラス(ValueObjクラス).

(なお, このクラスは現状では局所変数としてのみ生成されている)


```
    ((cite: hotspot/src/share/vm/opto/indexSet.hpp))
    //-------------------------------- class IndexSetIterator --------------------
    // An iterator for IndexSets.
    
    class IndexSetIterator VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用例(usage examples)
実際に使用する際にはこんな感じになる.


```
    ((cite: hotspot/src/share/vm/opto/chaitin.cpp))
          IndexSetIterator elements(s);
          uint lidx;
          while((lidx = elements.next()) != 0) {
    ...
          }
```

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* PhaseChaitin::set_was_low()
* PhaseChaitin::Pre_Simplify()
* PhaseChaitin::Simplify()
* PhaseChaitin::bias_color()
* PhaseChaitin::Select()
* PhaseChaitin::stretch_base_pointer_live_ranges()
* PhaseChaitin::dump()
* PhaseChaitin::interfere_with_live()
* PhaseChaitin::count_int_pressure()
* PhaseChaitin::count_float_pressure()
* PhaseChaitin::build_ifg_physical()
* PhaseChaitin::Split()
* PhaseIFG::add_vector
* PhaseIFG::SquareUp()
* PhaseIFG::Union()
* PhaseIFG::remove_node()
* PhaseIFG::re_insert()
* PhaseIFG::effective_degree()
* PhaseIFG::dump()
* PhaseIFG::dump()
* PhaseIFG::verify()
* PhaseLive::add_liveout()
* PhaseConservativeCoalesce::update_ifg()
* IndexSet::lrg_union()
* IndexSet::dump()
* IndexSet::verify()




### 詳細(Details)
See: [here](../doxygen/classIndexSetIterator.html) for details

---
