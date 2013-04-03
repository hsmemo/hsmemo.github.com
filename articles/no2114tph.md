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
void MemoryService::add_cms_perm_gen_memory_pool(CMSPermGenGen* cms_gen,
                                                 MemoryManager* mgr) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) MemoryService::add_cms_space() で CompactibleFreeListSpacePool のインスタンスを生成し,
      引数で渡された MemoryManager に登録する.
      ---------------------------------------- -}

	  MemoryPool* pool = add_cms_space(cms_gen->cmsSpace(),
	                                   "CMS Perm Gen",
	                                   false, /* is_heap */
	                                   cms_gen->reserved().byte_size(),
	                                   true   /* support_usage_threshold */);
	  mgr->add_pool(pool);
	}
	
```


