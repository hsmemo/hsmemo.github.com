---
layout: default
title: JNIHandles クラス関連のクラス (JNIHandles, JNIHandleBlock, 及びそれらの補助クラス(AlwaysAliveClosure, CountHandleClosure, VerifyHandleClosure))
---
[Top](../index.html)

#### JNIHandles クラス関連のクラス (JNIHandles, JNIHandleBlock, 及びそれらの補助クラス(AlwaysAliveClosure, CountHandleClosure, VerifyHandleClosure))

これらは, JNI の機能(より具体的に言うと JNI 参照(JNI references))を実現するためのクラス
(See: [here](notiXs9FLU.html) and [here](nowJltq41b.html) for details).

### 概要(Summary)
JNI 参照(JNI references)は, JNIHandles クラスと JNIHandleBlock クラスによって実装されている
(なお, JNIHandles クラスの方は AllStatic クラス).

実際の JNI 参照は「JNIHandleBlock オブジェクト内を指すポインタ」として実装されており,
それを操作するメソッドが JNIHandles クラスと JNIHandleBlock クラスに実装されている
(See: [here](notiXs9FLU.html) and [here](nowJltq41b.html) for details).



### クラス一覧(class list)

  * [JNIHandles](#noAerkw_h4)
  * [JNIHandleBlock](#noIw_PEBC3)
  * [AlwaysAliveClosure](#noniTIZx3V)
  * [CountHandleClosure](#noTN0qv-qB)
  * [VerifyHandleClosure](#noTbsSLEbI)


---
## <a name="noAerkw_h4" id="noAerkw_h4">JNIHandles</a>

### 概要(Summary)
JNI 参照(JNI references)に関する処理を納めた名前空間(AllStatic クラス).

なお JNI 参照には Local, Global, Weak Global の 3種類があるが, 内部的には全てこのクラスが担当している.



```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.hpp))
    // Interface for creating and resolving local/global JNI handles
    
    class JNIHandles : AllStatic {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

(主に JNI に関する箇所で)

### 備考(Notes)
static フィールドである JNIHandles::_deleted_handle はダミーの handle 値 
(消去された handle を示すためのもの) を入れておくフィールド.

(コメントによると「NULL は weak global JNI handle の処理で使っているので別の値を用意している」とのこと.
 (See: jni_DeleteLocalRef(), jni_DeleteGlobalRef(), jni_DeleteWeakGlobalRef()))


```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.hpp))
      static oop _deleted_handle;                         // Sentinel marking deleted handles
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.hpp))
      // Sentinel marking deleted handles in block. Note that we cannot store NULL as
      // the sentinel, since clearing weak global JNI refs are done by storing NULL in
      // the handle. The handle may not be reused before destroy_weak_global is called.
      static oop deleted_handle()   { return _deleted_handle; }
```




### 詳細(Details)
See: [here](../doxygen/classJNIHandles.html) for details

---
## <a name="noIw_PEBC3" id="noIw_PEBC3">JNIHandleBlock</a>

### 概要(Summary)
JNI 参照(JNI references)を束ねておくためのコンテナクラス.

JNI 用語で言うと「ローカル参照フレーム」のようなもの
(というか, 実際に JNI の PushLocalFrame()/PopLocalFrame() はこのコンテナが切り替わることで実装されている).
ただし HotSpot 内では, Local references だけでなく, 
Global references や Weak Global references も全て JNIHandleBlock で管理されている.

なお, 1つの JNIHandleBlock には最大で 32 個の JNI 参照しか格納できない (内部構造も参照).
このため Local, Global, Weak Global の各 JNI 参照は, 
それぞれが複数個の JNIHandleBlock によって管理されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.hpp))
    // JNI handle blocks holding local/global JNI handles
    
    class JNIHandleBlock : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* JNIHandles クラスの _global_handles フィールド (static フィールド)

  JNI Global References を束ねておくための JNIHandleBlock.
  
  (正確には, このフィールドは JNIHandleBlock の線形リストを格納するフィールド.
  JNIHandleBlock オブジェクトは _next フィールドで次の JNIHandleBlock オブジェクトを指せる構造になっている.
  この線形リストに JNI Global reference 用の全ての JNIHandleBlock オブジェクトがつながれている)


```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.cpp))
    JNIHandleBlock* JNIHandles::_global_handles       = NULL;
