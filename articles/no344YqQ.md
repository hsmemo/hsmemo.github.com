---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/generationSizer.hpp

### 名前(function name)
```
  GenerationSizer() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) GenerationSizer::initialize_flags() 及び
      TwoGenerationCollectorPolicy::initialize_size_info() を呼んで, 
      初期化を行う.
      ---------------------------------------- -}

	    // Partial init only!
	    initialize_flags();
	    initialize_size_info();
	  }
	
```


