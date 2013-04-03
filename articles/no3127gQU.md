---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/sparsePRT.cpp

### 名前(function name)
```
bool SparsePRT::add_card(RegionIdx_t region_id, CardIdx_t card_index) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#if SPARSE_PRT_VERBOSE
	  gclog_or_tty->print_cr("  Adding card %d from region %d to region %d sparse.",
	                card_index, region_id, _hr->hrs_index());
	#endif

  {- -------------------------------------------
  (1) RSHashTable::add_card() を呼んで記録処理を行う.
      (ただし半分以上のエントリが埋まっていたら, まず expand() してから呼び出す)
      ---------------------------------------- -}

	  if (_next->occupied_entries() * 2 > _next->capacity()) {
	    expand();
	  }
	  return _next->add_card(region_id, card_index);
	}
	
```


