---
layout: default
title: Parallel Compaction 用の GCTask のサブクラス (ThreadRootsMarkingTask, MarkFromRootsTask, RefProcTaskProxy, RefEnqueueTaskProxy, RefProcTaskExecutor, StealMarkingTask, StealRegionCompactionTask, UpdateDensePrefixTask, DrainStacksCompactionTask)
---
[Top](../index.html)

#### Parallel Compaction 用の GCTask のサブクラス (ThreadRootsMarkingTask, MarkFromRootsTask, RefProcTaskProxy, RefEnqueueTaskProxy, RefProcTaskExecutor, StealMarkingTask, StealRegionCompactionTask, UpdateDensePrefixTask, DrainStacksCompactionTask)

これらは, ParallelScavengeHeap の Parallel Compaction 処理
(UseParallelOldGC オプションが指定されている場合の Major GC 処理)で使用される補助クラス (See: [here](no28916egj.html) and [here](no28916Gft.html) for details).



### クラス一覧(class list)

  * [ThreadRootsMarkingTask](#noQxkK8n9o)
  * [MarkFromRootsTask](#noOo2KclfX)
  * [RefProcTaskProxy](#nohekyO8eV)
  * [RefEnqueueTaskProxy](#noPavmu2x5)
  * [RefProcTaskExecutor](#noHGosEIq8)
  * [StealMarkingTask](#noi2Up-9Dw)
  * [StealRegionCompactionTask](#noEBhzRtad)
  * [UpdateDensePrefixTask](#noGDpDfLYV)
  * [DrainStacksCompactionTask](#no8RItFIpJ)


---
## <a name="noQxkK8n9o" id="noQxkK8n9o">ThreadRootsMarkingTask</a>

### 概要(Summary)
PSParallelCompact::marking_phase() 内で使用される補助クラス(GCTaskクラス).

Parallel Compaction 処理における marking 処理時に,
strong roots から参照されているポインタに mark を付ける
(ただし, このクラスが処理する strong roots は JavaThread と VMThread だけ.
それ以外の strong roots は MarkFromRootsTask で処理する (See: MarkFromRootsTask)).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.hpp))
    // Tasks for parallel compaction of the old generation
    //
    // Tasks are created and enqueued on a task queue. The
    // tasks for parallel old collector for marking objects
    // are MarkFromRootsTask and ThreadRootsMarkingTask.
    //
    // MarkFromRootsTask's are created
    // with a root group (e.g., jni_handles) and when the do_it()
    // method of a MarkFromRootsTask is executed, it starts marking
    // form it's root group.
    //
    // ThreadRootsMarkingTask's are created for each Java thread.  When
    // the do_it() method of a ThreadRootsMarkingTask is executed, it
    // starts marking from the thread's roots.
    //
    // The enqueuing of the MarkFromRootsTask and ThreadRootsMarkingTask
    // do little more than create the task and put it on a queue.  The
    // queue is a GCTaskQueue and threads steal tasks from this GCTaskQueue.
    //
    // In addition to the MarkFromRootsTask and ThreadRootsMarkingTask
    // tasks there are StealMarkingTask tasks.  The StealMarkingTask's
    // steal a reference from the marking stack of another
    // thread and transitively marks the object of the reference
    // and internal references.  After successfully stealing a reference
    // and marking it, the StealMarkingTask drains its marking stack
    // stack before attempting another steal.
    //
    // ThreadRootsMarkingTask
    //
    // This task marks from the roots of a single thread. This task
    // enables marking of thread roots in parallel.
    //
    ...
    
    class ThreadRootsMarkingTask : public GCTask {
```




### 詳細(Details)
See: [here](../doxygen/classThreadRootsMarkingTask.html) for details

---
## <a name="noOo2KclfX" id="noOo2KclfX">MarkFromRootsTask</a>

### 概要(Summary)
PSParallelCompact::marking_phase() 内で使用される補助クラス(GCTaskクラス).

Parallel Compaction 処理における marking 処理時に,
strong roots から参照されているポインタに mark を付ける
(ただし, JavaThread と VMThread だけは ThreadRootsMarkingTask で処理する.
このクラスはそれ以外の strong roots 用 (See: ThreadRootsMarkingTask)).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.hpp))
    //
    // MarkFromRootsTask
    //
    // This task marks from all the roots to all live
    // objects.
    //
    //
    
    class MarkFromRootsTask : public GCTask {
```




### 詳細(Details)
See: [here](../doxygen/classMarkFromRootsTask.html) for details

---
## <a name="nohekyO8eV" id="nohekyO8eV">RefProcTaskProxy</a>

### 概要(Summary)
PSParallelCompact::marking_phase() 内で使用される補助クラス(GCTaskクラス).
(より正確には, そこから呼び出される RefProcTaskExecutor::execute(ProcessTask& task) 内で使用される補助クラス).

Parallel Compaction 処理における marking 処理時に,
参照オブジェクト(java.lang.ref オブジェクト)の mark 処理をマルチスレッド化するための補助クラス
(より具体的に言うと, コンストラクタ引数で渡された
AbstractRefProcTaskExecutor::ProcessTask オブジェクトを実行するためのクラス
(See: AbstractRefProcTaskExecutor::ProcessTask))


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.hpp))
    //
    // RefProcTaskProxy
    //
    // This task is used as a proxy to parallel reference processing tasks .
    //
    
    class RefProcTaskProxy : public GCTask {
```

### 内部構造(Internal structure)
参照オブジェクト処理のマルチスレッド化自体は RefProcTaskExecutor クラスが行っている.

各 RefProcTaskProxy オブジェクトには
RefProcTaskExecutor クラスによって
実行すべき AbstractRefProcTaskExecutor::ProcessTask オブジェクトが 1つ割り当てられるので, 単にそれを実行するだけ.

#### 参考(for your information): RefProcTaskProxy::do_it()
See: [here](no28916OMU.html) for details



### 詳細(Details)
See: [here](../doxygen/classRefProcTaskProxy.html) for details

---
## <a name="noPavmu2x5" id="noPavmu2x5">RefEnqueueTaskProxy</a>

### 概要(Summary)
??

...#TODO 内で使用される補助クラス(GCTaskクラス).
(より正確には, そこから呼び出される RefProcTaskExecutor::execute(EnqueueTask& task) 内で使用される補助クラス).

Parallel Compaction 処理における marking 処理時に,
参照オブジェクト(java.lang.ref オブジェクト)の mark 処理をマルチスレッド化するための補助クラス
(より具体的に言うと, コンストラクタ引数で渡された
AbstractRefProcTaskExecutor::EnqueueTask オブジェクトを実行するためのクラス
(See: AbstractRefProcTaskExecutor::ProcessTask))


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.hpp))
    //
    // RefEnqueueTaskProxy
    //
    // This task is used as a proxy to parallel reference processing tasks .
    //
    
    class RefEnqueueTaskProxy: public GCTask {
```

### 使われ方(Usage)
RefProcTaskExecutor::execute(EnqueueTask& task) 内で(のみ)使用されている.

が, この関数自体がどこからも使われていないような... #TODO

(ReferenceProcessor::enqueue_discovered_references() 自体は
PSParallelCompact::post_compact() 内で呼び出されているようだが, この関数は使われていない...)




### 詳細(Details)
See: [here](../doxygen/classRefEnqueueTaskProxy.html) for details

---
## <a name="noHGosEIq8" id="noHGosEIq8">RefProcTaskExecutor</a>

### 概要(Summary)
PSParallelCompact::marking_phase() 内で使用される補助クラス.

Parallel Compaction 処理で使用される AbstractRefProcTaskExecutor クラス
(つまり, Parallel Compaction 処理における参照オブジェクト(java.lang.ref オブジェクト)の処理をマルチスレッド化するためのクラス
 (See: AbstractRefProcTaskExecutor)).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.hpp))
    //
    // RefProcTaskExecutor
    //
    // Task executor is an interface for the reference processor to run
    // tasks using GCTaskManager.
    //
    
    class RefProcTaskExecutor: public AbstractRefProcTaskExecutor {
```




### 詳細(Details)
See: [here](../doxygen/classRefProcTaskExecutor.html) for details

---
## <a name="noi2Up-9Dw" id="noi2Up-9Dw">StealMarkingTask</a>

### 概要(Summary)
PSParallelCompact::marking_phase() 内で使用される補助クラス(GCTaskクラス).
(より正確には, PSParallelCompact::marking_phase() 内と,
そこから呼び出される RefProcTaskExecutor::execute(ProcessTask& task) 内で使用される補助クラス).

他の GCTaskThread の担当になっている mark 処理を奪ってきて実行するという GCTask.
キューの最後にこの GCTask を ParallelGCThreads 分だけ入れておくことで,
先に処理が終わった GCTaskThread が他のスレッドの仕事を奪って負荷分散するようになる
(この GCTask 自体は, 全ての GCTaskThread に仕事がなくなった時に終了する).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.hpp))
    //
    // StealMarkingTask
    //
    // This task is used to distribute work to idle threads.
    //
    
    class StealMarkingTask : public GCTask {
```




### 詳細(Details)
See: [here](../doxygen/classStealMarkingTask.html) for details

---
## <a name="noEBhzRtad" id="noEBhzRtad">StealRegionCompactionTask</a>

### 概要(Summary)
PSParallelCompact::compact() 内で使用される補助クラス(GCTaskクラス).
(より正確には, そこから呼び出される PSParallelCompact::enqueue_region_stealing_tasks() 内で使用される補助クラス).

他の GCTaskThread の担当になっているコンパクション処理を奪ってきて実行するという GCTask.
キューの最後にこの GCTask を ParallelGCThreads 分だけ入れておくことで,
先に処理が終わった GCTaskThread が他のスレッドの仕事を奪って負荷分散するようになる
(この GCTask 自体は, 全ての GCTaskThread に仕事がなくなった時に終了する).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.hpp))
    //
    // StealRegionCompactionTask
    //
    // This task is used to distribute work to idle threads.
    //
    
    class StealRegionCompactionTask : public GCTask {
```




### 詳細(Details)
See: [here](../doxygen/classStealRegionCompactionTask.html) for details

---
## <a name="noGDpDfLYV" id="noGDpDfLYV">UpdateDensePrefixTask</a>

### 概要(Summary)
PSParallelCompact::compact() 内で使用される補助クラス(GCTaskクラス).
(より正確には, そこから呼び出される PSParallelCompact::enqueue_dense_prefix_tasks() 内で使用される補助クラス).

Parallel Compaction 処理におけるコンパクション処理時に, dense prefix 部分の処理を行う
(live オブジェクト内のポインタの修正処理およびdead オブジェクトをダミーオブジェクトで上書きする処理).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.hpp))
    //
    // UpdateDensePrefixTask
    //
    // This task is used to update the dense prefix
    // of a space.
    //
    
    class UpdateDensePrefixTask : public GCTask {
```




### 詳細(Details)
See: [here](../doxygen/classUpdateDensePrefixTask.html) for details

---
## <a name="no8RItFIpJ" id="no8RItFIpJ">DrainStacksCompactionTask</a>

### 概要(Summary)
PSParallelCompact::compact() 内で使用される補助クラス(GCTaskクラス).
(より正確には, そこから呼び出される PSParallelCompact::enqueue_region_draining_tasks() 内で使用される補助クラス).

Parallel Compaction 処理におけるコンパクション処理時に,
live object を新しいアドレスに移動させ, それらの中にあるポインタを新しいアドレスに修正する処理を行う.

(ただし, dense prefix 部分については UpdateDensePrefixTask で処理する.
このクラスはそれ以外の部分用 (See: UpdateDensePrefixTask))


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.hpp))
    //
    // DrainStacksCompactionTask
    //
    // This task processes regions that have been added to the stacks of each
    // compaction manager.
    //
    // Trying to use one draining thread does not work because there are no
    // guarantees about which task will be picked up by which thread.  For example,
    // if thread A gets all the preloaded regions, thread A may not get a draining
    // task (they may all be done by other threads).
    //
    
    class DrainStacksCompactionTask : public GCTask {
```




### 詳細(Details)
See: [here](../doxygen/classDrainStacksCompactionTask.html) for details

---
