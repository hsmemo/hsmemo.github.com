---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp

### 名前(function name)
```
HeapWord* PSParallelCompact::first_src_addr(HeapWord* const dest_addr,
                                            SpaceId src_space_id,
                                            size_t src_region_idx)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(summary_data().is_region_aligned(dest_addr), "not aligned");
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  const SplitInfo& split_info = _space_info[src_space_id].split_info();
	  if (split_info.dest_region_addr() == dest_addr) {
	    // The partial object ending at the split point contains the first word to
	    // be copied to dest_addr.
	    return split_info.first_src_addr();
	  }
	
	  const ParallelCompactData& sd = summary_data();
	  ParMarkBitMap* const bitmap = mark_bitmap();
	  const size_t RegionSize = ParallelCompactData::RegionSize;
	
	  assert(sd.is_region_aligned(dest_addr), "not aligned");
	  const RegionData* const src_region_ptr = sd.region(src_region_idx);
	  const size_t partial_obj_size = src_region_ptr->partial_obj_size();
	  HeapWord* const src_region_destination = src_region_ptr->destination();
	
	  assert(dest_addr >= src_region_destination, "wrong src region");
	  assert(src_region_ptr->data_size() > 0, "src region cannot be empty");
	
	  HeapWord* const src_region_beg = sd.region_to_addr(src_region_idx);
	  HeapWord* const src_region_end = src_region_beg + RegionSize;
	
	  HeapWord* addr = src_region_beg;
	  if (dest_addr == src_region_destination) {
	    // Return the first live word in the source region.
	    if (partial_obj_size == 0) {
	      addr = bitmap->find_obj_beg(addr, src_region_end);
	      assert(addr < src_region_end, "no objects start in src region");
	    }
	    return addr;
	  }
	
	  // Must skip some live data.
	  size_t words_to_skip = dest_addr - src_region_destination;
	  assert(src_region_ptr->data_size() > words_to_skip, "wrong src region");
	
	  if (partial_obj_size >= words_to_skip) {
	    // All the live words to skip are part of the partial object.
	    addr += words_to_skip;
	    if (partial_obj_size == words_to_skip) {
	      // Find the first live word past the partial object.
	      addr = bitmap->find_obj_beg(addr, src_region_end);
	      assert(addr < src_region_end, "wrong src region");
	    }
	    return addr;
	  }
	
	  // Skip over the partial object (if any).
	  if (partial_obj_size != 0) {
	    words_to_skip -= partial_obj_size;
	    addr += partial_obj_size;
	  }
	
	  // Skip over live words due to objects that start in the region.
	  addr = skip_live_words(addr, src_region_end, words_to_skip);
	  assert(addr < src_region_end, "wrong src region");
	  return addr;
	}
	
```


