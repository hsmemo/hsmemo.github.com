---
layout: default
title: PtrQueue クラス関連のクラス (PtrQueue, BufferNode, PtrQueueSet)
---
[Top](../index.html)

#### PtrQueue クラス関連のクラス (PtrQueue, BufferNode, PtrQueueSet)

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, write barrier 処理を補佐するためのクラス (See: [here](no2114EV0.html) for details).

### 概要(Summary)
これらは, write barrier 処理で記録されるポインタ値を格納しておくためのクラス(の基底クラス).
例えば, SATB 方式の write barrier では変更前のポインタ値をどこかに記録しておく必要があるのでそういう用途で使う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/ptrQueue.hpp))
    // There are various techniques that require threads to be able to log
    // addresses.  For example, a generational write barrier might log
    // the addresses of modified old-generation objects.  This type supports
    // this operation.
```

なお, これらの機能は write barrier 以外のことにも使えそうだが, 
現状では PtrQueue クラスと PtrQueueSet クラスは, 
それぞれ DirtyCardQueue と ObjPtrQueue および DirtyCardQueueSet と SATBMarkQueueSet というサブクラスしか持たないため
(これらは全て write barrier 処理で使用されるクラス), 
事実上 write barrier 専用のクラスとなっている (See: [here](no2114EV0.html) for details).

PtrQueue クラスは, 実際にポインタが記録されるクラス.
バッファが一杯になると, PtrQueueSet から次のバッファを取得する.

PtrQueueSet クラスは, PtrQueue 用のバッファを管理するクラス.
バッファは内部では BufferNode オブジェクトとして管理されている.



### クラス一覧(class list)

  * [PtrQueue](#non63fXVSa)
  * [PtrQueueSet](#noE9KXrGvu)
  * [BufferNode](#no-jrMtw0J)


---
## <a name="non63fXVSa" id="non63fXVSa">PtrQueue</a>

### 概要(Summary)
write barrier 処理で何らかのポインタ値を記録するためのクラス (の基底クラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/ptrQueue.hpp))
    class PtrQueue VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
内部的には, _buf フィールドの配列にポインタを蓄えている.
なお, _buf フィールドの長さは _sz フィールドに格納されている.

_buf フィールド中の次に使用する位置は _index フィールドが指している. 
_index は最大値(_sz)から始まり, ポインタを入れる度にデクリメントされる (0 になるとそのバッファは一杯, ということになる)
(See: MacroAssembler::g1_write_barrier_pre(), generate_satb_log_enqueue()).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/ptrQueue.hpp))
      // The buffer.
      void** _buf;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/ptrQueue.hpp))
      // The index at which an object was last enqueued.  Starts at "_sz"
      // (indicating an empty buffer) and goes towards zero.
      size_t _index;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/ptrQueue.hpp))
      // The size of the buffer.
      size_t _sz;
```

また, "inactive" 状態の間はポインタを追加しても無視される. 状態は set_active() で変更できる. 
(SATB のバリアでは, ポインタの記録処理が必要なのは concurrent marking 中だけなので, 切り替えられるようにしている模様.
 write barrier 処理中でこれを確認してポインタを記録するかどうかを切り替えている.
 See: MacroAssembler::g1_write_barrier_pre())


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/ptrQueue.hpp))
      // Whether updates should be logged.
      bool _active;
```




### 詳細(Details)
See: [here](../doxygen/classPtrQueue.html) for details

---
## <a name="noE9KXrGvu" id="noE9KXrGvu">PtrQueueSet</a>

### 概要(Summary)
PtrQueue 用のバッファ(メモリ領域)を管理するクラス (の基底クラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/ptrQueue.hpp))
    // A PtrQueueSet represents resources common to a set of pointer queues.
    // In particular, the individual queues allocate buffers from this shared
    // set, and return completed buffers to the set.
    // All these variables are are protected by the TLOQ_CBL_mon. XXX ???
    class PtrQueueSet VALUE_OBJ_CLASS_SPEC {
```

各 PtrQueue はまず PtrQueueSet から空のバッファを取得し,
それが一杯になると PtrQueueSet に返却する (そして代わりに新しいバッファを確保する).
このため, PtrQueue オブジェクトは生成時に所属する PtrQueueSet を指定する必要がある.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/ptrQueue.hpp))
      // Initialize this queue to contain a null buffer, and be part of the
      // given PtrQueueSet.
      PtrQueue(PtrQueueSet* qset, bool perm = false, bool active = false);
