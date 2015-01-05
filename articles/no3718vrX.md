---
layout: default
title: Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： ParallelScavengeHeap の場合
---
[Up](noQ2dTyo8F.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： ParallelScavengeHeap の場合

--- 
## 概要(Summary)
メモリ確保の試みが全て失敗した場合, 最終的に CollectedHeap::mem_allocate() が呼び出される (See: [here](no28916Q0G.html) for details).
ParallelScavengeHeap は CollectedHeap::mem_allocate() をオーバーライドしているので, 
実際に呼び出されるのは ParallelScavengeHeap::mem_allocate() になる.

ParallelScavengeHeap::mem_allocate() 内では GC を実行してでも確保が試みられる.
なお, ParallelScavengeHeap::mem_allocate() には2種類の allocation policy が存在している (basic allocation policy, failed allocation policy).
これは, VM Operation が重い処理なので, 可能な限り VM Operation の回数を減らすための工夫である.

  * basic allocation policy は, safepoint 停止を必要としない手順は全て使用して確保を試みる.
    (lock を取ってもいいし, heap を expand してもいい)

  * failed allocation policy では, basic allocation policy では確保できなかった際に, safepoint 停止させて行われる. 
    GC やヒープの拡張を行い, どうしてもダメなら OutOfMemory 処理などを行う.
    (なお, failed allocation policy は VM Thread から呼び出される)

なお, 複数のスレッドが failed allocation policy に来た場合にも, 
VM Thread によって実際に実行される VM Operation は1回だけになる.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.cpp))
    // There are two levels of allocation policy here.
    //
    // When an allocation request fails, the requesting thread must invoke a VM
    // operation, transfer control to the VM thread, and await the results of a
    // garbage collection. That is quite expensive, and we should avoid doing it
    // multiple times if possible.
    //
    // To accomplish this, we have a basic allocation policy, and also a
    // failed allocation policy.
    //
    // The basic allocation policy controls how you allocate memory without
    // attempting garbage collection. It is okay to grab locks and
    // expand the heap, if that can be done without coming to a safepoint.
    // It is likely that the basic allocation policy will not be very
    // aggressive.
    //
    // The failed allocation policy is invoked from the VM thread after
    // the basic allocation policy is unable to satisfy a mem_allocate
    // request. This policy needs to cover the entire range of collection,
    // heap expansion, and out-of-memory conditions. It should make every
    // attempt to allocate the requested memory.
    
    // Basic allocation policy. Should never be called at a safepoint, or
    // from the VM thread.
    //
    // This method must handle cases where many mem_allocate requests fail
    // simultaneously. When that happens, only one VM operation will succeed,
    // and the rest will not be executed. For that reason, this method loops
    // during failed allocation attempts. If the java heap becomes exhausted,
    // we rely on the size_policy object to force a bail out.
```

## 備考(Notes)
ParallelScavengeHeap::mem_allocate() の処理の流れは
GenCollectorPolicy::mem_allocate_work() に似ている.


## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
ParallelScavengeHeap::mem_allocate()
-&gt; (1) まず PSYoungGen::allocate() で確保を試みる. (この確保が成功すれば以下の処理は行わない)
       -&gt; PSYoungGen::allocate()
          -&gt; (See: <a href="no28916rXC.html">here</a> for details)

   (2) 確保に成功するまで以下の処理を繰り返す.
       (メモリ確保に成功するか, あるいは成功しないと判断するまでループ)

       -&gt; (1) Heap_lock を取った状態で PSYoungGen::allocate() 及び PSOldGen::allocate() による確保処理を行う
              -&gt; PSYoungGen::allocate()
                 -&gt; (See: <a href="no28916rXC.html">here</a> for details)
              -&gt; PSOldGen::allocate()  (&lt;= Young Gen にいれるには大きすぎる確保要求の場合)
                 -&gt; PSOldGen::allocate_noexpand()
                 -&gt; PSOldGen::expand_and_allocate()
    
          (2) まだ確保できていなければ, GC を実行してから確保を試みる.
              -&gt; VM_ParallelGCFailedAllocation::doit()
                 -&gt; ParallelScavengeHeap::failed_mem_allocate()
    
                    -&gt; (1) まず PSScavenge::invoke() で Minor GC を行い, その後 Young Generation から確保してみる.
                           -&gt; PSScavenge::invoke()
                              -&gt; (See: <a href="no289165Un.html">here</a> for details)
                           -&gt; PSYoungGen::allocate()
                              -&gt; (See: <a href="no28916rXC.html">here</a> for details)
       
                       (2) 確保に失敗したら, ParallelScavengeHeap::invoke_full_gc() で簡単に Full GC を行った後, もう一度 Young Generation からの確保を試みる.
                           -&gt; ParallelScavengeHeap::invoke_full_gc()
                              (UseParallelOldGC オプションに応じて, 以下のどちらかを呼び出す)
                              -&gt; PSParallelCompact::invoke()
                                 -&gt; (See: <a href="no28916Gft.html">here</a> for details)
                              -&gt; PSMarkSweep::invoke()
                                 -&gt; (See: <a href="no2114YqK.html">here</a> for details)
                           -&gt; PSYoungGen::allocate()
                              -&gt; (See: <a href="no28916rXC.html">here</a> for details)
       
                       (3) まだ確保が成功していなければ, Old Generation から確保してみる.
                           -&gt; PSOldGen::allocate()
                              -&gt; (同上)
       
                       (4) まだ確保が成功していなければ, invoke_full_gc() で(今度は完璧に) Full GC を行い, その後 Young Generation からの確保を試みる.
                           -&gt; ParallelScavengeHeap::invoke_full_gc()
                              -&gt; (同上)
                           -&gt; PSYoungGen::allocate()
                              -&gt; (See: <a href="no28916rXC.html">here</a> for details)
       
                       (5) それでもまだ確保が成功していなければ, もう一度 Old Generation から確保を試みる.
                           (今度は完璧に Full GC した後なので, さっきとは結果が変わる可能性がある)
                           -&gt; PSOldGen::allocate()
                              -&gt; (同上)

          (3) GC 時間が制限を越えていれば NULL をリターン. そうでなければ, 確保が成功するまでループを繰り返す.
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### ParallelScavengeHeap::mem_allocate()
See: [here](no2480uaS.html) for details
### PSOldGen::allocate()
See: [here](no2480V5k.html) for details
### PSOldGen::allocate_noexpand()
(#Under Construction)

### PSOldGen::expand_and_allocate()
(#Under Construction)

### VM_ParallelGCFailedAllocation::doit()
See: [here](no2480iDr.html) for details
### ParallelScavengeHeap::failed_mem_allocate()
See: [here](no2480vNx.html) for details
### ParallelScavengeHeap::invoke_full_gc()
See: [here](no2480uhG.html) for details



## Subcategories
* [Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： ParallelScavengeHeap の場合 ： GC 処理の並列化の方針について  ](no28916egj.html)
* [Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： ParallelScavengeHeap の場合 ： Minor GC の処理](no289165Un.html)
* [Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： ParallelScavengeHeap の場合 ： Major GC の処理 (UseParallelOldGC 有効時)](no28916Gft.html)
* [Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： ParallelScavengeHeap の場合 ： Major GC の処理 (UseParallelOldGC 無効時)](no2114YqK.html)



