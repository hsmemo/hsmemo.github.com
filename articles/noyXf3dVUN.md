---
layout: default
title: Parallel Scavenge 用の GCTask のサブクラス (ScavengeRootsTask, ThreadRootsTask, StealTask, SerialOldToYoungRootsTask, OldToYoungRootsTask)
---
[Top](../index.html)

#### Parallel Scavenge 用の GCTask のサブクラス (ScavengeRootsTask, ThreadRootsTask, StealTask, SerialOldToYoungRootsTask, OldToYoungRootsTask)

これらは, ParallelScavengeHeap の Minor GC 処理
("Parallel Scavenge" 処理) で使用される補助クラス (See: [here](no28916egj.html) and [here](no289165Un.html) for details).


### クラス一覧(class list)

  * [ScavengeRootsTask](#noEsYxc41q)
  * [ThreadRootsTask](#no72HfF5do)
  * [StealTask](#nojYSO79fk)
  * [SerialOldToYoungRootsTask](#noIIZTpqFx)
  * [OldToYoungRootsTask](#no9rKf5wxb)


---
## <a name="noEsYxc41q" id="noEsYxc41q">ScavengeRootsTask</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス(GCTaskクラス).

strong roots から参照されているポインタに対して Scavenge 処理を行う
(ただし, JavaThread と VMThread だけは ThreadRootsTask で処理する. 
このクラスはそれ以外の strong roots 用 (See: ThreadRootsTask)).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psTasks.hpp))
    // ScavengeRootsTask
    //
    // This task scans all the roots of a given type.
    //
    //
    
    class ScavengeRootsTask : public GCTask {
```

### 使われ方(Usage)
PSScavenge::invoke_no_policy() 内で(のみ)使用されている (See: [here](no289165Un.html) for details).

### 内部構造(Internal structure)
このクラスが処理する strong roots には以下のような種別が存在する (要は「スレッドのスタック内以外の全て」ということだが...).

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psTasks.hpp))
      enum RootType {
        universe              = 1,
        jni_handles           = 2,
        threads               = 3,
        object_synchronizer   = 4,
        flat_profiler         = 5,
        system_dictionary     = 6,
        management            = 7,
        jvmti                 = 8,
        code_cache            = 9
      };
```




### 詳細(Details)
See: [here](../doxygen/classScavengeRootsTask.html) for details

---
## <a name="no72HfF5do" id="no72HfF5do">ThreadRootsTask</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス(GCTaskクラス).

strong roots から参照されているポインタに mark を付ける
(ただし, このクラスが処理する strong roots は JavaThread と VMThread だけ.
それ以外の strong roots は ScavengeRootsTask で処理する (See: ScavengeRootsTask)).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psTasks.hpp))
    // ThreadRootsTask
    //
    // This task scans the roots of a single thread. This task
    // enables scanning of thread roots in parallel.
    //
    
    class ThreadRootsTask : public GCTask {
```

### 使われ方(Usage)
PSScavenge::invoke_no_policy() 内で(のみ)使用されている (See: [here](no289165Un.html) for details).




### 詳細(Details)
See: [here](../doxygen/classThreadRootsTask.html) for details

---
## <a name="nojYSO79fk" id="nojYSO79fk">StealTask</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス(GCTaskクラス).

他の GCTaskThread の担当になっている scavenge 処理を奪ってきて実行するという GCTask.
キューの最後にこの GCTask を ParallelGCThreads 分だけ入れておくことで, 
先に処理が終わった GCTaskThread が他のスレッドの仕事を奪って負荷分散するようになる
(この GCTask 自体は, 全ての GCTaskThread に仕事がなくなった時に終了する).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psTasks.hpp))
    // StealTask
    //
    // This task is used to distribute work to idle threads.
    //
    
    class StealTask : public GCTask {
```

### 使われ方(Usage)
PSScavenge::invoke_no_policy() 内で(のみ)使用されている
(より正確には, PSScavenge::invoke_no_policy() 内と,
そこから呼び出される PSRefProcTaskExecutor::execute(ProcessTask& task) 内で使用される補助クラス)
(See: [here](no289165Un.html) for details).




### 詳細(Details)
See: [here](../doxygen/classStealTask.html) for details

---
## <a name="noIIZTpqFx" id="noIIZTpqFx">SerialOldToYoungRootsTask</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス(GCTaskクラス).

Perm 領域から Young 領域を指しているポインタの Scavenge 処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psTasks.hpp))
    // SerialOldToYoungRootsTask
    //
    // This task is used to scan for roots in the perm gen
    
    class SerialOldToYoungRootsTask : public GCTask {
```

### 使われ方(Usage)
PSScavenge::invoke_no_policy() 内で(のみ)使用されている (See: [here](no289165Un.html) for details).




### 詳細(Details)
See: [here](../doxygen/classSerialOldToYoungRootsTask.html) for details

---
## <a name="no9rKf5wxb" id="no9rKf5wxb">OldToYoungRootsTask</a>

ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス(GCTaskクラス).

Old 領域から Young 領域を指しているポインタの Scavenge 処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psTasks.hpp))
    // OldToYoungRootsTask
    //
    // This task is used to scan old to young roots in parallel
    
    class OldToYoungRootsTask : public GCTask {
```

### 使われ方(Usage)
PSScavenge::invoke_no_policy() 内で(のみ)使用されている (See: [here](no289165Un.html) for details).

### 内部構造(Internal structure)
Old 領域は量が多いため, 
stripe_number という数字で Old 領域内を複数に分割し, 
複数の GCTaskThread で並列処理できるようにしている.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psTasks.hpp))
      OldToYoungRootsTask(PSOldGen *gen, HeapWord* gen_top, uint stripe_number) :
        _gen(gen), _gen_top(gen_top), _stripe_number(stripe_number) { }
```




### 詳細(Details)
See: [here](../doxygen/classOldToYoungRootsTask.html) for details

---
