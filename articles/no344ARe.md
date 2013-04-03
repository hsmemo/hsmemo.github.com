---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.hpp

### 名前(function name)
```
  G1CollectorPolicy_BestRegionsFirst() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CollectionSetChooser オブジェクトを生成して, _collectionSetChooser フィールドに格納するだけ.
      ---------------------------------------- -}

	    _collectionSetChooser = new CollectionSetChooser();
	  }
	
```


