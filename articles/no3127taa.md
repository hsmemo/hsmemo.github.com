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
bool RSHashTable::add_card(RegionIdx_t region_ind, CardIdx_t card_index) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) RSHashTable::entry_for_region_ind_create() を呼んで
      対応する SparsePRTEntry を取得する.
      ---------------------------------------- -}

	  SparsePRTEntry* e = entry_for_region_ind_create(region_ind);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(e != NULL && e->r_ind() == region_ind,
	         "Postcondition of call above.");

  {- -------------------------------------------
  (1) SparsePRTEntry::add_card() を呼んで, 記録処理を行う.
      (もし成功したら, _occupied_cards もインクリメントしておく)
      ---------------------------------------- -}

	  SparsePRTEntry::AddCardResult res = e->add_card(card_index);
	  if (res == SparsePRTEntry::added) _occupied_cards++;

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#if SPARSE_PRT_VERBOSE
	  gclog_or_tty->print_cr("       after add_card[%d]: valid-cards = %d.",
	                         pointer_delta(e, _entries, SparsePRTEntry::size()),
	                         e->num_valid_cards());
	#endif

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(e->num_valid_cards() > 0, "Postcondition");

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return res != SparsePRTEntry::overflow;
	}
	
```


