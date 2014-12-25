---
layout: default
title: PSPromotionManager クラス 
---
[Top](../index.html)

#### PSPromotionManager クラス 



---
## <a name="noDi3C6lr7" id="noDi3C6lr7">PSPromotionManager</a>

### 概要(Summary)
ParallelScavengeHeap の Minor GC 処理 
("Parallel Scavenge" 処理) で使用される補助クラス (See: [here](no28916egj.html), [here](no3718vrX.html) and [here](no289165Un.html) for details).

オブジェクトのコピー処理やポインタの再配置処理, 及びそれらを並列化するための機能を提供している.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionManager.hpp))
    class PSPromotionManager : public CHeapObj {
```

なお, 各 PSPromotionManager オブジェクトはそれぞれ１つのスレッドからしか使用されない
(そして, PSPromotionManager オブジェクト内にはスレッドローカルなデータしか含まない).

(誰かデストラクタを書いてくれ, みたいなことが書いてあったりもするが...)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionManager.hpp))
    // psPromotionManager is used by a single thread to manage object survival
    // during a scavenge. The promotion manager contains thread local data only.
    //
    // NOTE! Be carefull when allocating the stacks on cheap. If you are going
    // to use a promotion manager in more than one thread, the stacks MUST be
    // on cheap. This can lead to memory leaks, though, as they are not auto
    // deallocated.
    //
    // FIX ME FIX ME Add a destructor, and don't rely on the user to drain/flush/deallocate!
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PSPromotionManager オブジェクトの _manager_array フィールドに(のみ)格納されている.
(正確には, このフィールドは PSPromotionManager の配列を格納するフィールド.
この中に, 全 GCTaskThread 用の PSPromotionManager オブジェクトが格納されている)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionManager.hpp))
    class PSPromotionManager : public CHeapObj {
    ...
      static PSPromotionManager**         _manager_array;
```

#### 生成箇所(where its instances are created)
PSPromotionManager::initialize() 内で(のみ)生成されている.

#### 参考(for your information): PSPromotionManager::initialize()
See: [here](no31727Vt.html) for details



### 詳細(Details)
See: [here](../doxygen/classPSPromotionManager.html) for details

---
