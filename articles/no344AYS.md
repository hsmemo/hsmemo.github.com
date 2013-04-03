---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningGenerations.cpp
### 説明(description)

```
// If boundary moving is being used, create the young gen and old
// gen with ASPSYoungGen and ASPSOldGen, respectively.  Revert to
// the old behavior otherwise (with PSYoungGen and PSOldGen).

```

### 名前(function name)
```
AdjoiningGenerations::AdjoiningGenerations(ReservedSpace old_young_rs,
                                           size_t init_low_byte_size,
                                           size_t min_low_byte_size,
                                           size_t max_low_byte_size,
                                           size_t init_high_byte_size,
                                           size_t min_high_byte_size,
                                           size_t max_high_byte_size,
                                           size_t alignment) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _virtual_spaces フィールド (AdjoiningVirtualSpaces オブジェクト) のコンストラクタを呼び出す.
      (やることは, 各フィールドに引数を初期値として設定する程度.
       (See: AdjoiningVirtualSpaces::AdjoiningVirtualSpaces())
      ---------------------------------------- -}

	  _virtual_spaces(old_young_rs, min_low_byte_size,
	                  min_high_byte_size, alignment) {

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(min_low_byte_size <= init_low_byte_size &&
	         init_low_byte_size <= max_low_byte_size, "Parameter check");
	  assert(min_high_byte_size <= init_high_byte_size &&
	         init_high_byte_size <= max_high_byte_size, "Parameter check");

  {- -------------------------------------------
  (1) 以下で, PSYoungGen オブジェクトと PSOldGen オブジェクト(あるいはそれらのサブクラスのオブジェクト)を生成し, 
      対応する仮想メモリ空間を commit する.
  
      UseAdaptiveGCBoundary オプションの値に応じて, 生成するオブジェクトが異なる.
      * UseAdaptiveGCBoundary が true の場合: 
        ASPSYoungGen オブジェクトと ASPSOldGen オブジェクトの生成を行う.
      * UseAdaptiveGCBoundary が true の場合: 
        PSYoungGen オブジェクトと PSOldGen オブジェクトの生成を行う.
      ---------------------------------------- -}

	  // Create the generations differently based on the option to
	  // move the boundary.

  {- -------------------------------------------
  (1) (以下は UseAdaptiveGCBoundary が true の場合)
      ---------------------------------------- -}

	  if (UseAdaptiveGCBoundary) {
	    // Initialize the adjoining virtual spaces.  Then pass the
	    // a virtual to each generation for initialization of the
	    // generation.
	
    {- -------------------------------------------
  (1.1) AdjoiningVirtualSpaces::initialize() で, 
        各世代に対応する PSVirtualSpace を作成して仮想メモリ空間を commit する.
        ---------------------------------------- -}

	    // Does the actual creation of the virtual spaces
	    _virtual_spaces.initialize(max_low_byte_size,
	                               init_low_byte_size,
	                               init_high_byte_size);
	
    {- -------------------------------------------
  (1.1) ASPSYoungGen オブジェクトと ASPSOldGen オブジェクトの生成を行う.
        ---------------------------------------- -}

	    // Place the young gen at the high end.  Passes in the virtual space.
	    _young_gen = new ASPSYoungGen(_virtual_spaces.high(),
	                                  _virtual_spaces.high()->committed_size(),
	                                  min_high_byte_size,
	                                  _virtual_spaces.high_byte_size_limit());
	
	    // Place the old gen at the low end. Passes in the virtual space.
	    _old_gen = new ASPSOldGen(_virtual_spaces.low(),
	                              _virtual_spaces.low()->committed_size(),
	                              min_low_byte_size,
	                              _virtual_spaces.low_byte_size_limit(),
	                              "old", 1);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    young_gen()->initialize_work();
	    assert(young_gen()->reserved().byte_size() <= young_gen()->gen_size_limit(),
	     "Consistency check");
	    assert(old_young_rs.size() >= young_gen()->gen_size_limit(),
	     "Consistency check");
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    old_gen()->initialize_work("old", 1);
	    assert(old_gen()->reserved().byte_size() <= old_gen()->gen_size_limit(),
	     "Consistency check");
	    assert(old_young_rs.size() >= old_gen()->gen_size_limit(),
	     "Consistency check");

  {- -------------------------------------------
  (1) (以下は UseAdaptiveGCBoundary が false の場合)
      ---------------------------------------- -}

	  } else {
	
    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    // Layout the reserved space for the generations.
	    ReservedSpace old_rs   =
	      virtual_spaces()->reserved_space().first_part(max_low_byte_size);
	    ReservedSpace heap_rs  =
	      virtual_spaces()->reserved_space().last_part(max_low_byte_size);
	    ReservedSpace young_rs = heap_rs.first_part(max_high_byte_size);
	    assert(young_rs.size() == heap_rs.size(), "Didn't reserve all of the heap");
	
    {- -------------------------------------------
  (1.1) PSYoungGen オブジェクトと PSOldGen オブジェクトの生成を行う.
        ---------------------------------------- -}

	    // Create the generations.  Virtual spaces are not passed in.
	    _young_gen = new PSYoungGen(init_high_byte_size,
	                                min_high_byte_size,
	                                max_high_byte_size);
	    _old_gen = new PSOldGen(init_low_byte_size,
	                            min_low_byte_size,
	                            max_low_byte_size,
	                            "old", 1);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // The virtual spaces are created by the initialization of the gens.
	    _young_gen->initialize(young_rs, alignment);
	    assert(young_gen()->gen_size_limit() == young_rs.size(),
	      "Consistency check");
	    _old_gen->initialize(old_rs, alignment, "old", 1);
	    assert(old_gen()->gen_size_limit() == old_rs.size(), "Consistency check");
	  }
	}
	
```