```




### 詳細(Details)
See: [here](../doxygen/classPtrQueueSet.html) for details

---
## <a name="no-jrMtw0J" id="no-jrMtw0J">BufferNode</a>

### 概要(Summary)
PtrQueueSet クラス用の補助クラス.

PtrQueueSet クラス内でバッファを管理するために使用されるクラス
(BufferNode クラスを使うとバッファを linked list 状にして管理できる).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/ptrQueue.hpp))
    class BufferNode {
```

なお, BufferNode オブジェクトはバッファとして使用する領域内に埋め込まれる 
(malloc() が使用するメタ情報のようなもの.
 より正確に言うと, 「バッファとして使用される領域よりも前の部分に BufferNode オブジェクトが埋め込まれている」)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/ptrQueue.hpp))
      // BufferNode is allocated before the buffer.
      // The chunk of memory that holds both of them is a block.
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. NEW_C_HEAP_ARRAY() 等でメモリを確保する 
   (この際には, 実際にバッファとして使う量に加えて BufferNode 用のヘッダ分も含めた量を確保する)

2. 取得したメモリに対して BufferNode::make_buffer_from_block() を呼ぶと, 
   実際にバッファとして使える領域の先頭アドレスが返される
   (先頭から BufferNode のヘッダ分だけ進めた地点のポインタが返される).

3. こうして作ったバッファは, BufferNode::new_from_buffer() で BufferNode オブジェクトに変換できる.

   (まぁポインタずらすだけだけど...)

4. 逆に, BufferNode オブジェクトからバッファを取り出したい場合は BufferNode::make_buffer_from_node() を使えばいい.

   (まぁポインタずらすだけだけど...)

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 PtrQueueSet オブジェクトの _completed_buffers_head フィールド
  
  使用済みのバッファ(= ポインタ値で一杯になったバッファ)を格納するフィールド.

  (正確には, このフィールドは BufferNode の線形リストを格納するフィールド.
  BufferNode オブジェクトは _next フィールドで次の BufferNode オブジェクトを指せる構造になっている.)

* 各 PtrQueueSet オブジェクトの _completed_buffers_tail フィールド
  
  _completed_buffers_head フィールドに格納されている線形リストの末尾を指すポインタ.

* 各 PtrQueueSet オブジェクトの _buf_free_list フィールド
  
  未使用のバッファを格納するフィールド
  (正確には, このフィールドは BufferNode の線形リストを格納するフィールド(フリーリスト)).

#### 生成箇所(where its instances are created)
メモリ領域の確保は PtrQueueSet::allocate_buffer() で行われている.

BufferNode オブジェクトとしての生成箇所は BufferNode::new_from_buffer() になる
(ただしこの中で使われている new は placement new なので, 上に書いたとおり, ここでメモリを確保しているわけではない).
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
(略) (See: [here](no2935YzN.html) for details)
-> G1RemSet::prepare_for_oops_into_collection_set_do()
   -> DirtyCardQueueSet::concatenate_logs()
      -> PtrQueueSet::enqueue_complete_buffer()
         -> BufferNode::new_from_buffer()

(略) (See: [here](no2935dGZ.html) and [here](no2935YzN.html) for details)
-> DirtyCardQueueSet::apply_closure_to_completed_buffer_helper()
   -> PtrQueueSet::enqueue_complete_buffer()
      -> (同上)

PtrQueue::~PtrQueue()
-> PtrQueue::flush()
   -> PtrQueueSet::enqueue_complete_buffer()
      -> (同上)

(略) (See: [here](no2114EV0.html) for details)
-> PtrQueue::locking_enqueue_completed_buffer()
   -> PtrQueueSet::enqueue_complete_buffer()
      -> (同上)

(略) (See: [here](no2114EV0.html) for details)
-> PtrQueueSet::process_or_enqueue_complete_buffer()
   -> PtrQueueSet::enqueue_complete_buffer()
      -> (同上)
```




### 詳細(Details)
See: [here](../doxygen/classBufferNode.html) for details

---
