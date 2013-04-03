---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/generationSpec.cpp

### 名前(function name)
```
Generation* GenerationSpec::init(ReservedSpace rs, int level,
                                 GenRemSet* remset) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 指定された名前に応じて, 以下の switch-case で
      適切な Generation クラスのコンストラクタを呼び出し, 
      生成された Generation オブジェクトをリターンする.
    
      (ただし, ConcurrentMarkSweep と ASConcurrentMarkSweep の場合は, コンストラクタ呼び出し後に 
       ConcurrentMarkSweepGeneration::initialize_performance_counters() を呼び出してからリターン)
      ---------------------------------------- -}

	  switch (name()) {
	    case Generation::DefNew:
	      return new DefNewGeneration(rs, init_size(), level);
	
	    case Generation::MarkSweepCompact:
	      return new TenuredGeneration(rs, init_size(), level, remset);
	
	#ifndef SERIALGC
	    case Generation::ParNew:
	      return new ParNewGeneration(rs, init_size(), level);
	
	    case Generation::ASParNew:
	      return new ASParNewGeneration(rs,
	                                    init_size(),
	                                    init_size() /* min size */,
	                                    level);
	
	    case Generation::ConcurrentMarkSweep: {
	      assert(UseConcMarkSweepGC, "UseConcMarkSweepGC should be set");
	      CardTableRS* ctrs = remset->as_CardTableRS();
	      if (ctrs == NULL) {
	        vm_exit_during_initialization("Rem set incompatibility.");
	      }
	      // Otherwise
	      // The constructor creates the CMSCollector if needed,
	      // else registers with an existing CMSCollector
	
	      ConcurrentMarkSweepGeneration* g = NULL;
	      g = new ConcurrentMarkSweepGeneration(rs,
	                 init_size(), level, ctrs, UseCMSAdaptiveFreeLists,
	                 (FreeBlockDictionary::DictionaryChoice)CMSDictionaryChoice);
	
	      g->initialize_performance_counters();
	
	      return g;
	    }
	
	    case Generation::ASConcurrentMarkSweep: {
	      assert(UseConcMarkSweepGC, "UseConcMarkSweepGC should be set");
	      CardTableRS* ctrs = remset->as_CardTableRS();
	      if (ctrs == NULL) {
	        vm_exit_during_initialization("Rem set incompatibility.");
	      }
	      // Otherwise
	      // The constructor creates the CMSCollector if needed,
	      // else registers with an existing CMSCollector
	
	      ASConcurrentMarkSweepGeneration* g = NULL;
	      g = new ASConcurrentMarkSweepGeneration(rs,
	                 init_size(), level, ctrs, UseCMSAdaptiveFreeLists,
	                 (FreeBlockDictionary::DictionaryChoice)CMSDictionaryChoice);
	
	      g->initialize_performance_counters();
	
	      return g;
	    }
	#endif // SERIALGC
	
	    default:
	      guarantee(false, "unrecognized GenerationName");
	      return NULL;
	  }
	}
	
```


