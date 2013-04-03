---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiGetLoadedClasses.cpp

### 名前(function name)
```
  Handle get_element(int index) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _list フィールド中の指定の index の要素をリターンする.
  
      (_list フィールド が NULL だったり index が配列長を超えている場合は, 空の Handle をリターン)
      ---------------------------------------- -}

	    if ((_list != NULL) && (index < _count)) {
	      return _list[index];
	    } else {
	      assert(false, "empty get_element");
	      return Handle();
	    }
	  }
	
```


