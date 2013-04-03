---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.cpp

### 名前(function name)
```
void PSMarkSweep::mark_sweep_phase3() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この関数では, 各ポインタをコンパクション後の新しいアドレスに書き換える処理を行う)
      ---------------------------------------- -}

	  // Adjust the pointers to reflect the new locations

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EventMark m("3 adjust pointers");
	  TraceTime tm("phase 3", PrintGCDetails && Verbose, true, gclog_or_tty);
	  trace("3");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ParallelScavengeHeap* heap = (ParallelScavengeHeap*)Universe::heap();
	  assert(heap->kind() == CollectedHeap::ParallelScavengeHeap, "Sanity");
	
	  PSYoungGen* young_gen = heap->young_gen();
	  PSOldGen* old_gen = heap->old_gen();
	  PSPermGen* perm_gen = heap->perm_gen();
	
  {- -------------------------------------------
  (1) まず, strong root 内に格納されているポインタの修正処理を行う.
    
      (なお, JNIHandles::weak_oops_do() については, 
       phase1 の中で ReferenceProcessor::process_discovered_references() が呼び出された段階で
       死んでいる Weak 参照については NULL になっている.
       このため, 今回は引数として PSAlwaysTrueClosure を使っているが, 
       これでも生きている Handle だけを全て辿ることが可能)
      ---------------------------------------- -}

	  // General strong roots.
	  Universe::oops_do(adjust_root_pointer_closure());
	  ReferenceProcessor::oops_do(adjust_root_pointer_closure());
	  JNIHandles::oops_do(adjust_root_pointer_closure());   // Global (strong) JNI handles
	  Threads::oops_do(adjust_root_pointer_closure(), NULL);
	  ObjectSynchronizer::oops_do(adjust_root_pointer_closure());
	  FlatProfiler::oops_do(adjust_root_pointer_closure());
	  Management::oops_do(adjust_root_pointer_closure());
	  JvmtiExport::oops_do(adjust_root_pointer_closure());
	  // SO_AllClasses
	  SystemDictionary::oops_do(adjust_root_pointer_closure());
	  //CodeCache::scavenge_root_nmethods_oops_do(adjust_root_pointer_closure());
	
	  // Now adjust pointers in remaining weak roots.  (All of which should
	  // have been cleared if they pointed to non-surviving objects.)
	  // Global (weak) JNI handles
	  JNIHandles::weak_oops_do(&always_true, adjust_root_pointer_closure());
	
	  CodeCache::oops_do(adjust_pointer_closure());
	  StringTable::oops_do(adjust_root_pointer_closure());
	  ref_processor()->weak_oops_do(adjust_root_pointer_closure());
	  PSScavenge::reference_processor()->weak_oops_do(adjust_root_pointer_closure());
	
  {- -------------------------------------------
  (1) 次に, MarkSweep::adjust_marks() を呼んで, PreservedMark 内のポインタを修正する.
      (See: PreservedMark)
      ---------------------------------------- -}

	  adjust_marks();
	
  {- -------------------------------------------
  (1) 最後に, 以下の関数を呼んで, 
      それぞれのヒープ領域について, その領域内の live オブジェクト中のポインタを修正する.
      * PSYoungGen::adjust_pointers() (New 領域用)
      * PSOldGen::adjust_pointers()   (Old 領域用)
      * PSOldGen::adjust_pointers()   (Perm 領域用)
      ---------------------------------------- -}

	  young_gen->adjust_pointers();
	  old_gen->adjust_pointers();
	  perm_gen->adjust_pointers();
	}
	
```