```

* JNIHandles クラスの _weak_global_handles フィールド (static フィールド)

  JNI Weak Global References を束ねておくための JNIHandleBlock.

  (正確には, このフィールドは JNIHandleBlock の線形リストを格納するフィールド.
  JNIHandleBlock オブジェクトは _next フィールドで次の JNIHandleBlock オブジェクトを指せる構造になっている.
  この線形リストに JNI Weak Global reference 用の全ての JNIHandleBlock オブジェクトがつながれている)


```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.cpp))
    JNIHandleBlock* JNIHandles::_weak_global_handles  = NULL;
```

* 各 Thread オブジェクトの _active_handles フィールド

  JNI Local References を束ねておくための JNIHandleBlock.
  
  (正確には, このフィールドは JNIHandleBlock の線形リストを格納するフィールド.
  JNIHandleBlock オブジェクトは _next フィールドで次の JNIHandleBlock オブジェクトを指せる構造になっている.
  この線形リストにそのスレッドの JNI Localreference 用の全ての JNIHandleBlock オブジェクトがつながれている)

  (より正確には, このフィールドは「JNIHandleBlock の線形リストの線形リスト」を格納するフィールド.
  JNIHandleBlock オブジェクトは _pop_frame_link フィールドで次の JNIHandleBlock オブジェクトを指せる構造になっている.
  _next フィールドで構成された線形リストの先頭の JNIHandleBlock オブジェクトは, _pop_frame_link フィールドで次の線形リストの先頭を指す.
  _next フィールドで構成された線形リストが 1つのローカル参照フレームに対応する.
  _pop_frame_link フィールドで構成された「線形リストの線形リスト」は, 
  そのスレッドが PushLocalFrame() で作成した全てのローカル参照フレームに対応する)
  

```cpp
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
      // Active_handles points to a block of handles
      JNIHandleBlock* _active_handles;
```

* 各 Thread オブジェクトの _free_handle_block フィールド

  スレッドローカルなフリーリスト. 
  あるスレッドが確保した JNIHandleBlock は, 不要になっても解放はされず, 代わりにこのスレッドローカルなフリーリストにつながれる.
  (See: JNIHandleBlock::allocate_block(), JNIHandleBlock::release_block())

  (正確には, このフィールドは JNIHandleBlock の線形リストを格納するフィールド(フリーリスト).
  JNIHandleBlock オブジェクトは _next フィールドで次の JNIHandleBlock オブジェクトを指せる構造になっている)


```cpp
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
      // One-element thread local free list
      JNIHandleBlock* _free_handle_block;
```

* JNIHandleBlock クラスの _block_free_list フィールド (static フィールド)

  大域的なフリーリスト (See: JNIHandleBlock::allocate_block(), JNIHandleBlock::release_block())

  (正確には, このフィールドは JNIHandleBlock の線形リストを格納するフィールド(フリーリスト).
  JNIHandleBlock オブジェクトは _next フィールドで次の JNIHandleBlock オブジェクトを指せる構造になっている.
  未使用な JNIHandleBlock オブジェクト(でスレッドローカルなフリーリストにつながれていないもの)は, 全てこの線形リストに格納されている)


```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.hpp))
      static JNIHandleBlock* _block_free_list;      // Free list of currently unused blocks
