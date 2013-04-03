---
layout: default
title: メモリアクセスに関する高レベル中間語(Ideal)クラス (MemNode, LoadNode, LoadBNode, LoadUBNode, LoadUSNode, LoadINode, LoadUI2LNode, LoadRangeNode, LoadLNode, LoadL_unalignedNode, LoadFNode, LoadDNode, LoadD_unalignedNode, LoadPNode, LoadNNode, LoadKlassNode, LoadNKlassNode, LoadSNode, StoreNode, StoreBNode, StoreCNode, StoreINode, StoreLNode, StoreFNode, StoreDNode, StorePNode, StoreNNode, StoreCMNode, LoadPLockedNode, LoadLLockedNode, SCMemProjNode, LoadStoreNode, StorePConditionalNode, StoreIConditionalNode, StoreLConditionalNode, CompareAndSwapLNode, CompareAndSwapINode, CompareAndSwapPNode, CompareAndSwapNNode, ClearArrayNode, StrIntrinsicNode, StrCompNode, StrEqualsNode, StrIndexOfNode, AryEqNode, MemBarNode, MemBarAcquireNode, MemBarReleaseNode, MemBarVolatileNode, MemBarCPUOrderNode, InitializeNode, MergeMemNode, MergeMemStream, PrefetchReadNode, PrefetchWriteNode)
---
[Top](../index.html)

#### メモリアクセスに関する高レベル中間語(Ideal)クラス (MemNode, LoadNode, LoadBNode, LoadUBNode, LoadUSNode, LoadINode, LoadUI2LNode, LoadRangeNode, LoadLNode, LoadL_unalignedNode, LoadFNode, LoadDNode, LoadD_unalignedNode, LoadPNode, LoadNNode, LoadKlassNode, LoadNKlassNode, LoadSNode, StoreNode, StoreBNode, StoreCNode, StoreINode, StoreLNode, StoreFNode, StoreDNode, StorePNode, StoreNNode, StoreCMNode, LoadPLockedNode, LoadLLockedNode, SCMemProjNode, LoadStoreNode, StorePConditionalNode, StoreIConditionalNode, StoreLConditionalNode, CompareAndSwapLNode, CompareAndSwapINode, CompareAndSwapPNode, CompareAndSwapNNode, ClearArrayNode, StrIntrinsicNode, StrCompNode, StrEqualsNode, StrIndexOfNode, AryEqNode, MemBarNode, MemBarAcquireNode, MemBarReleaseNode, MemBarVolatileNode, MemBarCPUOrderNode, InitializeNode, MergeMemNode, MergeMemStream, PrefetchReadNode, PrefetchWriteNode)

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, 主に「メモリアクセス」を表すクラス
(e.g. load, store, load-linked, store-conditional, CAS, memory barrier, prefetch, etc).

バイト幅や signed/unsigned による違いだけでなく, 配列長や narrow pointer 用,
あるいはオブジェクトからクラスを取得する load から card mark に書き込む store など, 様々な load/store が用意されている.


