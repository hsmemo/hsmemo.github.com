---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/collectorPolicy.cpp

### 名前(function name)
```
void MarkSweepPolicy::initialize_generations() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) initialize_perm_generation() で, Perm Generation 用の GenerationSpec を作成する.
      (See: initialize_perm_generation())
      ---------------------------------------- -}

	  initialize_perm_generation(PermGen::MarkSweepCompact);

  {- -------------------------------------------
  (1) GenerationSpec を格納する配列を確保する.
      (確保に失敗したら, ここで異常終了)
      ---------------------------------------- -}

	  _generations = new GenerationSpecPtr[number_of_generations()];
	  if (_generations == NULL)
	    vm_exit_during_initialization("Unable to allocate gen spec");
	
  {- -------------------------------------------
  (1) Young Generation と Old Generation 用の GenerationSpec を作成する.
      (もし GenerationSpec の作成に失敗したら, ここで異常終了)
      ---------------------------------------- -}

	  if (UseParNewGC && ParallelGCThreads > 0) {
	    _generations[0] = new GenerationSpec(Generation::ParNew, _initial_gen0_size, _max_gen0_size);
	  } else {
	    _generations[0] = new GenerationSpec(Generation::DefNew, _initial_gen0_size, _max_gen0_size);
	  }
	  _generations[1] = new GenerationSpec(Generation::MarkSweepCompact, _initial_gen1_size, _max_gen1_size);
	
	  if (_generations[0] == NULL || _generations[1] == NULL)
	    vm_exit_during_initialization("Unable to allocate gen spec");
	}
	
```