```

* JNIHandleBlock クラスの _block_list フィールド (static フィールド)

  デバッグ用(開発時用)のフィールド (#ifndef PRODUCT 時にしか定義されない).
  全ての JNIHandleBlock を格納したリスト 

  (格納している JNIHandleBlock オブジェクト自体は, 他の格納箇所と重複).

  (正確には, このフィールドは JNIHandleBlock の線形リストを格納するフィールド.
  JNIHandleBlock オブジェクトは _block_list_link フィールドで次の JNIHandleBlock オブジェクトを指せる構造になっている)

#### 生成箇所(where its instances are created)
JNIHandleBlock::allocate_block() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

* JNIHandles::_global_handles フィールドおよび JNIHandles::_weak_global_handles の初期化処理
  
  * JNIHandles::initialize()

* 各スレッドオブジェクト (のローカル参照フレーム) の初期化処理 

  * Threads::create_vm()
  * JavaThread::run()
  * WatcherThread::run()
  * VMThread::run()
  * ConcurrentMarkSweepThread::run()
  * ConcurrentGCThread::initialize_in_thread()
  * attach_current_thread()

* CompilerThread による JIT コンパイル処理の開始時 (= CompilerThread のローカル参照フレームの初期化処理)
  
  * CompileBroker::push_jni_handle_block()

* JVMTI のイベント通知処理時 (= コールバックの呼び出し処理時)

  * JvmtiEventMark::JvmtiEventMark()

* ランタイムによる Java のメソッドの呼び出し時

  * JavaCallWrapper::JavaCallWrapper()

* JNI の PushLocalFrame() が呼ばれた際
  
  * jni_PushLocalFrame()

* 使用中の JNIHandleBlock に空きがなくなった際

  * JNIHandleBlock::allocate_handle()

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

_handles というフィールドに oop の配列を保持しており, 最大 32 個の oop を入れることができる.
どこまで使っているかは _top というフィールドに保持している.

JNIHandle が 32 個以上になった場合には複数の JNIHandleBlock を使って管理する. 
その場合, _next フィールドで次の JNIHandleBlock へとリスト状につながる.

* oop 	_handles [block_size_in_oops]
  
  JNI 参照 (の参照先) を格納する oop 配列.
  現在は 1つの JNIHandleBlock に最大 32 個の oop を格納することができる.

  (なお, 現在の JNI 参照の実体は「この配列内の対応箇所を指すポインタ」になっている)
 

```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.hpp))
      enum SomeConstants {
        block_size_in_oops  = 32                    // Number of handles per handle block
      };
```

* int 	_top
  
  _handles 配列中で未使用な部分の先頭を示す. 
  _handles は先頭から使っていくため _top 以降の箇所は未使用.

* JNIHandleBlock * 	_next
  
  線形リストを構成するためのポインタ. 次の JNIHandleBlock を指す.
  
  (1つの JNIHandleBlock には最大で 32 個しか JNI 参照を確保できないため, 
  複数個の JNIHandleBlock からなる線形リストで JNI 参照が管理される)

  (また, 未使用の JNIHandleBlock の場合はフリーリストを構成するために使用される)

* JNIHandleBlock * 	_last

  線形リストを構成するためのポインタ. 
  _next による線形リスト中で現在使用している最後の JNIHandleBlock を指す.
  _next でつながるリストは先頭から使用していくため, 新しい JNI 参照の確保は _last から行われる.

* JNIHandleBlock * 	_pop_frame_link
  
  線形リストを構成するためのポインタ. 次の JNIHandleBlock を指す

  (このフィールドは JNI の PushLocalFrame()/PopLocalFrame() のためのもの).

* oop * 	_free_list
  
  _handles 中で delete された箇所を示すフィールド.
  
  (正確には _handles 中の全ての delete 箇所を示すフリーリスト.
  _handles 中の delete された箇所には, このフリーリスト中での次要素を指すポインタが埋め込まれる)
  
  (より正確には, delete された瞬間には JNIHandles::_deleted_handle の値が書き込まれ, 
  その後 JNIHandleBlock::rebuild_free_list() が呼ばれた際に, 全ての delete 箇所が 1本のフリーリストにつながれる)

* int 	_allocate_before_rebuild
  
  JNIHandleBlock::rebuild_free_list() を呼び出すタイミングを制御するフィールド.
  「あと何回 JNI 参照を確保行したら次回の JNIHandleBlock::rebuild_free_list() を呼び出すか」を示す.
  
  (このフィールドは JNIHandleBlock::rebuild_free_list() の処理後に値がセットされる.
  値は JNIHandleBlock::allocate_handle() を呼ぶ度に減少し, 
  0 になったら次の JNIHandleBlock::rebuild_free_list() が呼び出される)

* JNIHandleBlock * 	_block_list_link

  デバッグ用(開発時用)のフィールド (#ifndef PRODUCT 時にしか定義されない).
  
  線形リストを構成するためのポインタ. 
  JNIHandleBlock::_block_list のリストを作るために使用される.


```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.hpp))
      oop             _handles[block_size_in_oops]; // The handles
      int             _top;                         // Index of next unused handle
      JNIHandleBlock* _next;                        // Link to next block
    
      // The following instance variables are only used by the first block in a chain.
      // Having two types of blocks complicates the code and the space overhead in negligble.
      JNIHandleBlock* _last;                        // Last block in use
      JNIHandleBlock* _pop_frame_link;              // Block to restore on PopLocalFrame call
      oop*            _free_list;                   // Handle free list
      int             _allocate_before_rebuild;     // Number of blocks to allocate before rebuilding free list
    
      #ifndef PRODUCT
      JNIHandleBlock* _block_list_link;             // Link for list below
