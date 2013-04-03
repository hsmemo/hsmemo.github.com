---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.cpp

### 名前(function name)
```
void G1MarkSweep::mark_sweep_phase1(bool& marked_for_unloading,
                                    bool clear_all_softrefs) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  // Recursively traverse all live objects and mark them
	  EventMark m("1 mark object");
	  TraceTime tm("phase 1", PrintGC && Verbose, true, gclog_or_tty);
	  GenMarkSweep::trace(" 1");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  SharedHeap* sh = SharedHeap::heap();
	
  {- -------------------------------------------
  (1) まず, SharedHeap::process_strong_roots() を呼び出して root から参照されているオブジェクトを全て処理する.
      ---------------------------------------- -}

	  sh->process_strong_roots(true,  // activeate StrongRootsScope
	                           true,  // Collecting permanent generation.
	                           SharedHeap::SO_SystemClasses,
	                           &GenMarkSweep::follow_root_closure,
	                           &GenMarkSweep::follow_code_root_closure,
	                           &GenMarkSweep::follow_root_closure);
	
  {- -------------------------------------------
  (1) 次に, ReferenceProcessor::process_discovered_references() を呼び出して, 
      上の処理中に発見された参照オブジェクト(java.lang.ref オブジェクト) の処理を行う.
      ---------------------------------------- -}

	  // Process reference objects found during marking
	  ReferenceProcessor* rp = GenMarkSweep::ref_processor();
	  rp->setup_policy(clear_all_softrefs);
	  rp->process_discovered_references(&GenMarkSweep::is_alive,
	                                    &GenMarkSweep::keep_alive,
	                                    &GenMarkSweep::follow_stack_closure,
	                                    NULL);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Follow system dictionary roots and unload classes
	  bool purged_class = SystemDictionary::do_unloading(&GenMarkSweep::is_alive);
	  assert(GenMarkSweep::_marking_stack.is_empty(),
	         "stack should be empty by now");
	
	  // Follow code cache roots (has to be done after system dictionary,
	  // assumes all live klasses are marked)
	  CodeCache::do_unloading(&GenMarkSweep::is_alive,
	                                   &GenMarkSweep::keep_alive,
	                                   purged_class);
	  GenMarkSweep::follow_stack();
	
	  // Update subklass/sibling/implementor links of live klasses
	  GenMarkSweep::follow_weak_klass_links();
	  assert(GenMarkSweep::_marking_stack.is_empty(),
	         "stack should be empty by now");
	
	  // Visit memoized MDO's and clear any unmarked weak refs
	  GenMarkSweep::follow_mdo_weak_refs();
	  assert(GenMarkSweep::_marking_stack.is_empty(), "just drained");
	
	
	  // Visit interned string tables and delete unmarked oops
	  StringTable::unlink(&GenMarkSweep::is_alive);
	  // Clean up unreferenced symbols in symbol table.
	  SymbolTable::unlink();
	
	  assert(GenMarkSweep::_marking_stack.is_empty(),
	         "stack should be empty by now");
	}
	
```


