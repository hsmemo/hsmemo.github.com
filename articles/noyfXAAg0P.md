---
layout: default
title: ThreadCritical クラス 
---
[Top](../index.html)

#### ThreadCritical クラス 



---
## <a name="no5zP9qNSY" id="no5zP9qNSY">ThreadCritical</a>

### 概要(Summary)
スレッド間の排他処理を行うためのユーティリティ・クラス.

ThreadCritical クラスは, 対応する mutex を大域に 1つだけ保持しており, 
その mutex に対する排他処理(ロックの確保／開放処理)をソースコード上のスコープに合わせて行うことができるクラス(StackObjクラス).

なお, スレッド間の排他処理を行うためのクラスは幾つか存在している (See: [here](noFBHxUsnN.html) for details).
その中でも ThreadCritical は HotSpot の初期化作業のかなり早い段階でも使われるクラスなので, 
HotSpot 内の他の機構には頼らない実装になっている (See: [here](no3420_c0.html) for details).

なお, このクラスによるロックは reentrant である (= 同一スレッドが複数回ロックを取得できる).


```cpp
    ((cite: hotspot/src/share/vm/runtime/threadCritical.hpp))
    // ThreadCritical is used to protect short non-blocking critical sections.
    // This class must use no vm facilities that require initialization.
    // It is used very early in the vm's initialization, in allocation
    // code and other areas. ThreadCritical regions are reentrant.
    //
    // Due to race conditions during vm exit, some of the os level
    // synchronization primitives may not be deallocated at exit. It
    // is a good plan to implement the platform dependent sections of
    // code with resources that are recoverable during process
    // cleanup by the os. Calling the initialize method before use
    // is also problematic, it is best to use preinitialized primitives
    // if possible. As an example:
    //
    // mutex_t  mp  =  DEFAULTMUTEX;
    //
    // Also note that this class is declared as a StackObj to enforce
    // block structured short locks. It cannot be declared a ResourceObj
    // or CHeapObj, due to initialization issues.
    
    class ThreadCritical : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* ChunkPool の確保処理／解放処理
    * ChunkPool::allocate()
    * ChunkPool::free()
    * ChunkPool::free_all_but()
* PSScavenge::oop_promotion_failed()
* LazyClassPathEntry::resolve_entry()
* Thread::trace()
* IdealGraphPrinter::IdealGraphPrinter()




### 詳細(Details)
See: [here](../doxygen/classThreadCritical.html) for details

---