```




### 詳細(Details)
See: [here](../doxygen/classJNIHandleBlock.html) for details

---
## <a name="noniTIZx3V" id="noniTIZx3V">AlwaysAliveClosure</a>

### 概要(Summary)
保守運用機能のためのクラス (Dynamic Attach 機能及びスレッドダンプ機能用のクラス).

JNIHandles クラス内で使用される補助クラス.
JNI の Weak Global Handle を辿る処理で使用される Closure.


```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.cpp))
    class AlwaysAliveClosure: public BoolObjectClosure {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* JNIHandles::print_on()
* JNIHandles::verify()

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

```
* (?? 使用箇所が見当たらない)

  JNIHandles::print()
  -> JNIHandles::print_on()

* 保守運用機能のための処理 (Dynamic Attach 機能及びスレッドダンプ機能用の処理)

  VM_PrintJNI::doit()
  -> JNIHandles::print_on()

* デバッグ用(開発時用)の処理

   Universe::verify()
   -> JNIHandles::verify()
```

### 備考(Notes)
同名のクラスが hotspot/src/share/vm/memory/referenceProcessor.cpp にいたりするが
(そして処理もそっくりだが), 特に関係は無い模様.




### 詳細(Details)
See: [here](../doxygen/classAlwaysAliveClosure.html) for details

---
## <a name="noTN0qv-qB" id="noTN0qv-qB">CountHandleClosure</a>

### 概要(Summary)
保守運用機能のためのクラス (Dynamic Attach 機能及びスレッドダンプ機能用のクラス).

JNIHandles クラス内で使用される補助クラス.
JNI の Weak Global Handle を辿る処理で使用される Closure.


```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.cpp))
    class CountHandleClosure: public OopClosure {
```

### 使われ方(Usage)
JNIHandles::print_on() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
* 保守運用機能のための処理 (Dynamic Attach 機能及びスレッドダンプ機能用の処理)

  VM_PrintJNI::doit()
  -> JNIHandles::print_on()
```

### 備考(Notes)
同名のクラスが hotspot/src/share/vm/memory/referenceProcessor.cpp にいたりするが
(そして処理もそっくりだが), 特に関係は無い模様.




### 詳細(Details)
See: [here](../doxygen/classCountHandleClosure.html) for details

---
## <a name="noTbsSLEbI" id="noTbsSLEbI">VerifyHandleClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

JNIHandles クラス内で使用される補助クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.cpp))
    class VerifyHandleClosure: public OopClosure {
```

### 使われ方(Usage)
JNIHandles::verify() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Universe::verify()
-> JNIHandles::verify()
```

### 内部構造(Internal structure)
処理としては, 単に oopDesc::verify() を呼び出すだけ.


```cpp
    ((cite: hotspot/src/share/vm/runtime/jniHandles.cpp))
      virtual void do_oop(oop* root) {
        (*root)->verify();
      }
```




### 詳細(Details)
See: [here](../doxygen/classVerifyHandleClosure.html) for details

---
