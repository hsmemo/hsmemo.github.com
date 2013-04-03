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
  void allocate() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _count フィールドの個数分だけの Handle 配列を確保する.
      ---------------------------------------- -}

	    _list = NEW_C_HEAP_ARRAY(Handle, _count);
	    assert(_list != NULL, "Out of memory");
	    if (_list == NULL) {
	      _count = 0;
	    }
	  }
	
```


