---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1SATBCardTableModRefBS.cpp

### 名前(function name)
```
G1SATBCardTableModRefBS::G1SATBCardTableModRefBS(MemRegion whole_heap,
                                                 int max_covered_regions) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) スーパークラスの初期化
      ---------------------------------------- -}

	    CardTableModRefBSForCTRS(whole_heap, max_covered_regions)
	{

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _kind = G1SATBCT;
	}
	
```


