---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/genCollectedHeap.cpp

### 名前(function name)
```
char* GenCollectedHeap::allocate(size_t alignment,
                                 PermanentGenerationSpec* perm_gen_spec,
                                 size_t* _total_reserved,
                                 int* _n_covered_regions,
                                 ReservedSpace* heap_rs){
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const char overflow_msg[] = "The size of the object heap + VM data exceeds "
	    "the maximum representable size";
	
	  // Now figure out the total size.
	  size_t total_reserved = 0;
	  int n_covered_regions = 0;
	  const size_t pageSize = UseLargePages ?
	      os::large_page_size() : os::vm_page_size();
	
  {- -------------------------------------------
  (1) ヒープ領域として確保するサイズ(以下の total_reserved)を計算する
        (各世代の最大サイズ (以下の _gen_specs[i]->max_size()) を足し合わせ, 
         その後, Perm Gen の大きさ (以下の perm_gen_spec->max_size()) を足し込み,
         さらに Perm Gen に必要となるデータの領域分 (以下の perm_gen_spec->misc_data_size() + 
         perm_gen_spec->misc_code_size()) を足し,
         最後に (UseLargePages オプションが指定されていれば) large page size で丸める)
  
        (なお, 途中で値がオーバーフローしてしまったら, ここでエラー終了 (上記の overflow_msg を表示))
      ---------------------------------------- -}

	  for (int i = 0; i < _n_gens; i++) {
	    total_reserved += _gen_specs[i]->max_size();
	    if (total_reserved < _gen_specs[i]->max_size()) {
	      vm_exit_during_initialization(overflow_msg);
	    }
	    n_covered_regions += _gen_specs[i]->n_covered_regions();
	  }
	  assert(total_reserved % pageSize == 0,
	         err_msg("Gen size; total_reserved=" SIZE_FORMAT ", pageSize="
	                 SIZE_FORMAT, total_reserved, pageSize));
	  total_reserved += perm_gen_spec->max_size();
	  assert(total_reserved % pageSize == 0,
	         err_msg("Perm size; total_reserved=" SIZE_FORMAT ", pageSize="
	                 SIZE_FORMAT ", perm gen max=" SIZE_FORMAT, total_reserved,
	                 pageSize, perm_gen_spec->max_size()));
	
	  if (total_reserved < perm_gen_spec->max_size()) {
	    vm_exit_during_initialization(overflow_msg);
	  }
	  n_covered_regions += perm_gen_spec->n_covered_regions();
	
	  // Add the size of the data area which shares the same reserved area
	  // as the heap, but which is not actually part of the heap.
	  size_t s = perm_gen_spec->misc_data_size() + perm_gen_spec->misc_code_size();
	
	  total_reserved += s;
	  if (total_reserved < s) {
	    vm_exit_during_initialization(overflow_msg);
	  }
	
	  if (UseLargePages) {
	    assert(total_reserved != 0, "total_reserved cannot be 0");
	    total_reserved = round_to(total_reserved, os::large_page_size());
	    if (total_reserved < os::large_page_size()) {
	      vm_exit_during_initialization(overflow_msg);
	    }
	  }
	
  {- -------------------------------------------
  (1) ヒープ領域の確保場所として望ましい仮想アドレスを計算する.
  
      (なお, UseSharedSpaces オプションが指定されておらず, かつ UseCompressedOops オプションが指定されていた場合, 
      以下の if の中でメモリを確保してリターンする.
      それ以外のケースでは if を抜けた先でメモリを確保してリターンすることになる)
    
      (なお UseCompressedOops オプションが指定されており, かつ指定した仮想アドレスでの領域確保が失敗した場合は, 
      アドレスを変えて再度確保を試みる (Universe::ZeroBasedNarrowOop).
      それでもダメなら, heap base を用いる方法で確保を試みる (Universe::HeapBasedNarrowOop))
      ---------------------------------------- -}

	  // Calculate the address at which the heap must reside in order for
	  // the shared data to be at the required address.
	
	  char* heap_address;
	  if (UseSharedSpaces) {
	
	    // Calculate the address of the first word beyond the heap.
	    FileMapInfo* mapinfo = FileMapInfo::current_info();
	    int lr = CompactingPermGenGen::n_regions - 1;
	    size_t capacity = align_size_up(mapinfo->space_capacity(lr), alignment);
	    heap_address = mapinfo->region_base(lr) + capacity;
	
	    // Calculate the address of the first word of the heap.
	    heap_address -= total_reserved;
	  } else {
	    heap_address = NULL;  // any address will do.
	    if (UseCompressedOops) {
	      heap_address = Universe::preferred_heap_base(total_reserved, Universe::UnscaledNarrowOop);
	      *_total_reserved = total_reserved;
	      *_n_covered_regions = n_covered_regions;
	      *heap_rs = ReservedHeapSpace(total_reserved, alignment,
	                                   UseLargePages, heap_address);
	
	      if (heap_address != NULL && !heap_rs->is_reserved()) {
	        // Failed to reserve at specified address - the requested memory
	        // region is taken already, for example, by 'java' launcher.
	        // Try again to reserver heap higher.
	        heap_address = Universe::preferred_heap_base(total_reserved, Universe::ZeroBasedNarrowOop);
	        *heap_rs = ReservedHeapSpace(total_reserved, alignment,
	                                     UseLargePages, heap_address);
	
	        if (heap_address != NULL && !heap_rs->is_reserved()) {
	          // Failed to reserve at specified address again - give up.
	          heap_address = Universe::preferred_heap_base(total_reserved, Universe::HeapBasedNarrowOop);
	          assert(heap_address == NULL, "");
	          *heap_rs = ReservedHeapSpace(total_reserved, alignment,
	                                       UseLargePages, heap_address);
	        }
	      }
	      return heap_address;
	    }
	  }
	
  {- -------------------------------------------
  (1) ReservedHeapSpace オブジェクトを生成してヒープ領域として使うメモリ空間を確保し, 
      引数で渡されたポインタ先(heap_rs)に確保した領域を書き込む.
      (あわせて, total_reserved や n_covered_regions の値も呼び出し元に通知している)
      ---------------------------------------- -}

	  *_total_reserved = total_reserved;
	  *_n_covered_regions = n_covered_regions;
	  *heap_rs = ReservedHeapSpace(total_reserved, alignment,
	                               UseLargePages, heap_address);
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return heap_address;
	}
	
```


