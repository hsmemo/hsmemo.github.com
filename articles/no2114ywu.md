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
void PSParallelCompact::summarize_spaces_quick()
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 各ヒープ領域に対して ParallelCompactData::summarize() を呼び出し, 
      それぞれの領域中の live オブジェクト量を
      対応する SpaceInfo オブジェクトの SpaceInfo::new_top() に格納する.
      (ParallelCompactData::summarize() が
       引数で渡した new_top_addr に対して live オブジェクト量を書き込む)
    
      (ついでに, 各 SpaceInfo オブジェクトの dense_prefix 量についても 
       0 にリセットしておく)
      ---------------------------------------- -}

	  for (unsigned int i = 0; i < last_space_id; ++i) {
	    const MutableSpace* space = _space_info[i].space();
	    HeapWord** nta = _space_info[i].new_top_addr();
	    bool result = _summary_data.summarize(_space_info[i].split_info(),
	                                          space->bottom(), space->top(), NULL,
	                                          space->bottom(), space->end(), nta);
	    assert(result, "space must fit into itself");
	    _space_info[i].set_dense_prefix(space->bottom());
	  }
	
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      ---------------------------------------- -}

	#ifndef PRODUCT
	  if (ParallelOldGCSplitALot) {
	    provoke_split_fill_survivor(to_space_id);
	  }
	#endif // #ifndef PRODUCT
	}
	
```


