---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/classLoadingService.hpp

### 名前(function name)
```
  static void add_class_method_size(int size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数で指定された size 分だけ _class_methods_size を増加させる.
      (ただし, UsePerfData オプションが指定されていなければ何もしない)
      ---------------------------------------- -}

	    if (UsePerfData) {
	      _class_methods_size->inc(size);
	    }
	  }
	
```


