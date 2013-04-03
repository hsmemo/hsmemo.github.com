---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/code/nmethod.cpp

### 名前(function name)
```
jmethodID nmethod::get_and_cache_jmethod_id() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 処理対象の jmethod_id を _jmethod_id フィールドに記録しておく.
      (unload する場合は, これ以降はアクセスできなくなってしまうので)
      ---------------------------------------- -}

	  if (_jmethod_id == NULL) {
	    // Cache the jmethod_id since it can no longer be looked up once the
	    // method itself has been marked for unloading.
	    _jmethod_id = method()->jmethod_id();
	  }
	  return _jmethod_id;
	}
	
```


