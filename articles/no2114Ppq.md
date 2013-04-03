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
  static jlong loaded_class_count() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (shared の領域と shared ではない領域の) 両方の領域でロードされた回数の合計値をリターンする.
      ---------------------------------------- -}

	    return _classes_loaded_count->get_value() + _shared_classes_loaded_count->get_value();
	  }
	
```


