---
layout: default
title: ResourceArea クラス関連のクラス (ResourceArea, ResourceMark, DeoptResourceMark)
---
[Top](../index.html)

#### ResourceArea クラス関連のクラス (ResourceArea, ResourceMark, DeoptResourceMark)

これらは, メモリ管理用のクラス.
より具体的に言うと, ResourceObj 用のメモリ領域を管理するためのクラス (See: [here](no28916VHS.html) for details).


### クラス一覧(class list)

  * [ResourceArea](#noJpQYu2DA)
  * [ResourceMark](#noLONQwF_I)
  * [DeoptResourceMark](#nojFGEEUyi)


---
## <a name="noJpQYu2DA" id="noJpQYu2DA">ResourceArea</a>

### 概要(Summary)
ResourceObj 用のメモリ領域を管理するクラス.
ResourceObj (のサブクラス) のオブジェクトはこのクラスが管理するメモリ領域内に確保される (See: [here](no28916VHS.html) for details).

なお, このクラス自身は Arena のサブクラスとして定義されている (See: Arena).


```cpp
    ((cite: hotspot/src/share/vm/memory/resourceArea.hpp))
    //------------------------------ResourceArea-----------------------------------
    // A ResourceArea is an Arena that supports safe usage of ResourceMark.
    class ResourceArea: public Arena {
```




### 詳細(Details)
See: [here](../doxygen/classResourceArea.html) for details

---
## <a name="noLONQwF_I" id="noLONQwF_I">ResourceMark</a>

### 概要(Summary)
ResourceArea クラス用のユーティリティ・クラス(StackObjクラス) (See: [here](no28916VHS.html) for details).

ResourceObj オブジェクトの確保/開放をソースコード上のスコープに合わせて自動で行うことができる.
(参考: [Region-based memory management](http://en.wikipedia.org/wiki/Region-based_memory_management)).


```cpp
    ((cite: hotspot/src/share/vm/memory/resourceArea.hpp))
    //------------------------------ResourceMark-----------------------------------
    // A resource mark releases all resources allocated after it was constructed
    // when the destructor is called.  Typically used as a local variable.
    class ResourceMark: public StackObj {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
コンストラクタでカレントスレッドの ResourceArea の状態 (Chunk 等) を記録しておき, デストラクタで記録していた状態に戻すだけ.

#### 参考(for your information): ResourceMark::ResourceMark()
ResourceMark::initialize() を呼び出すだけ.


```cpp
    ((cite: hotspot/src/share/vm/memory/resourceArea.hpp))
      ResourceMark()               { initialize(Thread::current()); }
```

#### 参考(for your information): ResourceMark::initialize()
See: [here](no28916jZT.html) for details
#### 参考(for your information): ResourceMark::~ResourceMark()
See: [here](no31977Boa.html) for details
#### 参考(for your information): ResourceMark::reset_to_mark()
See: [here](no28916wjZ.html) for details



### 詳細(Details)
See: [here](../doxygen/classResourceMark.html) for details

---
## <a name="nojFGEEUyi" id="nojFGEEUyi">DeoptResourceMark</a>

### 概要(Summary)
ResourceMark の類似品. Deoptimization 処理の中で使用される (See: Deoptimization).

使用する際には, 普通の ResourceMark オブジェクトのようにスタック上に配置するのではなく, 
明示的に new/delete で確保／解放する. 

なおコメントによると, このクラスが作られたのには以下のような背景があるとのこと.

* ResourceMark オブジェクトは StackObj クラスのサブクラスであるため, 
  使う際には「スタック上に確保しておくことで, そのスコープの終了時にリソースを自動解放させる」という使い方になる.
  しかし deoptimization 処理では, 処理の途中でスタックフレームそのものを破棄する作業がはいるため, これでは上手くいかない.
  
* これに対して, DeoptResourceMark は CHeapObj のサブクラスとなっているため, 明示的に確保解放できる.

* なお, 普通の ResoruceMark クラスを CHeapObj のサブクラスにしてもよかったが, 
  そうすると (普通はスタック上に配置すべきである) ResoruceMark を誤用する危険が考えられたので, 
  CHeapObj のサブクラス版である DeoptResourceMark を改めて作ることにした.


```cpp
    ((cite: hotspot/src/share/vm/memory/resourceArea.hpp))
    //------------------------------DeoptResourceMark-----------------------------------
    // A deopt resource mark releases all resources allocated after it was constructed
    // when the destructor is called.  Typically used as a local variable. It differs
    // from a typical resource more in that it is C-Heap allocated so that deoptimization
    // can use data structures that are arena based but are not amenable to vanilla
    // ResourceMarks because deoptimization can not use a stack allocated mark. During
    // deoptimization we go thru the following steps:
    //
    // 0: start in assembly stub and call either uncommon_trap/fetch_unroll_info
    // 1: create the vframeArray (contains pointers to Resource allocated structures)
    //   This allocates the DeoptResourceMark.
    // 2: return to assembly stub and remove stub frame and deoptee frame and create
    //    the new skeletal frames.
    // 3: push new stub frame and call unpack_frames
    // 4: retrieve information from the vframeArray to populate the skeletal frames
    // 5: release the DeoptResourceMark
    // 6: return to stub and eventually to interpreter
    //
    // With old style eager deoptimization the vframeArray was created by the vmThread there
    // was no way for the vframeArray to contain resource allocated objects and so
    // a complex set of data structures to simulate an array of vframes in CHeap memory
    // was used. With new style lazy deoptimization the vframeArray is created in the
    // the thread that will use it and we can use a much simpler scheme for the vframeArray
    // leveraging existing data structures if we simply create a way to manage this one
    // special need for a ResourceMark. If ResourceMark simply inherited from CHeapObj
    // then existing ResourceMarks would work fine since no one use new to allocate them
    // and they would be stack allocated. This leaves open the possibilty of accidental
    // misuse so we simple duplicate the ResourceMark functionality here.
    
    class DeoptResourceMark: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JavaThread オブジェクトの _deopt_mark フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    class JavaThread: public Thread {
    ...
      // Deopt support
      DeoptResourceMark*  _deopt_mark;               // Holds special ResourceMark for deoptimization
```

#### 生成箇所(where its instances are created)
Deoptimization::fetch_unroll_info_helper() 内で(のみ)生成されている.

(この時点で JavaThread::_deopt_mark フィールドにセットされる)


```cpp
    ((cite: hotspot/src/share/vm/runtime/deoptimization.cpp))
    Deoptimization::UnrollBlock* Deoptimization::fetch_unroll_info_helper(JavaThread* thread) {
    ...
      // Allocate our special deoptimization ResourceMark
      DeoptResourceMark* dmark = new DeoptResourceMark(thread);
      assert(thread->deopt_mark() == NULL, "Pending deopt!");
      thread->set_deopt_mark(dmark);
```

#### 削除箇所(where its instances are deleted)
Deoptimization::cleanup_deopt_info() 内で削除されている.

(この時点で JavaThread::_deopt_mark フィールドも NULL に戻される)


```cpp
    ((cite: hotspot/src/share/vm/runtime/deoptimization.cpp))
    void Deoptimization::cleanup_deopt_info(JavaThread *thread,
                                            vframeArray *array) {
    ...
      // Deallocate any resource creating in this routine and any ResourceObjs allocated
      // inside the vframeArray (StackValueCollections)
    
      delete thread->deopt_mark();
      thread->set_deopt_mark(NULL);
      thread->set_deopt_nmethod(NULL);
```


### 内部構造(Internal structure)
機能としては ResourceMark と全く同じ (本当に C ヒープかスタック上かというだけの違い).

#### 参考(for your information): DeoptResourceMark::initialize()
See: [here](no28916WPN.html) for details
#### 参考(for your information): DeoptResourceMark::reset_to_mark()
See: [here](no28916JFH.html) for details



### 詳細(Details)
See: [here](../doxygen/classDeoptResourceMark.html) for details

---
