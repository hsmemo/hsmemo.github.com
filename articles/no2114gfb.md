---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/memoryService.cpp

### 名前(function name)
```
void MemoryService::add_compact_perm_gen_memory_pool(CompactingPermGenGen* perm_gen,
                                                     MemoryManager* mgr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) MemoryService::add_space() で ContiguousSpacePool のインスタンスを生成し, 
      引数で渡された MemoryManager に登録する.
    
      (なお, UseSharedSpaces オプションが指定されている場合には, 
       共有されている部分用に (read-only と read-write に分けて)
       さらに 2つの ContiguousSpacePool オブジェクトを MemoryService::add_space() で生成し, 
       引数で渡された MemoryManager に登録しておく)
      ---------------------------------------- -}

	  PermanentGenerationSpec* spec = perm_gen->spec();
	  size_t max_size = spec->max_size() - spec->read_only_size() - spec->read_write_size();
	  MemoryPool* pool = add_space(perm_gen->unshared_space(),
	                               "Perm Gen",
	                                false, /* is_heap */
	                                max_size,
	                                true   /* support_usage_threshold */);
	  mgr->add_pool(pool);
	  if (UseSharedSpaces) {
	    pool = add_space(perm_gen->ro_space(),
	                     "Perm Gen [shared-ro]",
	                     false, /* is_heap */
	                     spec->read_only_size(),
	                     true   /* support_usage_threshold */);
	    mgr->add_pool(pool);
	
	    pool = add_space(perm_gen->rw_space(),
	                     "Perm Gen [shared-rw]",
	                     false, /* is_heap */
	                     spec->read_write_size(),
	                     true   /* support_usage_threshold */);
	    mgr->add_pool(pool);
	  }
	}
	
```


