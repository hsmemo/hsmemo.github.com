---
layout: default
title: VMThread クラス関連のクラス (VMOperationQueue, VMThread, 及びそれらの補助クラス(VM_Dummy))
---
[Top](../index.html)

#### VMThread クラス関連のクラス (VMOperationQueue, VMThread, 及びそれらの補助クラス(VM_Dummy))

これらは, VM Operation 処理のためのクラス
(See: [here](no2480qPC.html) and [here](no2480eqy.html) for details).


### クラス一覧(class list)

  * [VMThread](#noCWaJy20C)
  * [VMOperationQueue](#noPV2PlKXC)
  * [VM_Dummy](#noRbwdSfHm)


---
## <a name="noCWaJy20C" id="noCWaJy20C">VMThread</a>

### 概要(Summary)
VM Operation を実行するためのスレッドクラス (See: [here](no2480qPC.html) and [here](no2480eqy.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/runtime/vmThread.hpp))
    // A single VMThread (the primordial thread) spawns all other threads
    // and is itself used by other threads to offload heavy vm operations
    // like scavenge, garbage_collect etc.
    //
    
    class VMThread: public NamedThread {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
VMThread クラスの _vm_thread フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
VMThread::create() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている (See: [here](no-la6kE9R.html) for details).

```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> VMThread::create()
```




### 詳細(Details)
See: [here](../doxygen/classVMThread.html) for details

---
## <a name="noPV2PlKXC" id="noPV2PlKXC">VMOperationQueue</a>

### 概要(Summary)
VMThread クラス用の補助クラス.

VMThread に対して VM Operation 要求を伝えるための優先順位付きキュー
(VMThread に対して何らかの VM Operation を要求する時には, 
 VM_Operation オブジェクトをこのキューに詰めることで通知する).


```cpp
    ((cite: hotspot/src/share/vm/runtime/vmThread.hpp))
    // Prioritized queue of VM operations.
    //
    // Encapsulates both queue management and
    // and priority policy
    //
    class VMOperationQueue : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
VMThread クラスの _vm_queue フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
VMThread::create() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている (See: [here](no-la6kE9R.html) for details).

```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> VMThread::create()
```

### 内部構造(Internal structure)
登録された VM_Operation オブジェクトは, 優先度毎の doubly linked list で管理している.

各リストは _queue というポインタ配列に格納されている.
各リストの長さは _queue_length フィールドに格納されている.

```cpp
    ((cite: hotspot/src/share/vm/runtime/vmThread.hpp))
      // We maintain a doubled linked list, with explicit count.
      int           _queue_length[nof_priorities];
      int           _queue_counter;
      VM_Operation* _queue       [nof_priorities];
```

なお, 現状の実装では優先度は３段階
(Safepoint が必要かどうかに応じて分けられている(? #TODO)).

```cpp
    ((cite: hotspot/src/share/vm/runtime/vmThread.hpp))
      enum Priorities {
         SafepointPriority, // Highest priority (operation executed at a safepoint)
         MediumPriority,    // Medium priority
         nof_priorities
      };
```

なお, 実行するのが GC 用の VM_Operation の場合は少し注意が必要になる
(実行中(あるいは実行予定)の VM_Operation オブジェクト自体が GC の調査対象になっていないと, 
 GC が終わったときにはポインタが不正になっている恐れがある).

そのため, 処理中(あるいは処理予定)の VM_Operation を入れておくための _drain_list というフィールドが用意されている
(この _drain_list フィールドは GC 時のチェック対象になっている.
 ここに登録しておけば処理途中で GC が起きてもポインタが不正にならないと保証される).

```cpp
    ((cite: hotspot/src/share/vm/runtime/vmThread.hpp))
      // we also allow the vmThread to register the ops it has drained so we
      // can scan them from oops_do
      VM_Operation* _drain_list;
```




### 詳細(Details)
See: [here](../doxygen/classVMOperationQueue.html) for details

---
## <a name="noRbwdSfHm" id="noRbwdSfHm">VM_Dummy</a>

### 概要(Summary)
VMOperationQueue クラス内で使用される補助クラス.

生成直後の VMOperationQueue オブジェクトには何も VM Operation 要求が入っていないが, 
VMOperationQueue は要求を circular double-linked list で管理しており, 
最低1つは要素がないとリストが作れない.
このため, ダミー要素として VM_Dummy オブジェクトが詰められる.

(以上の理由から, VMOperationQueue 内部の _queue リストは, 要素数が 1 の時に空リストを表す. 
そして 1 未満になることはない)


```cpp
    ((cite: hotspot/src/share/vm/runtime/vmThread.cpp))
    // Dummy VM operation to act as first element in our circular double-linked list
    class VM_Dummy: public VM_Operation {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 VMOperationQueue クラスの _queue フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
VMOperationQueue::VMOperationQueue() 内で(のみ)生成されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/vmThread.cpp))
    VMOperationQueue::VMOperationQueue() {
      // The queue is a circular doubled-linked list, which always contains
      // one element (i.e., one element means empty).
      for(int i = 0; i < nof_priorities; i++) {
    ...
        _queue[i] = new VM_Dummy();
        _queue[i]->set_next(_queue[i]);
        _queue[i]->set_prev(_queue[i]);
```




### 詳細(Details)
See: [here](../doxygen/classVM__Dummy.html) for details

---
