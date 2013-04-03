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
  static jlong class_method_data_size() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 現在ロードされているクラス内のメソッドの合計データサイズ(バイト数)をリターンする.
    
      (ただし, UsePerfData オプションが指定されていない場合には, 機能が有効になっていないので -1 を返す)
      ---------------------------------------- -}

	    return (UsePerfData ? _class_methods_size->get_value() : -1);
	  }
	
```


