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
  static jlong loaded_class_bytes() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (shared の領域と shared ではない領域の) 両方の領域でロードされたクラスの合計バイト数をリターンする.
    
      (ただし, UsePerfData オプションが指定されていない場合には, 機能が有効になっていないので -1 を返す)
      ---------------------------------------- -}

	    if (UsePerfData) {
	      return _classbytes_loaded->get_value() + _shared_classbytes_loaded->get_value();
	    } else {
	      return -1;
	    }
	  }
	
```


