---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp

### 名前(function name)
```
CMBitMapRO::CMBitMapRO(ReservedSpace rs, int shifter):
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  _bm((uintptr_t*)NULL,0),
	  _shifter(shifter) {
	  _bmStartWord = (HeapWord*)(rs.base());
	  _bmWordSize  = rs.size()/HeapWordSize;    // rs.size() is in bytes
	  ReservedSpace brs(ReservedSpace::allocation_align_size_up(
	                     (_bmWordSize >> (_shifter + LogBitsPerByte)) + 1));
	
	  guarantee(brs.is_reserved(), "couldn't allocate CMS bit map");
	  // For now we'll just commit all of the bit map up fromt.
	  // Later on we'll try to be more parsimonious with swap.
	  guarantee(_virtual_space.initialize(brs, brs.size()),
	            "couldn't reseve backing store for CMS bit map");
	  assert(_virtual_space.committed_size() == brs.size(),
	         "didn't reserve backing store for all of CMS bit map?");
	  _bm.set_map((uintptr_t*)_virtual_space.low());
	  assert(_virtual_space.committed_size() << (_shifter + LogBitsPerByte) >=
	         _bmWordSize, "inconsistency in bit map sizing");
	  _bm.set_size(_bmWordSize >> _shifter);
	}
	
```