### クラス一覧(class list)

  * [MemNode](#noqbXxOzlJ)
  * [LoadNode](#norAhXSVDe)
  * [LoadBNode](#noS2V1uxPR)
  * [LoadUBNode](#noJEN2iOly)
  * [LoadUSNode](#noVifROo82)
  * [LoadINode](#nouDeBBAYa)
  * [LoadUI2LNode](#noeWHAX1XA)
  * [LoadRangeNode](#nowZq7KDbY)
  * [LoadLNode](#noVwqBcvNZ)
  * [LoadL_unalignedNode](#no0losACK-)
  * [LoadFNode](#noH-M5eTxp)
  * [LoadDNode](#nollLA87E9)
  * [LoadD_unalignedNode](#noIALuMz2o)
  * [LoadPNode](#noJE3y0sY2)
  * [LoadNNode](#noMNH7bc3C)
  * [LoadKlassNode](#noMhy9CFbN)
  * [LoadNKlassNode](#noca4zOiTK)
  * [LoadSNode](#nouT9H7J9G)
  * [StoreNode](#no3VzfLwo5)
  * [StoreBNode](#noB3wmNDuF)
  * [StoreCNode](#noW_195Il0)
  * [StoreINode](#no-dyic72q)
  * [StoreLNode](#no6i4647kP)
  * [StoreFNode](#noIS5oTsRi)
  * [StoreDNode](#noepIj82zq)
  * [StorePNode](#nodsssOz1J)
  * [StoreNNode](#no8jxWTdwM)
  * [StoreCMNode](#nom1m53xIy)
  * [LoadPLockedNode](#nokiWT5M11)
  * [LoadLLockedNode](#no8QqCCwI7)
  * [SCMemProjNode](#noogmbgL_c)
  * [LoadStoreNode](#noAXFWs9FA)
  * [StorePConditionalNode](#nokJaBZCzx)
  * [StoreIConditionalNode](#noKgJKc1yI)
  * [StoreLConditionalNode](#no91fwpIsr)
  * [CompareAndSwapLNode](#no_nQYgJur)
  * [CompareAndSwapINode](#no3JIMHiEp)
  * [CompareAndSwapPNode](#no3JuaeXrU)
  * [CompareAndSwapNNode](#noi8ElPhB_)
  * [ClearArrayNode](#noAp4exCLg)
  * [StrIntrinsicNode](#no1Wkr-g1f)
  * [StrCompNode](#noEXWFdzni)
  * [StrEqualsNode](#no2_oiKJuW)
  * [StrIndexOfNode](#noQzQ7RSq_)
  * [AryEqNode](#nodi4HhtC3)
  * [MemBarNode](#nouboi-Dcd)
  * [MemBarAcquireNode](#noFzg6ME0H)
  * [MemBarReleaseNode](#noRT_j1n1e)
  * [MemBarVolatileNode](#noZLAGfTnb)
  * [MemBarCPUOrderNode](#noFvAGuZfj)
  * [InitializeNode](#noBLOaTm61)
  * [MergeMemNode](#noSWTVvDM0)
  * [MergeMemStream](#nopdJFv9S4)
  * [PrefetchReadNode](#noaZsM3UgC)
  * [PrefetchWriteNode](#noOA30pwoq)


---
## <a name="noqbXxOzlJ" id="noqbXxOzlJ">MemNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.
全てのメモリアクセス用の Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------MemNode----------------------------------------
    // Load or Store, possibly throwing a NULL pointer exception
    class MemNode : public Node {
```

### 内部構造(Internal structure)
(control input も含めて) 最大で5つの入力ノードを持つ.
それぞれの入力の意味は以下の通り (この enum の順で入力される).

* 1番目の入力Node : control input (他の Node の第1Nodeと同じ. Node への control flow 上の依存を示す)
* 2番目の入力Node : メモリアクセス間の依存を示すためのノード (後述)
* 3番目の入力Node : ロード/ストアの対象となるメモリアドレス
* 4番目の入力Node : (ストアの場合のみ使用される) ストアする値
* 5番目の入力Node : (StoreCM の場合のみ使用される) 対応する oop のストア Node

* 1番目の入力Node : Control (Node への control flow 上の依存を示す. 他の Node の第1Nodeと同じ.)
* 2番目の入力Node : Memory (メモリアクセス間での依存関係を示す #TODO)
* 3番目の入力Node : ロード/ストアの対象となるメモリアドレス
* 4番目の入力Node : (ストアの場合のみ使用される) ストアする値
* 5番目の入力Node : (StoreCM の場合のみ使用される) 対応する oop のストア Node


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      enum { Control,               // When is it safe to do this load?
             Memory,                // Chunk of memory is being loaded from
             Address,               // Actually address, derived from base
             ValueIn,               // Value to store
             OopStore               // Preceeding oop store, only in StoreCM
      };
```

Load ではこのうち3つが使われる


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadNode( Node *c, Node *mem, Node *adr, const TypePtr* at, const Type *rt )
        : MemNode(c,mem,adr,at), _type(rt) {
```

Store ではこのうち4つ, または5つ全てが使われる


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val )
        : MemNode(c,mem,adr,at,val) {
```


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val, Node *oop_store )
        : MemNode(c,mem,adr,at,val,oop_store) {
```

Load*Node と Store*Node の両方に対応するため, 引数の数が異なるコンストラクタが定義されている.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      MemNode( Node *c0, Node *c1, Node *c2, const TypePtr* at )
        : Node(c0,c1,c2   ) {
        init_class_id(Class_Mem);
        debug_only(_adr_type=at; adr_type();)
      }
      MemNode( Node *c0, Node *c1, Node *c2, const TypePtr* at, Node *c3 )
        : Node(c0,c1,c2,c3) {
        init_class_id(Class_Mem);
        debug_only(_adr_type=at; adr_type();)
      }
      MemNode( Node *c0, Node *c1, Node *c2, const TypePtr* at, Node *c3, Node *c4)
        : Node(c0,c1,c2,c3,c4) {
        init_class_id(Class_Mem);
        debug_only(_adr_type=at; adr_type();)
      }
```

(なお, メモリアクセス系の Node では, control input が 0 ではないケースもある.
 store については, 制御依存を無視して勝手にメモリを書き換えられるとまずい.
 load についても, 配列の境界チェック等を無視して先にロードするとメモリアクセスエラーを起こす危険がある)

(また, コンストラクタに渡される TypePtr 型の引数は, 
`#ifdef ASSERT` 時にのみ使用される debug 用の情報)


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    #ifdef ASSERT
      const TypePtr* _adr_type;     // What kind of memory is being addressed?
    #endif
```

#### 捕捉: 
メモリアクセスを示すノードに特徴的な入力ノードとして Memory というノードがある.
これはメモリアクセス間の依存を示すためのもの.
より具体的に言うと, 処理対象のアドレスがエイリアス解析により同値分類されたメモリ空間(memory slices)のどこに属するか, を示す.

(コンパイルのエントリポイントで, その時点における memory slices (memory state) が渡され,
その memory slices の中から各ロード／ストアは自分が対応するものに接続する)

同じ memory slice に属するメモリアクセス間では, メモリアクセスが順序づけられる.
異なる slice 間では順序づけは行われない (コンパイラが勝手にスケジューリングする).

(ただし, call 等の memory barrier となる場所では, memory slice 間でも依存が生じる)

(なお, メモリのオーダリングとごっちゃにしないように.
 ここで言う順序づけはシングルスレッドで(つまり自分自身から見て)破綻しないようにするという話.
 オーダリングは MP 環境のマルチスレッドで他スレッドから見て破綻しないようにするという話)

また, コンパイラから見て影響が読めない(alias analysis できないということ?)ような Node (call 等) が表れた場合や
明示的に順序づけが行われた場合(volatile memory operations 等)には, slices は束ねられる.
これらの Node は新しい memory state を作成し, 以後のロード／ストアはそちらに接続される.

#### 使用例:
* ロード系の Node の典型的な生成処理は, 以下のようである模様.

  memory state 中のその alias index に対応する Node (そのロードに対応するストア) が取得されて,
  それが生成する Node の入力ノードとして使用される.
  (そして, 生成した Node 自体は operand stack 内に納められたりする模様)

        Node* mem = memory(adr_idx);
        Node* ld;
        if (require_atomic_access && bt == T_LONG) {
          ld = LoadLNode::make_atomic(C, ctl, mem, adr, adr_type, t);
        } else {
          ld = LoadNode::make(_gvn, ctl, mem, adr, adr_type, t, bt);
        }
        return _gvn.transform(ld);


* ストア系の Node の典型的な生成処理は, 以下のようである模様.

  memory state 中のその alias index に対応する Node (そのストアにより上書きされるノード) が取得されて,
  それが生成する Node の入力ノードとして使用される.
  そして, 生成した Node 自体は memory state の対応する alias index の箇所に set_memory() される,

        Node *mem = memory(adr_idx);
        Node* st;
        if (require_atomic_access && bt == T_LONG) {
          st = StoreLNode::make_atomic(C, ctl, mem, adr, adr_type, val);
        } else {
          st = StoreNode::make(_gvn, ctl, mem, adr, adr_type, val, bt);
        }
        st = transform(st);
        set_memory(st, adr_idx);

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classMemNode.html) for details

---
## <a name="norAhXSVDe" id="norAhXSVDe">LoadNode</a>

### 概要(Summary)
MemNode クラスのサブクラスの1つ.
全てのロード(メモリ読み込み)用の Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadNode---------------------------------------
    // Load value; requires Memory and Address
    class LoadNode : public MemNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadNode( Node *c, Node *mem, Node *adr, const TypePtr* at, const Type *rt )
        : MemNode(c,mem,adr,at), _type(rt) {
        init_class_id(Class_Load);
      }
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classLoadNode.html) for details

---
## <a name="noS2V1uxPR" id="noS2V1uxPR">LoadBNode</a>

### 概要(Summary)
LoadNode クラスの具象サブクラスの1つ.
このクラスは byte 値のロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadBNode--------------------------------------
    // Load a byte (8bits signed) from memory
    class LoadBNode : public LoadNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadBNode( Node *c, Node *mem, Node *adr, const TypePtr* at, const TypeInt *ti = TypeInt::BYTE )
        : LoadNode(c,mem,adr,at,ti) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadBNode.html) for details

---
## <a name="noJEN2iOly" id="noJEN2iOly">LoadUBNode</a>

### 概要(Summary)
LoadNode クラスの具象サブクラスの1つ.
このクラスは unsigned byte 値のロード用.

(なお, このクラスは boolean 用)


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadUBNode-------------------------------------
    // Load a unsigned byte (8bits unsigned) from memory
    class LoadUBNode : public LoadNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadUBNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeInt* ti = TypeInt::UBYTE )
        : LoadNode(c, mem, adr, at, ti) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadUBNode.html) for details

---
## <a name="noVifROo82" id="noVifROo82">LoadUSNode</a>

### 概要(Summary)
LoadNode クラスの具象サブクラスの1つ.
このクラスは unsigned short (= char) 値のロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadUSNode-------------------------------------
    // Load an unsigned short/char (16bits unsigned) from memory
    class LoadUSNode : public LoadNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadUSNode( Node *c, Node *mem, Node *adr, const TypePtr* at, const TypeInt *ti = TypeInt::CHAR )
        : LoadNode(c,mem,adr,at,ti) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadUSNode.html) for details

---
## <a name="nouDeBBAYa" id="nouDeBBAYa">LoadINode</a>

### 概要(Summary)
LoadNode クラスの具象サブクラスの1つ.
このクラスは int 値のロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadINode--------------------------------------
    // Load an integer from memory
    class LoadINode : public LoadNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadINode( Node *c, Node *mem, Node *adr, const TypePtr* at, const TypeInt *ti = TypeInt::INT )
        : LoadNode(c,mem,adr,at,ti) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadINode.html) for details

---
## <a name="noeWHAX1XA" id="noeWHAX1XA">LoadUI2LNode</a>

### 概要(Summary)
AndLNode の最適化のための Node クラス.

LoadNode クラスの具象サブクラスの1つ.
このクラスは unsigned int (= unsigned 32bit) 値のロード用.

"LoadI + ConvI2L + AndL (0x00000000FFFFFFFF との and)" というパターンの式に対する最適化で使用される.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadUI2LNode-----------------------------------
    // Load an unsigned integer into long from memory
    class LoadUI2LNode : public LoadNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
AndLNode::Ideal() 内で(のみ)生成されている.


```
    ((cite: hotspot/src/share/vm/opto/mulnode.cpp))
    Node *AndLNode::Ideal(PhaseGVN *phase, bool can_reshape) {
    ...
      uint op = in1->Opcode();
    
      // Masking sign bits off of an integer?  Do an unsigned integer to
      // long load.
      // NOTE: This check must be *before* we try to convert the AndLNode
      // to an AndINode and commute it with ConvI2LNode because
      // 0xFFFFFFFFL masks the whole integer and we get a sign extension,
      // which is wrong.
      if (op == Op_ConvI2L && in1->in(1)->Opcode() == Op_LoadI && mask == CONST64(0x00000000FFFFFFFF)) {
        Node* load = in1->in(1);
        return new (phase->C, 3) LoadUI2LNode(load->in(MemNode::Control),
                                              load->in(MemNode::Memory),
                                              load->in(MemNode::Address),
                                              load->adr_type());
      }
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadUI2LNode(Node* c, Node* mem, Node* adr, const TypePtr* at, const TypeLong* t = TypeLong::UINT)
        : LoadNode(c, mem, adr, at, t) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadUI2LNode.html) for details

---
## <a name="nowZq7KDbY" id="nowZq7KDbY">LoadRangeNode</a>

### 概要(Summary)
LoadINode クラスのサブクラスの1つ.
このクラスは配列オブジェクト内の length フィールドのロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadRangeNode----------------------------------
    // Load an array length from the array
    class LoadRangeNode : public LoadINode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
GraphKit::load_array_length() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadRangeNode( Node *c, Node *mem, Node *adr, const TypeInt *ti = TypeInt::POS )
        : LoadINode(c,mem,adr,TypeAryPtr::RANGE,ti) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadRangeNode.html) for details

---
## <a name="noVwqBcvNZ" id="noVwqBcvNZ">LoadLNode</a>

### 概要(Summary)
LoadNode クラスの具象サブクラスの1つ.
このクラスは long 値のロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadLNode--------------------------------------
    // Load a long from memory
    class LoadLNode : public LoadNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadLNode( Node *c, Node *mem, Node *adr, const TypePtr* at,
                 const TypeLong *tl = TypeLong::LONG,
                 bool require_atomic_access = false )
        : LoadNode(c,mem,adr,at,tl)
        , _require_atomic_access(require_atomic_access)
      {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadLNode.html) for details

---
## <a name="no0losACK-" id="no0losACK-">LoadL_unalignedNode</a>

### 概要(Summary)
OSR 用の JIT コンパイル処理で使用される Node クラス.

LoadLNode クラスのサブクラスの1つ.
このクラスはアラインしていないアドレスからの long 値のロード用.

(OSR 用のコードでは, 
 まず Interpreter フレームから現在の実行状態(局所変数など)を読み取ってくる必要があり, 
 その際に使用される模様 (#TODO))


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadL_unalignedNode----------------------------
    // Load a long from unaligned memory
    class LoadL_unalignedNode : public LoadLNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
Parse::fetch_interpreter_state() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Parse::Parse()
-> Parse::load_interpreter_state()
   -> Parse::fetch_interpreter_state()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadL_unalignedNode( Node *c, Node *mem, Node *adr, const TypePtr* at )
        : LoadLNode(c,mem,adr,at) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadL__unalignedNode.html) for details

---
## <a name="noH-M5eTxp" id="noH-M5eTxp">LoadFNode</a>

### 概要(Summary)
LoadNode クラスの具象サブクラスの1つ.
このクラスは float 値のロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadFNode--------------------------------------
    // Load a float (64 bits) from memory
    class LoadFNode : public LoadNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadFNode( Node *c, Node *mem, Node *adr, const TypePtr* at, const Type *t = Type::FLOAT )
        : LoadNode(c,mem,adr,at,t) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadFNode.html) for details

---
## <a name="nollLA87E9" id="nollLA87E9">LoadDNode</a>

### 概要(Summary)
LoadNode クラスの具象サブクラスの1つ.
このクラスは double 値のロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadDNode--------------------------------------
    // Load a double (64 bits) from memory
    class LoadDNode : public LoadNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadDNode( Node *c, Node *mem, Node *adr, const TypePtr* at, const Type *t = Type::DOUBLE )
        : LoadNode(c,mem,adr,at,t) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadDNode.html) for details

---
## <a name="noIALuMz2o" id="noIALuMz2o">LoadD_unalignedNode</a>

### 概要(Summary)
OSR 用の JIT コンパイル処理で使用される Node クラス.

LoadLNode クラスのサブクラスの1つ.
このクラスはアラインしていないアドレスからの long 値のロード用.

(OSR 用のコードでは, 
 まず Interpreter フレームから現在の実行状態(局所変数など)を読み取ってくる必要があり, 
 その際に使用される模様 (#TODO))


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadD_unalignedNode----------------------------
    // Load a double from unaligned memory
    class LoadD_unalignedNode : public LoadDNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
Parse::fetch_interpreter_state() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Parse::Parse()
-> Parse::load_interpreter_state()
   -> Parse::fetch_interpreter_state()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadD_unalignedNode( Node *c, Node *mem, Node *adr, const TypePtr* at )
        : LoadDNode(c,mem,adr,at) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadD__unalignedNode.html) for details

---
## <a name="noJE3y0sY2" id="noJE3y0sY2">LoadPNode</a>

### 概要(Summary)
LoadNode クラスの具象サブクラスの1つ.
このクラスはポインタ値のロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadPNode--------------------------------------
    // Load a pointer from memory (either object or array)
    class LoadPNode : public LoadNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadPNode( Node *c, Node *mem, Node *adr, const TypePtr *at, const TypePtr* t )
        : LoadNode(c,mem,adr,at,t) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadPNode.html) for details

---
## <a name="noMNH7bc3C" id="noMNH7bc3C">LoadNNode</a>

### 概要(Summary)
LoadNode クラスの具象サブクラスの1つ.
このクラスは narrow oop 値のロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadNNode--------------------------------------
    // Load a narrow oop from memory (either object or array)
    class LoadNNode : public LoadNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadNNode( Node *c, Node *mem, Node *adr, const TypePtr *at, const Type* t )
        : LoadNode(c,mem,adr,at,t) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadNNode.html) for details

---
## <a name="noMhy9CFbN" id="noMhy9CFbN">LoadKlassNode</a>

### 概要(Summary)
LoadPNode クラスのサブクラス.
このクラスは klassOop 型のポインタ値のロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadKlassNode----------------------------------
    // Load a Klass from an object
    class LoadKlassNode : public LoadPNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LoadKlassNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadKlassNode( Node *c, Node *mem, Node *adr, const TypePtr *at, const TypeKlassPtr *tk )
        : LoadPNode(c,mem,adr,at,tk) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadKlassNode.html) for details

---
## <a name="noca4zOiTK" id="noca4zOiTK">LoadNKlassNode</a>

### 概要(Summary)
LoadNNode クラスのサブクラス.
このクラスは klassOop 型の narrow oop 値のロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadNKlassNode---------------------------------
    // Load a narrow Klass from an object.
    class LoadNKlassNode : public LoadNNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LoadKlassNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadNKlassNode( Node *c, Node *mem, Node *adr, const TypePtr *at, const TypeNarrowOop *tk )
        : LoadNNode(c,mem,adr,at,tk) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadNKlassNode.html) for details

---
## <a name="nouT9H7J9G" id="nouT9H7J9G">LoadSNode</a>

### 概要(Summary)
LoadNode クラスの具象サブクラスの1つ.
このクラスは short 値のロード用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadSNode--------------------------------------
    // Load a short (16bits signed) from memory
    class LoadSNode : public LoadNode {
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadSNode( Node *c, Node *mem, Node *adr, const TypePtr* at, const TypeInt *ti = TypeInt::SHORT )
        : LoadNode(c,mem,adr,at,ti) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadSNode.html) for details

---
## <a name="no3VzfLwo5" id="no3VzfLwo5">StoreNode</a>

### 概要(Summary)
MemNode クラスのサブクラスの1つ.
全てのストア(メモリ書き込み)用の Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StoreNode--------------------------------------
    // Store value; requires Store, Address and Value
    class StoreNode : public MemNode {
```

### 内部構造(Internal structure)
(control input も含めて) 4つまたは5つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つまたは 5つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val )
        : MemNode(c,mem,adr,at,val) {
        init_class_id(Class_Store);
      }
      StoreNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val, Node *oop_store )
        : MemNode(c,mem,adr,at,val,oop_store) {
        init_class_id(Class_Store);
      }
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classStoreNode.html) for details

---
## <a name="noB3wmNDuF" id="noB3wmNDuF">StoreBNode</a>

### 概要(Summary)
StoreNode クラスの具象サブクラスの1つ.
このクラスは byte 値のストア用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StoreBNode-------------------------------------
    // Store byte to memory
    class StoreBNode : public StoreNode {
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreBNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val ) : StoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStoreBNode.html) for details

---
## <a name="noW_195Il0" id="noW_195Il0">StoreCNode</a>

### 概要(Summary)
StoreNode クラスの具象サブクラスの1つ.
このクラスは char/short 値のストア用.

(ストアでは signed/unsigned の区別はないので char/short 両方に使用)


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StoreCNode-------------------------------------
    // Store char/short to memory
    class StoreCNode : public StoreNode {
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreCNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val ) : StoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStoreCNode.html) for details

---
## <a name="no-dyic72q" id="no-dyic72q">StoreINode</a>

### 概要(Summary)
StoreNode クラスの具象サブクラスの1つ.
このクラスは int 値のストア用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StoreINode-------------------------------------
    // Store int to memory
    class StoreINode : public StoreNode {
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreINode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val ) : StoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStoreINode.html) for details

---
## <a name="no6i4647kP" id="no6i4647kP">StoreLNode</a>

### 概要(Summary)
StoreNode クラスの具象サブクラスの1つ.
このクラスは long 値のストア用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StoreLNode-------------------------------------
    // Store long to memory
    class StoreLNode : public StoreNode {
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreLNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val,
                  bool require_atomic_access = false )
        : StoreNode(c,mem,adr,at,val)
        , _require_atomic_access(require_atomic_access)
      {}
```




### 詳細(Details)
See: [here](../doxygen/classStoreLNode.html) for details

---
## <a name="noIS5oTsRi" id="noIS5oTsRi">StoreFNode</a>

### 概要(Summary)
StoreNode クラスの具象サブクラスの1つ.
このクラスは float 値のストア用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StoreFNode-------------------------------------
    // Store float to memory
    class StoreFNode : public StoreNode {
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreFNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val ) : StoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStoreFNode.html) for details

---
## <a name="noepIj82zq" id="noepIj82zq">StoreDNode</a>

### 概要(Summary)
StoreNode クラスの具象サブクラスの1つ.
このクラスは double 値のストア用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StoreDNode-------------------------------------
    // Store double to memory
    class StoreDNode : public StoreNode {
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreDNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val ) : StoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStoreDNode.html) for details

---
## <a name="nodsssOz1J" id="nodsssOz1J">StorePNode</a>

### 概要(Summary)
StoreNode クラスの具象サブクラスの1つ.
このクラスはポインタ値のストア用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StorePNode-------------------------------------
    // Store pointer to memory
    class StorePNode : public StoreNode {
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StorePNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val ) : StoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStorePNode.html) for details

---
## <a name="no8jxWTdwM" id="no8jxWTdwM">StoreNNode</a>

### 概要(Summary)
StoreNode クラスの具象サブクラスの1つ.
このクラスは narrow oop 値のストア用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StoreNNode-------------------------------------
    // Store narrow oop to memory
    class StoreNNode : public StoreNode {
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 4つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreNNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val ) : StoreNode(c,mem,adr,at,val) {}
```




### 詳細(Details)
See: [here](../doxygen/classStoreNNode.html) for details

---
## <a name="nom1m53xIy" id="nom1m53xIy">StoreCMNode</a>

### 概要(Summary)
Garbage Collection 処理用の補助クラス.
Concurrent な GC アルゴリズム(CMS, G1GC)を使っている場合の card table の dirty 化処理(= 0 の書き込み)を表す StoreNode クラス.

(なお, G1GC や CMS でない場合には card table の dirty 化も普通の StoreBNode ノードで実装される.
 (See: GraphKit::write_barrier_post()))

なおコメントによると, 
「SafePoint 前の最後の StoreCM は維持されなければならない. また, 対応する "oop" store の後ろにくる必要がある.
  一方, 連続した同じ StoreCM は消していい.」
とのこと.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StoreCMNode-----------------------------------
    // Store card-mark byte to memory for CM
    // The last StoreCM before a SafePoint must be preserved and occur after its "oop" store
    // Preceeding equivalent StoreCMs may be eliminated.
    class StoreCMNode : public StoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
IdealKit::storeCM() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
GraphKit::post_barrier()
-> GraphKit::write_barrier_post()
   -> IdealKit::storeCM()
-> GraphKit::g1_write_barrier_post()
   -> GraphKit::g1_mark_card()
      -> IdealKit::storeCM()
```

### 内部構造(Internal structure)
(control input も含めて) 5つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 5つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreCMNode( Node *c, Node *mem, Node *adr, const TypePtr* at, Node *val, Node *oop_store, int oop_alias_idx ) :
        StoreNode(c,mem,adr,at,val,oop_store),
        _oop_alias_idx(oop_alias_idx) {
        assert(_oop_alias_idx >= Compile::AliasIdxRaw ||
               _oop_alias_idx == Compile::AliasIdxBot && Compile::current()->AliasLevel() == 0,
               "bad oop alias idx");
      }
```




### 詳細(Details)
See: [here](../doxygen/classStoreCMNode.html) for details

---
## <a name="nokiWT5M11" id="nokiWT5M11">LoadPLockedNode</a>

### 概要(Summary)
LoadPNode クラスのサブクラス.
このクラスは Load-locked 命令 (Load-linked 命令) によるポインタのロード用.

なおコメントによると, 
「Sparc や x86 上では (そんな命令ないので) 普通の pointer load でいい. 
 Power やそのお仲間 (ARM や MIPS や Alpha や...) では本当に load-linked にする必要がある」
とのこと.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadPLockedNode---------------------------------
    // Load-locked a pointer from memory (either object or array).
    // On Sparc & Intel this is implemented as a normal pointer load.
    // On PowerPC and friends it's a real load-locked.
    class LoadPLockedNode : public LoadPNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
PhaseMacroExpand::expand_allocate_common() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadPLockedNode( Node *c, Node *mem, Node *adr )
        : LoadPNode(c,mem,adr,TypeRawPtr::BOTTOM, TypeRawPtr::BOTTOM) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadPLockedNode.html) for details

---
## <a name="no8QqCCwI7" id="no8QqCCwI7">LoadLLockedNode</a>

### 概要(Summary)
LoadLNode クラスのサブクラス.
このクラスは Load-locked 命令 (Load-linked 命令) による long 値のロード用.

なおコメントによると, 
「Sparc や x86 上では (そんな命令ないので) 普通の long load でいい.」
とのこと.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadLLockedNode---------------------------------
    // Load-locked a pointer from memory (either object or array).
    // On Sparc & Intel this is implemented as a normal long load.
    class LoadLLockedNode : public LoadLNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_native_AtomicLong_get() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ.
これはスーパークラスである MemNode の入力ノードの先頭 3つに対応する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadLLockedNode( Node *c, Node *mem, Node *adr )
        : LoadLNode(c,mem,adr,TypeRawPtr::BOTTOM, TypeLong::LONG) {}
```




### 詳細(Details)
See: [here](../doxygen/classLoadLLockedNode.html) for details

---
## <a name="noogmbgL_c" id="noogmbgL_c">SCMemProjNode</a>

### 概要(Summary)
LoadStoreNode のサブクラス (Store*ConditionalNode または CompareAndSwap*Node) と組み合わせて使用する ProjNode クラス.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------SCMemProjNode---------------------------------------
    // This class defines a projection of the memory  state of a store conditional node.
    // These nodes return a value, but also update memory.
    class SCMemProjNode : public ProjNode {
```

Store*ConditionalNode は成否を示す condition code 値と変更後の memory state を出力するノードであり,
それぞれ BoolNode と SCMemProjNode によって取り出す.

また, CompareAndSwap*Node の場合は, 
結果が使用されない場合に最適化で消去されるのを防ぐため SCMemProjNode ノードを合わせて使うことが多い模様.

```
    ((cite: hotspot/src/share/vm/opto/library_call.cpp))
    bool LibraryCallKit::inline_unsafe_CAS(BasicType type) {
    ...
      case T_INT:
        cas = _gvn.transform(new (C, 5) CompareAndSwapINode(control(), mem, adr, newval, oldval));
    ...
      case T_LONG:
        cas = _gvn.transform(new (C, 5) CompareAndSwapLNode(control(), mem, adr, newval, oldval));
    ...
      case T_OBJECT:
    ...
          cas = _gvn.transform(new (C, 5) CompareAndSwapNNode(control(), mem, adr,
                                                              newval_enc, oldval_enc));
    ...
          cas = _gvn.transform(new (C, 5) CompareAndSwapPNode(control(), mem, adr, newval, oldval));
    ...
      // SCMemProjNodes represent the memory state of CAS. Their main
      // role is to prevent CAS nodes from being optimized away when their
      // results aren't used.
      Node* proj = _gvn.transform( new (C, 1) SCMemProjNode(cas));
      set_memory(proj, alias_idx);
```

### 内部構造(Internal structure)
control input は持たない. 入力ノードは 1つで, 処理対象の LoadStoreNode を指す
(正確には, 第1入力ノードは型の上ではどんな Node も設定可能になっているが, 実際には LoadStoreNode のサブクラスしか設定されない).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      SCMemProjNode( Node *src) : ProjNode( src, SCMEMPROJCON) { }
```




### 詳細(Details)
See: [here](../doxygen/classSCMemProjNode.html) for details

---
## <a name="noAXFWs9FA" id="noAXFWs9FA">LoadStoreNode</a>

### 概要(Summary)
Node クラスのサブクラスの1つ.
全てのアトミックなメモリ操作用の Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------LoadStoreNode---------------------------
    // Note: is_Mem() method returns 'true' for this class.
    class LoadStoreNode : public Node {
```

### 内部構造(Internal structure)
(control input も含めて) 5つの入力ノードを持つ.
それぞれの入力の意味は以下の通り (最初の 4つはスーパークラスである MemNode の入力ノードの先頭 4つに対応する).

* 1番目の入力Node : MemNode の入力ノードと同様 (control input)
* 2番目の入力Node : MemNode の入力ノードと同様 (メモリアクセス間の依存を示すためのノード)
* 3番目の入力Node : MemNode の入力ノードと同様 (ロード/ストアの対象となるメモリアドレス)
* 4番目の入力Node : MemNode の入力ノードと同様 (ストアする新しい値)
* 5番目の入力Node : ストア前の値に対する想定. この値と違っていればストアは失敗する.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      enum {
        ExpectedIn = MemNode::ValueIn+1 // One more input than MemNode
      };
```


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      LoadStoreNode( Node *c, Node *mem, Node *adr, Node *val, Node *ex);
```

### 備考(Notes)
なお, このクラスは以下のファイル中に書かれている Ideal クラスの一覧には出ていない.

* hotspot/src/share/vm/opto/opcodes.hpp
* hotspot/src/share/vm/opto/classes.hpp




### 詳細(Details)
See: [here](../doxygen/classLoadStoreNode.html) for details

---
## <a name="nokJaBZCzx" id="nokJaBZCzx">StorePConditionalNode</a>

### 概要(Summary)
LoadStoreNode クラスの具象サブクラスの1つ.
このクラスは store-conditional 命令によるポインタ値のストア用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StorePConditionalNode---------------------------
    // Conditionally store pointer to memory, if no change since prior
    // load-locked.  Sets flags for success or failure of the store.
    class StorePConditionalNode : public LoadStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
PhaseMacroExpand::expand_allocate_common() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 5つの入力ノードを持つ (See: LoadStoreNode).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StorePConditionalNode( Node *c, Node *mem, Node *adr, Node *val, Node *ll ) : LoadStoreNode(c, mem, adr, val, ll) { }
```

なお, 出力として成否を示す condition code 値と変更後の memory state を出す.
それぞれ BoolNode と SCMemProjNode によって取り出す.




### 詳細(Details)
See: [here](../doxygen/classStorePConditionalNode.html) for details

---
## <a name="noKgJKc1yI" id="noKgJKc1yI">StoreIConditionalNode</a>

### 概要(Summary)
LoadStoreNode クラスの具象サブクラスの1つ.
このクラスは store-conditional 命令による int 値のストア用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StoreIConditionalNode---------------------------
    // Conditionally store int to memory, if no change since prior
    // load-locked.  Sets flags for success or failure of the store.
    class StoreIConditionalNode : public LoadStoreNode {
```

なお StoreXConditionalNode という型も使われるが, 
`#ifdef _LP64` でない場合は, これは StoreIConditionalNode の別名.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define StoreXConditionalNode StoreIConditionalNode
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
PhaseMacroExpand::expand_lock_node() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 5つの入力ノードを持つ (See: LoadStoreNode).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreIConditionalNode( Node *c, Node *mem, Node *adr, Node *val, Node *ii ) : LoadStoreNode(c, mem, adr, val, ii) { }
```

なお, 出力として成否を示す condition code 値と変更後の memory state を出す.
それぞれ BoolNode と SCMemProjNode によって取り出す.




### 詳細(Details)
See: [here](../doxygen/classStoreIConditionalNode.html) for details

---
## <a name="no91fwpIsr" id="no91fwpIsr">StoreLConditionalNode</a>

### 概要(Summary)
LoadStoreNode クラスの具象サブクラスの1つ.
このクラスは store-conditional 命令による long 値のストア用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StoreLConditionalNode---------------------------
    // Conditionally store long to memory, if no change since prior
    // load-locked.  Sets flags for success or failure of the store.
    class StoreLConditionalNode : public LoadStoreNode {
```

なお StoreXConditionalNode という型も使われるが, 
`#ifdef _LP64` の場合は, これは StoreLConditionalNode の別名.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    #define StoreXConditionalNode StoreLConditionalNode
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* LibraryCallKit::inline_native_AtomicLong_attemptUpdate()
* PhaseMacroExpand::expand_lock_node()

### 内部構造(Internal structure)
(control input も含めて) 5つの入力ノードを持つ (See: LoadStoreNode).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StoreLConditionalNode( Node *c, Node *mem, Node *adr, Node *val, Node *ll ) : LoadStoreNode(c, mem, adr, val, ll) { }
```

なお, 出力として成否を示す condition code 値と変更後の memory state を出す.
それぞれ BoolNode と SCMemProjNode によって取り出す.




### 詳細(Details)
See: [here](../doxygen/classStoreLConditionalNode.html) for details

---
## <a name="no_nQYgJur" id="no_nQYgJur">CompareAndSwapLNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

LoadStoreNode クラスの具象サブクラスの1つ.
このクラスは CAS 命令による long 値のストア用
(= sun.misc.Unsafe.compareAndSwapLong() 用).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------CompareAndSwapLNode---------------------------
    class CompareAndSwapLNode : public LoadStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_unsafe_CAS() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_unsafe_CAS()
```

### 内部構造(Internal structure)
(control input も含めて) 5つの入力ノードを持つ (See: LoadStoreNode).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      CompareAndSwapLNode( Node *c, Node *mem, Node *adr, Node *val, Node *ex) : LoadStoreNode(c, mem, adr, val, ex) { }
```

なお, 出力として成否を示す bool 値と変更後の memory state を出す.
memory state は SCMemProjNode によって取り出す
(より正確に言うと, 出力は基本的には bool 値として扱われるが,
 その bool 値を SCMemProjNode に入力すると memory state を取り出すこともできる, という扱いになっている)

(この SCMemProjNode ノードには, 結果が使用されない場合に最適化で消去されるのを防ぐという役割もある模様)

```
    ((cite: hotspot/src/share/vm/opto/library_call.cpp))
    bool LibraryCallKit::inline_unsafe_CAS(BasicType type) {
    ...
      case T_INT:
        cas = _gvn.transform(new (C, 5) CompareAndSwapINode(control(), mem, adr, newval, oldval));
    ...
      case T_LONG:
        cas = _gvn.transform(new (C, 5) CompareAndSwapLNode(control(), mem, adr, newval, oldval));
    ...
      case T_OBJECT:
    ...
          cas = _gvn.transform(new (C, 5) CompareAndSwapNNode(control(), mem, adr,
                                                              newval_enc, oldval_enc));
    ...
          cas = _gvn.transform(new (C, 5) CompareAndSwapPNode(control(), mem, adr, newval, oldval));
    ...
      // SCMemProjNodes represent the memory state of CAS. Their main
      // role is to prevent CAS nodes from being optimized away when their
      // results aren't used.
      Node* proj = _gvn.transform( new (C, 1) SCMemProjNode(cas));
      set_memory(proj, alias_idx);
```




### 詳細(Details)
See: [here](../doxygen/classCompareAndSwapLNode.html) for details

---
## <a name="no3JIMHiEp" id="no3JIMHiEp">CompareAndSwapINode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

LoadStoreNode クラスの具象サブクラスの1つ.
このクラスは CAS 命令による int 値のストア用
(= sun.misc.Unsafe.compareAndSwapInt() 用).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------CompareAndSwapINode---------------------------
    class CompareAndSwapINode : public LoadStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_unsafe_CAS() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_unsafe_CAS()
```

### 内部構造(Internal structure)
(control input も含めて) 5つの入力ノードを持つ (See: LoadStoreNode).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      CompareAndSwapINode( Node *c, Node *mem, Node *adr, Node *val, Node *ex) : LoadStoreNode(c, mem, adr, val, ex) { }
```

なお, 出力として成否を示す bool 値と変更後の memory state を出す.
memory state は SCMemProjNode によって取り出す
(より正確に言うと, 出力は基本的には bool 値として扱われるが,
 その bool 値を SCMemProjNode に入力すると memory state を取り出すこともできる, という扱いになっている)

(この SCMemProjNode ノードには, 結果が使用されない場合に最適化で消去されるのを防ぐという役割もある模様)

```
    ((cite: hotspot/src/share/vm/opto/library_call.cpp))
    bool LibraryCallKit::inline_unsafe_CAS(BasicType type) {
    ...
      case T_INT:
        cas = _gvn.transform(new (C, 5) CompareAndSwapINode(control(), mem, adr, newval, oldval));
    ...
      case T_LONG:
        cas = _gvn.transform(new (C, 5) CompareAndSwapLNode(control(), mem, adr, newval, oldval));
    ...
      case T_OBJECT:
    ...
          cas = _gvn.transform(new (C, 5) CompareAndSwapNNode(control(), mem, adr,
                                                              newval_enc, oldval_enc));
    ...
          cas = _gvn.transform(new (C, 5) CompareAndSwapPNode(control(), mem, adr, newval, oldval));
    ...
      // SCMemProjNodes represent the memory state of CAS. Their main
      // role is to prevent CAS nodes from being optimized away when their
      // results aren't used.
      Node* proj = _gvn.transform( new (C, 1) SCMemProjNode(cas));
      set_memory(proj, alias_idx);
```




### 詳細(Details)
See: [here](../doxygen/classCompareAndSwapINode.html) for details

---
## <a name="no3JuaeXrU" id="no3JuaeXrU">CompareAndSwapPNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

LoadStoreNode クラスの具象サブクラスの1つ.
このクラスは CAS 命令によるポインタ値のストア用
(= sun.misc.Unsafe.compareAndSwapObject() 用).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------CompareAndSwapPNode---------------------------
    class CompareAndSwapPNode : public LoadStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_unsafe_CAS() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_unsafe_CAS()
```

### 内部構造(Internal structure)
(control input も含めて) 5つの入力ノードを持つ (See: LoadStoreNode).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      CompareAndSwapPNode( Node *c, Node *mem, Node *adr, Node *val, Node *ex) : LoadStoreNode(c, mem, adr, val, ex) { }
```

なお, 出力として成否を示す bool 値と変更後の memory state を出す.
memory state は SCMemProjNode によって取り出す
(より正確に言うと, 出力は基本的には bool 値として扱われるが,
 その bool 値を SCMemProjNode に入力すると memory state を取り出すこともできる, という扱いになっている)

(この SCMemProjNode ノードには, 結果が使用されない場合に最適化で消去されるのを防ぐという役割もある模様)

```
    ((cite: hotspot/src/share/vm/opto/library_call.cpp))
    bool LibraryCallKit::inline_unsafe_CAS(BasicType type) {
    ...
      case T_INT:
        cas = _gvn.transform(new (C, 5) CompareAndSwapINode(control(), mem, adr, newval, oldval));
    ...
      case T_LONG:
        cas = _gvn.transform(new (C, 5) CompareAndSwapLNode(control(), mem, adr, newval, oldval));
    ...
      case T_OBJECT:
    ...
          cas = _gvn.transform(new (C, 5) CompareAndSwapNNode(control(), mem, adr,
                                                              newval_enc, oldval_enc));
    ...
          cas = _gvn.transform(new (C, 5) CompareAndSwapPNode(control(), mem, adr, newval, oldval));
    ...
      // SCMemProjNodes represent the memory state of CAS. Their main
      // role is to prevent CAS nodes from being optimized away when their
      // results aren't used.
      Node* proj = _gvn.transform( new (C, 1) SCMemProjNode(cas));
      set_memory(proj, alias_idx);
```




### 詳細(Details)
See: [here](../doxygen/classCompareAndSwapPNode.html) for details

---
## <a name="noi8ElPhB_" id="noi8ElPhB_">CompareAndSwapNNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

LoadStoreNode クラスの具象サブクラスの1つ.
このクラスは CAS 命令による narrow oop 値のストア用
(= sun.misc.Unsafe.compareAndSwapObject() 用).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------CompareAndSwapNNode---------------------------
    class CompareAndSwapNNode : public LoadStoreNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_unsafe_CAS() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_unsafe_CAS()
```

### 内部構造(Internal structure)
(control input も含めて) 5つの入力ノードを持つ (See: LoadStoreNode).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      CompareAndSwapNNode( Node *c, Node *mem, Node *adr, Node *val, Node *ex) : LoadStoreNode(c, mem, adr, val, ex) { }
```

なお, 出力として成否を示す bool 値と変更後の memory state を出す.
memory state は SCMemProjNode によって取り出す
(より正確に言うと, 出力は基本的には bool 値として扱われるが,
 その bool 値を SCMemProjNode に入力すると memory state を取り出すこともできる, という扱いになっている)

(この SCMemProjNode ノードには, 結果が使用されない場合に最適化で消去されるのを防ぐという役割もある模様)

```
    ((cite: hotspot/src/share/vm/opto/library_call.cpp))
    bool LibraryCallKit::inline_unsafe_CAS(BasicType type) {
    ...
      case T_INT:
        cas = _gvn.transform(new (C, 5) CompareAndSwapINode(control(), mem, adr, newval, oldval));
    ...
      case T_LONG:
        cas = _gvn.transform(new (C, 5) CompareAndSwapLNode(control(), mem, adr, newval, oldval));
    ...
      case T_OBJECT:
    ...
          cas = _gvn.transform(new (C, 5) CompareAndSwapNNode(control(), mem, adr,
                                                              newval_enc, oldval_enc));
    ...
          cas = _gvn.transform(new (C, 5) CompareAndSwapPNode(control(), mem, adr, newval, oldval));
    ...
      // SCMemProjNodes represent the memory state of CAS. Their main
      // role is to prevent CAS nodes from being optimized away when their
      // results aren't used.
      Node* proj = _gvn.transform( new (C, 1) SCMemProjNode(cas));
      set_memory(proj, alias_idx);
```




### 詳細(Details)
See: [here](../doxygen/classCompareAndSwapNNode.html) for details

---
## <a name="noAp4exCLg" id="noAp4exCLg">ClearArrayNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.

メモリのゼロクリア処理を表す Node クラス.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------ClearArray-------------------------------------
    class ClearArrayNode: public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ClearArrayNode::clear_memory() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : メモリアクセス間の依存を示すためのノード (MemNode の入力ノードと同様)
* 3番目の入力Node : クリア対象の領域のワード数
* 4番目の入力Node : クリア対象の領域の先頭アドレス


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      ClearArrayNode( Node *ctrl, Node *arymem, Node *word_cnt, Node *base )
        : Node(ctrl,arymem,word_cnt,base) {
        init_class_id(Class_ClearArray);
      }
```




### 詳細(Details)
See: [here](../doxygen/classClearArrayNode.html) for details

---
## <a name="no1Wkr-g1f" id="no1Wkr-g1f">StrIntrinsicNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス (の基底クラス)

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StrIntrinsic-------------------------------
    // Base class for Ideal nodes used in String instrinsic code.
    class StrIntrinsicNode: public Node {
```

### 内部構造(Internal structure)
このクラス(のサブクラス)は (control input も含めて) 6つ, または 5つ, または 4つの入力ノードを持つ.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StrIntrinsicNode(Node* control, Node* char_array_mem,
                       Node* s1, Node* c1, Node* s2, Node* c2):
        Node(control, char_array_mem, s1, c1, s2, c2) {
      }
    
      StrIntrinsicNode(Node* control, Node* char_array_mem,
                       Node* s1, Node* s2, Node* c):
        Node(control, char_array_mem, s1, s2, c) {
      }
    
      StrIntrinsicNode(Node* control, Node* char_array_mem,
                       Node* s1, Node* s2):
        Node(control, char_array_mem, s1, s2) {
      }
```




### 詳細(Details)
See: [here](../doxygen/classStrIntrinsicNode.html) for details

---
## <a name="noEXWFdzni" id="noEXWFdzni">StrCompNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

StrIntrinsicNode クラスの具象サブクラスの1つ.
このクラスは文字列の比較演算用
(= java.lang.String.compareTo() 用).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StrComp-------------------------------------
    class StrCompNode: public StrIntrinsicNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::make_string_method_node() 内で(のみ)生成されている
(より正確に言うと Op_StrComp 定数を引数としてこの関数が呼び出された場合にのみ生成される).
そして, この関数は現在は以下のパスで(のみ) Op_StrComp 定数を引数として呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_string_compareTo()
      -> LibraryCallKit::make_string_method_node()
```

### 内部構造(Internal structure)
(control input も含めて) 6つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : メモリアクセス間の依存を示すためのノード (MemNode の入力ノードと同様)
* 3番目の入力Node : 比較対象の文字列(その1)
* 4番目の入力Node : 比較対象の文字列(その1)の長さ
* 5番目の入力Node : 比較対象の文字列(その2)
* 6番目の入力Node : 比較対象の文字列(その2)の長さ


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StrCompNode(Node* control, Node* char_array_mem,
                  Node* s1, Node* c1, Node* s2, Node* c2):
        StrIntrinsicNode(control, char_array_mem, s1, c1, s2, c2) {};
```




### 詳細(Details)
See: [here](../doxygen/classStrCompNode.html) for details

---
## <a name="no2_oiKJuW" id="no2_oiKJuW">StrEqualsNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

StrIntrinsicNode クラスの具象サブクラスの1つ.
このクラスは文字列の比較演算用
(= java.lang.String.equals() 用).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StrEquals-------------------------------------
    class StrEqualsNode: public StrIntrinsicNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::make_string_method_node() 内で(のみ)生成されている
(より正確に言うと Op_StrEquals 定数を引数としてこの関数が呼び出された場合にのみ生成される).
そして, この関数は現在は以下のパスで(のみ) Op_StrEquals 定数を引数として呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_string_equals()
      -> LibraryCallKit::make_string_method_node()
```

### 内部構造(Internal structure)
(control input も含めて) 5つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : メモリアクセス間の依存を示すためのノード (MemNode の入力ノードと同様)
* 3番目の入力Node : 比較対象の文字列(その1)
* 4番目の入力Node : 比較対象の文字列(その1)の長さ
* 5番目の入力Node : 比較対象の文字列(その2)


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StrEqualsNode(Node* control, Node* char_array_mem,
                    Node* s1, Node* s2, Node* c):
        StrIntrinsicNode(control, char_array_mem, s1, s2, c) {};
```




### 詳細(Details)
See: [here](../doxygen/classStrEqualsNode.html) for details

---
## <a name="noQzQ7RSq_" id="noQzQ7RSq_">StrIndexOfNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

StrIntrinsicNode クラスの具象サブクラスの1つ.
このクラスは文字列中の部分文字列の検出用
(= java.lang.String.indexOf() 用).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------StrIndexOf-------------------------------------
    class StrIndexOfNode: public StrIntrinsicNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::make_string_method_node() 内で(のみ)生成されている
(より正確に言うと Op_StrIndexOf 定数を引数としてこの関数が呼び出された場合にのみ生成される).
そして, この関数は現在は以下のパスで(のみ) Op_StrIndexOf 定数を引数として呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_string_indexOf()
      -> LibraryCallKit::make_string_method_node()
```

### 内部構造(Internal structure)
(control input も含めて) 6つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : メモリアクセス間の依存を示すためのノード (MemNode の入力ノードと同様)
* 3番目の入力Node : 部分文字列の検出対象となる文字列
* 4番目の入力Node : 部分文字列の検出対象となる文字列の長さ
* 5番目の入力Node : 検出する部分文字列を示す文字列
* 6番目の入力Node : 検出する部分文字列を示す文字列の長さ


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      StrIndexOfNode(Node* control, Node* char_array_mem,
                  Node* s1, Node* c1, Node* s2, Node* c2):
        StrIntrinsicNode(control, char_array_mem, s1, c1, s2, c2) {};
```




### 詳細(Details)
See: [here](../doxygen/classStrIndexOfNode.html) for details

---
## <a name="nodi4HhtC3" id="nodi4HhtC3">AryEqNode</a>

### 概要(Summary)
LibraryIntrinsic による最適化用の Node クラス.

StrIntrinsicNode クラスの具象サブクラスの1つ.
このクラスは配列の比較演算用
(= java.util.Arrays.equals() 用).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------AryEq---------------------------------------
    class AryEqNode: public StrIntrinsicNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_array_equals() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_array_equals()
```

### 内部構造(Internal structure)
(control input も含めて) 4つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : メモリアクセス間の依存を示すためのノード (MemNode の入力ノードと同様)
* 3番目の入力Node : 比較対象の配列(その1)
* 4番目の入力Node : 比較対象の配列(その2)


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      AryEqNode(Node* control, Node* char_array_mem, Node* s1, Node* s2):
        StrIntrinsicNode(control, char_array_mem, s1, s2) {};
```




### 詳細(Details)
See: [here](../doxygen/classAryEqNode.html) for details

---
## <a name="nouboi-Dcd" id="nouboi-Dcd">MemBarNode</a>

### 概要(Summary)
MultiNode クラスのサブクラスの1つ.
全てのメモリバリア処理用の Node クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------MemBar-----------------------------------------
    // There are different flavors of Memory Barriers to match the Java Memory
    // Model.  Monitor-enter and volatile-load act as Aquires: no following ref
    // can be moved to before them.  We insert a MemBar-Acquire after a FastLock or
    // volatile-load.  Monitor-exit and volatile-store act as Release: no
    // preceding ref can be moved to after them.  We insert a MemBar-Release
    // before a FastUnlock or volatile-store.  All volatiles need to be
    // serialized, so we follow all volatile-stores with a MemBar-Volatile to
    // separate it from any following volatile-load.
    class MemBarNode: public MultiNode {
```

### 内部構造(Internal structure)
(control input も含めて) 5つまたは6つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り.

* TypeFunc::Control

  ?? 使われているか?? (#TODO)

* TypeFunc::Memory

  ?? 使われているか?? (#TODO)

* TypeFunc::I_O

  (#TODO)

* TypeFunc::FramePtr

  (#TODO)

* TypeFunc::ReturnAdr

  (#TODO)

* TypeFunc::Parms (= MemBarNode::Precedent)
  
  precedent コンストラクタ引数が指定されていた場合には, ここに格納される.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      enum {
        Precedent = TypeFunc::Parms  // optional edge to force precedence
      };
```


```
    ((cite: hotspot/src/share/vm/opto/memnode.cpp))
    MemBarNode::MemBarNode(Compile* C, int alias_idx, Node* precedent)
      : MultiNode(TypeFunc::Parms + (precedent == NULL? 0: 1)),
        _adr_type(C->get_adr_type(alias_idx))
    {
      init_class_id(Class_MemBar);
      Node* top = C->top();
      init_req(TypeFunc::I_O,top);
      init_req(TypeFunc::FramePtr,top);
      init_req(TypeFunc::ReturnAdr,top);
      if (precedent != NULL)
        init_req(TypeFunc::Parms, precedent);
    }
```




### 詳細(Details)
See: [here](../doxygen/classMemBarNode.html) for details

---
## <a name="noFzg6ME0H" id="noFzg6ME0H">MemBarAcquireNode</a>

### 概要(Summary)
MemBarNode クラスの具象サブクラスの1つ.
このクラスは acquire barrier 用
(後続の load はこの命令を追い越せない).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    // "Acquire" - no following ref can move before (but earlier refs can
    // follow, like an early Load stalled in cache).  Requires multi-cpu
    // visibility.  Inserted after a volatile load or FastLock.
    class MemBarAcquireNode: public MemBarNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
MemBarNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
GraphKit::insert_mem_bar()
-> MemBarNode::make()

GraphKit::insert_mem_bar_volatile()
-> MemBarNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 5つまたは6つの入力ノードを持つ (See: MemBar).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      MemBarAcquireNode(Compile* C, int alias_idx, Node* precedent)
        : MemBarNode(C, alias_idx, precedent) {}
```




### 詳細(Details)
See: [here](../doxygen/classMemBarAcquireNode.html) for details

---
## <a name="noRT_j1n1e" id="noRT_j1n1e">MemBarReleaseNode</a>

### 概要(Summary)
MemBarNode クラスの具象サブクラスの1つ.
このクラスは release barrier 用
(先行する store はこの命令より後ろにリオーダされない).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    // "Release" - no earlier ref can move after (but later refs can move
    // up, like a speculative pipelined cache-hitting Load).  Requires
    // multi-cpu visibility.  Inserted before a volatile store or FastUnLock.
    class MemBarReleaseNode: public MemBarNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
MemBarNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
GraphKit::insert_mem_bar()
-> MemBarNode::make()

GraphKit::insert_mem_bar_volatile()
-> MemBarNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 5つまたは6つの入力ノードを持つ (See: MemBar).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      MemBarReleaseNode(Compile* C, int alias_idx, Node* precedent)
        : MemBarNode(C, alias_idx, precedent) {}
```




### 詳細(Details)
See: [here](../doxygen/classMemBarReleaseNode.html) for details

---
## <a name="noZLAGfTnb" id="noZLAGfTnb">MemBarVolatileNode</a>

### 概要(Summary)
MemBarNode クラスの具象サブクラスの1つ.
このクラスは先行する volatile store と後続の volatile load の順序付け用.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    // Ordering between a volatile store and a following volatile load.
    // Requires multi-CPU visibility?
    class MemBarVolatileNode: public MemBarNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* MemBarNode::make()
* GraphKit::gen_stub() (ただし #ifdef IA64 の場合のみ)

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

```
GraphKit::insert_mem_bar()
-> MemBarNode::make()

GraphKit::insert_mem_bar_volatile()
-> MemBarNode::make()
```

```
Compile::Compile()
-> GraphKit::gen_stub()
```

### 内部構造(Internal structure)
(control input も含めて) 5つまたは6つの入力ノードを持つ (See: MemBar).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      MemBarVolatileNode(Compile* C, int alias_idx, Node* precedent)
        : MemBarNode(C, alias_idx, precedent) {}
```




### 詳細(Details)
See: [here](../doxygen/classMemBarVolatileNode.html) for details

---
## <a name="noFvAGuZfj" id="noFvAGuZfj">MemBarCPUOrderNode</a>

### 概要(Summary)
MemBarNode クラスの具象サブクラスの1つ.
このクラスは, 実際のメモリバリア命令を出すわけではなく, 
JIT コンパイラ内での最適化によるリオーダを禁止するための Node
(= 同一コア上でのメモリアクセス命令の順序付け用).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    // Ordering within the same CPU.  Used to order unsafe memory references
    // inside the compiler when we lack alias info.  Not needed "outside" the
    // compiler because the CPU does all the ordering for us.
    class MemBarCPUOrderNode: public MemBarNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
MemBarNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
GraphKit::insert_mem_bar()
-> MemBarNode::make()

GraphKit::insert_mem_bar_volatile()
-> MemBarNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 5つまたは6つの入力ノードを持つ (See: MemBar).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      MemBarCPUOrderNode(Compile* C, int alias_idx, Node* precedent)
        : MemBarNode(C, alias_idx, precedent) {}
```




### 詳細(Details)
See: [here](../doxygen/classMemBarCPUOrderNode.html) for details

---
## <a name="noBLOaTm61" id="noBLOaTm61">InitializeNode</a>

### 概要(Summary)
MemBarNode クラスの具象サブクラスの1つ.
このクラスは, 確保したオブジェクトへの初期化処理(初期値のストア)とその後の safepoint との順序付け用.

(なお, より正確に言うと, オブジェクトのフィールドに対するゼロクリア処理もこのノードが表している)

なお, この Node は PhaseMacroExpand 時に AllocateNode と一緒に展開される.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    // Isolation of object setup after an AllocateNode and before next safepoint.
    // (See comment in memnode.cpp near InitializeNode::InitializeNode for semantics.)
    class InitializeNode: public MemBarNode {
```


```
    ((cite: hotspot/src/share/vm/opto/memnode.cpp))
    //===========================InitializeNode====================================
    // SUMMARY:
    // This node acts as a memory barrier on raw memory, after some raw stores.
    // The 'cooked' oop value feeds from the Initialize, not the Allocation.
    // The Initialize can 'capture' suitably constrained stores as raw inits.
    // It can coalesce related raw stores into larger units (called 'tiles').
    // It can avoid zeroing new storage for memory units which have raw inits.
    // At macro-expansion, it is marked 'complete', and does not optimize further.
    //
    // EXAMPLE:
    // The object 'new short[2]' occupies 16 bytes in a 32-bit machine.
    //   ctl = incoming control; mem* = incoming memory
    // (Note:  A star * on a memory edge denotes I/O and other standard edges.)
    // First allocate uninitialized memory and fill in the header:
    //   alloc = (Allocate ctl mem* 16 #short[].klass ...)
    //   ctl := alloc.Control; mem* := alloc.Memory*
    //   rawmem = alloc.Memory; rawoop = alloc.RawAddress
    // Then initialize to zero the non-header parts of the raw memory block:
    //   init = (Initialize alloc.Control alloc.Memory* alloc.RawAddress)
    //   ctl := init.Control; mem.SLICE(#short[*]) := init.Memory
    // After the initialize node executes, the object is ready for service:
    //   oop := (CheckCastPP init.Control alloc.RawAddress #short[])
    // Suppose its body is immediately initialized as {1,2}:
    //   store1 = (StoreC init.Control init.Memory (+ oop 12) 1)
    //   store2 = (StoreC init.Control store1      (+ oop 14) 2)
    //   mem.SLICE(#short[*]) := store2
    //
    // DETAILS:
    // An InitializeNode collects and isolates object initialization after
    // an AllocateNode and before the next possible safepoint.  As a
    // memory barrier (MemBarNode), it keeps critical stores from drifting
    // down past any safepoint or any publication of the allocation.
    // Before this barrier, a newly-allocated object may have uninitialized bits.
    // After this barrier, it may be treated as a real oop, and GC is allowed.
    //
    // The semantics of the InitializeNode include an implicit zeroing of
    // the new object from object header to the end of the object.
    // (The object header and end are determined by the AllocateNode.)
    //
    // Certain stores may be added as direct inputs to the InitializeNode.
    // These stores must update raw memory, and they must be to addresses
    // derived from the raw address produced by AllocateNode, and with
    // a constant offset.  They must be ordered by increasing offset.
    // The first one is at in(RawStores), the last at in(req()-1).
    // Unlike most memory operations, they are not linked in a chain,
    // but are displayed in parallel as users of the rawmem output of
    // the allocation.
    //
    // (See comments in InitializeNode::capture_store, which continue
    // the example given above.)
    //
    // When the associated Allocate is macro-expanded, the InitializeNode
    // may be rewritten to optimize collected stores.  A ClearArrayNode
    // may also be created at that point to represent any required zeroing.
    // The InitializeNode is then marked 'complete', prohibiting further
    // capturing of nearby memory operations.
    //
    // During macro-expansion, all captured initializations which store
    // constant values of 32 bits or smaller are coalesced (if advantageous)
    // into larger 'tiles' 32 or 64 bits.  This allows an object to be
    // initialized in fewer memory operations.  Memory words which are
    // covered by neither tiles nor non-constant stores are pre-zeroed
    // by explicit stores of zero.  (The code shape happens to do all
    // zeroing first, then all other stores, with both sequences occurring
    // in order of ascending offsets.)
    //
    // Alternatively, code may be inserted between an AllocateNode and its
    // InitializeNode, to perform arbitrary initialization of the new object.
    // E.g., the object copying intrinsics insert complex data transfers here.
    // The initialization must then be marked as 'complete' disable the
    // built-in zeroing semantics and the collection of initializing stores.
    //
    // While an InitializeNode is incomplete, reads from the memory state
    // produced by it are optimizable if they match the control edge and
    // new oop address associated with the allocation/initialization.
    // They return a stored value (if the offset matches) or else zero.
    // A write to the memory state, if it matches control and address,
    // and if it is to a constant offset, may be 'captured' by the
    // InitializeNode.  It is cloned as a raw memory operation and rewired
    // inside the initialization, to the raw oop produced by the allocation.
    // Operations on addresses which are provably distinct (e.g., to
    // other AllocateNodes) are allowed to bypass the initialization.
    //
    // The effect of all this is to consolidate object initialization
    // (both arrays and non-arrays, both piecewise and bulk) into a
    // single location, where it can be optimized as a unit.
    //
    // Only stores with an offset less than TrackedInitializationLimit words
    // will be considered for capture by an InitializeNode.  This puts a
    // reasonable limit on the complexity of optimized initializations.
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
MemBarNode::make() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
GraphKit::insert_mem_bar()
-> MemBarNode::make()

GraphKit::insert_mem_bar_volatile()
-> MemBarNode::make()
```

### 内部構造(Internal structure)
(control input も含めて) 5つまたは6つの入力ノードを持つ (See: MemBar).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      enum {
        Control    = TypeFunc::Control,
        Memory     = TypeFunc::Memory,     // MergeMem for states affected by this op
        RawAddress = TypeFunc::Parms+0,    // the newly-allocated raw address
        RawStores  = TypeFunc::Parms+1     // zero or more stores (or TOP)
      };
```


```
    ((cite: hotspot/src/share/vm/opto/memnode.cpp))
    InitializeNode::InitializeNode(Compile* C, int adr_type, Node* rawoop)
      : _is_complete(false),
        MemBarNode(C, adr_type, rawoop)
    {
      init_class_id(Class_Initialize);
    
      assert(adr_type == Compile::AliasIdxRaw, "only valid atp");
      assert(in(RawAddress) == rawoop, "proper init");
      // Note:  allocation() can be NULL, for secondary initialization barriers
    }
```




### 詳細(Details)
See: [here](../doxygen/classInitializeNode.html) for details

---
## <a name="noSWTVvDM0" id="noSWTVvDM0">MergeMemNode</a>

### 概要(Summary)

```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    //------------------------------MergeMem---------------------------------------
    // (See comment in memnode.cpp near MergeMemNode::MergeMemNode for semantics.)
    class MergeMemNode: public Node {
```





### 詳細(Details)
See: [here](../doxygen/classMergeMemNode.html) for details

---
## <a name="nopdJFv9S4" id="nopdJFv9S4">MergeMemStream</a>

### 概要(Summary)
MergeMemNode 中の Node をたどるためのイテレータクラス(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    class MergeMemStream : public StackObj {
```

### 使われ方(Usage)
#### 使用例(usage examples)

```
    ((cite: hotspot/src/share/vm/opto/graphKit.cpp))
        for (MergeMemStream mms(phi_mem); mms.next_non_empty(); ) {
          Node* m = mms.memory();
    ...
        }
```




### 詳細(Details)
See: [here](../doxygen/classMergeMemStream.html) for details

---
## <a name="noaZsM3UgC" id="noaZsM3UgC">PrefetchReadNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
メモリのプリフェッチ処理(load 用)を表す Node クラス.

(なお, 現状では LibraryIntrinsic からしか使用されていない.
 より具体的に言うと, sun.misc.Unsafe.prefetchRead() 
 及び sun.misc.Unsafe.prefetchReadStatic() 用の Node)


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    // Non-faulting prefetch load.  Prefetch for many reads.
    class PrefetchReadNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LibraryCallKit::inline_unsafe_prefetch() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_unsafe_prefetch()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : I/O state (#TODO)
* 3番目の入力Node : プリフェッチするメモリアドレス

(なお, control input はコンストラクタでは 0 が指定されているが, 
 その後すぐに現在のコントロールフロー (control()) に設定される
 (See: LibraryCallKit::inline_unsafe_prefetch()))


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      PrefetchReadNode(Node *abio, Node *adr) : Node(0,abio,adr) {}
```




### 詳細(Details)
See: [here](../doxygen/classPrefetchReadNode.html) for details

---
## <a name="noOA30pwoq" id="noOA30pwoq">PrefetchWriteNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.
メモリのプリフェッチ処理(load/store 用)を表す Node クラス.


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
    // Non-faulting prefetch load.  Prefetch for many reads & many writes.
    class PrefetchWriteNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* LibraryCallKit::inline_unsafe_prefetch()
* PhaseMacroExpand::prefetch_allocation()

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

```
LibraryIntrinsic::generate()
-> LibraryCallKit::try_to_inline()
   -> LibraryCallKit::inline_unsafe_prefetch()
```

```
PhaseMacroExpand::expand_allocate_common()
-> PhaseMacroExpand::prefetch_allocation()
```

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. 
それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : I/O state (#TODO)
* 3番目の入力Node : プリフェッチするメモリアドレス

(なお, control input はコンストラクタでは 0 が指定されているが, 
 LibraryCallKit::inline_unsafe_prefetch() のパスでは, 
 その後すぐに現在のコントロールフロー (control()) に設定される
 (See: LibraryCallKit::inline_unsafe_prefetch()))


```
    ((cite: hotspot/src/share/vm/opto/memnode.hpp))
      PrefetchWriteNode(Node *abio, Node *adr) : Node(0,abio,adr) {}
```

### 備考(Notes)
LibraryIntrinsic からの生成パスは
sun.misc.Unsafe.prefetchWrite() 及び sun.misc.Unsafe.prefetchWriteStatic() 用.




### 詳細(Details)
See: [here](../doxygen/classPrefetchWriteNode.html) for details

---
